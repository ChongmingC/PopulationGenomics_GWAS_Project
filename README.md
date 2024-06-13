Quality control part -->quality control.ipynb

GWAS analysis and pca -->GWAS_HEIGHT.ipynb

PRS prediction for height -->GWAS_HEIGHT_trained_PRS_prediction.ipynb

Significant variants for different dataset and p-value are attached.

pretrained_PGS000758.txt is the pretrained snp score files from this paper https://academic.oup.com/jcem/article/106/7/1918/6206752.

We generated other files from the origin data on the server using plink. The specific steps are showing below.

A. Do QC
A.1 missing Data
plink --bfile gwas_data --missing --out gwas_data --allow-no-sex
plink --bfile gwas_data --het --out gwas_data --allow-no-sex
plink --bfile gwas_data --remove wrong_het_missing_values.txt --make-bed --out gwas_data_mflt_hflt

A.2 Identification of duplicated or related individuals
plink --bfile gwas_data --indep-pairwise 500kb 5 0.2 --out gwas_data_id
plink --bfile gwas_data --extract gwas_data_id.prune.in --genome --min 0.185 --out gwas_data_id

A.3 filter missing data
plink --bfile gwas_data_mflt_hflt  --indep-pairwise 500kb 5 0.2 --out gwas_data_mflt_hflt --allow-no-sex
plink --bfile gwas_data_mflt_hflt --extract gwas_data_mflt_hflt.prune.in --genome --min 0.185 --out gwas_data_mflt_hflt --allow-no-sex

A.4 relateness
plink --bfile  gwas_data_mflt_hflt --remove wrong_ibd.txt --make-bed --out  gwas_data_mflt_hflt_iflt --allow-no-sex

B Do a pca plot
plink --bfile gwas_data_mflt_hflt_iflt --indep-pairwise 500kb 5 0.2 --out gwas_data_mflt_hflt_iflt --allow-no-sex
plink --bfile gwas_data_mflt_hflt_iflt --extract gwas_data_mflt_hflt_iflt.prune.in --pca 20 --out gwas_data_mflt_hflt_iflt --allow-no-sex


C.1 add phenotype information
plink --bfile gwas_data_mflt_hflt_iflt --pheno height_formatted.txt --allow-no-sex --make-bed --out gwas_data_mflt_hflt_iflt_height

C.2 Test for association with disease status using linear model
plink --bfile gwas_data_mflt_hflt_iflt_height --linear  --out gwas_data_mflt_hflt_iflt_height --allow-no-sex

D.1If you use half of the data set to calculate a polygenic score, how well does that score predict height on the other half?
#Get sample ID
awk '{print $1, $2}' gwas_data_mflt_hflt_iflt_height_removed.fam > all_samples.txt

# test and train model
plink --bfile gwas_data_mflt_hflt_iflt_height_removed --keep train_ids.txt --make-bed --out train_data --allow-no-sex
plink --bfile gwas_data_mflt_hflt_iflt_height_removed --keep test_ids.txt --make-bed --out test_data --allow-no-sex

#GWAS analysis
plink --bfile train_data --linear  --out train_data --allow-no-sex

#PGS
plink --bfile train_data --extract significant_snps.txt --make-bed --out train_data_significant --allow-no-sex
plink --bfile train_data --score snp_weights.txt 1 2 4 sum --out train_data_significant_profile --allow-no-sex
plink --bfile test_data --score snp_weights.txt 1 2 4 sum --out test_data_pgs --allow-no-sex

D.2Find a trained height PRS on the internet. How well does it predict the height in this data set?
plink --bfile gwas_data_mflt_hflt_iflt_height_removed --score pretrained_pgs.txt 1 2 4 sum --out pretrained_pgs_all --allow-no-sex






