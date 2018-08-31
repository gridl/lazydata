# lazydata: scalable data dependencies

`lazydata` is a minimalist library for including data dependencies into Python projects. 

**Problem**: Keeping data files in git (e.g. via git-lfs) results in a bloated repository that takes ages to pull.   

**Solution**: `lazydata` only stores references to data files in git, and syncs data files on-demand when they are needed.

**Why**: The semantics of code and data are different - code needs to be versioned to merge it, and data just needs to be kept in sync. `lazydata` achieves exactly this in a minimal way.    

**Benefits**:

- Keeps your git repository clean with just code, while enabling seamless access to any number of linked data files 
- Data consistency assured using file hashes and automatic versioning
- Choose your own remote storage backend: AWS S3 or directory over SSH

`lazydata` is primarily designed for machine learning and data science project.  

## Getting started 

In this section we'll show how to use `lazydata` on an example project.

### Installation

Install with pip:

```bash
$ pip install lazydata
```

### Add to your project

To enable `lazydata` run in your git repository root:

```bash
$ lazydata init 
```

This will initialise`lazydata.yml` which will hold the list of files managed by lazydata. 

### Storing a file

To add a file to the lazydata data store use `use("<path_to_file>")` in your code:

**my_script.py**
```python
import lazydata

# store the file when loading  
import pandas as pd
df = pd.read_csv(lazydata.use("data/my_big_table.csv"))

print("Data shape:" + df.shape)

```

Running the script will store the file:

```bash
$ python my_script.py
## lazydata: Storing a new file data/my_big_table.csv
## Data shape: (10000,100)
```

The file has now been stored in your local lazydata cache in `~/.lazydata-cache` and added to **lazydata.yml**:
```yaml
files:
  - path: data/my_big_table.csv
    hash: 2C94697198875B6E...
    usage: my_script.py

```

Locally stored files in your lazydata cache are hard-linked, meaning that they don't take any extra space on your hard drive (unless you delete or modify the original file in which case a copy of the file is kept). 

If you modify the data file and re-run the script, this will add another entry to the yml file with the new hash of the data file, i.e. data files are automatically versioned. If you don't want to keep past versions, simply remove them from the yml file. 

And you are done! This data file is now linked to your local repository.

### Sharing your stored files

To access your stored files from multiple machines add a remote storage backend where they can be uploaded. To use S3 as a retemo storage backend run:

```bash
$ lazydata add-remote s3://mybucket/lazydata
```

This will configure the S3 backend and also add it to `lazydata.yml` for future reference. Alternatively, you can also use `ssh://username@myserver/path` as a remote storage backend.

You can now git commit and push your  `my_script.py` and `lazydata.yml` files as you normally would. 
 
To copy the stored data files to S3 use:

```bash
$ lazydata push
```

When your collaborator pulls the latest version of the git repository, they will get the script and the `lazydata.yml` file as usual.  

Data files will be downloaded when your collaborator runs `my_script.py` and the `lazydata("my_big_table.csv")` is executed:

```bash
$ python my_script.py
## lazydata: Downloading stored file my_big_table.csv ...
## Data shape: (10000,100)
``` 

To get the data files without running the code, you can also use the command line utility:

```bash
# download just this file
$ lazydata pull my_big_table.csv

# download everything stored in this script
$ lazydata pull my_script.py

# download everything stored in this git commit
$ lazydata pull 653ca451

# download everything in HEAD
$ lazydata pull .
```

Because `lazydata.yml` is tracked by git you can safely make and switch git branches. 

### Advanced usage

You can achieve multiple data dependency scenarios by putting `lazydata.use()` into different parts of the code:

- Add to outputs of your data pipeline to store the outputs
- Add to `__init__(self)` to store files when the object is created
- Add to `__init__.py` to store files when the python module is used
- Add to `setup.py` to store files when a Python package is installed

### Coming soon... 

- Packaging multiple files into a portable dataset
- Add metadata to a stored file
- Visualise stored file provenance
- Storing data coming from databases and APIs

## Contributing

The library is licenced under Apache-2 licence. All contributions are welcome!
   