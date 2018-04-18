This docker executable runs MiXCR with the following versions:
* mixcr_version=2.1.9
* imgt_version=201802-5.sv2
* fastqc_version=0.11.7
* vdjtools_version=1.1.7

It has the following arguments:
* CHAINS - gets fed to --chains arg for mixcr align. From site:  
>>>
Target immunological chain list separated by “,”. Available values: IGH, IGL, IGK, TRA, TRB, TRG, TRD, IG (for all immunoglobulin chains), TCR (for all T-cell receptor chains), ALL (for all chains) . It is highly recomended to use the default value for this parameter in most cases at the align step. Filltering is also possible at the export step.
>>>
* RNA_SEQ - If set to true, it runs with rna seq parameters as described here: http://mixcr.readthedocs.io/en/master/rnaseq.html
* USE_EXISTING_VDJCA - looks for and uses existing VDJCA file if this is set to true
* SPECIES - hsa or 
* THREADS - gets fed to --species arg for mixcr align. From site:  
> Species (organism). Possible values: hsa (or HomoSapiens), mmu (or MusMusculus), rat (currently only TRB, TRA and TRD are supported), or any species from imported IMGT ® library...
* INPUT_PATH_1 - full path to fastq for read 1
* INPUT_PATH_2 - full path to fastq for read 1
* OUTPUT_DIR - path to folder where output should be stored. this needs to exist prior to running
* SAMPLE_NAME - name of sample.  This should be unique.

Steps to build an image and run it on slurm

login bioinf:
ssh dbortone@login.bioinf.unc.edu

need to list out all of the nodes that have docker partitions to make sure that the dokcer image is pushed to all of them
If you are building an image to a previously existing <sometool>:<version> you need to pull the changes to all of the docker nodes.
removing the image before rebuilding it isn't enough.
srun --pty -c 2 --mem 1g -w c6145-docker-2-0.local -p docker bash
cd /datastore/alldata/shiny-server/rstudio-common/dbortone/docker/mixcr/mixcr_2.1.9
docker build -t dockerreg.bioinf.unc.edu:5000/mixcr_2.1.9:2 .
docker push dockerreg.bioinf.unc.edu:5000/mixcr_2.1.9:2
exit
srun --pty -c 2 --mem 1g -w fc830-docker-2-0.local -p docker bash
docker pull dockerreg.bioinf.unc.edu:5000/mixcr_2.1.9:2
exit
srun --pty -c 2 --mem 1g -w r820-docker-2-0.local -p docker bash
docker pull dockerreg.bioinf.unc.edu:5000/mixcr_2.1.9:2

[dbortone@login2 ~]$ sinfo -p docker
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
docker       up   infinite      1    mix c6145-docker-2-0.local
docker       up   infinite      2   idle fc830-docker-2-0.local,r820-docker-2-0.local


docker variables are:
bash -c 'source /import/run_mixcr.sh \
 --chains "${CHAINS}" \
 --rna_seq "${RNA_SEQ}" \
 --use_existing_vdjca "${USE_EXISTING_VDJCA}" \
 --species "${SPECIES}" \
 --threads "${THREADS}" \
 --r1_path "${INPUT_PATH_1}" \
 --r2_path "${INPUT_PATH_2}" \
 --output_dir "${OUTPUT_DIR}" \
 --sample_name "${SAMPLE_NAME}"'

# run on bioinf
mkdir -p /datastore/nextgenout2/share/labs/imgf/datasets/Sharpless_IP61/mixcr/AE001-1T-SH-TCR_CGTACTAG-TAGATCGC_S3_L001/
sbatch AE001-1T-SH-TCR_CGTACTAG-TAGATCGC_S3_L001.job

#!/bin/bash
#SBATCH --job-name p108_Pre_Tumor
#SBATCH --partition docker
#SBATCH --time UNLIMITED
#SBATCH --mincpus 12
#SBATCH --mem-per-cpu 2g
#SBATCH --output /datastore/nextgenout2/share/labs/imgf/datasets/Merck1520_IP59/mixcr/p108_Pre_Tumor/_output.txt
#SBATCH --error /datastore/nextgenout2/share/labs/imgf/datasets/Merck1520_IP59/mixcr/p108_Pre_Tumor/_error.txt

docker run --rm=true \
  -v /datastore:/datastore:shared \
  -e CHAINS=ALL \
  -e RNA_SEQ=true \
  -e USE_EXISTING_VDJCA=true \
  -e SPECIES=hsa \
  -e THREADS=12 \
  -e INPUT_PATH_1=/datastore/nextgenout4/seqware-analysis/illumina/170724_UNC18-D00493_0434_BCB95DANXX/1520-108-PRETX-RNA06_ACAGTG_S27_L008_R1_001.fastq.gz \
  -e INPUT_PATH_2=/datastore/nextgenout4/seqware-analysis/illumina/170724_UNC18-D00493_0434_BCB95DANXX/1520-108-PRETX-RNA06_ACAGTG_S27_L008_R2_001.fastq.gz \
  -e OUTPUT_DIR=/datastore/nextgenout2/share/labs/imgf/datasets/Merck1520_IP59/mixcr/p108_Pre_Tumor/ \
  -e SAMPLE_NAME=p108_Pre_Tumor \
  mixcr_2.1.9:1

# for interactive session
srun --pty -c 1 --mem 1g -p docker -w c6145-docker-2-0.local docker run -v /datastore:/datastore:shared  -it dockerreg.bioinf.unc.edu:5000/mixcr_2.1.9:2 bash