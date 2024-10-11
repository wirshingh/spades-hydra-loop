# Run spades on hydra in a loop
### Summary
This job file will run spades on hydra in a loop using trimmed reads. Results will be outputed to a directory named 'spades_All_results' which will contain sub-directories with results for each sample. 
In addition, this directory will also contain a directory named 'contigs_renamed' which will have copies of the final contigs renamed with sample IDs.

For instructions on how to prepare the job file see "To Run this Job" below.

```
# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 10
#$ -q mThM.q
#$ -l mres=120G,h_data=12G,h_vmem=200G,himem
#$ -l ssd_res=200G -v SSD_SAVE_MAX=0
#$ -cwd
#$ -j y
#$ -N spades
#$ -o spades.log
#
# ----------------Modules------------------------- #
module load tools/ssd
module load bio/spades
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS


# Create a variable by entering full path to trimmed reads directory
SAMPLEDIR_TRM="path to trimmed reads"

#Create a directory by entering full path the base project directory, where job file is located
SAMPLEDIR_BASE="path to base project directory"

# Create a directory that will contain all spades results
mkdir -p ${SAMPLEDIR_BASE}/spades_All_results

# Create a directory for renamed final contigs for each sample
mkdir -p ${SAMPLEDIR_BASE}/spades_All_results/contigs_renamed

# Begin loop that will generate a sample ID based on the file name of trimmed reads
for GETSAMPLENAME in ${SAMPLEDIR_TRM}/*_R1_PE_trimmed.fastq.gz; do
SAMPLENAME=$(basename "${GETSAMPLENAME}" _R1_PE_trimmed.fastq.gz)

# Create a sample-specific directory for sample results
mkdir -p ${SAMPLEDIR_BASE}/spades_All_results/${SAMPLENAME}_spades_results

# Run spades for each sample and output results to sample-specifice folder
spades.py \
-o ${SAMPLEDIR_BASE}/spades_All_results/${SAMPLENAME}_spades_results \
--pe1-1 ${SAMPLEDIR_TRM}/${SAMPLENAME}_R1_PE_trimmed.fastq.gz \
--pe1-2 ${SAMPLEDIR_TRM}/${SAMPLENAME}_R2_PE_trimmed.fastq.gz \
--pe1-s ${SAMPLEDIR_TRM}/${SAMPLENAME}_R0_SE_trimmed.fastq.gz \
-t $NSLOTS --tmp-dir $SSD_DIR

# Copy final contigs for each sample into the directory 'contigs_renamed' and rename it with sample ID
cp ${SAMPLEDIR_BASE}/spades_All_results/${SAMPLENAME}_spades_results/contigs.fasta ${SAMPLEDIR_BASE}/spades_All_results/contigs_renamed/${SAMPLENAME}_spades_contigs.fasta

done


#
echo = `date` job $JOB_NAME

```

### To Run this Job
The trimmed reads (R1, R2, and R0) file names must end in '_R1_PE_trimmed.fastq.gz', '_R2_PE_trimmed.fastq.gz' and '_R0_SE_trimmed.fastq.gz'. Alternatively, the job file can be edited to match the trimmed reads file names of the user accordingly.

If assemblies without the single end reads (the SE reads file) are desired, simply delete the spades command line '--pe1-s ${SAMPLEDIR_TRM}/${SAMPLENAME}_R0_SE_trimmed.fastq.gz' from the script.

Two items need to be copied to the script by the user:

1. SAMPLEDIR_TRM="path to trimmed reads"

   Afteer the '=' paste the full path the trimmed reads directory.

3. SAMPLEDIR_BASE="path to base project directory"

   After the '=' paste the full path to the base project directory, the directory that contains the job file.

After adding the required items save the job file as 'spades_loop.job' and run it on hydra (qsub spades_loop.job).

