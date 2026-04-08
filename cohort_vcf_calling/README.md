# Breadfruit SNP Array (HPC workflow)

This dir contains the workflow for generating per-sample alignments and cohort variant calls (VCF/gVCF) from PacBio HiFi reads.

## Sample-specific processing

The first step will run this per-sample:

1. **Concatenate raw HiFi FASTQs** for the sample into `01_raw_hifi/`
2. **Subset reads** to a fixed size (~20 GB) for more comparable coverage across samples
3. **Align reads** to the reference genome using `minimap2 (map-hifi)` and produce a sorted/indexed BAM
4. **Call variants** with DeepVariant (PACBIO model) to produce per-sample VCF and gVCF  
   - We use the diploid model pragmatically and will treat genotypes as ALT present/absent downstream (ignore dosage)
   
Submit the job with a sample name as the first argument in the proj `wd`:

```bash
sbatch 0A_Subset_Align_CallSNP.sh HART038
```

Workhorse: 

```bash
SAMPLE=${1:?Missing SAMPLE argument}
WD=/project/coffea_pangenome/Breadfruit_SNP_Array
RAW_DATA=/project/coffea_pangenome/Artocarpus/RawData/HiFi
GENOME=${WD}/genome/HART001.fa

t=24

# Subset reads so that we have roughly equal inputs, HART038 only has 10x so it will be low 
echo "Subsetting Reads for: ${SAMPLE}"
cat ${RAW_DATA}/${SAMPLE}__* > ${WD}/01_raw_hifi/${SAMPLE}.HiFi.fastq.gz
bbduk.sh in=${WD}/01_raw_hifi/${SAMPLE}.HiFi.fastq.gz out=${WD}/02_subset_reads/${SAMPLE}.20gb.fastq.gz maxbasesout=20000000000

# Align, with more relaxed parameters since jackfruit is ~20 MY diverged
echo "Aligning Reads for: ${SAMPLE}"
minimap2 -ax map-hifi -t ${t} \
    -R @RG\\tID:${SAMPLE}\\tPL:PACBIO\\tLB:${SAMPLE}\\tSM:${SAMPLE} ${GENOME} \
    ${WD}/02_subset_reads/${SAMPLE}.20gb.fastq.gz 2> ${WD}/03_bams/${SAMPLE}.minimap.log | \
    samtools view -F 4 -bS - | \
    samtools sort -@ ${t} -o ${WD}/03_bams/${SAMPLE}.sorted.bam
samtools index ${WD}/03_bams/${SAMPLE}.sorted.bam

# Call variants, just use diploid deep variant model since we will ignore dosage 
echo "Calling SNPs for: ${SAMPLE}"
DEEPVAR="/project/coffea_pangenome/Breadfruit_SNP_Array/containers/deepvariant_v1.10.0.sif"
apptainer exec \
    -B /project/coffea_pangenome:/project/coffea_pangenome \
    ${DEEPVAR} run_deepvariant \
    --make_examples_extra_args='small_model_call_multiallelics=false' \
    --model_type PACBIO \
    --ref ${GENOME} \
    --reads ${WD}/03_bams/${SAMPLE}.sorted.bam \
    --output_vcf ${WD}/04_vcfs/${SAMPLE}.pt.vcf.gz \
    --output_gvcf ${WD}/04_vcfs/${SAMPLE}.pt.gvcf.gz \
    --sample_name ${SAMPLE} \
    --num_shards ${t} \
    --postprocess_cpus ${t}
```

