# Lesson plan


## Logging in to JupyterLab

Go to https://jupyterhub.greatplains.net and use your KU/Gmail/Github login. 

Select Genomics Workshop Supplement and click the start server button. This will take a while. In the meantime, discuss the following (slides):
 - RNAseq  
 - analytical pipeline  
 - data 

BK: add slides and slide numbers here. 

When the JupyterHub server is ready.. 

Explain that there are a number of tools installed on the image, and that for the purposes of this workshop we are only going to be using Terminal and RStudio. 

Click on Terminal. 

## Command Line prep/review

Change prompt to show only the working directory and the dollar sign. 
```
PS1="\w \$ "
```

Use `pwd` to print the current directory. 

```
$ pwd
/home/jovyan
```

Use `ls` to list the contents of the current directory. There is nothing in the home directory. 
Let's make a folder for our project and grab a copy of the data. We will be working with a subset of the original data for purposes of this demonstration. 

```
mkdir rnaseq_project

mkdir rnaseq_project/raw_reads 
```

Verify that we created the directory in the right place
```
ls rnaseq_project
```


Copy the raw data from the shared directory

```
cp -r /home/shared/raw_reads/* rnaseq_project/raw_reads/
```


### Conda

Conda is a package manager - it makes software installation easier. Here are the steps we are going to follow:

1. create a conda environment 
2. install the software that we need
3. activate the environment 


First, let's make sure conda is installed:

```
which conda
```
If the command returns a path to the conda executable file it means that it's installed. 

```
conda create -n rnaseq
```

The `-n` argument is required; provides the name for the environment which we will be using to activate the environment. 

```
conda activate rnaseq

```
Conda returns with a message saying that we need to configure to our shell. Let's do that:
```
conda init bash
```
It is recommended that we restart our shell. Due to the nature of the instances we are using, we are going to do a quick reset of the file that was modified with the following command:
```
source ~/.bashrc
```
Now there is an indication next to our shell prompt that we are in the base conda environment which is empty as we haven't installed anything. Let's go back to the `rnaseq` environment we created earlier. Let's try to activate it again:
```
conda activate rnaseq
``` 
The command doesn't return any messages to the terminal but the change of environment is reflected in the parentheses where it should now say `rnaseq` and not `base`.  
All we have done so far is create and activate an environment. Now, we are going to install the software that we need. The programs that we are using are available in a specialized repository for biomedical research tools called `bioconda`, which we will specify in our conda commnand using the `-c` argument.  
```
conda install -c bioconda fastp kallisto tree
```
We will be asked to confirm that we want to install `fastp`, `kallisto`, `tree` and their dependencies. We will type 'y' and press enter. 

To verify that the programs are installed use the which command again:
```
which fastp
# /opt/conda/envs/rnaseq/bin/fastp
which conda
# /opt/conda/envs/rnaseq/bin/kallisto 
```

(OPTIONAL) We can create the conda environment and install the software with one command:
```
conda create -n rnaseq -c bioconda fastp kallisto 
```

Let's review: We have created a conda environment, installed the software, and have copied the data to the `raw_reads` directory. Let's take a look 

We can use the utility `tree` to visualize the structure of the project directory
``` 
tree rnaseq_project
```


## fastp

```
cd rnaseq_project
```

Let's run fastp for one sample and see what the outputs look like.

```
cd raw_reads/SRR11799527

ls

# SRR11799527_1.sub_1m.fastq.gz  SRR11799527_2.sub_1m.fastq.gz
```


```
fastp -i SRR11799527_1.sub_1m.fastq.gz -I SRR11799527_2.sub_1m.fastq.gz -o SRR11799527_1.sub_1m.filt.fastq.gz -O SRR11799527_2.sub_1m.filt.fastq.gz 
```


```
Read1 before filtering:
total reads: 1000000
total bases: 38000000
Q20 bases: 37342182(98.2689%)
Q30 bases: 37080399(97.58%)

Read1 after filtering:
total reads: 984566
total bases: 37412428
Q20 bases: 36821469(98.4204%)
Q30 bases: 36576625(97.766%)

Read2 before filtering:
total reads: 1000000
total bases: 38000000
Q20 bases: 36695499(96.5671%)
Q30 bases: 36267713(95.4413%)

Read2 aftering filtering:
total reads: 984566
total bases: 37412428
Q20 bases: 36450127(97.4279%)
Q30 bases: 36070733(96.4138%)

Filtering result:
reads passed filter: 1969132
reads failed due to low quality: 30542
reads failed due to too many N: 326
reads failed due to too short: 0
reads with adapter trimmed: 552
bases trimmed due to adapters: 2226

JSON report: fastp.json
HTML report: fastp.html

fastp -i SRR11799527_1.sub_1m.fastq.gz -I SRR11799527_2.sub_1m.fastq.gz -o SRR11799527_1.sub_1m.filt.fastq.gz -O SRR11799527_2.sub_1m.filt.fastq.gz 
fastp v0.12.4, time used: 5 seconds
```

Let's see the folder contents
```
ls
# fastp.html  fastp.json  SRR11799527_1.sub_1m.fastq.gz  SRR11799527_1.sub_1m.filt.fastq.gz  SRR11799527_2.sub_1m.fastq.gz  SRR11799527_2.sub_1m.filt.fastq.gz
```

Describe outputs. You can download the html report by navigating to the folder in the left sidebar, right click on `fastp.html`, and select download. 

(Walk learners through the report)

Now, let's look at the other options that might be of of interest to us:

```

fastp -i SRR11799527_1.sub_1m.fastq.gz \
    -I SRR11799527_2.sub_1m.fastq.gz \
    -o SRR11799527_1.sub_1m.filt.fastq.gz \
    -O SRR11799527_2.sub_1m.filt.fastq.gz \
    -h SRR11799527.fastp.report.html \
    -j SRR11799527.fastp.report.json \
    -w 3 \
    --correction \
    --cut_by_quality3 \
    --cut_window_size 5 \
    --cut_mean_quality 30 \
    --overrepresentation_analysis 

```
The output is very similar to the first run. 

```
Read1 before filtering:
total reads: 1000000
total bases: 38000000
Q20 bases: 37342182(98.2689%)
Q30 bases: 37080399(97.58%)

Read1 after filtering:
total reads: 972840
total bases: 36784668
Q20 bases: 36331552(98.7682%)
Q30 bases: 36124503(98.2053%)

Read2 before filtering:
total reads: 1000000
total bases: 38000000
Q20 bases: 36695499(96.5671%)
Q30 bases: 36267713(95.4413%)

Read2 aftering filtering:
total reads: 972840
total bases: 36719684
Q20 bases: 35973931(97.9691%)
Q30 bases: 35650501(97.0883%)

Filtering result:
reads passed filter: 1945680
reads failed due to low quality: 54060
reads failed due to too many N: 234
reads failed due to too short: 26
reads with adapter trimmed: 488
bases trimmed due to adapters: 2451
reads corrected by overlap analysis: 2223
bases corrected by overlap analysis: 2434

JSON report: SRR11799527.fastp.report.json
HTML report: SRR11799527.fastp.report.html

fastp -i SRR11799527_1.sub_1m.fastq.gz -I SRR11799527_2.sub_1m.fastq.gz -o SRR11799527_1.sub_1m.filt.fastq.gz -O SRR11799527_2.sub_1m.filt.fastq.gz -h SRR11799527.fastp.report.html -j SRR11799527.fastp.report.json -w 3 --correction --cut_by_quality3 --cut_window_size 5 --cut_mean_quality 30 --overrepresentation_analysis 
fastp v0.12.4, time used: 10 seconds
```

Checking the directory contents shows that we have the filtered reads files and the properly named fastp reports. Let's clean up the directory and create a simple script that runs fastp for a single sample based on the command we just used. 

```
rm *.html *.json *.filt.fastq.gz
ls
# SRR11799527_1.sub_1m.fastq.gz  SRR11799527_2.sub_1m.fastq.gz
```

```
cd ../..

pwd
# /home/jovyan/rnaseq_project
```

In nano:
```
nano run_fastp.sh

```

```
SAMPLE=${1}
echo ${SAMPLE}

# reminder that each sample has its own folder containing all fastq files associated with that sample
DATA_RAW=/home/jovyan/rnaseq_project/raw_reads/${SAMPLE}
# each sample will have its own folder with the trimmed fastq files
## first, we generate the name of the folder
DATA_FILT=/home/jovyan/rnaseq_project/trimmed_reads/${SAMPLE}
## next, we create the directory with mkdir
mkdir -p ${DATA_FILT}

# now we are ready to run fastq to do read trimming
## -h HTML report output path
## -j JSON report output path
## -w sets the number of threads fastp will use; more threads means it runs faster, up to 16 cores
## 

fastp -i ${DATA_RAW}/${SAMPLE}_1.fastq.gz \
    -I ${DATA_RAW}/${SAMPLE}_2.fastq.gz \
    -o ${DATA_FILT}/${SAMPLE}_1.filt.fastq.gz \
    -O ${DATA_FILT}/${SAMPLE}_2.filt.fastq.gz \
    -h ${DATA_FILT}/${SAMPLE}.fastp.report.html \
    -j ${DATA_FILT}/${SAMPLE}.fastp.report.json \
    -w 3 \
    --correction \
    --cut_by_quality3 \
    --cut_window_size 5 \
    --cut_mean_quality 30 \
    --overrepresentation_analysis 
```

let's run it for one sample:
```
bash run_fastp.sh SRR11799527
```

It you see the fastp output, the command completed successfully. We can verify the outputs by checking the `trimmed_reads` directory:

```
ls trimmed_reads
# SRR11799527

ls trimmed_reads/SRR11799527
# SRR11799527_1.sub_1m.filt.fastq.gz  SRR11799527_2.sub_1m.filt.fastq.gz  SRR11799527.fastp.report.html  SRR11799527.fastp.report.json
```

Since this works for one sample, let's run it for all 20 samples. To do that we will need the SRR ID's for all the samples. We have created a file with that information - copy it from the shared space. 

```
pwd 
# /home/jovyan/rnaseq_project

cp /home/shared/SRR_IDs.txt . 
```

```
cat SRR_IDs.txt


SRR11799527
SRR11799528
SRR11799529
SRR11799530
SRR11799531
SRR11799532
SRR11799533
SRR11799534
SRR11799535
SRR11799536
SRR11799537
SRR11799538
SRR11799539
SRR11799540
SRR11799541
SRR11799542
SRR11799543
SRR11799544
SRR11799545
SRR11799546
```

```
while read SRRID
do
  bash run_fastp.sh ${SRRID}
done < SRR_IDs.txt
```





## Kallisto

Discuss Kallisto. (back to slides)


In order to quantify the RNA-seq data for our samples, we need the transcript sequences for _D. melanogaster_. Open a new browser tab, and search for [flybase](https://flybase.org/). Go to downloads, and click on Genomes (FTP). Next, click on Drosophila melanogaster, and then navigate to the most recent release `dmel_r6.50_FB2023_01/` (at the bottom of the page). Since we need the transcript sequences, we will look in the `fasta` folder. The file that we are interested in is called `dmel-all-transcript-r6.50.fasta.gz`. Right click and select "Copy link" on that file name. We can use that link to download the reference transciptome. 

Check pwd and ensure evetyone is at the same place: home dir

```
wget http://ftp.flybase.net/genomes/Drosophila_melanogaster/dmel_r6.50_FB2023_01/fasta/dmel-all-transcript-r6.50.fasta.gz
```

Now, let's move the file we just downloaded to a new directory in the `rnaseq_project` directory. 

```
mkdir rnaseq_project/refs

mv dmel-all-transcript-r6.50.fasta.gz rnaseq_project/refs/
```

Next, we have to prepare the reference transcriptome for the quantification step: 

```
cd rnaseq_project/refs/
kallisto index -i dmel-all-transcript-r6.50.idx  dmel-all-transcript-r6.50.fasta.gz
```
(this will take about a minute)

```
ls
```






