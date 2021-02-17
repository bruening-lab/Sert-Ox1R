SHELL=/bin/bash
RAW_FILES_FOLDER=/beegfs/v0/bruening_group/CCG/2020-02-xing-rnaseq
SINGULARITY=singularity exec $(SINGULARITY_IMAGES)/mytidyverse-base-3.6.2-1.simg

all: clean \
	results/DESeq2

# CCG Sample ID,Sample Name,Condition
# 117638,10684 BAT,CTRL
# 117640,11253 BAT,CTRL
# 117642,11560 BAT,CTRL
# 117644,11563 BAT,CTRL
# 117646,11708 BAT,Sert-Ox1R
# 117648,11243 BAT,Sert-Ox1R
# 117650,11613 BAT,Sert-Ox1R
# 117652,11601 BAT,Sert-Ox1R
# 117654,10684 SM,CTRL
# 117656,11253 SM,CTRL
# 117658,11560 SM,CTRL
# 117660,11563 SM,CTRL
# 117662,11708 SM,Sert-Ox1R
# 117664,11243 SM,Sert-Ox1R
# 117666,11613 SM,Sert-Ox1R
# 117668,11601 SM,Sert-Ox1R
# 117670,11391 liver,CTRL
# 117672,11274 liver,CTRL
# 117674,11512 liver,CTRL
# 117676,11502 liver,CTRL
# 117678,11390 liver,Sert-Ox2R
# 117680,11296 liver,Sert-Ox2R
# 117682,11515 liver,Sert-Ox2R
# 117684,11510 liver,Sert-Ox2R

raw_data:
	mkdir raw_data
	ln -s ${RAW_FILES_FOLDER}/A006200069_117638_S76_L002_R1_001.fastq.gz raw_data/BAT_10684_CTRL_R1.fastq.gz # 10684 BAT CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117638_S76_L002_R2_001.fastq.gz raw_data/BAT_10684_CTRL_R2.fastq.gz # 10684 BAT CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117640_S77_L002_R1_001.fastq.gz raw_data/BAT_11253_CTRL_R1.fastq.gz # 11253 BAT CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117640_S77_L002_R2_001.fastq.gz raw_data/BAT_11253_CTRL_R2.fastq.gz # 11253 BAT CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117644_S79_L002_R1_001.fastq.gz raw_data/BAT_11563_CTRL_R1.fastq.gz # 11563 BAT CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117644_S79_L002_R2_001.fastq.gz raw_data/BAT_11563_CTRL_R2.fastq.gz # 11563 BAT CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117646_S80_L002_R1_001.fastq.gz raw_data/BAT_11708_Sert-Ox1R_R1.fastq.gz # 11708 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117646_S80_L002_R2_001.fastq.gz raw_data/BAT_11708_Sert-Ox1R_R2.fastq.gz # 11708 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117648_S81_L002_R1_001.fastq.gz raw_data/BAT_11243_Sert-Ox1R_R1.fastq.gz # 11243 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117648_S81_L002_R2_001.fastq.gz raw_data/BAT_11243_Sert-Ox1R_R2.fastq.gz # 11243 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117650_S82_L002_R1_001.fastq.gz raw_data/BAT_11613_Sert-Ox1R_R1.fastq.gz # 11613 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117650_S82_L002_R2_001.fastq.gz raw_data/BAT_11613_Sert-Ox1R_R2.fastq.gz # 11613 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117652_S83_L002_R1_001.fastq.gz raw_data/BAT_11601_Sert-Ox1R_R1.fastq.gz # 11601 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117652_S83_L002_R2_001.fastq.gz raw_data/BAT_11601_Sert-Ox1R_R2.fastq.gz # 11601 BAT Sert-Ox1R
	ln -s ${RAW_FILES_FOLDER}/A006200069_117658_S86_L002_R1_001.fastq.gz raw_data/BAT_11560_CTRL_R1.fastq.gz # 11560 SM CTRL
	ln -s ${RAW_FILES_FOLDER}/A006200069_117658_S86_L002_R2_001.fastq.gz raw_data/BAT_11560_CTRL_R2.fastq.gz # 11560 SM CTRL

nextflow:
	curl -s https://get.nextflow.io | bash

mpisf-rnaseq.config:
	# Copy config file
	cp ${NEXTFLOW_CONFIGS}/mpisf-rnaseq.config mpisf-rnaseq.config

results: nextflow mpisf-rnaseq.config raw_data
	# Run nextflow
	./nextflow run nf-core/rnaseq \
		-r 1.4 \
		--genome GRCm38 \
		--pseudo_aligner salmon \
		-c ${shell readlink -f mpisf-rnaseq.config} \
		--reads '${shell readlink -f raw_data}/*{_R1,_R2}.fastq.gz'

results/DESeq2: results
	$(SINGULARITY) Rscript -e 'nfRNAseqDESeq2::run_differential_expression(path_config_json = "${shell readlink -f .}/groups.json", path_salmon_counts = "${shell readlink -f .}/results/salmon/salmon_merged_gene_counts.csv", out_path = "${shell readlink -f .}/results/DESeq2", biomart_attributes = c("external_gene_name", "gene_biotype"))'

results/goterm-analyses: results/DESeq2
	# Create GO-term analysis for all comparisons
	$(SINGULARITY) Rscript -e 'nfRNAseqDESeq2::goterm_analysis_of_all_comparisons(deseq2_diff_path = "${shell readlink -f .}/results/DESeq2/deseq2_diff.csv", out_path = "${shell readlink -f .}/results/goterm-analyses", simplify_ontologies = TRUE, do_gse = TRUE)'

.PHONY: clean
clean:
	-rm -r raw_data
	-rm mpisf-rnaseq.config
	-rm nextflow
	-rm -r .nextflow
	-rm .nextflow.log*
	-rm -r results

.PHONY: clean_work_files
clean_work_files:
	rm -r work
