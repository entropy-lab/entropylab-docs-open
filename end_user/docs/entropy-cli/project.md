# An entropy project
 
An entropy project is defined as a directory which has a hidden subdirectory inside called `.entropy`. 

To initialize a new project, open the command line and change into the directory you wish to work from.
Then type `entropy init`. 

This generates the `.entropy` directory and populates it with a directory called `hdf5`
and a file called `entropy.db`.

## The database

The `entropy.db` file is a sqlite database. Sqlite is a simple, lightweight 
database that does not require a server, or any setup for that matter. 
However, entropy can also be used with different SQL DB variants. This is
made simple because the implementation is built with [SQL Alchemy](https://www.sqlalchemy.org/). If you need support 
for a different DB, please open an [issue on the entropy github repository](https://github.com/entropy-lab/entropy/issues).

The database is where we save details about the experimental runs: who ran the experiment, when did they run 
it, where the results are etc. 

In addition, the database contains information about instruments and other resources you registered. More information 
about resources can be found in the `Pipeline` section of the documentation.

## HDF5

The HDF5 file format is a way to save, potentially very large, datasets. It 
is a good selection for saving experimental results for several reasons. 
It allows to access parts of the file without loading it all to memory, so even multi gigabyte 
data sets (or much more) are not a problem. In addition, it allows keeping metadata 
alongside the data (units for example) and is cross-platform. You can read more [here](https://www.hdfgroup.org/solutions/hdf5/).

Entropy saves a single HDF5 file per experimental run. These files are all contained in the HDF5 folder and named according
to their run identifier. This makes data sharing quite easy - pick your HDF5 file and email it to your favorite collaborator. 

## Database migrations

In the database world, if you change the database in some way you need to _migrate_ it. 
This does not mean changing the DB by adding data to it, it means a change to how the data is stored or what data is stored. 
So if the database gets some new tables, new columns to a table or if something is removed then the data needs to be migrated 
from the old _schema_ to the new. 

If you have an existing DB file and we've made changes to the DB, you can use the `upgrade` CLI command to migrate it to a new version.
If you updated `entropylab` and you need to do this you will know from either the release notes or because of an error message. 

Running the migration is as simple as running `entoropy upgrade` in the project folder.

