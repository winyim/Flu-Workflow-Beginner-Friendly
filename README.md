# Flu Workflow (Beginner-Friendly)
Mubareka Lab Flu Sequencing Bioinformatics Workflow

## Commonly used commands in Unix shell
<div align="center">

|Command|Short For|Purpose
:-------------------------:|:-------------------------:|:-------------------------:
`ls`|listing|view contents of current directory
`pwd`|print working directory|outputs current working directory
`cd`|change directory|allows you to change your working directory
`mkdir`|make directory|make a new folder in your current directory
`mv`|move|move folders or files to a target directory
`rm`|remove|removes a file <br /> `-r` (recursive) parameter can be used to remove a directory/folder.
`cp`|copy| copies a file/folder(with -r flag).
`.`|here|used to indicate current directory
`..`|one up|used to indicate parent directory (directory one level up)
`cat`|concatenate|create files, view contents of files, concatenate files
`nano`|GNU nano|command line text editor

</div>

`cd` examples:

- This will change your current directory to the `Downloads` folder, given this folder is listed in your current directory.

    ```bash
    cd Downloads
    ```

- or you can use `..` to move to the parent  of the current directory. (move up one level)

    ```bash
    cd ..
    ```

Usage of `mkdir`:

- make a directory called `example_folder`

```bash
mkdir example_folder
```

Usage of `mv`:

```bash
mv [folder/filename] [destination]
```

>_The_ `mv` _command can also be used to rename file and folder names._

```bash
mv  [folder/filnename] [newname]
```

# Setting up your workspace

## Windows Subsystem for Linux (WSL) 

[WSL Installation Guide]

If you are using a Windows system, WSL would be required.  First,  we need to install a Linux distribution from the Microsoft Store.  

>Open Microsoft store and search for "Debian".  Click "Get".

>After its downloaded, click "Open".  Wait for the installation and follow the prompts.

Now we can check that we have set up our WSL environment correctly.

>Open a command prompt terminal and enter the following. 

```bash
wsl -l -v
```
- `-l` lists distributions
- `-v` 'verbose', shows detailed information about all distributions

The expected output should be:

```bash
NAME    STATE      VERSION
Debian  Stopped    2
```
You have no succesfully installed WSL! 

To enter WSL all you need to do is type:

```bash
wsl
```
## Installing tmux 
[tmux] allows you to multitask in terminal by creating multiple panes.

To install:

```bash
sudo apt-get install tmux
```

Before we enter our tmux environment, let's copy a tmux configuration for better shortcuts to our `$HOME` directory. 

```bash
cp /mnt/q/Winfield/scripts/.tmux.conf ~/
```

Now we can enter tmux enviroment:

```bash
tmux
```

>To view the shortcuts `nano ~/.tmux.conf`

<div align="center">

Common tmux shortcuts
|tmux shortcut|Purpose|
:-------------------------:|:-------------------------:
|ctrl + a| bind-key (activates tmux command)|
|bind-key h | horizonal split|
|bind-key v | vertical split|
|alt + arrowkey |select pane in the direction of arrowkey|
|ctrl + D| closes a pane|

</div>

## Install Conda

We will be following [this guide] to install Miniconda on Linux. 

Download the "Miniconda3 Linux 64-bit" installer for Python 3.9. Move the installer to your `$HOME` directory. 

Then run the installer making sure you are in your `$HOME` directory.

```bash
bash Miniconda3-py39_4.12.0-Linux-x86_64.sh
```

>Follow the Miniconda installer prompts. 

To check your installation you will have to restart your WSL or reload your ~/.bashrc file.
This file is a script file that is executed upon a user logging in.  So instead of re-logging in we can simply reload it (AKA re-sourcing it)

```bash 
source ~/.bashrc 
```

Now we can check that Conda is installed.

```bash
conda
```
>This should output the help/man page for Conda. If this doesn't show up you have done something wrong.

## Install Bioconda

After installing Conda, we need to add the Bioconda channel to our conda. Conda channels are the locations where packages are stored. Bioconda will give us access to packages we will need in the future. [Here](https://bioconda.github.io/conda-package_index.html#) is a list of packages Bioconda provides.

Adding Bioconda:

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority flexible
```
Then update Conda:

```bash 
conda update conda
```

## Create a Conda environment and install our packages

Conda environments are self-contained and help manage dependencies while keeping projects isolated. 

The following will create a new environment and install FastQC, Trim-Galore and IRMA.  We will go through each package separately later in this guide.
```bash
conda create -n [env_name] -c bioconda fastqc trim-galore irma
```

# It's GO TIME!

## Set up our working directory

It is finally time to begin an analysis! First we activate our Conda environment.

```bash
conda activate [env_name]
```

Let's copy our data to our `$HOME` directory.

```bash
cp /mnt/q/Winfield/jbs_test ~/
```
Enter the directory after copying it. 

```bash
cd ~/jbs_test
```


## Step 1 - Check quality of sequence data with FastQC

[FastQC Github](https://github.com/s-andrews/FastQC)

FastQC is available through a graphical interface and can be downloaded for `use` on Windows.  Available [here]. But for our purposes we will be using it through command line.

First we will make a directory to store our FastQC output:

```bash
mkdir fastqc_raw
```

Next we can run FastQC on  our raw reads:

```bash
fastqc /reads/dir/*.fastq.gz -o fastqc_raw
```

- `/reads/dir/` replace with directory that contain the Illumina raw reads
- `*.fastq.gz` this just tells FastQC to run quality checks on anything inside the folder that ends with the ".fastq.gz"
- `-o fastqc_raw` indicates the output folder you want to place all your FastQC results

We can now view our Fastqc results.

```bash
cd fastqc_raw
open [html_filename]
```

Pay attention to the per base sequence quality and sequence length distribution.
Here are some examples of what to look for in the FastQC results:
|Good|Bad|
:-------------------------:|:-------------------------:
![goodexample](https://user-images.githubusercontent.com/95540789/196755849-9e052b09-8d07-4d9d-9150-71231bf40c68.JPG)|![badexample](https://user-images.githubusercontent.com/95540789/196755876-33c6a36b-6059-4166-8040-4ece29513366.JPG)
![gooddist](https://user-images.githubusercontent.com/95540789/196755903-e8244189-8f11-4cd4-a147-49778464c121.JPG)|![badexample](https://user-images.githubusercontent.com/95540789/196755921-875f5985-486b-4659-85a7-314fba49aec2.JPG)

## Step 2 - Quality trim and remove adapters with Trim-Galore

[TrimGalore Github](https://github.com/FelixKrueger/TrimGalore)

Trim-Galore uses both [Cutadapt] and FastQC.  Trim-Galore's default setting for Phred score cutoff is 33. If higher Phred score is necessary use `--phred64` parameter.

First we need to make a new directory for FastQC to output the results.

```bash
mkdir fastqc_trimmed
```

Then we can run Trim-Galore with the following parameters.

```bash
 trim_galore --stringency 5 --dont_gzip  --length 30 --fastqc --fastqc_args "-o fastqc_trimmed" --paired -a CTGTCTCTTATA  -a2 CTGTCTCTTATA  -o [outputfolder] [READ1] [READ2]
```

- `--stringency 5` overlap with adapter sequence required to trim a sequence. 5 base pairs of overlapping sequence will be trimmed off the 3' end of any read
- `--dont_gzip` output files wont be gzip compressed
- `--length 30` discard reads shorter than 30bp
- `--fastqc` run FastQC in default mode on the resulting fastq file once trimming is complete
- `--fastqc-args` passes arguments to FastQC
- `-o fastqc_trimmed` argument for FastQC indicating the output foldername
- `--paired` performs length trimming of quality/adapter trimmed reads for paired-end files
- `-a` adapter sequence to be trimmed
- `-a2` adapter sequence to be trimmed off read2 of paired-end files
- `-o` all outputs will be written into specified directory instead of current directory

Once Trim-Galore has finished, check resulting FastQC outputs to check quality of reads and whether or not any adapters are present after trimming.
Example of FastQC results of before and after Trim-Galore trimming (adapters should be removed):
Before             |  After
:-------------------------:|:-------------------------:
![beforetrim](https://user-images.githubusercontent.com/95540789/196755664-53c5f524-ae44-40b7-bb44-fc9d67cf54bd.JPG)|![aftertrim](https://user-images.githubusercontent.com/95540789/196755727-9ffe53eb-c0b6-4918-babd-b30a5fe5bdd0.JPG)


## Step 3 - IRMA: Iterative Refinement Meta-Assembler

[IRMA Usage](https://wonder.cdc.gov/amd/flu/irma/)
[IRMA Manuscript](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-016-3030-6)
To run IRMA you need to (1) indicate the module (our case `FLU`), (2) the paired-end trimmed illumina reads you wish to run and (3) the output name for the given sample. But first you need to be in the directory where your trimmed fastqs are.

```bash
IRMA FLU [READ1]_R1.fq [READ2]_R2.fq [SAMPLE_NAME]
```

After IRMA has been completed, you can explore the directories that it has created.

<!-- ## Bash/Python Scripting for Batch Resulting

As you may have noticed, steps 2 and 3 can only be performed on one sample at a time. Obviously this is not ideal when trying to result multiple samples at the same time. -->


  [here]: <https://www.bioinformatics.babraham.ac.uk/projects/fastqc/>
  [Cutadapt]: <https://github.com/marcelm/cutadapt>	
  [WSL Installation Guide]: <https://learn.microsoft.com/en-us/windows/wsl/install>
  [tmux]: <https://github.com/tmux/tmux/wiki>
  [this guide]: <https://conda.io/projects/conda/en/latest/user-guide/install/linux.html>
