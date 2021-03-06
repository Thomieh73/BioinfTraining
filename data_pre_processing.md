# Data pre-processing

## Datasets used for this session

Dataset used during this session can be found in the following location within abel:
```
/work/projects/nn9305k/tmp/Files_for_Dec14/
```

## NB: Replace <your_user_name> with your abel username

Create a new folder called _Data_pre_processing_Dec14_ in your home area and move there.
```
cd /work/projects/nn9305k/home/<your_user_name>/
mkdir Data_pre_processing_Dec14
cd Data_pre_processing_Dec14
```

Create three folders here and move to the _data_ folder:
```
mkdir data
mkdir raw_fastqc
mkdir trim

cd data
```

Type the following command to link the files (not copy):
```
ln -s /work/projects/nn9305k/tmp/Files_for_Dec14/*fq.gz .
```


## Fastq quality check

We will use [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) to check the quality of raw sequenced data.
It is important to check most of the graphs and make sure that the data represents the sample that was library prep'd and sequenced. For more information regarding the graphs, either visit the above website or [check this video](https://www.youtube.com/watch?v=bz93ReOv87Y)

--------
**Task**
1. Run _fastqc_ on the file _Ha_R1.fq.gz_ (in the login node).
2. type in the following (don't include the `$`).

```
$ fastqc Ha_R1.fq.gz
```

3. Try to find help for _fastqc_ and discuss what flags one can use to process multiple samples.
  You will use _-t_ option to use multiple threads. One thread will analyse one file at a time.
4. Use _SLURM_ to process the other four files.

_SLURM script_
```
#!/bin/bash
#
# Job name:
#SBATCH --job-name=raw_fastq
#
# Project: 
#SBATCH --account=nn9305k
#
# Wall clock limit: 
#SBATCH --time=01:00:00
#
#SBATCH --ntasks=4 
#
# Max memory usage: 
## A good suggestion here is 4GB
#SBATCH --mem-per-cpu=4Gb
## Set up job environment

source /cluster/bin/jobsetup
  
module load fastqc
fastqc -t 4 Br_R* Ed_R*
```

5. Move the _html_ and _zip_ files to _raw_fastqc_
```
cd ../raw_fastqc
mv ../data/*html .
mv ../data/*.zip .
```

6. Copy the _raw_fastqc_ folder from abel to Biolinux in a folder called _Data_pre_processing_Dec14_ in _Desktop_ using scp.
  Option _-r (stands for recursively)_ will help in copying folders and all the content inside (and do not forget the _'.'_ at the end of the command. 
  
  **In Biolinux:**
```
cd 
cd Desktop
mkdir Data_pre_processing_Dec14
cd Data_pre_processing_Dec14
scp -r <your_user_name>@abel.uio.no:/work/projects/nn9305k/home/<your_user_name>/Data_pre_processing_Dec14/raw_fastqc .
```

7. Go through the html files and discuss.

--------
## Trimmomatic - adapter trimming and removing

We wll use [Trimmomatic](http://www.usadellab.org/cms/index.php?page=trimmomatic) to trim/remove adapter and low quality reads.
This tool is NOT available via _module load_ in abel but available at _/work/projects/nn9305k/bin/_. Make sure you know where the adapter sequences are available.

--------
**Task**
1. Move to _trim_ folder.
```
cd ../trim
```

2. Run _trimmomatic-0.36.jar_ on _Ha_R1.fq.gz_ file.

_SLURM script_
```
#!/bin/bash
#
# Job name:
#SBATCH --job-name=trim
#
# Project:
#SBATCH --account=nn9305k
#
# Wall clock limit:
#SBATCH --time=01:00:00
#
#SBATCH --ntasks=12
#
# Max memory usage:
## A good suggestion here is 4GB
#SBATCH --mem-per-cpu=4Gb
## Set up job environment
  
source /cluster/bin/jobsetup

java -jar /work/projects/nn9305k/bin/trimmomatic-0.36.jar SE -threads 12 -phred33 ../data/Ha_R1.fq.gz Ha_trim_R1.fq.gz ILLUMINACLIP:/work/projects/nn9305k/db_flatfiles/trimmomatic_adapters/TruSeq3-SE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:15 CROP:75
```

| Trimmomatic       | Description |
| ---               | --- |
| _SE_              | stands for single reads |
| _-threads_        | to utilize 12 threads and use multiple CPUs in parallel |
| _-phred33_        | specifying the quality encoding of fastq files generated by Illumina sequencers |
| _ILLUMINACLIP_    | to trim/remove adapter reads using an adapter file and a sliding window (2:30:10) |
| _LEADING_         | to remove low quality bases in the beginning of the read |
| _TRAILING_        | to remove low quality bases from the end of the read |
| _SLIDINGWINDOW_   | to trim/remove low quality reads using a sliding window (4:15) |
| _MINLEN_          | reads with minimum length to be retained. In general this should be _36_ but since we are dealing with miRNAs here, it should be _15_ |
| _CROP_            | to trim the reads to a specific length |


3. Run _fastqc_ on the output fastq files and copy the html and zip to BioLinux and view them in the browser
--------

# Homework

Use trimmomatic to trim/remove adapters and low quality reads in _Br_R1.fq.gz_ and _Br_R2.fq.gz_ (or/and _Ed_R1.fq.gz_ and _Ed_R2.fq.gz_)

1. Remember that you are working with paired end data (Change _SE_ to _PE_). 
2. There are two input files and four output files.
3. Use _TruSeq3-PE-2.fa_ instead of _TruSeq3-SE.fa_ since we are dealing with paired end reads.
4. Change _MINLEN_ parameter to _36_.
5. Use appropriate value for _CROP_ (check the fastqc output for raw reads and use the correct value).
6. Also, remember to change _#SBATCH --mem-per-cpu=4Gb_ to _#SBATCH --mem-per-cpu=12Gb_. This is a bigger job and needs more memory (12Gb instead of 4Gb).

_SLURM script_
```
#!/bin/bash
#
# Job name:
#SBATCH --job-name=trim
#
# Project:
#SBATCH --account=nn9305k
#
# Wall clock limit:
#SBATCH --time=01:00:00
#
#SBATCH --ntasks=12
#
# Max memory usage:
## A good suggestion here is 4GB
#SBATCH --mem-per-cpu=12Gb
## Set up job environment

source /cluster/bin/jobsetup
  
java -jar /work/projects/nn9305k/bin/trimmomatic-0.36.jar PE -threads 12 -phred33 ../data/Br_R1.fq.gz ../data/Br_R2.fq.gz Br_trim_R1.fq.gz Br_trim_R1_UNPAIRED.fq.gz Br_trim_R2.fq.gz Br_trim_R2_UNPAIRED.fq.gz ILLUMINACLIP:/work/projects/nn9305k/db_flatfiles/trimmomatic_adapters/TruSeq3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36 CROP:150
  
java -jar /work/projects/nn9305k/bin/trimmomatic-0.36.jar PE -threads 12 -phred33 ../data/Ed_R1.fq.gz ../data/Ed_R2.fq.gz Ed_trim_R1.fq.gz Ed_trim_R1_UNPAIRED.fq.gz Ed_trim_R2.fq.gz Ed_trim_R2_UNPAIRED.fq.gz ILLUMINACLIP:/work/projects/nn9305k/db_flatfiles/trimmomatic_adapters/TruSeq3-PE-2.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36 CROP:150
```

## FastX-Toolkit

I mentioned about FastX-toolkit [FastX-Toolkit](http://hannonlab.cshl.edu/fastx_toolkit/index.html) and showed one example to _collapse_ fastq reads to identify the list of unique reads. There are more then 10 tools in this toolkit and do read the above page to familiarize yourself with these tools. 

Fastx-Toolkit does not accept _compressed_ files. So we need to _uncompress_ the _.gz_ file to normal _fastq_ file using _gunzip_.
In folder _trim_:
```
gunzip -c Ha_trim_R1.fq.gz > Ha_trim_R1.fq
fastx_collapser -v -i Ha_trim_R1.fq -o Ha_trim_R1_Collapse.fq -Q33 
```

