# decahose-filter
The decahose-filter tool is a general tool developed at the University of Michigan and written in PySpark, and it can be used to filter Twitter Decahose data to create a that match a keyword list. The purpose of this tool is to provide a way to generate small subsets of data of interest as a first, preprocessing, step for research. This allows users to avoid the expensive computational effort of filtering the data each time in order to perform further analysis. Therefore, this tool is most useful for subsets of data that will be used multiple times since the filtering would only need to be performed once.

## Tool Location
The tool can be cloned from this repository or found in Turbo storage at the following location:
```bash
/nfs/turbo/twitter-decahose/tools/decahose-filter/decahose_filter.py
```

## How to Run the Tool
The decahose-filter tool is a simple command-line utility that can be run on the Cavium-ThunderX Hadoop cluster or on the Great Lakes cluster. There are only three command-line arguments that must be supplied to the tool:

* keyword-file - This is the path to the file containing a list of keywords of interest to filter the data. This should be a plaintext file with each keyword on a separate line. This argument can be specified as `--keyword-file` or simply as `-k`
* input - This is the path to the input file(s) to be processed. This can be a HDFS path (if using Cavium-ThunderX) or a linux path (such as /nfs/turbo or a local file). This argument can be specified as `--input` or simply as `-i`.
* output - This is the path to the output directory under which the resulting subset of data be stored. If using Cavium-ThunderX, this will be an output directory in HDFS, and if using Great Lakes, this will be an output directory on the Linux filesystem. This argument can be specified as `--output` or simply as `-o`.

### Running the Tool on Cavium-ThunderX
The tool can be run with the `spark-submit` command on Cavium-ThunderX such as the following example:
```bash
spark-submit --master yarn --num-executors <REQUESTED_NUM_EXECUTORS> --executor-memory <REQUESTED_EXECUTOR_MEMORY> --executor-cores <REQUESTED_EXECUTOR_CORES> /nfs/turbo/twitter-decahose/tools/decahose-filter/decahose_filter.py  -k <PATH_TO_KEYWORD_LIST_FILE> -i <INPUT_PATH> -o <OUTPUT_PATH_IN_HDFS>
```
Replace `<REQUESTED_NUM_EXECUTORS>`, `<REQUESTED_EXECUTOR_MEMORY>`, `<REQUESTED_EXECUTOR_CORES>`, `<PATH_TO_KEYWORD_LIST_FILE>`, `<INPUT_PATH>`, and `<OUTPUT_PATH_IN_HDFS>` with appropriate values for your particular filtering job that you are running. Note that you will need to request an appropriate number of cores and memory depending on the size of your data and how much parallelism you would like to request for speeding up processing. As a general rule of thumb, the total memory requested should be at least the size of the input data in gigabytes (GB).

As a concrete example, the following would run the tool for all the files for the date 2020-07-04, using the keyword list file `COVIDTerms.txt` and output the data under the path `/user/arburks/decahose_filter_test` in HDFS:

```bash
spark-submit --master yarn --num-executors 10 --executor-memory 4g --executor-cores 4 /nfs/turbo/twitter-decahose/tools/decahose-filter/decahose_filter.py -k COVIDTerms.txt -i /data/twitter/decahose/2020/decahose.2020-07-04*bz2 -o /user/arburks/decahose_filter_test
```

Also, note that the input file is assumed by Spark/Hadoop to be located in HDFS. If we wanted to use a file located on the Linux filesystem, we could add `file://` at the beginning of the path.

### Running the tool on Great Lakes
The tool can be run on the Great Lakes cluster either by submitting it as a batch job using the `sbatch` command or running it in an interactive job that was started with the `srun` command.

The following is an example sbatch script for running the tool on Great Lakes as a batch job. Note that you will need to specify your root account name and set the requested resources as needed for your job. Note that you should request one more core (cpus-per-task) than the total number of executors that you wish to have. This extra core will be used by the PySpark driver. See https://arc.umich.edu/greatlakes/user-guide for more general information on running jobs on Great Lakes.

```bash
#!/bin/bash
#SBATCH --job-name JOBNAME
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --cpus-per-task=21
#SBATCH --mem-per-cpu=8g
#SBATCH --time=02:00:00
#SBATCH --account=test
#SBATCH --partition=standard
#SBATCH --mail-type=BEGIN,END

module add python3.7-anaconda
module add spark/3.0.3

spark-submit --num-executors 20 --executor-memory 8g /nfs/turbo/twitter-decahose/tools/decahose-filter/decahose_filter.py/ -k /home/arburks/COVIDTerms.txt -i file:///nfs/turbo/twitter-decahose/decahose/raw/decahose.2020-07-31.* -o /home/arburks/decahose_filter_test
```

The above sbatch script could then be run as a batch job on Great Lakes by running the following command (assuming that the script is named `decahose-filter.py`:
```bash
sbatch decahose-filter.py
```

Similarly, if desired, the script can be run interactively in on Great Lakes with an interactive job using the `srun` command. Run the `srun` command, specifying all the necessary resources being requested, and then once the interactive session begins, the tool can be run by entering the last three lines of the above script.

