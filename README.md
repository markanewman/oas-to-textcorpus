# OAS To Text Corpus

The [National Institutes of Health](https://nih.gov) has provided an excelent [data source](https://www.ncbi.nlm.nih.gov/pmc/tools/textmining/) for text mining.
Not only does it cover Medical journals, but other ones from mathmatics to chemestry.
The purpose of this repo is to convert the PMC Open Access Subset from the given format into the normal text corpus format.
I.E. one document per file, one sentence per line, pargraphs have a blank line between them.

# Prerequisites

The following packages need to be installed.
I recommend using [Chocolatey](https://chocolatey.org/install).

* [7-zip](https://www.7-zip.org/)
* [Python](https://www.python.org/downloads/)

  
```{ps1}
if('Unrestricted' -ne (Get-ExecutionPolicy)) { Set-ExecutionPolicy Bypass -Scope Process -Force }
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
refreshenv

choco install 7zip.install -y
choco install python3 -y
```

# Modules

All scripts have been tested on Python 3.8.2.
The below modules are need to run the scripts.
The scripts were tested on the noted versions, so YMMV.
**Note**: not all modules are required for all scripts.
If this it the first time running the scripts, the modules will need to be installed.
They can be installed by navigating to the `~/code` folder, then using the below code.

* nltk 3.4.5
* progressbar2 3.47.0
* lxml 4.5.0
* wget 3.2

```{shell}
pip install -r requirments.txt
python -c "import nltk;nltk.download('punkt')"
```

# Steps

The below document describes how to recreate the text corpus.
It assumes that a particular path structure will be used, but the commands can be modified to target a different directory structure without changing the code.
I am choosing the `d:/oas` directory because my d drive is big enough to hold everything.

1. Clone this repo then open a shell to the `~/code` directory.
2. Retrieve the dataset.
```{shell}
python download_oas.py -dest d:/oas
``` 
3. Extract the data in-place.
   This needs done twice due to the `*.tar.gz` compression.
   * **NOTE 1**: I have personally observed some amount of _sampeling_ between downloads for the OAS.
     I.E. The 2019-05-18 version of `comm_use.A-B.xml.tar.gz` has some files lacking in the one from 2020-03-23.
     As of 2020-03-24, accross all 8 files, this amounted to ~1700 articles.
     It _may_ be helpful to keep some amount of _personal_ archive here as some articles are in one bulk dump and not the next.
     I have the included the delete commands to cleanup the folder in case there is a space concern.
   * **NOTE 2**: I have personally observed some amount of PCMID _reuse_ accross different folders.
     I.E. `PMC6174971.nxml` is in both the `~/Glob_Chall` and `~/Global_Chall` folders.
     As of 2020-03-24, accross all 8 files, this amounted to ~20 articles.
     As of now, this apperes to simply be the same file in multiple folders.
     The extract command assumes this to be true hence the `-aos` switch.
```{shell}
"C:/Program Files/7-Zip/7z.exe" e -od:/oas "d:/oas/*.xml.tar.gz"
"C:/Program Files/7-Zip/7z.exe" e -aos -od:/oas/raw "d:/oas/*.xml.tar"
del "d:\oas\*.xml.tar.gz"
del "d:\oas\*.xml.tar"
```
4. [Extract](./code/extract_metadata.py) the metadata.
   This will create a single `metadata.csv` containing some useful information.
   In general this would be used as part of segementation or as part of a MANOVA.
   Some of the files provide by NIH do not parse.
   These _incomplete_ files are filtered out of the final folders and noted in `{{file-out}}error.csv`
```{shell}
python extract_metadata.py -in d:/oas/raw -out d:/oas/metadata.csv
```
5. [Convert](./code/convert_to_corpus.py) the raw JATS files into the nomal folder corpus format.
   This will create a text corpus folder at the location I.E. `./corpus` containing 2 sub folders, one for the abstract and one for the body.
   Some of the files provide by NIH do not parse.
   These _incomplete_ files are filtered out of the final folders and noted in `{{folder-out}}.error.csv`
```{shell}
python convert_to_corpus.py -in d:/oas/raw -out d:/oas/corpus
```