## WaPOR Notebooks

<table class = "docutils align-default">
   <thead>
      <tr class="row-odd" style="text-align:center">
         <th class="head">Name</th>
         <th class="head" width = "150">Colab</th>
      </tr>
   </thead>
   <tbody>
      <tr class="row-even">
         <td>WaPOR v2 API</td>
         <td style="text-align:center"><a href="https://colab.research.google.com/github/un-fao/FAO-Water-Applications/blob/main/WaPOR/WaPORv2_API.ipynb"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="colab v2"/></a></td>
      </tr>
      <tr class="row-odd">
        <td>WaPOR v3 API</td>
        <td style="text-align:center"><a href="https://colab.research.google.com/github/un-fao/FAO-Water-Applications/blob/main/WaPOR/WaPORv3_API.ipynb"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="colab v3"/></a></td>
    </tr>
   </tbody>
</table>

## Running the notebooks localy

To create a local environment with package versions similar as used on Google Colab, first run the following code on Colab to create a `yml`-file:

```python
from datetime import datetime as dt
from platform import python_version

reqs = {
    "ee": "earthengine-api", 
    "requests": "requests", 
    "numpy": "numpy", 
    "matplotlib": "matplotlib", 
    "osgeo.gdal": "gdal", 
}

modules = {}

for req, repo_name in reqs.items():
  try:
    modules[req] = __import__(req)
  except ImportError:
    print(f"Error importing `{req}`.")

date = dt.now().strftime('%Y%m%d')
yml_lines = [
    f"# run: conda env create --file colab_{date}.yml",
    f"name: colab_{date}",
    "dependencies:",
    f"- python=={python_version()}",
    "- ipykernel",
    "- pip",
    ]

for name, mod in modules.items():
  version = getattr(modules[name], "__version__", None)
  if isinstance(version, type(None)):
    yml_lines.append(reqs[name])
  else:
    yml_lines.append(f"- {reqs[name]}=={version}")

with open(f"colab_{date}.yml", "w") as outfile:
  outfile.write('\n'.join(str(i) for i in yml_lines))
```

Then download the file and run `conda env create --file colab_20230921.yml` to create the environment.