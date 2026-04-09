# Generation of Error model for point mutations
This study uses a site and mutation specific error model as described in Walkes et. al. Haematologica. 2017;102(9):1549-1557  
[PMID: 28572161](https://doi.org/10.3324/haematol.2017.169136 "doi link")  
Scripts used for model generation can be found in the ``ErrorModel`` folder 

## Steps to generate the error model
1. Variant calling for Biological Negative Controls (BNCs).  
Generation of error model requires variant calling data of BNCs. 30 BNCs samples were sequenced using the smMIP panel and analysed using the pipeline till the variant calling step (COMBINE_VCF process in the sal_mips.nf file)  
The following steps 2-4 are to be carried out for individual samples.

2. Exclusion of variants with VAF > 0.2  
Sites with VAF > 0.2 were excluded following the methods mentioned in the [article](https://doi.org/10.3324/haematol.2017.169136 "doi link"). This was acheived using the remove_variants_gtr_20.pl script. Command used was 
	```
	perl remove_variants_gtr_20.pl BNC.vcf.gz > BNC.real_removed
	```  
3. Separating multiple variants at the same position  
Positions with more than one ALT variant were split into separate lines using the following command.  
	```
	perl print_multiple_variants_at_same_location.pl BNC.real_removed > BNC_combined.real_removed
	```

4. Adding alt count values for positions without variants  
This step will generate a file of altcount of each base (A/T/G/C) and tagcount (depth for each probe) for all positions covered by smMIPS probes.  
This step requires 2 input files
	- A text file containing a **non overlapping list of probes**
	- Output of Step 3  

	The following command produces an output file with an extension of .filled  
	```
	perl fill_empty_mips.pl mips_mrd_bal210125_nooverlap.txt BNC_combined.real_removed
	```  

5. Generation of beta matrix  
	Files with *.filled extension for 30 BNC samples were used in this step to generate the beta matrix
	```
	./beta_distribution.py --samples *.filled --output beta_matrix.txt
	```
	This script assumes a fixed error rate of 1/15,000 for sites with no variants detected in one or more BNCs as mentioned in the supplementary methods of the [article](https://doi.org/10.3324/haematol.2017.169136 "doi link").  
	This can be modified by altering the value of `default_error_rate` variable in this script