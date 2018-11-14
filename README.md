# RNAseq tutorial

Let's work from this study:
https://trace.ncbi.nlm.nih.gov/Traces/study/?acc=SRP117267


## Download data from SRA
Get the accession IDs through the run selector: 
https://www.ncbi.nlm.nih.gov/Traces/study/?WebEnv=NCID_1_43615871_130.14.18.97_5555_1542128126_2560564383_0MetA0_S_HStore&query_key=4
The indivduals runs can be downloaded in a number of ways.
```
wget/FTP root: ftp://ftp-trace.ncbi.nih.gov
ascp root: anonftp@ftp.ncbi.nlm.nih.gov:
```

### By wget or FTP
Downloading run by wget or FTP:
```
wget ftp://ftp-trace.ncbi.nih.gov/sra/sra-instant/reads/ByRun/sra/SRR/SRR602/SRR6026440/SRR6026440.sra
```
### By ascp (Aspera connect) 
Downloading the same file by ascp:
```
[path_to_ascp_binary]/ascp -i [path_to_Aspera_key]/asperaweb_id_dsa.openssh -k 1 –T -l200m anonftp@ftp.ncbi.nlm.nih.gov:/sra/sra-instant/reads/ByRun/sra/SRR/SRR602/SRR6026440/SRR6026440.sra [local_target_directory]
```
### With the SRA toolkit 
```
fastq-dump  -I --split-files SRR6026440
```
or 
```
prefetch SRR_Acc_List.txt
```
### In R 
```
library(SRAdb) 
sqlfile <- file.path(system.file('extdata', package='SRAdb'), 'SRAmetadb_demo.sqlite')
sra_con <- dbConnect(SQLite(),sqlfile)
getSRAfile( "SRR6026440", sra_con, destDir = getwd(), fileType = 'sra', srcType = 'ftp', makeDirectory = FALSE, method = 'curl', ascpCMD = NULL )
```
https://www.bioconductor.org/packages/release/bioc/vignettes/SRAdb/inst/doc/SRAdb.pdf


## Map data
### Using STAR 
Download STAR from here: https://github.com/alexdobin/STAR 
Download genome index files: http://labshare.cshl.edu/shares/gingeraslab/www-data/dobin/STAR/STARgenomes/   
Or generate own by downloading genome (fa) and gene annotation files (gtf): https://www.gencodegenes.org/human/
Follow instructions from manual: https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf 
Or use these scripts:
Mapping:
```
STAR --genomeDir $genome --readFilesIn $read1 $read2 --runThreadN 5 --quantMode GeneCounts TranscriptomeSAM
```
where
```
$genome <- genome directory 
$read1 <- fastq file for read1  
$read2 <- fastq file for read2 (if paired end)
```

## Analyze! Visualize! 
### Using R 
Once mapping is done, we read in the ReadsPerGene.out.tab file. 
```
data <- read.table("ReadsPerGene.out.tab")
```
This has the gene counts for all transcripts in your gtf file. 

