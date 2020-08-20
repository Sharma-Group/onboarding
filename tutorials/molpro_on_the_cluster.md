# Running MOLPRO on Summit (or Blanca)

This tutorial is meant to give students all the necessary resources to run MOLPRO calculation on CU Boulder research clusters. If you have any questions contact James Smith.


## Installing MOLPRO
- Log on to the group MOLPRO account and go to the download page in the toolbar on the left.

- Download the latest version of MOLPRO by clicking on the link.

- Move this file to the desired location on the cluster and unpack it, for example:
```bash
gunzip molpro-mpp-2015.1.35.linux_x86_64_openmp.sh.gz
```

- Run the script and follow the instructions, note that you do not have root access on the cluster and will have to install it somewhere in your projects directory.

- And we're done! MOLPRO should be installed and ready to use.

## Generating MOLPRO input

- Generating input files from scratch can be a pain so we suggest that you use a program like [Avogadro](https://avogadro.cc/) to get started. This will format the geometry correctly and can create simple input files, which you can edit in your favorite text editor.

- Download and install [Avogadro](https://avogadro.cc/).

- Create a molecule in the viewer and then click on the `Extensions` menu in the top toolbar. From here you can select `MOLPRO` and set a few basis settings.

- Save the file and move it to the cluster, or copy it there directly.

## Running MOLPRO

- Create a directory for the job you plan to run

- Copy the bash script show below:

```bash
#!/bin/bash
#SBATCH --nodes 1
#SBATCH --job-name MOLPRO
#SBATCH --time=00:05:00
#SBATCH --export=NONE
#SBATCH --exclusive


#############################
### Environment Variables ###
#############################

export I_MPI_FABRICS=shm:tcp
export TMP="/rc_scratch/jasm3285/"
#export TMP="/scratch/summit/jasm3285/"
export TEMP=$TMP
export TMPDIR=$TMP


################################
### MOLRPO Calc Input (EDIT) ###
################################

INPUT_FILE="allene.inp" # Input filename base (no extension)
MOLPRO_EXEC="/projects/jasm3285/molpro/bin/molpro"
export OMP_NUM_THREADS=28 # Might need to change this to 24 for SUMMIT


################################
### Run MOLPRO (DO NOT EDIT) ###
################################

$MOLPRO_EXEC $INPUT_FILE  --omp-num-threads $OMP_NUM_THREADS --no-xml-output


exit
```

- Edit the three variables in the `MOLPRO Calc Input (EDIT)` section to the appropriate paths and values.

- Submit the above script as using to the queue, e.g. if the script shown above is called `submit.sh` then we would submit the job with the command:

```$ sbatch submit.sh```
