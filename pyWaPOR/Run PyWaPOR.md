# PyWaPOR 3.5

- **Source**: [https://bitbucket.org/cioapps/pywapor/src/master/](https://bitbucket.org/cioapps/pywapor/src/master/)
- **Documentation**: [https://www.fao.org/aquastat/py-wapor/](https://www.fao.org/aquastat/py-wapor/)
---
## What's needed
- Python >=3.7
- Install pywapor using [conda](https://docs.conda.io/projects/conda/en/latest/user-guide/concepts/environments.html) or [miniforge](https://github.com/conda-forge/miniforge). See [tutorial](https://courses.gisopencourseware.org/mod/book/view.php?id=430&chapterid=1427)
```cmd
conda create -n pywapor_env -c conda-forge pywapor
## OR
mamba create -n pywapor_env pywapor
```
---
### Create user accounts at
- [https://urs.earthdata.nasa.gov](https://urs.earthdata.nasa.gov/) (MODIS, SRTM, CHIRPS, MERRA2, and VIIRS)
	- Go to Applications > Authorized Apps
   	- Add 'NASA GESDISC DATA ARCHIVE' and 'LP DAAC OPeNDAP' to the list by clicking [![Button]][Link]
- [https://viewer.terrascope.be](https://viewer.terrascope.be)(PROBA-V)
- [https://cds.climate.copernicus.eu](https://cds.climate.copernicus.eu) (ERA5 and AgERA5)
- [https://earthexplorer.usgs.gov](https://earthexplorer.usgs.gov) (Landsat)
- [https://dataspace.copernicus.](https://dataspace.copernicus.eu/)(Sentinel-2 and Sentinel-3)
---
- Other data products that do not require user account: COPERNICUS (GLO30, GLO90), GEOS5 (inst3_2d_asm_Nx), GLOBCOVER (2009_V2.3_Global), STATICS (WaPOR2, WaPOR3) and VIIRSL1 (VNP02IMG)
---
## Choose your working environment
- Cloud computer
	- Google Colab (free with some [limits](https://research.google.com/colaboratory/faq.html#idle-timeouts))
```Python
pip install pywapor
```
---
- Your own computer
	```cmd
	conda activate pywapor_env
	#OR
	mamba activate pywapor_env
	```
	- Jupyter notebook (need to be installed to python environment)
	 	```cmd
		conda install jupyterlab
		#OR
		mamba install jupyterlab	
	  	```
	- Python console (CLI)
		
---
## Step 1: Choose your study case
- Region of interest
	- Latitude min,max
	- Longitude min, max
- Season
	- Start date, end date
---
```Python
import pywapor
project_folder = r"Test_240523" #Path to folder
bb = [33.1479429498060583, 14.2657100971198449, 33.2874918465625242, 14.3487734799492763] # [xmin, ymin, xmax, ymax] #Wad_Helal
period = ["2022-10-01", "2023-04-30"] 
# Set up a project.
project = pywapor.Project(project_folder, bb, period)
```
---
## Step 2: Configure input data
You can choose a default configuration or customize one. 
### Option 1: Default configurations 
For example, the input configuration for WaPOR3 Level 3 can be set as follows:
```Python
project.load_configuration(name = "WaPOR3_level_3")
```
See the list of default configurations: https://bitbucket.org/cioapps/pywapor/src/master/pywapor/configs/ 

⚠ WARNING: WaPOR3 default configurations might fail or take long time to download due to long queued ERA5 request.

---
### Option 2: Custom configurations
For a custom configuration, you need to create a python dictionary including the data product name for each type of input data.
The downloading tool only supports certain data products. See the list of data products supported: https://www.fao.org/aquastat/py-wapor/data_sources.html. 
When defining a custom configuration, you need to use a valid product name from this list. 

Using other data products that are not supported is also possible via [sideloading](https://colab.research.google.com/github/un-fao/FAO-Water-Applications/blob/main/pyWaPOR/sideload.ipynb). However, the documentation for sideloading using version 3.5 is not yet available.

---
- Custom configuration example
```Python
summary = {
            '_ENHANCE_': {"bt": ["pywapor.enhancers.dms.thermal_sharpener.sharpen"],},
            '_EXAMPLE_': 'SENTINEL2.S2MSI2A_R20m',
            '_WHITTAKER_': {'SENTINEL2.S2MSI2A_R20m':
				{'method':'linear'},
				#{'lmbdas': 1000.0,'method': 'whittaker'}, #whittaker interpolation takes too long 
                            'VIIRSL1.VNP02IMG':
				{'method':'linear'},
				#{'a': 0.85,'lmbdas': 1000.0,'method': 'whittaker'}}, #whittaker interpolation takes too long
				},
            'elevation': {'COPERNICUS.GLO30'},
            'meteorological': {'GEOS5.inst3_2d_asm_Nx'},
            'optical': {'SENTINEL2.S2MSI2A_R20m'},
            'precipitation': {'CHIRPS.P05'},
            'soil moisture': {'FILE:{folder}{sep}se_root_out*.nc'},
            'solar radiation': {'MERRA2.M2T1NXRAD.5.12.4'},
            'statics': {'STATICS.WaPOR3'},
            'thermal': {'VIIRSL1.VNP02IMG'}
            }
project.load_configuration(summary = summary)
```
---
- Save configuration 
```Python
import os
project.configuration.to_json(os.path.join(project_folder,"configuration.json"))
```
---
## Step 3: Download data
- Enter account and passwords
```Python
project.set_passwords()
```
- Start downloader
```Python
datasets = project.download_data()
```
---
## Step 4: Run model
- Run the models with set configuration and default parameters
```Python
se_root_in = project.run_pre_se_root()
se_root = project.run_se_root()
et_look_in = project.run_pre_et_look()
#et_look = project.run_et_look() #no custom chunksize available yet in version 3.5.2
# run et_look with custom chunksize
et_look = pywapor.et_look.main(et_look_in, et_look_version = 'v3', chunks = {"time_bins": 1, "x": 1000, "y": 1000})
```
---
# Debugging
- Check error message in **project_folder/log.txt**
- Try to understand and fix errors. 
- OR give up and report as an [issue](https://bitbucket.org/cioapps/pywapor/issues) on bitbucket repository
---
### Common errors:
- Installation errors:
	- GDAL DLL: see this https://gis.stackexchange.com/questions/430538/importing-gdal-in-qgis-and-pycharm-not-working/456857#456857  
- Server issues: 
	- Incorrect authentication => reset account and passwords
		```Python
		pywapor.collect.accounts.setup('NASA')
		```
	- "Server error 502 Server Error: Bad Gateway" => seems to happen when using one user account to request data in more than one processes. Make sure you are downloading data in only one place.
	- Timeout, etc. => check internet connection, no fix
	- Long queued CDS request => check **project_folder/ERA5/CDS_log.txt**, no fix
- Configuration error: 
	- Missing/invalid values (e.g. product name, method name) => Check documentation and source
---
- Used all available RAM => optimize chunksize
- NameError: Could not download url => check the printed url in the error message
- Empty downloaded files => delete, and re-run
---
# Custom parameters
- Changing model parameters (or entire variables) can be done in between the `project.run_pre_se_root()` and `project.run_se_root()` steps (same goes for `et_look`) by making changes to the `xarray.Dataset` returned by `project.run_pre_se_root` (and stored at `project.se_root_in`):
```Python
print(se_root_in["r0_bare"].values)
se_root_in["r0_bare"] = 0.32
print(se_root_in["r0_bare"].values)
print(project.se_root_in["r0_bare"].values)
```

---
[Button]: https://img.shields.io/badge/APPROVE_MORE_APPLICATIONS-blue?style=for-the-badge
[Link]: https://urs.earthdata.nasa.gov/application_search 
