# **Running Calculations on Blanca (or Summit)**

Unless otherwise noted the symbols &lt; > indicate something you should change! Usually to a username, password, or account-specific path. The $ symbol indicates bash shell lines where the user inputs command line arguments and lines without it indicate output from a command. Do not copy the $ to the command line when you try to run the commands.

## Logging into the cluster

```bash
$ ssh <identikey-username>@login.rc.colorado.edu
```

You'll then see the prompt:

```bash
Password:
```

You may notice that when you type your password doesn't show up. Don't worry it's normal!

Enter `duo:<identikey-password>` with no spaces and you should receive a notification on the Duo app on your phone. Once you press 'OK' in the app, you're in.

## Modular Software

When you first log on to the node you'll be on a login node. Do **not** run calculations or compile code on these nodes. You can however submit jobs to the queue from them, I'll talk more about that later.

On clusters like Summit and Blanca, there are many users that all want to use different software. These packages can sometimes come into conflict with one another, for example if two users need two different versions of the same compiler. To circumvent this, softwares, compilers, and other things can be loaded and unloaded as **modules**. When you ssh into any node, unless you have modified your ~/.bashrc file no modules are loaded and you are given a "clean slate."

The modules you need to load depends on what your program/software requires. The one exception to this is the slurm/blanca and the slurm/summit modules and one of these is required to submit jobs to a queue (the module you pick depends on where you want your calculations to run, blanca or summit).

To see what modules are available you can simply type `module avail` in your terminal and a list of the available modules will be printed to the terminal. _Note_ that this list may change as you load some modules due to conflicts and dependencies so make sure to check `module avail` if you run into issues.

To load the modules just enter `module load <module-name>` in the terminal and to unload enter `module unload <module-name>`. If `module load` is too many characters for you, you can use the shortcut `ml <module-name>`.

You can check the currently loaded modules with the command `module list` or looking at the letters next to modules when you use the `module avail` command.

Luck for us the only module we'll need for now is _gaussian/16_avx2_.

## The Queue

Since a large number of people are trying to use a limited amount of resources, computing clusters usually have a queueing system to handle who gets priority. In order to run jobs on the cluster, we have to "reserve" computational assets by submitting jobs to this queue.

You will use a bash script to submit to this queue and this file will contain settings for the job as well as specifications for the resources your job requires. This could be the number of processors or nodes, the maximum amount of time for your job, etc. **NOTE: THE MORE RESOURCES YOUR ASK FOR THE LONGER IT WILL TAKE FOR YOUR JOB TO START SO DON'T BE GREEDY.**

You need to load the queue module before you can submit them to the cluster. There are two options either `slurm/blanca` or `slurm/summit`. If you have access to only one, then that's the one you have to use, but if you have a choice, try to choose the one with the fewer calculations running. you can load/unload these modules using the `module load` command discussed above. 

## Running Gaussian16 Calculations

You need two things in order to run a Gaussian16 calculation on the cluster:
  1. Gaussian16 (g16) input file (which you can generate on your local machine with Gaussview)
  2. A batch file to submit your calculation to the queue.

Shown below is an example of a batch script (which I always call `submit.sh`) to submit two g16 jobs. Note that these jobs will run sequentially. You can submit multiple calculations in a single file just remember that your max wall time applies to all your calculations together.

You can include the running of jobs outside the directory where the bash script is kept, but I strongly suggest you keep your `submit.sh` in the same directory as the g16 inputs you want to run.

```bash
#!/bin/bash
#SBATCH --nodes 1
#SBATCH --job-name ALE-Misc
#SBATCH --time=24:00:00
#SBATCH --exclusive
#SBATCH --export=NONE


module load gaussian/16_avx2


export I_MPI_FABRICS=shm:tcp

export TMP="/rc_scratch/<indetikey-username>/" # Blanca Cluster
# export TMP="/summit/scratch/<indetikey-username>" # Summit Cluster
export TEMP=$TMP
export TMPDIR=$TMP
export GAUSS_SCRDIR=$TMP


export GAUSS_MDEF=60GB
export OMP_NUM_THREADS=28 # Blanca cluster
# export OMP_NUM_THREADS=28 # Summit cluster


########################
### Run Calculations ###
########################

g16 "F[Si]1(F)(F)F[Ti](Cl)(Cl)(Cl)Cl1_ub3lyp_sdd.com"  
g16 "F[Ti]1(F)(F)F[Ti](Cl)(Cl)(Cl)Cl1_ub3lyp_sdd.com"  

exit
```

Once you have created your bash script and g16 input file you can submit your job to the queue using the the `sbatch` command from the terminal.

```bash
$ sbatch submit.sh
```

After you submit the job you can check its status in two main ways

1.  The first way lets you view all of the jobs submitted by your user name.

    ```bash
    $ squeue --user <identikey-username>
           JOBID   PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           3213539 blanca-ib ALE-Misc jasm3285 PD       0:00      1 (Resources)
    ```

2.  The second lets you view all the jobs on our group's nodes (which will include your own). This method also allows you to see how busy our nodes are and decide whether you should submit them to our nodes on summit or blanca.

    ```bash
    $ squeue | grep "blanca-sh"
           JOBID   PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           3213539 blanca-sh ALE-Misc jasm3285 PD       0:00      1 (Resources)
           3213540 blanca-sh ALE-Misc sash2428 PD       0:00      1 (Resources)
           3213541 blanca-sh ALE-Misc sash2428 PD       0:00      1 (Resources)
           3213542 blanca-sh ALE-Misc sash2428 PD       0:00      5 (Resources)
    ```

You can check if your calculation has actually converged by searching its output file (FILENAME.log) for the word Converged using the `grep` command.

```bash
$ grep -A 4 "Converged" FILENAME.log
```

## Copying files to and from the cluster

First a quick review of some basic bash commands for copying and moving.

The copying command use the following syntax `cp <file-to-copy-path> <copied-file-path>`. An example is shown below.

```bash
$ ls
file1 file2 file3
$ cp file1 file1_copy
$ ls
file1 file1_copy file2 file3
```

The mv command use the following syntax `mv <current-file-path> <new-file-path>`. An example is shown below. Note that the file at `<current-file-path >` will be gone.

```bash
$ ls
file1 file2 file3
$ cp file1 file1_new
$ ls
file1_new file2 file3
```

There is no rename command in bash, but you accomplish this using the `mv` command:

```bash
$ mv <current-name> <new-name>
```

Now that we've reviews how to move things around in bash terminals, let's talk about how to copy files from your local machine to a server (like Blanca/Summit) or vice versa. The syntax will be very similar to `cp` shown above except that we will use `scp` for secure copy. In the example below, once the command is entered a `Password:` prompt will appear, just like logging in and you will need to enter `duo:<identikey-password>`.

```bash
$ scp <file-path> <identikey-username>@login.rc.colorado.edu:<file-destination-path>
Password:
```

## Try it out

Now you should have all the tools you need to run Gaussian16 calculations on the cluster so try it out.
