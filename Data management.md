# **Data management** *i.e. why so many files D:*

Suggested read/based off: [hbctraining](https://github.com/hbctraining/Intro-to-rnaseq-hpc-salmon-flipped/blob/main/lessons/01_intro-to-RNAseq.md)  

## **The Dataset** *i.e. this is your plastilina* 
The data I am using comes from a pilot study on schizophrenia that measures differences between case and control samples in a specific tissue.  
**Raw data**
The raw data can be found in the cluster. I.e. Only I have it.
**Metadata**
In addition to the raw sequence data we also need to collect information about the data, also known as metadata. We are usually quick to want to begin analysis of the sequence data (FASTQ files), but how useful is it if we know nothing about the samples that this sequence data originated from?

This is the preliminary info we have on our data:



## **Planning and organization** *i.e. folder fengshui*
For each experiment you work on and analyze data for, it is considered best practice to get organized by creating a planned storage space (directory structure). We will start by creating a directory that we can use for the rest of the workshop. First, make sure that you are in your home directory.

```
rnaseq
  ├── logs
  ├── meta
  ├── raw_data  
  ├── results
  └── scripts
```

This is a generic structure and can be tweaked based on personal preference and the analysis workflow.

- **logs**: to keep track of the commands run and the specific parameters used, but also to have a record of any standard output that is generated while running the command.
- **meta**: for any information that describes the samples you are using, which we refer to as metadata.
- **raw_data**: for any unmodified (raw) data obtained prior to computational analysis here, e.g. FASTQ files from the sequencing center. We strongly recommend leaving this directory unmodified through the analysis.
- **results**: for output from the different tools you implement in your workflow. Create sub-folders specific to each tool/step of the workflow within this folder.
- **scripts**: for scripts that you write and use to run analyses/workflow.

```
mkdir logs meta raw_data results scripts
```


## **Documentation**
In your lab notebook, you likely keep track of the different reagents and kits used for a specific protocol. Similarly, recording information about the tools used in the workflow is important for documenting your computational experiments.

- Make note of the software you use. Do your research and find out what tools are best for the data you are working with. Don't just work with tools that you are able to easily install.
- Keep track of software versions. Keep up with the literature and make sure you are using the most up-to-date versions.
Record information on parameters used and summary statistics at every step (e.g., how many adapters were removed, how many reads did not align)
- A general rule of thumb is to test on a single sample or a subset of the data before running your entire dataset through. This will allow you to debug quicker and give you a chance to also get a feel for the tool and the different parameters.
- Different tools have different ways of reporting log messages to the terminal. You might have to experiment a bit to figure out what output to capture. You can redirect standard output with the > symbol which is equivalent to 1> (standard out); other tools might require you to use 2> to re-direct the standard error instead.

## **README files**
After setting up the directory structure it is useful to have a README file within your project directory. This is a plain text file containing a short summary about the project and a description of the files/directories found within it. An example README is shown below. It can also be helpful to include a README within each sub-directory with any information pertaining to the analysis.

```
## README ##  
## This directory contains data generated during the Introduction to RNA-seq workshop  
## Date:   
  
There are five subdirectories in this directory:  
  
raw_data : contains raw data  
meta:  contains...  
logs:  
results:  
scripts:  
  
```
