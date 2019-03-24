# Cisco Egg and Larvae Transcriptome analysis 2017 

## All scripts combined

**Overall pipeline:** [Raw](#Raw-stats) -> [Fastqc](#Fastqc (on the raw files)) -> [Trim](#Trim) -> [fastqc](#Fastqc (on the cleaned files)) -> [File prep](#File-prep) -> [Trinity](#Trinity) ->  [TrinityStats.pl](#TrinityStats.pl) -> [BUSCO](#BUSCO) -> [Transdecoder](#Transdecoder) -> [Salmon](#Salmon) -> [Deseq2](#DESeq2) -> [BLAST](#BLAST) -> [GOterm](#DESeq2-and-BLAST-results-->-GOterms/Function-(using-Uniprot)) -> [Functional Enrichment](#Functional-Enrichment) 

All scripts/programs were run on a VACC (Vermont Advanced Computer Core) server unless otherwise specified.  Due to this, all script have a header required by the VACC to specify the amount of space and/or the number of hours the job is allowed to run.  If there is not header its possible the code was run directly on the server. 



#Raw stats
Summary stats from the quality report provided by the sequencing facility (Polar Genomics; Cornell Seq)

|        | Total raw reads | Avg # raw reads per indiv | Total cleaned reads | Avg # cleaned reads per indiv |
| ------ | --------------- | ------------------------- | ------------------- | ----------------------------- |
| Eggs   | 212,461,345     | 17,705,112                | 147,769,201         | 12,314,100                    |
| Larvae | 205,075,546     | 17,089,629                | 147,978,219         | 12,331,518                    |
| All    | 417,536,891     | 17,397,370                | 295,747,420         | 12,322,809                    |

The fasta file format (usually) has 4 lines for each read: 

**1st line**: info on sequencer etc
**2nd line**: actual sequence (should be 150base long)
**3rd line**: +
**4th line**: Quality scores for each base (link in tutorial will tell you what each means; J and K are good)

Here's a reference for understanding Quality (Phred) scores: http://www.drive5.com/usearch/manual/quality_score.html

Typically, we accept bases with Q >= 30, which is equivalent to a 0.1% chance of error (40 is 0.01% error, 20 is 1% error, 10 is 10% error).

#Fastqc (on the raw files)

Fastqc (v0.11.3) is "A quality control tool for high throughput sequence data."

**Info on the program:** https://www.bioinformatics.babraham.ac.uk/projects/fastqc/ 

I ran Fastqc before any trimming to see the initial quality of my raw reads



Ran two separate scripts to maximize efficiency on the server:

**File Name:** "fastqc_job.script" 

**Date run:** 8/30/17

**Code:** 

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=16gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=10:00:00
# Name of job.
#PBS -N fastqc
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw
echo "fastqc LC6_R1.fastq.gz" `bluemoon-user1.uvm.edu`
## /users/h/l/hlachanc/programs/fastqc/fastqc
for i in `ls ~/data/raw/E*.gz`;do ~/programs/fastqc/fastqc ${i};done
```



**File Name:** "fastqc_L.script"

**Date run:** 8/30/17

**Code:** 

``` 
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=16gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=10:00:00
# Name of job.
#PBS -N fastqc
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw
echo "fastqc LC6_R1.fastq.gz" `bluemoon-user1.uvm.edu`
## /users/h/l/hlachanc/programs/fastqc/fastqc
for i in `ls ~/data/raw/L*.gz`;do ~/programs/fastqc/fastqc ${i};done
```



# Trim

Overall quality was good but still wanted to clean the raw reads up a bit so I used the program Trimmomatic (v0.33) to remove adaptors and primers and clean up the ends of the reads.

**Info on Trimmomatic: ** http://www.usadellab.org/cms/?page=trimmomatic

**File Name:** "trim_EC1.script" (a script like this for each individual) 

**Date run:** 12/18/17

**Code:** (example from 1 of the 24 scripts)

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=50gb,vmem=50gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=10:00:00
# Name of job.
#PBS -N trimmomatic 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea


## cd $HOME/data/raw/

## /users/h/l/hlachanc/programs/Trimmomatic-0.33

#!/bin/bash
  java -classpath /users/h/l/hlachanc/programs/Trimmomatic-0.33/trimmomatic-0.33.jar org.usadellab.trimmomatic.TrimmomaticPE \
        -threads 10 \
        -phred33 \
        ~/data/raw/EC6_R1.fastq.gz \
        ~/data/raw/EC6_R2.fastq.gz \
        ~/data/raw/cleanreads/EC6_R1_clean_paired.fq \
        ~/data/raw/cleanreads/EC6_R1_clean_unpaired.fq \
        ~/data/raw/cleanreads/EC6_R2_clean_paired.fq \
        ~/data/raw/cleanreads/EC6_R2_clean_unpaired.fq\
        ILLUMINACLIP:/gpfs1/home/h/l/hlachanc/programs/Trimmomatic-0.33/adapters/TruSeq3-PE.fa:2:30:10 \
        LEADING:20 \
        TRAILING:20 \
        SLIDINGWINDOW:6:20 \
        HEADCROP:12 \
        MINLEN:35
```



# Fastqc (on the cleaned files)

Ran Fastqc (v0.11.3) after cleaning the reads to ensure they meet basic standards and/or weren't over trimmed

**Info on Fastqc:** https://www.bioinformatics.babraham.ac.uk/projects/fastqc/ 

Ran two separate scripts to maximize efficiency on the server:

**File Name:** "fastqc2_E.script" 

**Date run:** 9/2/17

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=16gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=10:00:00
# Name of job.
#PBS -N fastqc
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw
echo "fastqc LC6_R1.fastq.gz" `bluemoon-user1.uvm.edu`
## /users/h/l/hlachanc/programs/fastqc/fastqc
for i in `ls ~/data/raw/cleanreads/E*.fq`;do ~/programs/fastqc/fastqc ${i};done
```



**File Name:** "fastqc2_L.script"

**Date run:** 9/2/17

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=16gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=10:00:00
# Name of job.
#PBS -N fastqc
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw
echo "fastqc LC6_R1.fastq.gz" `bluemoon-user1.uvm.edu`
## /users/h/l/hlachanc/programs/fastqc/fastqc
for i in `ls ~/data/raw/cleanreads/L*.fq`;do ~/programs/fastqc/fastqc ${i};done
```



# Other (optional) steps 

Between now and running Trinity you could run programs/commands to **cat** and/or **normalize** the files.  I tried these steps but ran into a few issues (i.e. Trinity didn't like the cat files, a collaborator suggested running non-normalized files, etc) so ***my final product did not include either*** option but the code to cat and normalize can be found below:

## Cat

**Note:** see Melissa's "Notes_2-14-17.md" typora file for examples of cat. One example is below:

```
cat 08_5-08_H_0_R1.cl.pd.fq 08_5-11_S_1_R1.cl.pd.fq 08_5-14_S_1_R1.cl.pd.fq 08_5-17_S_2_R1.cl.pd.fq 08_5-20_S_3_R1.cl.pd.fq 10_5-08_H_0_R1.cl.pd.fq 10_5-11_H_0_R1.cl.pd.fq 10_5-14_H_0_R1.cl.pd.fq 10_5-17_H_0_R1.cl.pd.fq 10_5-20_S_2_R1.cl.pd.fq 35_6-12_H_0_R1.cl.pd.fq 35_6-15_H_0_R1.cl.pd.fq 35_6-18_H_0_R2.cl.pd.fq 35_6-21_H_0_R1.cl.pd.fq 36_6-12_S_1_R1.cl.pd.fq 36_6-15_S_2_R1.cl.pd.fq 36_6-18_S_3_R1.cl.pd.fq > 08-11-35-36_R1.cl.pd.fq
```



This command will concatenate files together.  Concatenate means it will copy and paste info in each file one after the other in the order you list the files.  I will cat Egg R1, Egg R2, Larva R1, Larva R2, and EL R1 and EL R2 so that I will have concatenated files to normalize then use to assemble 3 different trinity assemblies (one for just egg, one for just larva, and one with both egg and larva reads) . When cating use the ***clean_paired.fq*** files.   Below are the command to cat each

Egg R1

```
cat EA2_R1_clean_paired.fq EA3_R1_clean_paired.fq EA4_R1_clean_paired.fq EA5_R1_clean_paired.fq EA7_R1_clean_paired.fq EA8_R1_clean_paired.fq EC1_R1_clean_paired.fq EC2_R1_clean_paired.fq EC3_R1_clean_paired.fq EC4_R1_clean_paired.fq EC5_R1_clean_paired.fq EC6_R1_clean_paired.fq > catE_R1_clean_paired.fq
```



Egg R2

```
cat EA2_R2_clean_paired.fq EA3_R2_clean_paired.fq EA4_R2_clean_paired.fq EA5_R2_clean_paired.fq EA7_R2_clean_paired.fq EA8_R2_clean_paired.fq EC1_R2_clean_paired.fq EC2_R2_clean_paired.fq EC3_R2_clean_paired.fq EC4_R2_clean_paired.fq EC5_R2_clean_paired.fq EC6_R2_clean_paired.fq > catE_R2_clean_paired.fq
```



Larva R1

```
cat LA2_R1_clean_paired.fq LC3_R1_clean_paired.fq LA4_R1_clean_paired.fq LA6_R1_clean_paired.fq LA7_R1_clean_paired.fq LA8_R1_clean_paired.fq LC1_R1_clean_paired.fq LC2_R1_clean_paired.fq LC3_R1_clean_paired.fq LC4_R1_clean_paired.fq LC5_R1_clean_paired.fq LC6_R1_clean_paired.fq > catL_R1_clean_paired.fq
```



Larva R2

```
cat LA2_R2_clean_paired.fq LC3_R2_clean_paired.fq LA4_R2_clean_paired.fq LA6_R2_clean_paired.fq LA7_R2_clean_paired.fq LA8_R2_clean_paired.fq LC1_R2_clean_paired.fq LC2_R2_clean_paired.fq LC3_R2_clean_paired.fq LC4_R2_clean_paired.fq LC5_R2_clean_paired.fq LC6_R2_clean_paired.fq > catL_R2_clean_paired.fq
```



EL R1

```
cat EA2_R1_clean_paired.fq EA3_R1_clean_paired.fq EA4_R1_clean_paired.fq EA5_R1_clean_paired.fq EA7_R1_clean_paired.fq EA8_R1_clean_paired.fq EC1_R1_clean_paired.fq EC2_R1_clean_paired.fq EC3_R1_clean_paired.fq EC4_R1_clean_paired.fq EC5_R1_clean_paired.fq EC6_R1_clean_paired.fq LA2_R1_clean_paired.fq LC3_R1_clean_paired.fq LA4_R1_clean_paired.fq LA6_R1_clean_paired.fq LA7_R1_clean_paired.fq LA8_R1_clean_paired.fq LC1_R1_clean_paired.fq LC2_R1_clean_paired.fq LC3_R1_clean_paired.fq LC4_R1_clean_paired.fq LC5_R1_clean_paired.fq LC6_R1_clean_paired.fq > catEL_R1_clean_paired.fq
```



EL R2

```
cat EA2_R2_clean_paired.fq EA3_R2_clean_paired.fq EA4_R2_clean_paired.fq EA5_R2_clean_paired.fq EA7_R2_clean_paired.fq EA8_R2_clean_paired.fq EC1_R2_clean_paired.fq EC2_R2_clean_paired.fq EC3_R2_clean_paired.fq EC4_R2_clean_paired.fq EC5_R2_clean_paired.fq EC6_R2_clean_paired.fq LA2_R2_clean_paired.fq LC3_R2_clean_paired.fq LA4_R2_clean_paired.fq LA6_R2_clean_paired.fq LA7_R2_clean_paired.fq LA8_R2_clean_paired.fq LC1_R2_clean_paired.fq LC2_R2_clean_paired.fq LC3_R2_clean_paired.fq LC4_R2_clean_paired.fq LC5_R2_clean_paired.fq LC6_R2_clean_paired.fq > catEL_R2_clean_paired.fq
```



I ran each of the above cat commands (one after the other) and they all seem to have worked.  Each command took ~ 1min to run.  Run in "cleanreads" folder on 10-13-17 around 3:15pm



## Normalization

Use the bbnorm method

An example from Melissa's "digital_norm_bbnorm.md" file is below:

```
./bbnorm.sh in1=08-11-35-36_R1.cl.pd.fq in2=08-11-35-36_R2.cl.pd.fq out1=normalized1.fq out2=normalized2.fq target=100
```

This process basically takes out only unique reads so that the transcriptome assembly process has less data to sift through and can run the "trinity" command faster. I will need to normalize each of the "cat*.fq" files I just generated:



Egg R1 and R2

```
./bbnorm.sh in1=catE_R1_clean_paired.fq in2=catE_R2_clean_paired.fq out1=normalized_E_1.fq out2=normalized_E_2.fq target=100
```

*Ran this and it did not work.  Said did not recognize the command.  I tried Googling and it didn't list a place to download a file/program so I'm not sure how to fix this.  Maybe try to run as a script. (with all the VACC pre information included)



10-16-17: I made a script with the above info (file called bbnorm_E.script) and ran with below command:



```
qsub bbnorm_E.script
```



**File Names:** "bbnorm_E.script"

**Date run:** 10/16/17 

**Code: ** 

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=50gb,vmem=24gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=10:00:00
# Name of job.
#PBS -N trimmomatic 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

#cd $HOME/data/raw/cleanreads

#!/bin/bash
./bbnorm.sh in1=catE_R1_clean_paired.fq in2=catE_R2_clean_paired.fq out1=normalized_E_1.fq out2=normalized_E_2.fq target=100
```

*Not sure if it worked or not...no further notes (bad Hannah...)

# File prep

We had to edit the files to be in a format that is accepted by Trinity.  This entailed two separate steps described below:

1) add /1 or /2 to the end of the sequence name to indicate if it was read 1 or 2

**File Names:** "editR1.script" and "editR2.script"

**Date run:** 12/15/17 

**Code: ** "editR1.script"

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=24gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=01:00:00
# Name of job.
#PBS -N add1 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw/cleanreads

#sed -i '/@E00595/ s_$_/1_' LC1_R1_clean_paired.fq > LC1_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LC2_R1_clean_paired.fq > LC2_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LC3_R1_clean_paired.fq > LC3_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LC4_R1_clean_paired.fq > LC4_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LC5_R1_clean_paired.fq > LC5_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LC6_R1_clean_paired.fq > LC6_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LA2_R1_clean_paired.fq > LA2_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LA3_R1_clean_paired.fq > LA3_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LA4_R1_clean_paired.fq > LA4_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LA6_R1_clean_paired.fq > LA6_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LA7_R1_clean_paired.fq > LA7_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' LA8_R1_clean_paired.fq > LA8_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EA2_R1_clean_paired.fq > EA2_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EA3_R1_clean_paired.fq > EA3_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EA4_R1_clean_paired.fq > EA4_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EA5_R1_clean_paired.fq > EA5_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EA7_R1_clean_paired.fq > EA7_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EA8_R1_clean_paired.fq > EA8_R1_clean_paired_1.fq
sed -i '/@E00595/ s_$_/1_' EC1_R1_clean_paired.fq > EC1_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EC2_R1_clean_paired.fq > EC2_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EC3_R1_clean_paired.fq > EC3_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EC4_R1_clean_paired.fq > EC4_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EC5_R1_clean_paired.fq > EC5_R1_clean_paired_1.fq
#sed -i '/@E00595/ s_$_/1_' EC6_R1_clean_paired.fq > EC6_R1_clean_paired_1.fq

```

**Code: ** "editR2.script"

``` 
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=24gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=01:00:00
# Name of job.
#PBS -N add2 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw/cleanreads

sed -i '/@E00595/ s_$_/2_' LC1_R2_clean_paired.fq > LC1_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LC2_R2_clean_paired.fq > LC2_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LC3_R2_clean_paired.fq > LC3_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LC4_R2_clean_paired.fq > LC4_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LC5_R2_clean_paired.fq > LC5_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LC6_R2_clean_paired.fq > LC6_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LA2_R2_clean_paired.fq > LA2_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LA3_R2_clean_paired.fq > LA3_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LA4_R2_clean_paired.fq > LA4_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LA6_R2_clean_paired.fq > LA6_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LA7_R2_clean_paired.fq > LA7_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' LA8_R2_clean_paired.fq > LA8_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EA2_R2_clean_paired.fq > EA2_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EA3_R2_clean_paired.fq > EA3_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EA4_R2_clean_paired.fq > EA4_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EA5_R2_clean_paired.fq > EA5_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EA7_R2_clean_paired.fq > EA7_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EA8_R2_clean_paired.fq > EA8_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EC1_R2_clean_paired.fq > EC1_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EC2_R2_clean_paired.fq > EC2_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EC3_R2_clean_paired.fq > EC3_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EC4_R2_clean_paired.fq > EC4_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EC5_R2_clean_paired.fq > EC5_R2_clean_paired_2.fq
sed -i '/@E00595/ s_$_/2_' EC6_R2_clean_paired.fq > EC6_R2_clean_paired_2.fq
```



2) remove `_2:N:0` and `_1:N:0` from read names

**File Name:** "edit1No.script" and "edit2N0.script"

**Date run:** 12/15/17 

**Code:** "edit1No.script" 

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=24gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=01:00:00
# Name of job.
#PBS -N edit2N0 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw/cleanreads

#sed -i -e 's/ 1:N:0//g' LC1_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LC2_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LC3_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LC4_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LC5_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LC6_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LA2_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LA3_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LA4_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LA6_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LA7_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' LA8_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EA2_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EA3_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EA4_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EA5_R1_clean_paired.fq
#sed -i -e 's/ 1:N:0//g' EA7_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EA8_R1_clean_paired.fq 
sed -i -e 's/ 1:N:0//g' EC1_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EC2_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EC3_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EC4_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EC5_R1_clean_paired.fq 
#sed -i -e 's/ 1:N:0//g' EC6_R1_clean_paired.fq 

```

**Code:** "edit1No.script" 

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=1,mem=24gb,vmem=24gb
#PBS -q sharedq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=01:00:00
# Name of job.
#PBS -N edit2N0
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw/cleanreads
#sed -i -e 's/ 2:N:0//g' LC1_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LC2_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LC3_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LC4_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LC5_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LC6_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LA2_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LA3_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LA4_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LA6_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LA7_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' LA8_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EA2_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EA3_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EA4_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EA5_R2_clean_paired.fq
#sed -i -e 's/ 2:N:0//g' EA7_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EA8_R2_clean_paired.fq 
sed -i -e 's/ 2:N:0//g' EC1_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EC2_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EC3_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EC4_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EC5_R2_clean_paired.fq 
#sed -i -e 's/ 2:N:0//g' EC6_R2_clean_paired.fq 
```



# Trinity

Trinity (v2.4.0) is probably the most important program I ran.  Trinity is "a novel method for the efficient and robust de novo reconstruction of transcriptomes from RNA-seq data." It takes ~2days (~1week?) for each script to run.

**Info on Trinity:** https://github.com/trinityrnaseq/trinityrnaseq/wiki and http://cbsu.tc.cornell.edu/lab/doc/Trinity_workshop_Part1.pdf

**Brief description of how Trinity works:**

"Trinity partitions the sequence data into many individual de Bruijn graphs, each representing the transcriptional complexity at a given gene or locus, and then processes each graph independently to extract full-length splicing isoforms and to tease apart transcripts derived from paralogous genes.  Briefly, the process works like so:

- *Inchworm* assembles the RNA-seq data into the unique sequences of transcripts, often generating full-length transcripts for a dominant isoform, but then reports just the unique portions of alternatively spliced transcripts.
- *Chrysalis* clusters the Inchworm contigs into clusters and constructs complete de Bruijn graphs for each cluster.  Each cluster represents the full transcriptonal complexity for a given gene (or sets of genes that share sequences in common).  Chrysalis then partitions the full read set among these disjoint graphs.
- *Butterfly* then processes the individual graphs in parallel, tracing the paths that reads and pairs of reads take within the graph, ultimately reporting full-length transcripts for alternatively spliced isoforms, and teasing apart transcripts that corresponds to paralogous genes."

## *De novo* assemblies

I created 3 *de novo* assemblies:

- an assembly with **all** egg and larva reads

- an assembly with **only** egg reads

- an assembly with **only** larva reads


**File Name:** "trinityall_111517.script"

**Date run:** 12/18/17

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=24,mem=500gb,vmem=500gb
#PBS -q l1wkpoolq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=168:00:00
# Name of job.
#PBS -N trinityall 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw/cleanreads/

#export PATH= /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/jre/bin:${PATH}

~/programs/trinityrnaseq-Trinity-v2.4.0/Trinity --seqType fq --right EA2_R2_clean_paired.fq,EA3_R2_clean_paired.fq,EA4_R2_clean_paired.fq,EA5_R2_clean_paired.fq,EA7_R2_clean_paired.fq,EA8_R2_clean_paired.fq,EC1_R2_clean_paired.fq,EC2_R2_clean_paired.fq,EC3_R2_clean_paired.fq,EC4_R2_clean_paired.fq,EC5_R2_clean_paired.fq,EC6_R2_clean_paired.fq,LA2_R2_clean_paired.fq,LC3_R2_clean_paired.fq,LA4_R2_clean_paired.fq,LA6_R2_clean_paired.fq,LA7_R2_clean_paired.fq,LA8_R2_clean_paired.fq,LC1_R2_clean_paired.fq,LC2_R2_clean_paired.fq,LC3_R2_clean_paired.fq,LC4_R2_clean_paired.fq,LC5_R2_clean_paired.fq,LC6_R2_clean_paired.fq  --left EA2_R1_clean_paired.fq,EA3_R1_clean_paired.fq,EA4_R1_clean_paired.fq,EA5_R1_clean_paired.fq,EA7_R1_clean_paired.fq,EA8_R1_clean_paired.fq,EC1_R1_clean_paired.fq,EC2_R1_clean_paired.fq,EC3_R1_clean_paired.fq,EC4_R1_clean_paired.fq,EC5_R1_clean_paired.fq,EC6_R1_clean_paired.fq,LA2_R1_clean_paired.fq,LC3_R1_clean_paired.fq,LA4_R1_clean_paired.fq,LA6_R1_clean_paired.fq,LA7_R1_clean_paired.fq,LA8_R1_clean_paired.fq,LC1_R1_clean_paired.fq,LC2_R1_clean_paired.fq,LC3_R1_clean_paired.fq,LC4_R1_clean_paired.fq,LC5_R1_clean_paired.fq,LC6_R1_clean_paired.fq --normalize_max_read_cov 50  --CPU 20 --max_memory 4G --no_bowtie --output ~/data/raw/cleanreads/trinityall_out_dir
 
```



**File Name:** "trinityE_111517.script"

**Date run:** 1/8/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=24,mem=500gb,vmem=500gb
#PBS -q l1wkpoolq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=168:00:00
# Name of job.
#PBS -N trinityE 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd /gpfs2/scratch/hlachanc/cleanreads

#export PATH= /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/jre/bin:${PATH}

~/programs/trinityrnaseq-Trinity-v2.4.0/Trinity --seqType fq --right EA2_R2_clean_paired.fq,EA3_R2_clean_paired.fq,EA4_R2_clean_paired.fq,EA5_R2_clean_paired.fq,EA7_R2_clean_paired.fq,EA8_R2_clean_paired.fq,EC1_R2_clean_paired.fq,EC2_R2_clean_paired.fq,EC3_R2_clean_paired.fq,EC4_R2_clean_paired.fq,EC5_R2_clean_paired.fq,EC6_R2_clean_paired.fq  --left EA2_R1_clean_paired.fq,EA3_R1_clean_paired.fq,EA4_R1_clean_paired.fq,EA5_R1_clean_paired.fq,EA7_R1_clean_paired.fq,EA8_R1_clean_paired.fq,EC1_R1_clean_paired.fq,EC2_R1_clean_paired.fq,EC3_R1_clean_paired.fq,EC4_R1_clean_paired.fq,EC5_R1_clean_paired.fq,EC6_R1_clean_paired.fq --normalize_max_read_cov 50  --CPU 20 --max_memory 4G --no_bowtie --output /gpfs1/home/h/l/hlachanc/output/trinityE_out_dir

```



**File Name:** "trinityL_111517.script"

**Date run:** 12/18/17

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=24,mem=500gb,vmem=500gb
#PBS -q l1wkpoolq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=168:00:00
# Name of job.
#PBS -N trinityL
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd $HOME/data/raw/cleanreads/

#export PATH= /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-1.b12.el7_4.x86_64/jre/bin:${PATH}

~/programs/trinityrnaseq-Trinity-v2.4.0/Trinity --seqType fq --right LA2_R2_clean_paired.fq,LC3_R2_clean_paired.fq,LA4_R2_clean_paired.fq,LA6_R2_clean_paired.fq,LA7_R2_clean_paired.fq,LA8_R2_clean_paired.fq,LC1_R2_clean_paired.fq,LC2_R2_clean_paired.fq,LC3_R2_clean_paired.fq,LC4_R2_clean_paired.fq,LC5_R2_clean_paired.fq,LC6_R2_clean_paired.fq  --left LA2_R1_clean_paired.fq,LC3_R1_clean_paired.fq,LA4_R1_clean_paired.fq,LA6_R1_clean_paired.fq,LA7_R1_clean_paired.fq,LA8_R1_clean_paired.fq,LC1_R1_clean_paired.fq,LC2_R1_clean_paired.fq,LC3_R1_clean_paired.fq,LC4_R1_clean_paired.fq,LC5_R1_clean_paired.fq,LC6_R1_clean_paired.fq --normalize_max_read_cov 50  --CPU 20 --max_memory 4G --no_bowtie --output ~/data/raw/cleanreads/trinityL_out_dir
 
```



## TrinityStats.pl

When you download the Trinity program you automatically download a number of additional, helpful tools, including a pearl script called **"TrinityStats.pl" ** which prints some stats about your trinity assembly.  More info and examples of TrinityStats.pl can be found in the link below, but generally you want fewer, longer contigs and a decent N50.  My stats look similar to the example in the link below so I moved on to the next steps using my "Trinity (all)" assembly.

Info on TrinityStats.pl: https://github.com/trinityrnaseq/trinityrnaseq/wiki/Transcriptome-Contig-Nx-and-ExN50-stats 

***Note:*** to run this pearl script you don't need to create a separate script with the additional server commands.  Simply run it directly on the server using the following command:

```
pathway/to/folder/with/Trinitystats.pl pathway/to/folder/with/Trinityall.fasta
```

**Results**: Trinity (all)

```

################################
## Counts of transcripts, etc.
################################
Total trinity 'genes':  365661
Total trinity transcripts:      650023
Percent GC: 47.13

########################################
Stats based on ALL transcript contigs:
########################################

        Contig N10: 3157
        Contig N20: 2396
        Contig N30: 1893
        Contig N40: 1487
        Contig N50: 1132

        Median contig length: 372
        Average contig: 682.92
        Total assembled bases: 443916348


#####################################################
## Stats based on ONLY LONGEST ISOFORM per 'GENE':
#####################################################

        Contig N10: 2629
        Contig N20: 1751
        Contig N30: 1172
        Contig N40: 791
        Contig N50: 568

        Median contig length: 327
        Average contig: 504.35
        Total assembled bases: 184420517
```

**Results**: Trinity (Egg only)

```
################################
## Counts of transcripts, etc.
################################
Total trinity 'genes':  269618
Total trinity transcripts:      470880
Percent GC: 46.92

########################################
Stats based on ALL transcript contigs:
########################################

        Contig N10: 3310
        Contig N20: 2550
        Contig N30: 2040
        Contig N40: 1634
        Contig N50: 1265

        Median contig length: 388
        Average contig: 730.42
        Total assembled bases: 343941757


#####################################################
## Stats based on ONLY LONGEST ISOFORM per 'GENE':
#####################################################

        Contig N10: 2850
        Contig N20: 1972
        Contig N30: 1371
        Contig N40: 931
        Contig N50: 645

        Median contig length: 334
        Average contig: 536.20
        Total assembled bases: 144569172
```

**Results**: Trinity (Larvae only)

```

################################
## Counts of transcripts, etc.
################################
Total trinity 'genes':  256589
Total trinity transcripts:      463392
Percent GC: 47.17

########################################
Stats based on ALL transcript contigs:
########################################

        Contig N10: 3007
        Contig N20: 2308
        Contig N30: 1857
        Contig N40: 1484
        Contig N50: 1156

        Median contig length: 392
        Average contig: 704.08
        Total assembled bases: 326265092


#####################################################
## Stats based on ONLY LONGEST ISOFORM per 'GENE':
#####################################################

        Contig N10: 2631
        Contig N20: 1858
        Contig N30: 1322
        Contig N40: 928
        Contig N50: 654

        Median contig length: 334
        Average contig: 534.13
        Total assembled bases: 137052436
```



# BUSCO

BUSCO (v.3.0.2) stands for **B**enchmarking **U**niversal **S**ingle-**C**opy **O**rthologs. BUSCO "provides quantitative measures for the assessment of genome assembly, gene set, and transcriptome completeness, based on evolutionarily-informed expectations of gene content from near-universal single-copy orthologs selected from [OrthoDB *v9*](http://orthodb.org/)"

Information on BUSCO: https://busco.ezlab.org/

I ran BUSCO for each of the three *de novo* assemblies I created; I included an example of the output for "Tall" (aka the transcriptome assembled with ALL samples):

**File Name:** "busco_tall_12418.script"

**Date run:** 1/24/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb
#PBS -q l1wkpoolq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=168:00:00
# Name of job.
#PBS -N buscoTall
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

python /gpfs2/scratch/hlachanc/busco/busco_tall/run_BUSCO.py -i /gpfs2/scratch/hlachanc/busco/busco_tall/Trinityall.fasta -o BUSCO_Tall_12518 -l /gpfs2/scratch/hlachanc/busco/busco_tall/actinopterygii_odb9/ -m tran

```

**Output:** Prints multiple files including a "missing list" a "full table" and a "short summary" Below is the ***"short summary"*** 

```
# BUSCO version is: 3.0.2 
# The lineage dataset is: actinopterygii_odb9 (Creation date: 2016-02-13, number of species: 20, number of BUSCOs: 4584)
# To reproduce this run: python /gpfs2/scratch/hlachanc/busco/busco_tall/run_BUSCO.py -i /gpfs2/scratch/hlachanc/busco/busco_tall/Trinityall.fasta -o BUSCO_Tall_12518 -l /gpfs2/scratch/hlachanc/busco/busco_tall/actinopterygii_odb9/ -m transcriptome -c 1
#
# Summarized benchmarking in BUSCO notation for file /gpfs2/scratch/hlachanc/busco/busco_tall/Trinityall.fasta
# BUSCO was run in mode: transcriptome

	C:71.9%[S:18.4%,D:53.5%],F:18.6%,M:9.5%,n:4584

	3297	Complete BUSCOs (C)
	843	Complete and single-copy BUSCOs (S)
	2454	Complete and duplicated BUSCOs (D)
	853	Fragmented BUSCOs (F)
	434	Missing BUSCOs (M)
	4584	Total BUSCO groups searched
```



**File Name:** "busco_tl_11818.script"

**Date run:** 1/18/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb
#PBS -q poolmemq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=30:00:00
# Name of job.
#PBS -N buscoTL
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

python /gpfs2/scratch/hlachanc/busco/busco_tl/run_BUSCO.py -i /gpfs2/scratch/hlachanc/busco/busco_tl/TrinityL.fasta -o BUSCO_TL -l /gpfs2/scratch/hlachanc/busco/busco_tl/actinopterygii_odb9/ -m tran

```



**File Name:** "busco_te_13018.script"

**Date run:** 1/30/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=20,mem=250gb,vmem=250gb
#PBS -q l1wkpoolq
# It should be allowed to run for up to 1 hour.
#PBS -l walltime=168:00:00
# Name of job.
#PBS -N buscoTE
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start, job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

python /gpfs2/scratch/hlachanc/busco/busco_te/run_BUSCO.py -i /gpfs2/scratch/hlachanc/busco/busco_te/TrinityE.fasta -o BUSCO_TE_13018 -l /gpfs2/scratch/hlachanc/busco/busco_te/actinopterygii_odb9/ -m tran

```



# Transdecoder

***Additional method to check quality but I don't think I put much weight on these results.***

Transcdecoder (v0.33)  "is a tool we built to identify likely coding regions within transcript sequences.  It **identifies long open reading frames (ORFs)** within transcripts and scores them according to their sequence composition."

**Info on Transdecoder:** https://github.com/TransDecoder/TransDecoder/wiki 

FIRST run **"TransDecoder.longestorf"** then **"TransDecoder.predict"**



1) **TransDecoder.LongOrfs** 

**File Name:** "transdecoder_all_4318.script"

**Date run:** 4/8/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=24,mem=250gb,vmem=250gb
#PBS -q poolmemq 
## It should be allowed to run for up to 1 hour.
#PBS -l walltime=30:00:00
# Name of job.
#PBS -N TransDecoder_all 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start,  job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd /gpfs2/scratch/hlachanc/cleanreads

spack load transdecoder 

TransDecoder.LongOrfs -t Trinity_transcripts.fasta


```

2) **TransDecoder.Predict**

**File Name:** "transdecoder_predict_all_4318.script"

**Date run:** 4/8/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=24,mem=250gb,vmem=250gb
#PBS -q poolmemq 
## It should be allowed to run for up to 1 hour.
#PBS -l walltime=30:00:00
# Name of job.
#PBS -N TransDecoder_all 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start,  job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd /gpfs2/scratch/hlachanc/cleanreads

spack load transdecoder 

TransDecoder.Predict -t Trinity_transcripts.fasta

```

ouputs in "Trinity_transcripts.fasta.transdecoder_dir"

More notes in purple/green/blue notebook p.72



# Salmon

Now that we've assembled our transcriptome and assessed the quality using **TrinityStats.pl** and **BUSCO** we can move on to aligning our reads against the assembly.

Salmon (v_0.9.1) is a "quasi-mapping" method that is an alternative to methods such as BWA.

**Information on Salmon:** https://salmon.readthedocs.io/en/latest/salmon.html and https://combine-lab.github.io/salmon/

***NOTE:*** This is where you could also use BWA (either aln or mem) however we had trouble getting those to work and decided to use Salmon.  It worked well so moved forward with results from Salmon.

I ran Salmon for the *de novo* assembly that used **all** the reads (usually referred to "Trinityall") I created scripts for each individual cisco (ie EA2) so there are 24 scripts but below is an example of one. 

**File Name:** "salmon_3518_EA2.script"

**Date run:** 3/6/18

**Code:**

```
## This job needs 1 compute node with 1 processor per node.
#PBS -l nodes=1:ppn=24,mem=250gb,vmem=250gb
#PBS -q poolmemq 
## It should be allowed to run for up to 1 hour.
#PBS -l walltime=30:00:00
# Name of job.
#PBS -N salmon 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start,  job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea

cd /gpfs2/scratch/hlachanc/cleanreads

#/gpfs1/arch/x86_64-rhel7/salmon-0.9.1/bin/salmon index -t Trinity.fasta -i Trinityall_index


/gpfs1/arch/x86_64-rhel7/salmon-0.9.1/bin/salmon quant -i Trinityall_index -p 8 -l A -1  EA2_R1_clean_paired.fq -2 EA2_R2_clean_paired.fq -o quants/EA2
```

**Output:** can be found in the "quants" folder "/gpfs2/scratch/hlachanc/cleanreads/quants" on the server.  a description of the various output files can be found here: https://salmon.readthedocs.io/en/latest/file_formats.html#fileformats



# DESeq2

DESeq2 (v1.20.1) is run in **R** and is used to detect differentially expressed genes when comparing treatments, etc. 

**Info on DESeq2:** http://bioconductor.org/packages/devel/bioc/vignettes/DESeq2/inst/doc/DESeq2.html

NOTE: Trevor said to adjust p-value to 0.01 (lower is better)

###***NOTE:*** DESeq2 is run in R

For my transcriptome, I ran few different designs:
***1) tank and stage:*** test effect of stage controlling for tank
***2) stage:*** looks at effect of stage without controlling for anything
***3) tank:***  looks at effect of tank without controlling for anything
***4) stage and tank:*** test effect of tank controlling for stage
***5) stage + tank + stage:tank:*** test effect of tank controlling for stage and comparing to interaction of stage:tank. (right?) 

*NOTE:* An **interaction** tells you if effect is due to stage *AND* tank vs stage alone or tank alone.

***6) EGG ONLY: tank*** : Using only the egg data, testing effect of tank (controls for stage in different way then in design 1)
***7) LARVAE ONLY: tank***: Using only the larvae data, testing effect of tank (controls for stage in different way then in design 1)

For the manuscript I focused on the results from design 5, 6, and 7

**File Name:** DESeq2_ciscolight_6418.R

**Date run:** (a few times between 6-4-18 and 7-25-18)

**Code:**

```
setwd("C:/Users/Hannah/Desktop/Grad School/Pilot Study 2017/data analysis/RScripts/DESeq2_cisco")
#source("https://bioconductor.org/biocLite.R")
#biocLite("DESeq2")
#BiocInstaller::biocValid("DESeq2")
library("DESeq2")

library("ggplot2")

countsTable <- read.delim('matrix.counts.matrix', header=TRUE, stringsAsFactors=TRUE, row.names=1)
countData <- as.matrix(countsTable)
storage.mode(countData) = "integer"
head(countData)

#colData <- data.frame(row.names=colnames(countData))
conds <- read.delim("colData.txt", header=TRUE, stringsAsFactors=TRUE, row.names=1)
head(conds)
colData <- as.data.frame(conds)
head(colData)


#################### MODEL NUMBER 1: tank + stage

dds <- DESeqDataSetFromMatrix(countData = countData, colData = colData, design = ~ tank + stage)
# In this typical model, the "health effect" represents the overall effect of health status 
# controlling for differences due to location page 26-27 manual DESeq2.pdf
# The last term in the model is what is tested.  In this case health. You need to rearrange the order 
# of the factors in the model to test for the overall effect of a diff. factor.
# This is not the same as an interaction.


dim(dds)
#[1] 13053    77

dds <- dds[ rowSums(counts(dds)) > 100, ]
dim(dds)
#[1] 12954    77  # at > 100; we loose many fewer genes


# For practice, let's work with fewer genes so that the models can run faster...
#dds <- dds[sample(nrow(dds), 1200), ]
#dim(dds)

colData(dds)$stage <- factor(colData(dds)$stage, levels=c("Egg","Larvae")) #sets that "egg is the reference

dds <- DESeq(dds) 

res <- results(dds)
res <- res[order(res$padj),]
head(res)
# log2 fold change (MAP): health S vs H 
# Wald test p-value: health S vs H 
# DataFrame with 6 rows and 6 columns
# baseMean log2FoldChange     lfcSE
# <numeric>      <numeric> <numeric>
#   TRINITY_DN46709_c0_g1_TRINITY_DN46709_c0_g1_i2_g.23139_m.23139   95.58364       2.916172 0.5141065
# TRINITY_DN45370_c1_g1_TRINITY_DN45370_c1_g1_i1_g.19244_m.19244  326.18876       1.495569 0.2910832
# TRINITY_DN45983_c5_g1_TRINITY_DN45983_c5_g1_i3_g.20837_m.20837 1292.39567       1.514289 0.2904796
# TRINITY_DN44127_c0_g1_TRINITY_DN44127_c0_g1_i1_g.16286_m.16286  250.18334       2.081641 0.4477985
# TRINITY_DN45492_c0_g1_TRINITY_DN45492_c0_g1_i5_g.19587_m.19587  276.71581       1.706664 0.3722197
# TRINITY_DN46924_c3_g1_TRINITY_DN46924_c3_g1_i1_g.23942_m.23942  253.74577       1.146243 0.2648761
# stat           pvalue
# <numeric>        <numeric>
#   TRINITY_DN46709_c0_g1_TRINITY_DN46709_c0_g1_i2_g.23139_m.23139  5.672312 0.00000001408832
# TRINITY_DN45370_c1_g1_TRINITY_DN45370_c1_g1_i1_g.19244_m.19244  5.137942 0.00000027776334
# TRINITY_DN45983_c5_g1_TRINITY_DN45983_c5_g1_i3_g.20837_m.20837  5.213065 0.00000018574588
# TRINITY_DN44127_c0_g1_TRINITY_DN44127_c0_g1_i1_g.16286_m.16286  4.648611 0.00000334177720
# TRINITY_DN45492_c0_g1_TRINITY_DN45492_c0_g1_i5_g.19587_m.19587  4.585099 0.00000453770438
# TRINITY_DN46924_c3_g1_TRINITY_DN46924_c3_g1_i1_g.23942_m.23942  4.327470 0.00001508320259
# padj
# <numeric>
#   TRINITY_DN46709_c0_g1_TRINITY_DN46709_c0_g1_i2_g.23139_m.23139 0.000006001624
# TRINITY_DN45370_c1_g1_TRINITY_DN45370_c1_g1_i1_g.19244_m.19244 0.000039442394
# TRINITY_DN45983_c5_g1_TRINITY_DN45983_c5_g1_i3_g.20837_m.20837 0.000039442394
# TRINITY_DN44127_c0_g1_TRINITY_DN44127_c0_g1_i1_g.16286_m.16286 0.000355899272
# TRINITY_DN45492_c0_g1_TRINITY_DN45492_c0_g1_i5_g.19587_m.19587 0.000386612413
# TRINITY_DN46924_c3_g1_TRINITY_DN46924_c3_g1_i1_g.23942_m.23942 0.001070907384

summary(res, alpha=0.01)
summary(res)

# out of 1199 with nonzero total read count
# adjusted p-value < 0.1
# LFC > 0 (up)     : 27, 2.3% 
# LFC < 0 (down)   : 14, 1.2% 
# outliers [1]     : 31, 2.6% 
# low counts [2]   : 743, 62% 
# (mean count < 25)

#################### MODEL NUMBER 2: Stage

ddss <- DESeqDataSetFromMatrix(countData = countData, colData = colData, design = ~ stage)
dim(ddss)

ddss <- ddss[ rowSums(counts(ddss)) > 100, ]

ddss <- DESeq(ddss)

ress <- results(ddss)
ress <- ress[order(ress$padj),]
head(ress)

summary(ress, alpha=0.01)
summary(ress)

#################### MODEL NUMBER 3: Tank

ddst <- DESeqDataSetFromMatrix(countData = countData, colData = colData, design = ~ tank)
dim(ddst)

ddst <- ddst[ rowSums(counts(ddst)) > 100, ]

ddst <- DESeq(ddst)

rest <- results(ddst)
rest <- rest[order(rest$padj),]
head(rest)

summary(rest, alpha=0.01)
summary(rest)

resultsNames(ddst)

#################### MODEL NUMBER 4: stage + tank

ddsst <- DESeqDataSetFromMatrix(countData = countData, colData = colData, design = ~ stage + tank)
dim(ddsst)

ddsst <- ddsst[ rowSums(counts(ddsst)) > 100, ]

ddsst <- DESeq(ddsst)

resst <- results(ddsst)
resst <- resst[order(resst$padj),]
head(resst)

summary(resst, alpha=0.01)
summary(resst)
resultsNames(ddsst)


######## Model 5: design = ~ stage + tank + stage:tank #############
ddsstst <- DESeqDataSetFromMatrix(countData = countData, colData = colData, design = ~ stage + tank + stage:tank)
dim(ddsstst)

ddsstst <- ddsstst[ rowSums(counts(ddsstst)) > 100, ]

ddsstst <- DESeq(ddsstst)

resstst <- results(ddsstst)
resstst <- resstst[order(resstst$padj),]
head(resstst)

summary(resstst, alpha=0.01)
summary(resstst)
# out of 120145 with nonzero total read count
# adjusted p-value < 0.1
# LFC > 0 (up)       : 129, 0.11%
# LFC < 0 (down)     : 130, 0.11%
# outliers [1]       : 279, 0.23%
# low counts [2]     : 0, 0%
# (mean count < 3)
# [1] see 'cooksCutoff' argument of ?results
# [2] see 'independentFiltering' argument of ?results
resultsNames(ddsstst)
# > resultsNames(ddsstst)
# [1] "Intercept"           "stage_Larvae_vs_Egg" "tank_C_vs_A"
# [4] "stageLarvae.tankC"
res_int <- results(ddsstst, name="stageLarvae.tankC", alpha = 0.05)
res_int <- res_int[order(res_int$padj),]
summary(res_int)
head(res_int)
write.csv(resstst, file = "resstst.csv")

##PCA with stage and tank sep
vsdlt <- varianceStabilizingTransformation(ddsstst, blind=FALSE)
pcaData <- plotPCA(vsdlt, intgroup = c( "tank", "stage"), returnData = TRUE)
pcaData
percentVar <- round(100 * attr(pcaData, "percentVar"))
ggplot(pcaData, aes(x = PC1, y = PC2, color = tank, shape = stage)) + geom_point(size =3) +
  xlab(paste0("PC1: ", percentVar[1], "% variance")) +
  ylab(paste0("PC2: ", percentVar[2], "% variance")) +
  coord_fixed()

###############Model 6: EGG ONLY, design= tank

colData <- subset(colData, stage=="Egg")
head(colData)

countData <- countData[, c(1:12)]
head(countData)

##deseq

ddset <- DESeqDataSetFromMatrix(colData = colData, countData = countData, design = ~ tank)
dim(ddset)

ddset <- ddset[ rowSums(counts(ddset)) > 100, ]

ddset <- DESeq(ddset)

reset <- results(ddset)
reset <- reset[order(reset$padj),]
head(reset)

summary(reset, alpha=0.01)
summary(reset)
# out of 80951 with nonzero total read count
# adjusted p-value < 0.1
# LFC > 0 (up)       : 196, 0.24%
# LFC < 0 (down)     : 278, 0.34%
# outliers [1]       : 243, 0.3%
# low counts [2]     : 0, 0%
# (mean count < 6)
# [1] see 'cooksCutoff' argument of ?results
# [2] see 'independentFiltering' argument of ?results
resultsNames(ddset)
#"Intercept"   "tank_C_vs_A"
write.csv(reset, file = "reset.csv")


##PCA
vsdet <- varianceStabilizingTransformation(ddset, blind=FALSE)
plotPCA(vsdet, intgroup=c("tank"))

plotPCA(vsdet, intgroup = c("tank", "stage"))


###############Model 7: LARVAE ONLY, design= tank

colData <- subset(colData, stage=="Larvae")
head(colData)

countData <- countData[, c(13:24)]
head(countData)

##deseq

ddslt <- DESeqDataSetFromMatrix(colData = colData, countData = countData, design = ~ tank)
dim(ddslt)

ddslt <- ddslt[ rowSums(counts(ddslt)) > 100, ]

ddslt <- DESeq(ddslt)

reslt <- results(ddslt)
reslt <- reslt[order(reslt$padj),]
head(reslt)

summary(reslt, alpha=0.01)
summary(reslt)

resultsNames(ddslt)
#write.csv(reslt, file = "reslt.csv")

##PCA
vsdlt <- varianceStabilizingTransformation(ddslt, blind=FALSE)
plotPCA(vsdlt, intgroup=c("tank"))


###############Model: Tank A ONLY, design= stage

colData <- subset(colData, tank=="A")
head(colData)

countData <- countData[, c(1:6, 13:18)]
head(countData)

##deseq

ddsta <- DESeqDataSetFromMatrix(colData = colData, countData = countData, design = ~ stage)
dim(ddsta)

ddsta <- ddsta[ rowSums(counts(ddsta)) > 100, ]

ddsta <- DESeq(ddsta)

resta <- results(ddsta)
resta <- resta[order(resta$padj),]
head(resta)

summary(resta, alpha=0.01)
summary(resta)

resultsNames(ddsta)

##PCA
vsdta <- varianceStabilizingTransformation(ddsta, blind=FALSE)
plotPCA(vsdta, intgroup=c("stage"))

###############Model: Tank C ONLY, design= stage

colData <- subset(colData, tank=="C")
head(colData)

countData <- countData[, c(7:12, 19:24)]
head(countData)

##deseq

ddstc <- DESeqDataSetFromMatrix(colData = colData, countData = countData, design = ~ stage)
dim(ddstc)

ddstc <- ddstc[ rowSums(counts(ddstc)) > 100, ]

ddstc <- DESeq(ddstc)

restc <- results(ddstc)
restc <- restc[order(restc$padj),]
head(restc)

summary(restc, alpha=0.01)
summary(restc)

resultsNames(ddstc)

##PCA
vsdtc <- varianceStabilizingTransformation(ddstc, blind=FALSE)
plotPCA(vsdtc, intgroup=c("stage"))


################################################################
# Data quality assessment by sample clustering and visualization 

plotMA(ress, main="DESeq2", ylim=c(-2,2))

plotMA(rest, main="DESeq2", ylim=c(-2,2))

####Volcano plot##########
which(resstst$padj <0.05)
sig.genes.stst <- resstst[which(resstst$padj <0.05),]
head(sig.genes.stst)

with(resstst, plot(log2FoldChange, -log10(pvalue), pch=20, main="Differential gene expression of 24hr Light vs 24hr Dark", xlim=c(-40,40), ylim=c(0,15)))

points(sig.genes.stst$log2FoldChange,-log10(sig.genes.stst$pvalue), col='red', pch=20)

####Volcano plot #2
which(rest$padj <0.05)
sig.genes.t <- rest[which(rest$padj <0.05),]
head(sig.genes.t)

with(rest, plot(log2FoldChange, -log10(pvalue), pch=20, main="Differential gene expression of 24hr Light vs 24hr Dark", xlim=c(-4,4), ylim=c(0,45)))

points(sig.genes.t$log2FoldChange,-log10(sig.genes.t$pvalue), col='red', pch=20)

## Check out one of the genes to see if it's behaving as expected....
#d <- plotCounts(dds, gene="TRINITY_DN44744_c1_g2_TRINITY_DN44744_c1_g2_i2_g.17686_m.17686", intgroup=(c("health","day","location")), returnData=TRUE)
#d
#p <- ggplot(d, aes(x= health, y=count, shape = location)) + theme_minimal() + theme(text = element_text(size=20), panel.grid.major = element_line(colour = "grey"))
#p <- p + geom_point(position=position_jitter(w=0.3,h=0), size = 3) + ylim(0,500)
#p


## Check out one of the genes to see interaction between score, health and expression....
d <- plotCounts(ddsstst, gene="TRINITY_DN38539_c0_g1_i1", intgroup=(c("tank", "stage")), returnData=TRUE)d
p <- ggplot(d, aes(x= tank, y=count, shape = stage, color = tank)) + theme_minimal() + theme(text = element_text(size=20), panel.grid.major = element_line(colour = "grey"))
p <- p + geom_point(position=position_jitter(w=0.3,h=0), size = 3) 
p

#p <- ggplot(d, aes(x=score, y=count, color=health, group=health)) 
#p <- p +  geom_point() + stat_smooth(se=FALSE,method="loess") +  scale_y_log10()
#p

############## PCA plots
vsdt <- varianceStabilizingTransformation(ddst, blind=FALSE)

plotPCA(vsdt, intgroup=c("tank"))

vsds <- varianceStabilizingTransformation(ddss, blind=FALSE)
plotPCA(vsds, intgroup=c("stage"))

vsd <- varianceStabilizingTransformation(dds, blind=FALSE)
plotPCA(vsd, intgroup=c("stage"))

vsdstst <- varianceStabilizingTransformation(ddsstst, blind=FALSE)
plotPCA(vsdstst, intgroup=c("tank"))

###########More advance PCA plot from Melissas beetle DESeq2 code
#####  PCA
rld <- rlog(ddsstst, blind=FALSE)
vsd <- vst(ddsstst, blind=FALSE)

data <- plotPCA(vsd,intgroup=c("stage","tank"), returnData=TRUE)
percentVar <- round(100 * attr(data, "percentVar"))

#data$devstage <- factor(data$devstage, levels=c("L3L","PP1","PD1","AD4"), labels = c("L3L","PP1","PD1","AD4"))

#pdf(file="PCA_sex_by_stage.pdf", height=5.5, width=5.5)
ggplot(data, aes(PC1, PC2, color=tank, shape=stage)) +
  geom_point(size=4, alpha=0.85) +
  xlab(paste0("PC1: ",percentVar[1],"% variance")) +
  ylab(paste0("PC2: ",percentVar[2],"% variance")) +
  theme_minimal()
dev.off()

############Venn Diagrams
install.packages("VennDiagram")
library(VennDiagram)
# A simple two-set diagram 
venn.plot <- draw.pairwise.venn(130, 10, 129, c("First", "Second"))
grid.draw(venn.plot)


############# export results to excel
#install.packages("xlsx")
#library(xlsx)

write.csv(rest, file = "rest.csv")

write.csv(ress, file = "ress.csv")
```



### Results print out for each of the 3 designs I focused on:

1) **Model 5: design = ~ stage + tank + stage:tank**

* Results at **padj 0.1**:

  ```
  out of 120145 with nonzero total read count
  adjusted p-value < 0.1
  LFC > 0 (up)       : 129, 0.11%
  LFC < 0 (down)     : 130, 0.11%
  outliers [1]       : 279, 0.23%
  low counts [2]     : 0, 0%
  (mean count < 3)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```

* Results at **padj 0.05**:

  ```
  out of 120145 with nonzero total read count
  adjusted p-value < 0.05
  LFC > 0 (up)       : 90, 0.075%
  LFC < 0 (down)     : 97, 0.081%
  outliers [1]       : 279, 0.23%
  low counts [2]     : 0, 0%
  (mean count < 3)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```

* Results at **padj 0.01**:

  ```
  out of 120145 with nonzero total read count
  adjusted p-value < 0.01
  LFC > 0 (up)       : 54, 0.045%
  LFC < 0 (down)     : 57, 0.047%
  outliers [1]       : 279, 0.23%
  low counts [2]     : 0, 0%
  (mean count < 3)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```



2) **Model 6: EGG ONLY, design= tank**

- Results at **padj 0.1**:

  ```
  out of 80951 with nonzero total read count
  adjusted p-value < 0.1
  LFC > 0 (up)       : 196, 0.24%
  LFC < 0 (down)     : 278, 0.34%
  outliers [1]       : 243, 0.3%
  low counts [2]     : 0, 0%
  (mean count < 6)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```

- Results at **padj 0.05**:

  ```
  out of 80951 with nonzero total read count
  adjusted p-value < 0.05
  LFC > 0 (up)       : 141, 0.17%
  LFC < 0 (down)     : 181, 0.22%
  outliers [1]       : 243, 0.3%
  low counts [2]     : 0, 0%
  (mean count < 6)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```

- Results at **padj 0.01**:

  ```
  out of 80951 with nonzero total read count
  adjusted p-value < 0.01
  LFC > 0 (up)       : 75, 0.093%
  LFC < 0 (down)     : 96, 0.12%
  outliers [1]       : 243, 0.3%
  low counts [2]     : 0, 0%
  (mean count < 6)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```


2) **Model 7: LARVAE ONLY, design= tank**

- Results at **padj 0.1**:

  ```
  out of 82588 with nonzero total read count
  adjusted p-value < 0.1
  LFC > 0 (up)       : 1587, 1.9%
  LFC < 0 (down)     : 2035, 2.5%
  outliers [1]       : 288, 0.35%
  low counts [2]     : 20693, 25%
  (mean count < 13)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```

- Results at **padj 0.05**:

  ```
  out of 82588 with nonzero total read count
  adjusted p-value < 0.05
  LFC > 0 (up)       : 970, 1.2%
  LFC < 0 (down)     : 1331, 1.6%
  outliers [1]       : 288, 0.35%
  low counts [2]     : 20693, 25%
  (mean count < 13)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```

- Results at **padj 0.01**:

  ```
  out of 82588 with nonzero total read count
  adjusted p-value < 0.01
  LFC > 0 (up)       : 400, 0.48%
  LFC < 0 (down)     : 591, 0.72%
  outliers [1]       : 288, 0.35%
  low counts [2]     : 20693, 25%
  (mean count < 13)
  [1] see 'cooksCutoff' argument of ?results
  [2] see 'independentFiltering' argument of ?results
  ```



# BLAST

BLAST (v2.7.1)  "finds regions of similarity between biological sequences. The program compares nucleotide or protein sequences to sequence databases and calculates the statistical significance."

**Info on BLAST:** https://blast.ncbi.nlm.nih.gov/Blast.cgi 
**Additional resources for on Sequence Based Function Annotation using BLAST:** https://biohpc.cornell.edu/doc/annotation2017_1.pdf and https://community.gep.wustl.edu/repository/course_materials_WU/annotation/Basics_of_BLAST.pdf ***both very handy!!***

I used BLAST to compare my results to three databases:

- **Atlantic salmon database** (found here: <ftp://ftp.ncbi.nih.gov/genomes/Salmo_salar>)
- **Uniref90** (found here: https://www.uniprot.org/downloads)
- **nr databased** (found here: <ftp://ftp.ncbi.nlm.nih.gov/blast/db/FASTA/nr.gz>)

***Note #1:*** I had difficulty analyzing/finding GO terms for the Atlantic Salmon and nr results so moving foward I focused on the results from the **uniref90** comparison. 

***Note #2:*** BLAST runs take **~15.5 hrs**; Therefore, if you can, its best to allocate a longer run time on the server (I used "poolmemq" but perhaps "l1wkpoolq" would allow the longer runs to finish without quitting.)

***Note #3:*** The input files were too big so we **split** it into 6 smaller files and ran blast against each file (so there will be 6 scripts for each databased comparison).  The code to spilt the file is below:

**File Name:** `fasta_split_ae_52418.script`

**Date run:** 5/24/18

**Code:**

```
#PBS -l nodes=1:ppn=10,mem=50gb,vmem=50gb
#PBS -q poolmemq 
##PBS -l walltime=30:00:00
#PBS -N Fasta_split 
# Join STDERR TO STDOUT.  (omit this if you want separate STDOUT AND STDERR)
#PBS -j oe   
# Send me mail on job start,  job end and if job aborts
#PBS -M hlachanc@uvm.edu
#PBS -m bea


##adapted from aevans code
##split â€“d numbers the output, versus alphabetized, and the T_all on the end is the output prefix, creating T_all00 T_all01 T_all02 (etc.) output files.
 
awk '/^>/ {printf("\n%s\n",$0);next; } { printf("%s",$0);}  END {printf("\n");}' < Trinity_all.fasta > Trinity_all_single.fasta
tail -n +2 Trinity_all_single.fasta >Trinity_all_single-1.fasta
split -dl 200000 Trinity_all_single-1.fasta T_all

```

### 

### BLAST(blastx): Uniref90 (example of 1 out of 6 scripts)

blastx = nucleotide query  vs. protein database

**File Name:** "blast_uniprot_T_all00_52418.script"

**Date run:** 5/24/18

**Code:**

```
#PBS -l nodes=1:ppn=14,mem=240gb,vmem=240gb
#PBS -l walltime=30:00:00
#PBS -q poolmemq
#PBS -N blast_uniprot_T_all00
#PBS -j oe
#PBS -M hlachanc@uvm.edu
#PBS -m bea

##adapted from aevans code

spack load blast-plus
spack load perl
cd $PBS_O_WORKDIR
#gunzip -c uniref90.fasta.gz|makeblastdb -in -  -dbtype prot -title uniref90 -out uniref90_database
blastx -db uniref90_database -query T_all00 -out blast_Tall_uniprot_T_all00.out -max_target_seqs 1 -outfmt 6 -evalue 1e-5 -num_threads 12
```

### BLAST(blastp): NR (example of 1 out of 6 scripts)

blastp = protein query  vs. protein database

**File Name:** "blast_nr_T_all00_52418.script"

**Date run:** 6/6/18

**Code:**

```
#PBS -l nodes=1:ppn=14,mem=240gb,vmem=240gb
#PBS -l walltime=30:00:00
#PBS -q poolmemq
#PBS -N blast_nr_T_all00
#PBS -j oe
#PBS -M hlachanc@uvm.edu
#PBS -m bea

##adapted from aevans code

spack load blast-plus
spack load perl
cd $PBS_O_WORKDIR
#gunzip -c nr.gz|makeblastdb -in -  -dbtype prot -title nr -out nr_database
blastp -db nr_database -query T_all00 -out blast_Tall_nr_T_all00.out -max_target_seqs 1 -outfmt 6 -evalue 1e-5 -num_threads 12

```

### BLAST(tblastx): Atlantic Salmon (example of 1 out of 6 scripts)

TBLASTX searches translated nucleotide databases using a translated nucleotide query (tblastx = translated query  vs. translated database)

**File Name:** "blast_salmon_T_all00_52418.script"

**Date run:** 5/25/18

**Code:**

```
#PBS -l nodes=1:ppn=14,mem=240gb,vmem=240gb
#PBS -l walltime=30:00:00
##PBS -q poolmemq
#PBS -N blast_salmon_T_all00
#PBS -j oe
#PBS -M hlachanc@uvm.edu
#PBS -m bea

##adapted from aevans code

spack load blast-plus
spack load perl
cd $PBS_O_WORKDIR
## makeblastdb -in salmon_rna.fa -dbtype nucl -out salmon_database
tblastx -db salmon_database -query T_all00 -out blast_Tall_salmon_T_all00.xml -max_target_seqs 1 -outfmt 14 -evalue 1e-5 -num_threads 12
```



# DESeq2 and BLAST results -> GOterms/Function (using Uniprot)

Below are steps to take the DESeq2 and BLAST results and turn them into a list of GOterms/functions 

###1) Output results from DESeq2

* Run **"Model:Egg ONLY, design = tank"** in [DESeq2_ciscolight_6418.R](#DESeq2)

* output the results as a .csv using the following code as the end of the Model code:

  ```
  write.csv(reset, file = "reset.csv") 
  ```

### 2) Get a list of UniRef90 IDs associated with the genes identified by the DESeq2 design

Using the following steps to create a file with either just the ***top significant genes*** or ***all genes*** from identified in each DESeq2 design.

* **Create a list of top significant genes:**

  * Look at the .csv file you created and filter based on **padj** (smallest to largest)

  * Delete sequences higher then 0.05 and save as "reset_top258.csv"

  * Save a .txt version of the top sig genes file which includes only the seqIDs (new .txt file called "top258et_seqID.txt") 

    * Create .txt file by going to **"save as"** in excel and saving it as a **"Text (Tab delimited) (*.txt)"** file
    * this file is used to pull out the uniprot IDs we need
    * ***NOTE:*** sometimes the format is wonky so you can take the text and paste it in a working "*_seqID.txt" file and save as (new file name) so you ensure you have the correct format.

  * Using a bash script based on Andy Evans' recommendations you can combine the seqIDs with the geneIDs and create a file with both called "'top"

    **Bash script:** "filertop258et_6618.script"
    **Code:** *drag files over with WINscp and run directly on server(?)*

    ```
    for i in `cat top258et_seqID.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > top258et_seq_geneID.txt
    ```

* **Create a list of all genes**

  * Save only the **seqIDs** from the .csv file generated by DESeq2 as a .txt file by going to **"save as" **in excel and saving it as a **"Text (Tab delimited) (*.txt)"** file

  * Use a similar bash script as above to pull out the UniRef90 ID's that are associated with the genes output from the DESeq2 design the .txt file is based off of:

    **Bash script:** "filteret_all_11519.script"
    **Code:** *drag files over with WINscp and run directly on server(?)*

    ```
    for i in `cat reset_seqID_all.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reset_seq_geneID.txt
    ```

###3) Upload list of UniRef90 IDs to Uniprot website

* Go to the UniProt Website and click on **"Retrieve/ID mapping"**: https://www.uniprot.org/uploadlists/
* Open the .txt file we generated in step 2 in excel. Copy the column of UniRef90 IDs and paste the list into the "1. Provide your identifiers" box on the UniProt website. 
* Under "2. Select option" convert from **UniRef90** to **UniProtKB**
* Click submit

### 4) Save UniProt results

* ***Basic Download:***
  * Click the button called **"Download"** located above the table of results
  * Ensure **"Download all"** is clicked and the format is set to **"Excel"**
  * Open the file and save it to your desired location (my results are located in: GradSchool\PiloStudy2017\dataanalysis\results)
* ***Advanced Download:***
  * Click **"Columns"** located above the results table
  * Check the boxes next to whatever columns you want to include then click save. I recommend the following:
    * Entry Name
    * Gene names 
    * Organism
    * Protein Names
    * Any/all of the options listed under **"Gene Ontology (GO)"** (But especially **"Gene ontology IDs"** and **"Gene ontology (GO)"**)
  * Click the button called **"Download"** located above the table of results
  * Ensure **"Download all"** is clicked and the format is set to **"Excel"**
  * Open the file and save it to your desired location (my results are located in: GradSchool\PiloStudy2017\dataanalysis\results)

### 5) Additional Research

* Using the results provided by UniProt, take time to look up each gene to learn more about the function and how it could possibly play into the different effects in the study. 



### ***Additional files for the other DESeq2 designs***

* **Model: Larvae ONLY, design = tank**

  * deleted seq above 9.88 E-05 padj (save top 210 sig)

    * **.csv file name:** reslt_top210.csv

    * **.txt file with just seq IDs:** top210lt_seqID.txt

    * **Bash script file name**: filtertop210lt_6618.script

      * **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat top210lt_seqID.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > top210lt_seq_geneID.txt
        ```

    * **.txt file with seq IDs and gene IDs:** top210lt_seq_geneID.txt

  * deleted seq above 0.05 padj

    - **.csv file name:** reslt_05.csv

    - **.txt file with just seq IDs:** reslt_seqID_05.txt

    - **Bash script file name**: "filterlt_05_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reslt_seqID_05.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reslt_seq_geneID_05.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reslt_seq_geneID_05.txt

  * deleted seq above 0.01 padj

    - **.csv file name:** reslt_01.csv

    - **.txt file with just seq IDs:** reslt_seqID_01.txt

    - **Bash script file name**: "filterlt_01_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reslt_seqID_01.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reslt_seq_geneID_01.txt
        ```

    - **.txt file with seq IDs and gene IDs:** 

  * deleted seq above 0.1 padj

    - **.csv file name:** reslt_1.csv

    - **.txt file with just seq IDs:** reslt_seqID_1.txt

    - **Bash script file name**: "filterlt_1_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reslt_seqID_1.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reslt_seq_geneID_1.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reslt_seq_geneID_1.txt

  * ALL genes

    - **.csv file name:** reslt.csv

    - **.txt file with just seq IDs:** reslt_seqID_all.txt

    - **Bash script file name**: "filterlt_all_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reslt_seqID_all.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reslt_seq_geneID_all.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reslt_seq_geneID_all.txt

* **Model: design = ~ stage + tank + stage:tank**

  * deleted seq above 0.05 padj 

    * **.csv file name:** resstst_05.csv

    * **.txt file with just seq IDs:** resstst_seqID_05.txt

    * **Bash script file name**: "filterstst_05_12219.script"

      * **Code:** *drag files over with WINscp and run directly on server(?)*

      ```
      for i in `cat resstst_seqID_05.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > resstst_seq_geneID_05.txt
      ```

    * **.txt file with seq IDs and gene IDs:** resstst_seq_geneID_05.txt

  * deleted seq above 0.01 padj

    - **.csv file name:** resstst_01.csv

    - **.txt file with just seq IDs:** resstst_seqID_01.txt

    - **Bash script file name**: "filterstst_01_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat resstst_seqID_01.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > resstst_seq_geneID_01.txt
        ```

    - **.txt file with seq IDs and gene IDs:** resstst_seq_geneID_01.txt

  * deleted seq above 0.1 padj

    - **.csv file name:** resstst_1.csv

    - **.txt file with just seq IDs:** resstst_seqID_1.txt

    - **Bash script file name**: "filterstst_1_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat resstst_seqID_1.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > resstst_seq_geneID_1.txt
        ```

    - **.txt file with seq IDs and gene IDs:** resstst_seq_geneID_1.txt

  - ALL genes

    - **.csv file name:** resstst_.csv

    - **.txt file with just seq IDs:** resstst_seqID_all.txt

    - **Bash script file name**: "filterstst_all_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat resstst_seqID_all.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > resstst_seq_geneID_all.txt
        ```

    - **.txt file with seq IDs and gene IDs:** resstst_seq_geneID_all.txt

* **Model: Egg ONLY, design = tank**

  * deleted seq above 0.05 padj

    - **.csv file name:** reset_05.csv

    - **.txt file with just seq IDs:** reset_seqID_05.txt

    - **Bash script file name**: "filteret_05_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reset_seqID_05.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reset_seq_geneID_05.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reset_seq_geneID_05.txt

  * deleted seq above 0.01 padj

    - **.csv file name:** reset_01.csv

    - **.txt file with just seq IDs:** reset_seqID_01.txt

    - **Bash script file name**: "filteret_01_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reset_seqID_01.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reset_seq_geneID_01.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reset_seq_geneID_01.txt

  * deleted seq above 0.1 padj

    - **.csv file name:** reset_1.csv

    - **.txt file with just seq IDs:** reset_seqID_1.txt

    - **Bash script file name**: "filteret_1_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reset_seqID_1.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reset_seq_geneID_1.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reset_seq_geneID_1.txt

  * ALL genes

    - **.csv file name:** reset.csv

    - **.txt file with just seq IDs:** reset_seqID_all.txt

    - **Bash script file name**: "filteret_all_12219.script"

      - **Code:** *drag files over with WINscp and run directly on server(?)*

        ```
        for i in `cat reset_seqID_all.txt`; do grep ${i} blast_Tall_uniprot_T_all0*.out; done > reset_seq_geneID_all.txt
        ```

    - **.txt file with seq IDs and gene IDs:** reset_seq_geneID_all.txt


# Functional Enrichment

Most Transcriptomics papers run a functional enrichment analysis to (**elaborate**)... There are many scripts/programs that run functional enrichment analyses.  There two that were recommended to me by my collaborators were **GO_MWU.R** and **Panther**. I started with GO_MWU.R however I ran into a few issues so I wanted to run a second method to validate my results (or lack of results) which is when I ran Panther.



**Functional enrichment (suggested path from meeting with Melissa 4/3/18)**: Transdecoder -> BLAST -> uniprot website (take uniprot IDs from BLAST_uniref and get GO terms) -> get go terms from other blast runs, create a master table by merging the outputs and then use that table for GOMWU.Rnm 



## GO_MWU.R

GO_MWU.R (v_) is an R script that "uses continuous measure of significance (such as fold-change or -log(p-value) ) to identify GO categories that are significantly enriches with either up- or down-regulated genes. The advantage - no need to impose arbitrary significance cutoff."

This method was used in our Ecological genomics class (Git hub link: https://github.com/hmlrocks/Ecological-Genomics2/blob/master/Notebook.md#id-section9)

**Info on GO_MWU.R:** https://github.com/z0on/GO_MWU 

Before running GO_MWU.R I had to create the **input** and **annotation** files required. The overall process is described below:

- **Input:** 
  - using the results file from the DESeq2 run (***res(et/lt/stst)***) calculate the **negative log 10 pvalue**.
    - use the following function to calc neg log pvalue: `=-log(pvalue)`
  - save a new file of just the **"seqID"** and **"neglogpval"** columns
    - new file: `res(et/lt/stst)_neglogpval.csv`
  - Open the new file in Notepad ++; copy the data and paste it into the input file we used in the Ecological Genomics Class; (yes headers!) save as a new file (so you don't erase the class file)
    - This is because I ran into MANY formatting issues which I was unable to resolve no matter how I tried.  Since the class file worked, Melissa suggested pasting my data into that file (removing the class data of course) and it magically started to work!
    - New files: `input_(et/lt/stst))_72718`
- **Annotation Table:** To get the annotation table we had to create and merge many files.  For more detail see handwritten notes in notebook vol 2.  A brief description below:
  - run R code to merge files: `mergefilesforannotationtable.R`
    - output: `annotationtable_all.csv`
  - edit output to include only "seqID" and "GOterms"
    - from excel, save as: `ann_all_edited.txt` *Needs to be tab delim*
  - Open the new file in Notepad ++; copy the data and paste it into the input file we used in the Ecological Genomics Class; (no headers!) save as a new file (so you don't erase the class file)
    - New file: `annotationtable_all_72718`
      - *use this file for all three subsets*

 

I ran GO_MWU.R three times with 3 different designs I ran in DESeq2 - ***NONE of the designs showed Functional enrichment***

**File Name:** "GO_MWU_et_72718.R"

**Date run:** 7/31/18

**Code:**

```
setwd("C:/Users/Hannah/Desktop/Grad School/Pilot Study 2017/data analysis/Func Enrich cisco")# Can run for all 6 inputs, mean and max for each of the three pop. pair FSTs and can do for each GO division BP, MF, CC
# Edit these to match your data file names: 
input_et="input_et_72718" # two columns of comma-separated values: gene id, continuous measure of significance. To perform standard GO enrichment analysis based on Fisher's exact test, use binary measure (0 or 1, i.e., either sgnificant or not).
  ## Make sure the input and goAnnotations filea are saved as Unix (LF) - open in TextWrangler, save as, change from Classic Mac to Unix (LF)!!!
goAnnotations="annotationtable_all_72718" # two-column, tab-delimited, one line per gene, multiple GO terms separated by semicolon. If you have multiple lines per gene, use nrify_GOtable.pl prior to running this script.
goDatabase="go.obo" # download from http://www.geneontology.org/GO.downloads.ontology.shtml
goDivision="BP" # either MF, or BP, or CC
source("gomwu.functions.R")

# Calculating stats. It takes ~3 min for MF and BP. Do not rerun it if you just want to replot the data with different cutoffs, go straight to gomwuPlot. If you change any of the numeric values below, delete the files that were generated in previos runs first.
gomwuStats(input_et, goDatabase, goAnnotations, goDivision, perlPath="perl", # replace with full path to perl executable if it is not in your system's PATH already
largest=0.1,  # a GO category will not be considered if it contains more than this fraction of the total number of genes
smallest=5,   # a GO category should contain at least this many genes to be considered 
clusterCutHeight=0.25) # threshold for merging similar (gene-sharing) terms. 
#	Alternative="g" # by default the MWU test is two-tailed; specify "g" or "l" of you want to test for "greater" or "less" instead)
# do not continue if the printout shows that no GO terms pass 10% FDR.

```

**Results:**

```
go.obo annotationtable_all_72718 input_et_72718 BP largest=0.1 smallest=5cutHeight=0.25

Run parameters:

largest GO category as fraction of all genes (largest)  : 0.1
         smallest GO category as # of genes (smallest)  : 5
                clustering threshold (clusterCutHeight) : 0.25

-----------------
retrieving GO hierarchy, reformatting data...

-------------
go_reformat:
Genes with GO annotations, but not listed in measure table: 4

Terms without defined level (old ontology?..): 0
-------------
-------------
go_nrify:
372 categories, 148 genes; size range 5-14.8
	55 too broad
	238 too small
	79 remaining

removing redundancy:

calculating GO term similarities based on shared genes...
16 non-redundant GO categories of good size
-------------
Continuous measure of interest: will perform MWU test
0  GO terms at 10% FDR
```



**File Name:** "GO_MWU_lt_72718.R"

**Date run:** 7/31/18

**Code:**

```
setwd("C:/Users/Hannah/Desktop/Func Enrich class")
# Can run for all 6 inputs, mean and max for each of the three pop. pair FSTs and can do for each GO division BP, MF, CC
# Edit these to match your data file names: 
input_lt="input_lt_72718" # two columns of comma-separated values: gene id, continuous measure of significance. To perform standard GO enrichment analysis based on Fisher's exact test, use binary measure (0 or 1, i.e., either sgnificant or not).
  ## Make sure the input and goAnnotations filea are saved as Unix (LF) - open in TextWrangler, save as, change from Classic Mac to Unix (LF)!!!
goAnnotations="annotationtable_all_72718" # two-column, tab-delimited, one line per gene, multiple GO terms separated by semicolon. If you have multiple lines per gene, use nrify_GOtable.pl prior to running this script.
goDatabase="go.obo" # download from http://www.geneontology.org/GO.downloads.ontology.shtml
goDivision="BP" # either MF, or BP, or CC
source("gomwu.functions.R")


# Calculating stats. It takes ~3 min for MF and BP. Do not rerun it if you just want to replot the data with different cutoffs, go straight to gomwuPlot. If you change any of the numeric values below, delete the files that were generated in previos runs first.
gomwuStats(input_lt, goDatabase, goAnnotations, goDivision, perlPath="perl", # replace with full path to perl executable if it is not in your system's PATH already
largest=0.1,  # a GO category will not be considered if it contains more than this fraction of the total number of genes
smallest=5,   # a GO category should contain at least this many genes to be considered 
clusterCutHeight=0.25)) # threshold for merging similar (gene-sharing) terms. 
#	Alternative="g" # by default the MWU test is two-tailed; specify "g" or "l" of you want to test for "greater" or "less" instead)
# do not continue if the printout shows that no GO terms pass 10% FDR.
```

**Results:** 

```
go.obo annotationtable_all_72718 input_lt_72718 BP largest=0.1 smallest=5cutHeight=0.25

Run parameters:

largest GO category as fraction of all genes (largest)  : 0.1
         smallest GO category as # of genes (smallest)  : 5
                clustering threshold (clusterCutHeight) : 0.25

-----------------
retrieving GO hierarchy, reformatting data...

-------------
go_reformat:
Genes with GO annotations, but not listed in measure table: 144

Terms without defined level (old ontology?..): 0
-------------
-------------
go_nrify:
359 categories, 136 genes; size range 5-13.6
	68 too broad
	234 too small
	57 remaining

removing redundancy:

calculating GO term similarities based on shared genes...
15 non-redundant GO categories of good size
-------------
Continuous measure of interest: will perform MWU test
0  GO terms at 10% FDR
```



**File Name:**  "GO_MWU_stst_72718.R"

**Date run:** 7/31/18

**Code:**

```
setwd("C:/Users/Hannah/Desktop/Func Enrich class")
# Can run for all 6 inputs, mean and max for each of the three pop. pair FSTs and can do for each GO division BP, MF, CC
# Edit these to match your data file names: 
input_stst="input_stst_72718" # two columns of comma-separated values: gene id, continuous measure of significance. To perform standard GO enrichment analysis based on Fisher's exact test, use binary measure (0 or 1, i.e., either sgnificant or not).
  ## Make sure the input and goAnnotations filea are saved as Unix (LF) - open in TextWrangler, save as, change from Classic Mac to Unix (LF)!!!
goAnnotations="annotationtable_all_72718" # two-column, tab-delimited, one line per gene, multiple GO terms separated by semicolon. If you have multiple lines per gene, use nrify_GOtable.pl prior to running this script.
goDatabase="go.obo" # download from http://www.geneontology.org/GO.downloads.ontology.shtml
goDivision="BP" # either MF, or BP, or CC
source("gomwu.functions.R")


# Calculating stats. It takes ~3 min for MF and BP. Do not rerun it if you just want to replot the data with different cutoffs, go straight to gomwuPlot. If you change any of the numeric values below, delete the files that were generated in previos runs first.
gomwuStats(input_stst, goDatabase, goAnnotations, goDivision, perlPath="perl", # replace with full path to perl executable if it is not in your system's PATH already
largest=0.1,  # a GO category will not be considered if it contains more than this fraction of the total number of genes
smallest=5,   # a GO category should contain at least this many genes to be considered 
clusterCutHeight=0.25) # threshold for merging similar (gene-sharing) terms. 
#	Alternative="g" # by default the MWU test is two-tailed; specify "g" or "l" of you want to test for "greater" or "less" instead)
# do not continue if the printout shows that no GO terms pass 10% FDR.
```

**Results:**

```
go.obo annotationtable_all_72718 input_stst_72718 BP largest=0.1 smallest=5cutHeight=0.25

Run parameters:

largest GO category as fraction of all genes (largest)  : 0.1
         smallest GO category as # of genes (smallest)  : 5
                clustering threshold (clusterCutHeight) : 0.25

-----------------
retrieving GO hierarchy, reformatting data...

-------------
go_reformat:
Genes with GO annotations, but not listed in measure table: 2

Terms without defined level (old ontology?..): 0
-------------
-------------
go_nrify:
372 categories, 148 genes; size range 5-14.8
	55 too broad
	238 too small
	79 remaining

removing redundancy:

calculating GO term similarities based on shared genes...
16 non-redundant GO categories of good size
-------------
Continuous measure of interest: will perform MWU test
0  GO terms at 10% FDR
```



***Try GO_MWU.R with ALL genes not just top significant?***



## Panther

Panther is an online tool recommended by Trevor Krabbenhoft.  

**Info on Panther:** http://www.pantherdb.org/ 
**Steps on how to run Panther for enrichment analysis:** http://geneontology.org/page/go-enrichment-analysis

I found that Panther wasn't very thorough (for my purposes) since the closest reference it has is zebrafish (*Danio rerio*).  Therefore I just went off the GO_MWU.R results (which unfortunately showed no functional enrichment)


