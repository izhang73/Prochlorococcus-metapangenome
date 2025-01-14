# Prochlorococcus-metapangenome
Annika Gomez and Irene Zhang

Delmont, T. O., & Eren, A. M. (2018). Linking pangenomes and metagenomes: the Prochlorococcus metapangenome. PeerJ, 6, e4320. doi:10.7717/peerj.4320 

We propose to use the 31 isolate genomes for Prochlorococcus that Delmont and Eren downloaded from the NCBI database (totaling 55 MB, available at https://figshare.com/articles/Prochlorococus_FASTA_files_for_isolates_and_SAGs/5447221 along with 25 metagenomes from the Atlantic Ocean from the TARA Oceans project as opposed to the 93 used in the paper (about 1 GB). We will filter these reads for quality using the parameters delineated by Delmont and Eren and annotate functional genes within these genomes. We will distinguish between environmental core genes and environmental accessory genes for each genome using anvi’o based upon a coverage value threshold (25%) for each gene across all genomes. From this we can compute the Prochlorococcus pangenome in anvi’o. We will visualize the results of our analysis and recreate Figures 1-4 using anvi’o (http://merenlab.org/software/anvio/) and the ggplot2 library for R if necessary. 

# Repository Structure and Files

data/ : contains raw data for metagenomes and isolates, quality-filtered fastqs, text files required for associating reads to metagenomes, config files for quality filtering, and any files required to clean and assemble data for anvi'o
    
    Atlantic_Sample_IDs.txt : list of all TARA fastq files associated with Atlantic Ocean metagenomes 
    Atlantic_samples.txt : associates fasta filenames with metagenome reads 1 and 2 for Atlantic Ocean TARA metagenomes
    ftp-links-for-raw-data-files.txt: ftp links for Atlantic Ocean TARA metagenomes, we tried to use this to download TARA data prior to receiving them from Maria 
    PROCHLOROCOCCUS-FASTA-FILES.tar.gz : zipped Prochlorococcus fasta files downloaded from Meren's blog, same ones as used in their analysis
    internal-genomes.txt : internal genomes file received from Meren's blog 
    samples.txt : original list associating fasta filenames with metagenome reads for TARA, which we filtered to get Atlantic_samples.txt
    sets.txt: list of locations only (ANW, ANE, etc.)
    config-files/ : directory for all config files generated during quality filtering metagenomes
        all-config-files/ : all config .ini files for TARA
        atlantic-config-files/ : only Atlantic Ocean config files      
    PROCHLOROCOCCUS-FASTA-FILES/ : directory containing unzipped Prochlorococcus fasta files and SAGs
        Maria_SAGs : directory containing SAGs provided by Maria
        quality-filtered-fastqs/ : quality-filtered Atlantic Ocean TARA fastq files associated with metagenomes
            *QUALITY-PASSED* files are outputs from quality filtering step, denoised and cleaned fastqs
            *STATS.txt files summarize reads passing quality filtering and other information for each metagenome 

databases/  : contains database and collections files output by anvi'o workflow, text files needed for anvi'o scripts, and anything generated by anvi'o that isn't a visualization
    
        EQPAC1-ENV-DETECTION.txt : Additional data file for anvi-interactive to show which EQPAC1 genes occurred systemmatically across metagenomes given the ‘fraction of median coverage’ criterion
        
        EQPAC1-GENE-COVs.txt : Tab-delimited matrix with coverage values for genes from EQPAC1 across each metagenome for a given bin
        
        MIT9314-ENV-DETECTION.txt : Additional data file for anvi-interactive to show which MIT9314 genes occurred systemmatically across metagenomes given the ‘fraction of median coverage’ criterion
        
        MIT9314-GENE-COVs.txt : Tab-delimited matrix with coverage values for genes from MIT9314 across each metagenome for a given bin
        
        A* folders: Each contains PROFILE.db, which is the anvi'o profile database for that individual metagenomic sample. 
        
        Prochlorococcus-MERGED: Contains PROFILE.db, which is the anvi'o profile database of all the metagenomic samples merged into one. 
        
        Prochlorococcus-CONTIGS.db: Anvi'o contigs database generated from the isolate genomes and SAGs
        
        Prochlorococcus-ISOLATE-PAN: contains the result of the anvi'o pangenome analysis on ONLY the isolate genomes. 
        
        Prochloroccocus-PAN: contains the resul of the anvi'o pangenome analysis on all the genomes, isolates AND SAGs. 
        
        prochlorococcus-bowtie.*: results of the bowtie-build command, which built databases used in mapping the metagenomic reads to the genomes. 

envs/ : contains the .yaml conda environment file necessary to install packages and anything needed to run these analyses

jupyter-notebooks/ : contains final jupyter notebook containing our images and comparisons and the jupyter notebook used in creating the collections and internal-genomes files. 

logs/  : all .log files generated from slurm scripts we used. 
        
        bowtie_*.log : logs from bowtie-map.sh
        quality-filter.log : log from quality-filtering.sh
        anvi-meta-pan-genome.log : log from anvi-meta-pan-genome.sh
        *pangenome*.log : logs from pangenome.sh
        anvi-db*.log : logs from anvi-db-cogs-hmms.sh

output/ : summary files from anvi'o and images from our analysis
    
        genes-in-two-genomes.png: Meren's EQPAC1 and MIT314 image (Fig 3)
        Prochlorococcus_Metapangenome images: Meren's metapangenome visualization (Fig 2)
        Prochlorococcus-SUMMARY/ : summary files from visualization of contigs db
        Prochlorococcus-Isolate-PAN-SUMMARY/ : summary files form pangenome visualization
        Prochlorococcus_EQPAC1_MIT9314.png : our Fig 3
        Genes_in__MIT9314___mode__Standard_.png : our MIT9314 visualization only
        Genes_in__EQPAC1___mode__Standard_ : our EQPAC1 visualization only
        Prochloroccocus_ISOLATE_PAN : our pangenome visualization (Fig 2)

README.md  : this README describing our workflow

scripts/ : any scripts, bash or otherwise, we submitted to the cluster

    quality-filtering.sh : script to quality filter TARA Atlantic Ocean metagenomes to generate *QUALITY-PASSED*.fastq and *STATs files and associate fastqs to metagenomes
    anvi-meta-pan-genome.sh : characterizes core and accessory genes and add information to Prochloroccocus-ISOLATE-PAN-PAN.db 
    anvi-db-cogs-hmms.sh : script to run Blast and HMMs on contigs
    anvi-profile.sh	: makes anvi'o profiles for each metagenome sample
    bowtie-map.sh : runs Bowtie
    collection-file.sh : makes the collection file 
    pangenome.sh : computes the pangenome
    

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

We also included in the analysis 5 new SAGs (courtesy of Maria), which were downloaded into the folder *Maria_SAGs*. The SAG IDs are:

    SAG_id    Accession
    AG-893-K05    ERS3871084
    AH-321-C14    ERS3869306
    AG-349-G23    ERS3869306
    AG-422-K10    ERS3877429
    AG-404-D14    ERS3876520
    
We located the ftp directories for these SAGS at NCBI using the BioProject accession PRJEB33281. After downloading correct assemblies, these files were transferred onto Poseidon.
To unzip each of these files: 

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
    
The contents of a config file are as follows:

    project_name = ANE_004_05M
    researcher_email = u@example.edu
    input_directory = /vortexfs1/home/izhang/TARA
    output_directory = /vortexfs1/omics/env-bio/collaboration/Prochlorococcus-metapangenome/data

    [files]
    pair_1 = ERR599003_1.fastq.gz,ERR598955_1.fastq.gz
    pair_2 = ERR599003_2.fastq.gz,ERR598955_2.fastq.gz

For the actual quality filtering, we made the slurm script quality-filtering.sh and submitted:

    sbatch scripts/quality-filtering.sh

Since quality filtering requires significant time to run and the maximum time we could request on Poseidon was 20 hours, we had to rerun this script several times to perform QC on each metagenome. In addition, several metagenomes had multiple fasta files associated with each read. As each fasta file in the TARA directory was located in its own subdirectory, and the .ini config files cannot read from multiple directories, we had to copy these fasta files into our home directory for this step to work. 

The contents of quality-filtering.sh are as follows:
    
    for sample in `awk '{print $1}' Atlantic_samples.txt`
    do
        if [ "$sample" == "sample" ]; then continue; fi
        iu-filter-quality-minoche $sample.ini --ignore-deflines
    done

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
    
We summarized the results of the pangenome analysis with the following command:

    anvi-summarize -p databases/Prochlorococcus-ISOLATE-PAN/Prochlorococcus-ISOLATE-PAN-PAN.db -g databases/Prochlorococcus-ISOLATE-PAN-GENOMES.db -C default -o output/Prochlorococcus-ISOLATE-PAN-SUMMARY

# Linking pangenome to environment:

To characterize the ratio of environmentally accessory genes (EAGs) to environmentally core genes (ECGs) in each gene cluster in the pangenome, we classified genes with less than 25% of the median coverage of all genes found in the genome as EAGs. This is the same threshold used by Delmont and Eren. We used this command which we ran through the slurm script anvi-meta-pan-genome.sh:

    anvi-meta-pan-genome -p Prochlorococcus-ISOLATE-PAN/Prochloroccocus-ISOLATE-PAN-PAN.db -g Prochlorococcus-ISOLATE-PAN-  GENOMES.db -i ../data/internal-genomes.txt --fraction-of-median-coverage 0.25 
    
    sbatch scripts/anvi-meta-pangenome.sh

To visualize the distribution of genes in a single genome across a set of metagenomes, we chose to look at the same genomes as Delmont and Eren: MIT9314 and EQPAC1 in our Atlantic Ocean metagenomes. We ran the following commands using the same 25% threshold as above:

    anvi-script-gen-distribution-of-genes-in-a-bin -c Prochlorococcus-CONTIGS.db -p Prochlorococcus-MERGED/PROFILE.db -b MIT9314 -C Genomes --fraction-of-median-coverage 0.25 

    anvi-script-gen-distribution-of-genes-in-a-bin -c Prochlorococcus-CONTIGS.db -p Prochlorococcus-MERGED/PROFILE.db -b EQPAC1 -C Genomes --fraction-of-median-coverage 0.25 

# Visualizations:
    
We visualized Figure 2 using the following command:

    anvi-display-pan -p Prochlorococcus-ISOLATE-PAN/Prochloroccocus-ISOLATE-PAN-PAN.db -g Prochlorococcus-ISOLATE-PAN-GENOMES.db --title "Prochlorococcus Metapangenome” --server-only
    
We attempted to attach the state file GENES-PROFILE.json file from Meren's blog for visualization of Figure 3 using the command:
    
    anvi-import-state -s GENES-PROFILE.json -p Prochloroccocus-PAN/Prochloroccocus-PAN-PAN.db -n default
    
While this command works, when we attempt to visualize this PAN.db using anvi-interactive or anvi-display-pan, we run into an issue where the state file was created on the interactive server version 1, while we are using version 3. When the server tries to convert this to version 3, it is stuck on a loading screen and does not progress for hours. As a result, we remade the PAN.db file and PROFILE.db file as above.

To visualize Figure 3 for EQPAC1 and MIT9314:
    
    anvi-interactive -p databases/Prochlorococcus-MERGED/PROFILE.db -c databases/Prochlorococcus-CONTIGS.db -C Genomes --gene-mode -b EQPAC1 -d databases/EQPAC1-GENE-COVs.txt -A databases/EQPAC1-ENV-DETECTION.txt --title "Prochlorococcus EQPAC1 genes across TARA Oceans Project metagenomes" --server-only

    anvi-interactive -p databases/Prochlorococcus-MERGED/PROFILE.db -c databases/Prochlorococcus-CONTIGS.db -C Genomes --gene-mode -b MIT9314 -d databases/MIT9314-GENE-COVs.txt -A databases/MIT9314-ENV-DETECTION.txt --title "Prochlorococcus MIT9314 genes across TARA Oceans Project metagenomes" --server-only

Figure 4 B and C: visualizing read recruitment to isolate genomes and SAGs
To visualize metagenomic read recruitment to several of the genomes in our collection, we used the Anvi'o interactive interface in gene mode. To more closely replicate the figure presented in the paper, we adjusted the figure through the interactive interface by viewing the plot as a 360 degree circle. For example, to visualize isolate NATL2A, we ran the command:

    anvi-interactive -p databases/Prochlorococcus-MERGED/PROFILE.db -c databases/Prochlorococcus-CONTIGS.db -C Genomes --gene-mode -b NATL2A --server-only

As we are running anvi-interactive on Poseidon, to visualize the server run this command on a local machine. The localhost port number is provided by anvi'o when run with the --server-only flag, and we used the Poseidon node which we are logged onto:

     ssh -N -f -L localhost:8082:localhost:8082 username@poseidon-l1.whoi.edu

Then, open a browser window and navigate to the localhost (http://localhost:8082/ in this case).

To exit out of the server, we kill it using on the local machine:
    
    lsof -ti:8082 | xargs kill -9ed
