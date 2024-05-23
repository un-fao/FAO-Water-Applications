# PyWaPOR 3.5

- **Source**: [https://bitbucket.org/cioapps/pywapor/src/master/](https://bitbucket.org/cioapps/pywapor/src/master/)
- **Documentation**: [https://www.fao.org/aquastat/py-wapor/](https://www.fao.org/aquastat/py-wapor/)
---
## What's needed
- Python >=3.7
- Install pywapor using [conda](https://docs.conda.io/projects/conda/en/latest/user-guide/concepts/environments.html)
```cmd
conda create -n pywapor_env -c conda-forge pywapor
```
---
### Create user accounts at
- [https://urs.earthdata.nasa.](https://www.google.com/url?q=https%3A%2F%2Furs.earthdata.nasa.gov)[gov](https://www.google.com/url?q=https%3A%2F%2Furs.earthdata.nasa.gov) (MODIS, SRTM, CHIRPS and MERRA2)
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
- Your own computer
	- Jupyter notebook (need to be installed to python environment)
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
project_folder = r"Test_240523"
bb = [33.1479429498060583, 14.2657100971198449, 33.2874918465625242, 14.3487734799492763] 
# [xmin, ymin, xmax, ymax] #Wad_Helal
period = ["2022-10-01", "2023-04-30"]
# Set up a project.
project = pywapor.Project(project_folder, bb, period)
```
---
## Step 2: Configure input data
- List of data products supported: https://www.fao.org/aquastat/py-wapor/data_sources.html 
- List of default configurations: https://bitbucket.org/cioapps/pywapor/src/master/pywapor/configs/ 
```Python
project.load_configuration(name = "WaPOR3_level_3")
```

---
- Custom configuration example
```Python
summary = {
            '_ENHANCE_': {"bt": ["pywapor.enhancers.dms.thermal_sharpener.sharpen"],},
            '_EXAMPLE_': 'SENTINEL2.S2MSI2A_R20m',
            '_WHITTAKER_': {'SENTINEL2.S2MSI2A_R20m':  
					            {'lmbdas': 1000.0,                                             'method': 'whittaker'},
                            'VIIRSL1.VNP02IMG': 
								{'a': 0.85,'lmbdas': 1000.0,                                   'method': 'whittaker'}},
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
et_look = project.run_et_look()
```
---
# Debugging
- Check error message in **project_folder/log.txt**
- Try to understand and fix errors. 
- OR give up and report as an [issue](https://bitbucket.org/cioapps/pywapor/issues?status=new&status=open) on bitbucket repository
---
### Common errors:
- Server issues: 
	- Incorrect authentication
	- Gateway error, Timeout, etc.
	- Long queued CDS request => check **project_folder/ERA5/CDS_log.txt**
- Configuration error: 
	- Missing/invalid values (e.g. product name, method name) => Check documentation and source
---
# Custom parameters
- Changing model parameters (or entire variables) can be done in between the `project.run_pre_se_root()` and `project.run_se_root()` steps (same goes for `et_look`) by making changes to the `xarray.Dataset` returned by `project.run_pre_se_root` (and stored at `project.se_root_in`):
```Python
print(se_root_in["r0_bare"].values)
se_root_in["r0_bare"] = 0.32
print(se_root_in["r0_bare"].values)
print(project.se_root_in["r0_bare"].values)
```