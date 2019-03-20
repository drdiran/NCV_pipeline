# NCV_pipeline
###########################################################
## README file for non-coding variant pipeline        #####
## Developer: Oyediran Akinrinade                     #####
###########################################################

The pipeline has five components
- 1. NCV_pipeline_v1.sh - qsub (or NCV_pipeline_v1_Interactive_Bash.sh for interactive runs)
- 2. DeepSEA_pipeline.sh
- 3. RegulomeDB_Query_Pipeline.R (optional )
- 4. motifBreakR_pipeline.R
- 5. Aggregate_Results.R (To be added later)

The pipelines are to be run sequentially:
# Step 1
 - 1. NCV_pipeline_v1.sh: This takes a 'fileName.vcf.gz' as input, decomposes/splits multi-allelics,
 updates dbsnp ID, annotates with vep/92, filters for SNVs overlapping known Ensembl Regulatory Features,
 and generates a bunch of files: vcf, txt and, bed files, needed for the other scripts.

# How to run NCV_pipeline_v1.sh:

 bash path_to_Script/NCV_pipeline_v1.sh - a fileName.vcf.gz

 - The above command submits the job to the cluster and generates the following outputs (required for next steps):
# output files from NCV_pipeline_v1.sh
 - fileName_vep92_gnomADgenome.vcf (needed for DeepSEA_pipeline.sh)
 - fileName_vep92_gnomADgenome_Regulatory_SNVs_RegDB_Input.txt (needed for RegulomeDB_Query_Pipeline.R)
 - fileName_vep92_gnomADgenome_Regulatory_SNVs_MotifbreakR_Input.bed (needed for motifBreakR_pipeline.R)
 - filename_vep92_gnomADgenome_Regulatory_SNVs.txt (needed for the last step - Aggregate_Results.R)
 - fileName_vep92_gnomADgenome_Regulatory_SNVs_Coverage.txt (needed for the last step - Aggregate_Results.R)

 NB: All these are written to the CWD

 # Step 2: DeepSEA prediction
 - 2.DeepSEA_pipeline.sh: This takes as input fileName_vep92_gnomADgenome.vcf from step 1
 # How to use:
bash path_to_Script/DeepSEA_pipeline.sh -a fileName_vep92_gnomADgenome.vcf
This command splits the vcf file by chr and qsub chromosome-level jobs to the cluster.
All analysis for this step are done in ./DeepSEA_Analysis and results are written to ./DeepSEA_Result.
Ten files are written to ./DeepSEA_Result directory and the outputs are needed in the final stage (Aggregate_Results.R).

 # Step 3 (optional) - already added to step 1
 - 3. RegulomeDB_Query_Pipeline.R: Depends on the output of step 1 (filename_vep92_gnomADgenome_Regulatory_SNVs_RegDB_Input.txt).
 NB: I have incorporated this in step 1 but I wouldn't know how hpf will handle this since it
 requires data scrapping from webpage. If it works, step 1 should outputs the following two files
 in addition to the ones listed above:
  - fileName_vep92_gnomADgenome_Regulatory_SNVs_RegDB_RegDB_Scores.RData
  - fileName_vep92_gnomADgenome_Regulatory_SNVs_RegDB_RegDB_Scores.txt

  If these two files are not generated by step 1, please run the command below interactively from the CWD:
  NB: it requires R/3.4.3
#################################################################################################################
module load R/3.4.3
Rscript path_to_Script/RegulomeDB_Query_Pipeline.R -r fileName_vep92_gnomADgenome_Regulatory_SNVs_RegDB_Input.txt

This should generate the two files above
#################################################################################################################
I can look more closely at the behavior of this part later

# Step 4 (optional) - already added to step 1
- 4. motifBreakR_pipeline.R: Should be run if file 'fileName_vep92_gnomADgenome_Regulatory_SNVs_MotifbreakR_MotifBreakR_Scores.RData' is not generated by step 1.
Input: fileName_vep92_gnomADgenome_Regulatory_SNVs_MotifbreakR_Input.bed
This step can be run interactively or submitted to the cluster:

Interactive: Rscript path_to_Script/motifBreakR_pipeline.R -r fileName_vep90_gnomADgenome_Regulatory_SNVs_MotifbreakR_Input.bed
Cluster: run the submit_motifBreakR_job.sh script in the NCV_pipeline but modify the input file in the script before running
NB: I have just added this step to NCV_pipeline_v1.sh and I think it should work. If it works, another RData will be written to the CWD with the suffix - "MotifBreakR_Scores.RData"
If this file exists in the CWD, you don't have to run step 4, as I have incorporated it into step 1 - NCV_pipeline_v1.sh

# Step 5
This step aggregates all the results from various steps and outputs an annotated file (text & RData)
containing all annotations needed for downstream analyses.
The script loads cardiac regulome database and promoter and enhancer regions active in human adult and the
developing heart, and annotates the variants based on these information.
By using allele frequency information, conservation score, activity in heart, and effect prediction by
founr tools, the script classifies each SNV into two classes: tier 1 & tier 2.
