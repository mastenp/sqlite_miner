# SQLite Miner
By: Jon Baumann, [Ciofeca Forensics](https://www.ciofecaforensics.com)

## About
This script mines SQLite databases for hidden gems that might be overlooked. It identifies for the forensic examiner which databases, tables, and columns had which potential types of files within them. For an explanation of why I wrote this script, please see [this blog entry](https://www.ciofecaforensics.com/2017/10/23/mining-hidden-gems-with-sqlite-miner/).

## How It Works
This script searches identified SQLite databases to find files that are hidden in blob objects within the database. The `fun_stuff.pl` file controls the regular expressions that will be matched to assert that a given blob is a given file type. SQLite Miner supports magic numbers at any offset.

## Usage
### Individual Files
This script is run by perl on a command line. The easiest usage is to look at one SQLite database, which is accomplished by running `perl sqlite_miner.pl --file=<path to SQLite database>`. When this is run, the script will create a folder in the output folder named `YYYY_MM_DD_<database_name>`. For example, running this on NotesStore.sqlite today will generate `2017_10_21_NoteStore.sqlite`. Importantly, at the beginning of the run, the script will copy the target SQLite database into this folder and work from the copied database, instead of the original. Also within that folder will be a file `results.csv` that contains a line-by-line list of each blob that is identified as potentially beng a known file. If the `--export` option is set, the folder will also contain an export folder that has all of the files which were recognized saved within them. If the `--decompress` option is set, the copied database will be updated with any decompressed data that is identified.

### Entire Directories
For a larger outlook, the script can be run to recursively look at an entire directory with `perl sqlite_miner.pl --dir=<path to directory>`. That will cause the script to recursively walk through every file under that directory, check the file's header to see if it is SQLite format 3, and run each identified SQLite file as if it had been done using the `--file=` option above. The only difference is all of the results from that entire folder will be stored under an output directory named `YYYY_MM_DD_<folder_name>`. For example, running this on /home/test/backup_unpacked today will generate `2017_10_21_backup_unpacked/`. The `results.csv` will contain all results from the entire directory, but each specific database will have its own output folder within the overall directory.

### Android Backup Files
The script can also be used to open an Adnroid backup without unpacking it first with `perl sqlite_miner.pl --android-backup=<path to backup>`. That will cause the script to open the Android backup, decompress the TAR portion to a temporary folder, then iteratively walk through the TAR file, exporting any files that have a SQLite format 3 header. At that point the script will behave the same as using the `--dir=` option described above. When done, the script will remove the temporary folder that the TAR file was saved in, but please be aware that for large, full, backups, this may result in an additional 1GB+ of space (the same amount of space as if you decompressed the file in order to run the `--dir=` option against an actual folder). 

In addition, while this script attempts to cut down on memory usage, for large files tar may run out of memory and crash. If that occurs, decompress the file separately, then run this script with the `--dir` option on the exported data. Finally, decompressing the Android backup and working through the TAR will add time on to the script's processing. For example, a 1GB backup.ab file took roughly an additional 45 seconds to run against the Android backup, as compared to the exported files on a not-very-powerful computer. That said, decompressing the files takes roughly as long.

### Protobufs
If you want to try to find protobufs, use the `--protobufs` switch on the command line. This is somewhat experimental code and is not optimal for large databases as it literally tries to parse every blob as a protobuf and records when a blob correctly parses. It requires you to have the `protoc` [package](https://github.com/protocolbuffers/protobuf) installed. This may have false positives for binary data which appears to start like a protobuf. 

If you are on Windows, the easiest way to install protoc is to download [this package](https://github.com/protocolbuffers/protobuf/releases/download/v3.13.0/protoc-3.13.0-win64.zip) and put the file `bin/protoc.exe` into `C:\Windows\system32`. 

### Options
The required options that are currently supported are (one of):
1. `--file=`: This option tells the script where to find the SQLite you want to mine. 
2. `--dir=`: This option tells the script where to find a directory to recursively search for SQLite format 3 database files and to parse each of them as if the `--file` option was called on them above.
3. `--android-backup=`: This option tells the script where to find an Android backup file to recursively search for SQLite format 3 database files and to parse each of them as if the `--file` option was called on them above.

The optional arguments are:
1. `--decompress`: This option tells the script to decompress any compressed data it knows it can unpack and replace the original data with the decompressed data to provide the examiner with a plaintext view. Note, this option drastically increases the run time as now the script is reading in the comrpessed object, decompressing it, and writing it back into the database.
2. `--export`: This option tells the script to export any of the files it recognizes and saves them in an `export` folder with an appropriate file extension. Note, this option can drastically increase the size of your results, especially when used in conjunction with the `--decompress` option.
3. `--help`: This option prints the usage information.
4. `--output=`: This option sets the output directory to not be the default `output/`.
5. `--verbose`: This option provides more feedback about the script as it runs.
6. `--very-verbose`: As above, but more so.
7. `--protobufs`: This option turns on a brute force check of every blob to see if it parses as a protobuf. This may have false positives. 

## Requirements
This script requires the following Perl packages:
1. Archive::Tar
1. Data::Dumper
2. File::Copy
8. File::Find
4. File::Path
3. File::Spec::Functions
4. Getopt
5. IO::Uncompress
6. POSIX
7. Time::HiRes

