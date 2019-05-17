# Mothur_oligo_batch
This repository was created by Ruben Props to run both mothur and oligotyping via a batch script on the University of Michigan HPC Flux system on 16S rRNA fastq files produced by Illumina MiSeq sequencing.

To Run the batch script (Mothur.batch.taxass), please follow the directions below.  
**01/26/2017 -- Added Mothur.batch.general and make_phyloseq_general for general classification with SILVA**

## 1. Sign into the University of Michigan Flux system.

## 2. Navigate to your scratch folder.

## 3. Install Oligotyping software in your local directory
```R
module load python-anaconda3/201607
pip install --user oligotyping
```
To check to make sure that the path to the oligotyping package is correct, try using a command from the oligotyping module:

```
oligotype -h
```
If the output brings up the manual for the `oligotype` command, you have successfully installed the oligotyping software!

If the ouput says "-bash: oligotype: command not found", then it is possible that your path to the oligotyping package is incorrect.  Therefore, we need to fix our path.  To do so we will edit the .bash_profile file in our home directory.  Therefore, you will need to fix your path by typing the following in your current directory:
```R
cd 	#This will take you to your *home* directory
ls -a 	#Show all files in your home directory, including hidden ones (like your .bash_profile file!)
```
Then under the line that includes "PATH=$PATH:$HOME/bin" that you have the following:
```R
export PATH=/home/your_unique_name/.local/bin:/:$PATH  # Be sure to change "rprops" to YOUR USER NAME!
```
If you do not have the above line, please edit it with nano:

```
nano .bash_profile
```

## 4. Load mothur, R, and ncbi-blast.
```R
module load mothur R ncbi-blast
```
For the TaxAss script make sure you have the <code>dplyr</code> and <code>reshape</code> R packages installed in your local R version.

```
R # open R
install.packages(c("dplyr", "reshape","splitstackshape","data.table", "gtools")) # install the packages
source("https://bioconductor.org/biocLite.R")
biocLite("phyloseq") 
quit() # exit R
```

## 5. Copy the `mothur.batch.taxass` file into your the folder with the fastq files.

## 6. **Make sure the sample names of your fastq files are correct. Mothur does **not** accept: ':', '-' or '/' !**
If you have to rename your files, you can run the following line of code:
```R
find /scratch/vdenef_fluxm/your_unique_name/path_to_fastq_files -type f -exec rename '-' '' {} \;
```
The above code  will remove "-" from your sample names. You may have to run rerun this several times if you have multiple "-" in your sample names.

## 7. To select for a taxon of interest to perform oligotyping analysis on, run the following code:
**If you want to oligotype everything do *not* run this.** 

Replace <code> Bacteria;Proteobacteria;Betaproteobacteria;Burkholderiales;betI;betI_A </code> with your taxon of interest.
```
sed -i "s/Bacteria/Bacteria;Proteobacteria;Betaproteobacteria;Burkholderiales;betI;betI_A/g" mothur.batch.taxass
```

## 8. Make stability file (file with sample names and corresponding fastq files for that sample).
Should you get an error during <code>make.contigs</code> then check if the &#95;R1&#95; and &#95;R2&#95; pattern does not occur in the sample name.
```
paste <(ls *_R1_*.fastq | awk -F"_" '{print $1}') <(ls *_R1_*.fastq) <(ls *_R2_*.fastq) > stability.file
```
### You can remove hidden characters (often caused by editing in windows) as follows:
```
dos2unix mothur.batch.taxass
dos2unix mothur.batch.taxass.pbs
```

## 9. You're ready to go!  Option A:  Run the `mothur.batch.taxass` or option B: submit a job to flux with the `mothur.batch.taxass.pbs`.
```R
mothur mothur.batch.taxass  #Option A
 # OR
qsub mothur.batch.taxass.pbs  #Option B
```

## Optional step 10: Create a phyloseq object with the `make_phyloseq.R` script.
```
Rscript make_phyloseq.R mothur.shared mothur.cons.taxonomy ./stability-m0.10-A0-M0-d4/MATRIX-COUNT.txt ./stability-m0.10-A0-M0-d4/oligo.taxonomy
```
There are 4 inputs in the above code:
 1. mothur `shared` file
 2. mothur `cons.taxonomy` file 
 3. oligotyping `shared` file 
 4. oligotyping `tax` file  
 
You can specify the full path (as above for the oligotyping data which was located in a different folder) or you may write the file names if you are in the same working directory.

**Note** that instead of copying this script in every new directory, you can refer to the script by its full path in the `rprops` nfs directory: `/nfs/vdenef-lab/Shared/Ruben/scripts_taxass/make_phyloseq.R`
