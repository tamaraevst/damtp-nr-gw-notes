# Pre-workshop instructions

1. Please request Fawcett account [here](https://www.maths.cam.ac.uk/computing/faculty-hpc-system-fawcett).
2. Please configure your ssh access appropriately, by using instructions [here](https://www.maths.cam.ac.uk/computing/fawcett-ssh-access).
3. Make sure you can login to Fawcett with no issues. 
4. Download all the data files in the `download` directory of this repository into your home directory on Fawcett. 
5. Please have some sort of plotting tool on hand, I personally use `gnuplot` for quick plotting, which can be easily installed using `homebrew`.
6. Please download Visit 2.13 on your laptop (executables can be found [here](https://sd.llnl.gov/simulation/computer-codes/visit/executables) 

# GRChombo
GRChombo is an open-source code for numerical relativity simulations. It is written entirely in C++14, using hybrid MPI/OpenMP parallelism and
vector intrinsics to achieve good performance on the latest architectures.
Furthermore, it makes use of the Chombo library for adaptive mesh refinement
to allow automatic increasing and decreasing of the grid resolution in regions
of arbitrary shape and topology.

Below is a (not exhaustive) list of relevant literature you should read at some point, if you would like to work with the code or understand its aspects in more detail:

1. Lessons for adaptive mesh refinement in numerical relativity: https://arxiv.org/abs/2112.10567
2. GRChombo : Numerical Relativity with Adaptive Mesh Refinement: https://arxiv.org/abs/1503.03436
3. Katy's thesis with lots of insights into numerical relativity and GRChombo: https://arxiv.org/abs/1704.06811

Please also consider joining the Slack channel, where you can ask questions about the code and hear about the latest updates in the community. 

## Where do I find the code?

All of the public codes can be found by visiting the page of the GRTLCollaboration: https://github.com/GRTLCollaboration

Over the last few years, GRChombo's capabilities have been immensely expanded and the following public versions/extensions are now available:

1. [GRChombo](https://github.com/GRTLCollaboration/GRChombo/tree/main) -- the original version of the code with capabilities of evolving binary black hole systems in GR, single Kerr black hole and a scalar field on Kerr background. 
2. [GRDzhadzha](https://github.com/GRTLCollaboration/GRDzhadzha) -- a code to evolve matter on curved spacetimes with an analytic time and space dependence.
3. [GRFolres](https://github.com/GRTLCollaboration/GRFolres) -- for evolving balck hole spacetimes in modified theories of gravity, including the four-derivative scalar-tensor theory of gravity and Cubic Horndeski theory of gravity.
4. [TwoPunctures](https://github.com/GRTLCollaboration/TwoPunctures) -- Bowen-York initial data for two puncture black holes using a single domain spectral method, to be used along with GRChombo source code.

In this workshop, we will focus on compiling and runnig the original version of GRChombo, i.e. the first item in the above list. 

## Fawcett Tutorial

This is an adapted version of Miren's/Amelia's workshop, which can be found here: https://github.com/GRTLCollaboration/GRChombo/tree/training/fawcett 

We will assume that you have an account set up on Fawcett, and that you are happy with using GitHub. You can find more information about Fawcett [here](https://www.maths.cam.ac.uk/computing/faculty-hpc-system-fawcett). Please load all the files in the `download` folder of this repository into your home directory on Fawcett. The files to load are:

1. Fawcett.Make.defs.local
2. fawcett-modules.sh
3. fawcett-jobscript
4. params.txt

To run simulations with GRChombo, first we need to get a copy of both the
Chombo and GRChombo repositories.

### Getting Chombo and GRChombo

The GRChombo copy of the Chombo repository can be found
[here](https://github.com/GRTLCollaboration/Chombo). To get a copy, clone it using a
command such as
```bash
git clone https://github.com/GRTLCollaboration/Chombo.git ~/Chombo
```
e.g. to clone into your home directory (which we will assume below).

Next, clone the GRChombo repository using a command
such as
```bash
git clone https://github.com/GRTLCollaboration/GRChombo.git ~/GRChombo
```

Now we can start building the Chombo libraries

### Building Chombo
Before we can build the Chombo libraries, we need to first set the configuration
variables that the build system uses. These are set in the file
`Chombo/lib/mk/Make.defs.local`. This will differ for each system you build on,
but we try to add this file for clusters we have used \[GR\]Chombo on before in `GRChombo/InstallNotes/MakeDefsLocalExamples`

There is a ready-to-use Fawcett.Make.defs.local file in the `download` folder of this repository for use on the `cosmosx` and `skylake`
partitions on Fawcett. You should now copy this to the right place in your Chombo
directory using the command: 
```bash
cp ~/Fawcett.Make.defs.local ~/Chombo/lib/mk/Make.defs.local
```
For more details on what the variables are in the Make.defs.local file, have a
look at
[this wiki page](https://github.com/GRTLCollaboration/GRChombo/wiki/Compiling-Chombo).

Next, we need to load all the necessary modules. We will use the Intel compiler with
Intel MPI and a compatible HDF5 module. This can be done by running: 
```bash
source ~/fawcett-modules.sh
```
Note that you can check currently loaded modules in your environment using 
```bash 
module list
```
Modules can be removed using 
```bash
module unload <module_name>
```
Modules available on the cluster can be found via
```bash
module avail
```

Finally we can build the Chombo libraries with
```bash
cd ~/Chombo/lib
make lib -j <num tasks>
```
Here a sensible number of tasks will depend on whether you are building
on the login node (in which case choose a small number such as 4) or if you are
in an interactive job (which case feel free to use up to about twice
as many tasks as you have cores available).

### Building the GRChombo BinaryBH example
First we need to let GRChombo know where to look for the Chombo libraries.
This is done with the `CHOMBO_HOME` environment variable and can be set using
the command
```bash
export CHOMBO_HOME=${HOME}/Chombo/lib
```
You might want to add this line to your `~/.bashrc` file so that every time you
start a new shell, the environment variable is automatically set.

Now we are ready to build with the commands
```bash
cd ~/GRChombo/Examples/BinaryBH
make all -j <num tasks>
```
Assuming everything compiled and linked sucessfully, you should have an
executable called something like (the name will change depending on the Chombo
configuration variables)
```
Main_BinaryBH3d_ch.Linux.64.mpiicpc.ifort.OPTHIGH.MPI.OPENMPCC.ICPX2022.0.ex
```

### Running the example
In general, on most computing clusters, you should not run jobs from your home
directory.
Usually there is some separate parallel storage to be used for simulations
(often called the "scratch" partition) and you should consult the documentation
of the particular cluster on where to find it. On Fawcett, this is
the `/nfs/st01/hpc-gr-epss` directory. First, let's create a subdirectory
for our simulation
```bash
mkdir -p /nfs/st01/hpc-gr-epss/${USER}/BinaryBH
```
and let's copy our binary across to the parent of this new directory using 
a command such as
```bash
cp ~/GRChombo/Examples/BinaryBH/Main_BinaryBH3d_ch.Linux.64.mpiicpc.ifort.OPTHIGH.MPI.OPENMPCC.ICPX2022.0.ex \
/nfs/st01/hpc-gr-epss/${USER}/BinaryBH/
```
We will use Miren's parameter file from previous trainig sessions. It contains initial data
for the quasicircular inspiral of an equal-mass BH binary which should do
about 10 orbits (though due to low resolution and approximate initial data,
it will probably be a fair number fewer - it would also take around 2 days to
finish). Let's copy it across to our simulation
directory that we created:
```bash
cp ~/fawcett-params.txt /nfs/st01/hpc-gr-epss/${USER}/BinaryBH/params.txt
```
We will need to create a jobscript to submit to the scheduler.
```bash
cp ~/fawcett-jobscript /nfs/st01/hpc-gr-epss/${USER}/BinaryBH/jobscript
```
You will need to edit a few lines in this template before you can submit it.
Use your favourite available text editor to provide the full path to the
executable i.e. edit the line 
```bash
EXEC="/nfs/st01/hpc-gr-epss/<crsid>/BinaryBH/Main_BinaryBH3d_ch.Linux.64.mpiicpc.ifort.OPTHIGH.MPI.OPENMPCC.ICPX2022.0.ex"
```
where `<crsid>` is your username. Once you are happy,
save and exit your editor and submit it to the scheduler using
```bash
cd /nfs/st01/hpc-gr-epss/${USER}/BinaryBH && sbatch jobscript
```
Assuming there are available resources, the job should start shortly.
You can query the status of the job with the command
```bash
squeue -u $USER
```
Within the first minute of the job starting, you should be able to see the
output from each MPI rank in `pout/pout.n` where `n` is the rank number.
To monitor the output of this, you can use the command
```bash
tail -f pout/pout.0
```
Note that there isn't much output during the initial setup after the loading of
parameters so don't panic if it appears nothing is happening for a bit.
You should also check the main job output in the `slurm-<jobid>.out` file
to check that it hasn't crashed.

Congratulations, you have just run your first GRChombo simulation!

### Visualisation in Paraview

In this section, we will outline how to get set up with visualisation using
Paraview on Fawcett. It is a bit fiddly, so we will go over the bare minimum
to get up and running. For more detailed information on what Paraview can do,
have a look at the
[tutorial](https://www.paraview.org/Wiki/images/b/bc/ParaViewTutorial56.pdf)
(or ask Amelia!). Adapted from instructions provided by Kacper Kornet.

First, we need to set up _direct_ ssh access to fawcett (i.e. not by manually `ssh`ing into a 
maths/DAMTP computer first). Instructions on how to do this
can be found [here](https://www.maths.cam.ac.uk/computing/fawcett-ssh-access). Please set this up
in advance of the tutorial.

Second, we need you to download Paraview v5.6.0 (note that is important that
you download this specific version and not a different one) onto your local machine i.e. the
laptop/desktop you are using to access fawcett. This can be done
[here](https://www.paraview.org/download/). Please do this in advance of the tutorial.

Before we tunnel to fawcett, we need to add a server on the Paraview
client. To do this, open Paraview, and click the connect icon near the top left corner,
then 'Add Server'. Change the name to whatever you like e.g. 'localhost'. All
other settings should be the default:  

Server type: Client/Server  
Host: localhost  
Port: 11111  

Click Configure. On the next screen, set 

Startup Type: Manual

Accept by clicking Save and Close. Once you have set this up, you won't need
to do it again.

Now we need to start the Paraview server on fawcett. Ssh into fawcett - as
outlined above, this should already be configured with ssh forwarding, and so
shouldn't require explicit login via another machine. Load Paraview using

```bash
module load paraview/5.6.0/upstream
```

Now run the command
```bash
pvserver
```

It should print something like:

```bash
Waiting for client...
Connection URL: cs://mn01:11111
Accepting connection(s): mn01:11111
```

Note that the number 11111 can be different if someone else
is already running pvserver. In what follows, we will refer to it as `<fawcett_port>`.

(For large datasets, it is a good idea to submit a job to the queue and run
pvserver with a modified environment

```bash
OSPRAY_THREADS=8 KNOB_MAX_WORKER_THREADS=8 pvserver
```

where 8 is the number of cores reserved for the job. However for the purposes
of this tutorial we should be fine without.)

We next need to set up the tunnel to fawcett. On your local machine, run

```bash
ssh -NL 11111:localhost:<fawcett_port> crsid@fawcett.maths.cam.ac.uk
```

It may ask for authentication and then look as though it has 'hung' (i.e. no prompt).

On your local Paraview client, again click  the connect icon and choose the connection
configured in the earlier step. This should now have opened a connection to
fawcett that you can use to access your data using `File->Open`. 
You are now ready to start visualising your binary black holes! 
We will go over some basics of using Paraview in the hands-on tutorial.

For more detailed information, consult the
[wiki](https://github.com/GRChombo/GRChombo/wiki/).
