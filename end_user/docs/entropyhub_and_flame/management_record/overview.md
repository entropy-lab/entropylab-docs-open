# Management of records

## Components

## Job submission, tracking and versioning

## Raw data explorer

Once the job is finished, runtime saves all data (node outputs with retention
period set to 2) into a single HDF5 file per job, together with all metadata
about the job. These files can be seen in **Data** panel of the Entropy Hub.
Select from the list experiment run, and use HDF5 Explorer to explore the
content of the file.

Each file root, node and data saves metadata information about job, that node,
and output descriptions respectively. These can be seen by selecting **Inspect**
switch.

Note that HDF5 Explorer can visualise only native HDF5 data, which are
N-dimensional arrays. To save such from Python NodeIO output you can use

```python
output.set(some_output_value=2)
output.set(some_output_1D_array=[1,2,3])
output.set(some_output_2D_array=[[1,2],[3,4]])
```

Setting output multiple times will generate an additional 1D array in the HDF5
output. Timestamps when the data records are written are saved in
`<value name>_time` variables

Any other JSON compatible JSON dictionary will be saved as string, and they can
be accessed directly programmatically for use and analysis in other programs -
see
[Accessing data when entropy hub is not running](#accessing-data-when-entropy-hub-is-not-running)
below

## Looking at data with user-defined views

## Notebook

## Data export & backup

See the follwing section.

## Accessing data when entropy hub is not running

All data is stored in the `data_store` folder. Experimental data, together with
all relevant metadata about the workflow and job that produced the data, is
stored in `data_store/<project prefix>/<jobid>.hdf5`, with one file
corresponding to one job (one experimental run).

Some additional information is saved for debugging purposes: standard output and
standard error output of all the nodes participating in the workflow are saved
for each job under `data_store/project prefix>/<job id>/<node name>.log`.

Finally workflow files and parameter files are saved in corresponding git
repositories `data_store/git/<project prefix>/workflow.git` and
`data_store/git/<project prefix>/params.git` respectively. HDF5 data files as
attributes of the root contain information on the job, including corresponding
git SHA1 commit tags for workflow and parameters that can be used for manual
extraction of corresponding run, when Entropy Hub is not running (when Entropy
Hub is running, this is all automatically done from the web GUI).
