# Prochlorococcus-metapangenome
Annika Gomez and Irene Zhang

Delmont, T. O., & Eren, A. M. (2018). Linking pangenomes and metagenomes: the 
Prochlorococcus metapangenome. PeerJ, 6, e4320. doi:10.7717/peerj.4320 

We propose to use the 31 isolate genomes for Prochlorococcus that Delmont and Eren \
downloaded from the NCBI database (totaling 55 MB, available at  \
https://figshare.com/articles/Prochlorococus_FASTA_files_for_isolates_and_SAGs/5447221 \
along with 25 metagenomes from the Atlantic Ocean from the TARA Oceans project as opposed \
to the 93 used in the paper (about 1 GB). We will filter these reads for quality using the \
parameters delineated by Delmont and Eren and annotate functional genes within these \
genomes. We will distinguish between environmental core genes and environmental accessory \
genes for each genome using anvi’o based upon a coverage value threshold (25%) for each \
gene across all genomes. From this we can compute the Prochlorococcus pangenome in anvi’o. \
We will visualize the results of our analysis and recreate Figures 1-4 using anvi’o \
(http://merenlab.org/software/anvio/) and the ggplot2 library for R if necessary. 

# Anvi'o installation

We installed anvi'o 6.1 using conda according to the instructions found here: http://merenlab.org/2016/06/26/installation-v2/

    conda create -n anvio-6.1 python=3.6
    conda activate anvio-6.1
    conda install -y -c conda-forge -c bioconda anvio=6.1
    conda install -y diamond=0.9.14

We checked the installation using:

    anvi-self-test --suite mini

And saved the environment using:

    conda env export > anvio-6.1.yml

After attempting to run a later step, we realized that another necessary part of the installation was setting up the NCBI COG database: 
    
    anvi-setup-ncbi-cogs


# Downloading data

Download Prochlorococcus isolate genome & SAG fasta files: 

    curl -L https://ndownloader.figshare.com/files/9416614 -o PROCHLOROCOCCUS-FASTA-FILES.tar.gz
    tar -xzvf PROCHLOROCOCCUS-FASTA-FILES.tar.gz
   
We used the TARA metagenomes available on Poseidon (provided by Maria) to conserve space in our directory.
TARA metagenomes were available at : /vortexfs1/omics/data/tara/PRJEB1787

The file PROCHLOROCOCCUS-FASTA-FILES contains: *CONTIGS-FOR-ISOLATES.fa* and *CONTIGS-FOR-SAGs.fa*
To combine these into a single fasta file, run the following command:

    cat CONTIGS-FOR-ISOLATES.fa CONTIGS-FOR-SAGs.fa > Prochlorococcus-genomes.fa

We also included in the analysis 5 new SAGs (courtesy of Maria), which were downloaded into the folder *Maria_SAGs*. To unzip each of these files: 

    for fasta in Maria_SAGs/
    do
    gunzip $fasta
    done 

We then combined all the genome sequences into a single file...

    cat Maria_SAGs/*.fna Prochlorococcus-genomes.fa >> all-genome-seqs.fa

...And edited the deflines of the new SAGs to be consistent with the ones used in the paper:

    awk '{ if (substr($1,1,3) == ">CA") print ">" substr($10,1,10) substr($10,16,length($10)-1); else print $0}' all-genome-seqs.fa > all-genome-seqs-fixed.fa

For example, a defline in the *PROCHLOROCOCCUS-FASTA-FILES/CONTIGS-FOR-ISOLATES.fa* file is:

">AS9601-00000001"; where "AS9601" represents the name of the isolate, and "00000001" represents the contig within that genome. The file *CONTIGS-FOR-SAGs.fa* follows the same format. 
  
However, the deflines in the new SAGs we downloaded are formated as follows:
 
">CACAYO010000001.1 uncultured Prochlorococcus sp. isolate AG-349-G23 genome assembly, contig: AG-349-G23_NODE_1, whole genome shotgun sequence"
The awk command above extracts the genome (AG-349-G23) and contig number (1) from the defline and formats it in a way that is consistent with the deflines given above. 

# Metagenome Quality Filtering

Download file listing 93 Prochlorococcus metagenomes:

    wget http://merenlab.org/data/tara-oceans-mags/files/sets.txt
    wget http://merenlab.org/data/tara-oceans-mags/files/samples.txt

We filtered for Atlantic metagenomes only from the samples.txt file, which associates reads with sample IDs, and saved this
files as Atlantic_samples.txt.

To perform quality filtering on the metagenomes (removing noise), we used the illumina-utils library. The iu-gen-configs command generates config files (.ini file extension) which are used in the downstream steps to associate raw reads with the correct sample IDs and locations. 
    
    iu-gen-configs Atlantic_samples.txt

For the actual quality filtering, we made the slurm script quality-filtering.txt. Since quality filtering requires significant time to run and the maximum time we could request on Poseidon was 20 hours, we had to rerun this script several times to perform QC on each metagenome. In addition, several metagenomes had multiple fasta files associated with each read. As each fasta file in the TARA directory was located in its own subdirectory, and the .ini config files cannot read from multiple directories, we had to copy these fasta files into our home directory for this step to work. 

The .STATS file for quality-filtering.txt contains the following information for each metagenome:

    cat ANW_146_05M-STATS.txt 
    number of pairs analyzed : 169471567
    total pairs passed       : 159790673 (%94.29 of all pairs)


# Creating Anvi'o contigs database:
- An anvio contigs database stores the sequence information from each of the inputted contigs, predicts ORFs, and adds information generated from searching these contigs against a reference databases, using HMMs and BLAST, to uncover function of genes populating the database
    
Before generating the database, we using the following command to edit the deflines in the fasta files to anvi'o's specifications:

    anvi-script-reformat-fasta data/PROCHLOROCOCCUS-FASTA-FILES/all-genome-seqs-fixed.fa -o seqs-fixed.fa -l 0 --simplify-names --report-file report-deflines.tab
 
 We then used the *anvi-db-cogs-hmms.sh* script to generate a contigs database and populate it with information from an HMM and BLAST search.
 
    sbatch scripts/anvi-db-cogs-hmms.sh 

# Mapping metagenome reads:
We began by building a bowtie database from the fasta file containing each isolate and SAG genome:

    bowtie2-build data/PROCHLOROCOCCUS-FASTA-FILES/all-genome-seqs-fixed.fa databases/prochlorococcus-bowtie
    
Then used bowtie to map the metagenomic reads to the *Prochlorococcus* genomes using the script bowtie-map.sh. This script also converts the SAM output to a BAM file, then sort and index it. 

    sbatch scripts/bowtie-map.sh
    
 The bowtie outputs were then used to create Anvi'o profiles for each of the metagenomic samples using the anvi-profile.sh script.
    
    sbatch scripts/anvi-profile.sh
 
 The profiles were then merged into a single profile using the following command:
 
    anvi-merge databases/A*/PROFILE.db -o databases/Prochlorococcus-merged -c databases/Prochlorococcus-CONTIGS.db

# Collection file
The purpose of a collection file in Anvi'o is to tell the program which contigs correspond to which genome. Prior to this step, the contigs database was treated as one large group of contigs, as opposed to many individual genomes. We used the Jupyter notebook Make-Collections-File.ipynb to create a file containing each of the splits stored in the Anvi'o database and the corresponding genome from which that split originated. We could not use the provided file due to the addition of new genomes. The collection file can be found in data/PROCHLOROCOCCUS-FASTA-FILES/final_Prochlorococcus-GENOME-COLLECTION.txt. 

We used the following command to read the information in the collection file into the contigs database:

    anvi-import-collection final_Prochlorococcus-GENOME-COLLECTION.txt -c databases/Prochlorococcus-CONTIGS.db -p databases/Prochlorococcus-MERGED/PROFILE.db -C Genomes

# Pangenome analysis:

To begin the pangenome analysis, we needed to create an internal-genomes.txt file. The process for creating this file can be found in the jupyter notebook Make-Collections-File.ipynb. We repeated this process twice, once including all the isolate and SAG genomes, and once only including the isolate genomes. The figures presented in this analysis represent the repeated pangenome analysis with only the isolate genomes, as this is what was done in the paper. The script pangenome.sh was used to run the pangenome analysis.

    sbatch scripts/pangenome.sh
 
 We visualized the pangenome using the following command:
 
    anvi-display-pan -p databases/Prochlorococcus-ISOLATE-PAN/Prochlorococcus-ISOLATE-PAN-PAN.db -g databases/Prochlorococcus-ISOLATE-PAN-GENOMES.db --server-only
 
The server-only flag instructs anvi'o to load the information into the server without opening a browser. We were then able to launch the interactive mode by shh-ing into the correct local host, using the same procedure as done in class to launch jupyter notebooks from the cluster. 

# Generating summary files:

We summarized the results of the profiling of the contigs database (mapping metagenomic reads to isolate genomes and SAGs) using the following command:

    anvi-summarize -c databases/Prochlorococcus-CONTIGS.db -p databases/Prochlorococcus-MERGED/PROFILE.db -C Genomes --init-gene-coverages -o output/Prochlorococcus-SUMMARY

# Linking pangenome to environment:

To 

anvi-meta-pan-genome -p Prochlorococcus-ISOLATE-PAN/Prochloroccocus-ISOLATE-PAN-PAN.db -g Prochlorococcus-ISOLATE-PAN-GENOMES.db -i ../data/internal-genomes.txt --fraction-of-median-coverage 0.25 


# Contribution Statement ALG

ALG generated the anvi’o contigs database and assigned functions to genes using HMMer and the NCBI COGs database, recruited metagenome reads to contigs, and profiled read recruitment to generate a merged profile database and anvi’o collection. ALG computed and visualized the Prochlorococcus pangenome.


# Contribution Statement IZ

IZ downloaded required FASTA files for isolates and single-amplified genomes (SAGs) and SAGs provided by Dr. Maria Pachiadaki. IZ performed quality filtering on Atlantic Ocean TARA metagenomes and associated metagenomes with sample metadata. IZ classified genes as environmental core (ECG) or environmental accessory (EAG) and visualized this distribution across metagenomes. 
