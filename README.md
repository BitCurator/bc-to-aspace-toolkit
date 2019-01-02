# bc-to-aspace-toolkit

[![GitHub issues](https://img.shields.io/github/issues/bitcurator/bc-to-aspace-toolkit.svg)](https://github.com/bitcurator/bc-to-aspace-toolkit/issues)
[![GitHub forks](https://img.shields.io/github/forks/bitcurator/bc-to-aspace-toolkit.svg)](https://github.com/bitcurator/bc-to-aspace-toolkit/network)

Metadata transfer scripts - BitCurator to ASpace

## Setup and Installation

This script is intended to be run in the BitCurator environment (or a similarly configured Linux host). It requires the bulk_extractor tool to be present, and depends on pandas and pathlib. The pandas and pathlib dependencies are not included by default in the BitCurator environment; they will be automatically installed by the setup script if not present. 

First, open a terminal and check out the lastest version of this repo from GitHub:

```shell
git clone https://github.com/bitcurator/bc-to-aspace-toolkit
```

To install, run the following commands:

```shell
cd bc-to-aspace-toolkit
python3 setup.py build
sudo python3 setup.py install
```

## Running the script

A simple example with the included sample disk image is provided here.

This script assumes you have a working ArchivesSpace instance running on another host, or in a VM accessible from your host. Need a simple way to get a test instance of ArchivesSpace up and running? See our simple Vagrant deployment option at https://github.com/bitcurator/aspace-vagrant. The script also assumes the **brunnhilde.py** script is installed.

Ensure you're in the bc-to-aspace-toolkit directory, then copy the provided sample image to your home directory:

```shell
bcadmin@ubuntu:~$ cd ~/bc-to-aspace-toolkit
bcadmin@ubuntu:~$ cp sample-disk-images/nps-2010-emails.E01 ~/
```

The Brunnhilde script uses a tool provided by The Sleuth Kit, **tsk_recover**, to extract files from disk images. The **tsk_recover** tool will make a best effort to autodetect disk image type, file system type, and partition offset, but this does not always work. We know this is likely an Expert Witness (E01) file from the file extension, and we can use the **mmls** tool to find the remaining information:

```shell
bcadmin@ubuntu:~/bc-to-aspace-toolkit$ cd ~/
bcadmin@ubuntu:~$ mmls nps-2010-emails.E01 
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000000000   0000000001   Unallocated
002:  000:000   0000000001   0000020479   0000020479   Win95 FAT32 (0x0b)
```

This tells us there is a FAT32 file system located at sector 1 (512 bytes into the image). We can pass this information through to **tsk_recover** via Brunnhilde as follows:

```shell
bcadmin@ubuntu:~$ brunnhilde.py -z --tsk_imgtype ewf --tsk_fstype fat --tsk_sector_offset 1 -d nps-2010-emails.E01 /home/bcadmin brunnhilde-reports
```

This extracts all files into the directory **brunnhilde-reports** in our home directory, **/home/bcadmin**. Note the -z flag at the beginning, which ensures Siegfried will be run, and the -d flag before the path to the disk image. Also note the process may take some time.

Change directory into the **brunnhilde-reports** directory and examine the contents. Then, change directory into the **csv_reports** directory that it contains and examine those contents:

```shell
bcadmin@ubuntu:~$ cd brunnhilde-reports/
bcadmin@ubuntu:~/brunnhilde-reports$ ls
carved_files  dfxml.xml  report.html    tree.txt
csv_reports   logs       siegfried.csv
bcadmin@ubuntu:~/brunnhilde-reports$ cd csv_reports/
bcadmin@ubuntu:~/brunnhilde-reports/csv_reports$ ls
duplicates.csv  formats.csv         mimetypes.csv     warnings.csv
errors.csv      formatVersions.csv  unidentified.csv  years.csv
```

We need the **formats.csv** and **siegfried.csv** files to complete our transfer to our ArchivesSpace instance with **bc_to_as.py**. Prior to running the script, we need to create a folder structure that matches our repository, project, and dataset information. For this example, we will assume our ArchivesSpace instance is **clean** and contains no existing repositories.

First, move back to the home directory and create folder corresponding to a new repository:

```shell
bcadmin@ubuntu:~$ cd ~/
bcadmin@ubuntu:~$ mkdir ossarcflow_repository
```

Now, make a new project directory inside the repository directory. Repository directories can contain more than one project.

```shell
bcadmin@ubuntu:~$ cd ossarcflow_repository/
bcadmin@ubuntu:~/ossarcflow_repository$ mkdir project1
bcadmin@ubuntu:~/ossarcflow_repository$ 
```

Now, change into the project directory and make a directory corresponding to our dataset (we'll copy the CSV files we created earlier into this directory next):

```shell
bcadmin@ubuntu:~/ossarcflow_repository$ cd project1/
bcadmin@ubuntu:~/ossarcflow_repository/project1$ mkdir SET1_brunnout
```

Now copy the relevant CSV files over (note that formats.csv and siegfried.csv have different source locations):

```shell
bcadmin@ubuntu:~/ossarcflow_repository/project1$ cp ~/brunnhilde-reports/csv_reports/formats.csv SET1_brunnout/
bcadmin@ubuntu:~/ossarcflow_repository/project1$ cp ~/brunnhilde-reports/siegfried.csv SET1_brunnout/
```

**[MORE TO COME...]**

## What's in this repository

- **json-templates**: JSON templates for the relevant metadata to be created / transfered
- **sample-data**: Small sample data sets using Brunnhilde output for an existing disk image (not included)
- **bc_to_as.py**: Script to create and transfer the relevant objects via the ASpace API

## Documentation, help, and other information

Additional in-progress documentation at: https://docs.google.com/document/d/1-pU__nBYSgjHhfcxYhQLdci9rxSgo1XiXpLEFgQSkCE

## License(s)

Unless otherwise indicated, software items in this repository are distributed under the terms of the GNU General Public License v3.0. See the LICENSE file for additional details.

## Development Team and Support

Product of the OSSArcFlow research team.
