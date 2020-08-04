# Sequence-based-filtering-method-for-16S-rRNA-sequencing

Here we proposed a simple-to-use sequence-based filtering method for the removal of contaminants in low biomass 16S rRNA sequencing approaches.

Pipeline contributors: Cristina Zubiria Barrera, Dr. Magdalena Stock and Dr. Tilman Klassert.



# 1. Filter the sequences of your blank control(s) from your demultiplexed fasta file.

	filter_fasta.py 
		-f $PWD/seqs.fna 
		-p Blank 
		-o $PWD/Blank_seqs.fna
		
		
# 2. Cluster the sequences from the Blank control(s) at 99% 

	pick_otus.py 
		-i $PWD/Blank_seqs.fna 
		-m uclust 
		-o $PWD/otus_Blank.only_uclust_99 
		-s 0.99 
		-g 2 
		--threads 8
	
	
# 3. Use the most representative sequences from the largest clusters (i.e. the “X” top clusters witch contain >1% of the total reads).  

	awk '$0=$1": "NF' $PWD/otus_Blank.only_uclust_99/Blank_seqs_otus.txt |sort -k 2 -n |tail -n X >$PWD/Blank_Xlargestclusters.txt

	pick_rep_set.py 
		-i $PWD/otus_Blank.only_uclust_99/Blank_seqs_otus.txt 	
		-f $PWD/Blank_seqs.fna 
		-m most_abundant 
		-o $PWD/rep_set_Blank_only.fna
	
	awk '{print $1}' $PWD/Blank_Xlargestclusters.txt |tr -s ':' ' ' |tr -d "[:blank:]" >$PWD/Blank_Xlargestclusters.txt_IDs.txt

	filter_fasta.py 
		-f $PWD/rep_set_Blank_only.fna 
		-o $PWD/rep_set_Blank_Xlargestclusters.fna 
		-s $PWD/Blank_Xlargestclusters.txt_IDs.txt

--> At this point you have picked the most representative sequences assigned to the X largest clusters of your blank control(s).
	

# 4.  Obtain all reads from these representative sequences in these X clusters.

	grep -Ff $PWD/Blank_Xlargestclusters.txt_IDs.txt $PWD/otus_Blank.only_uclust_99/Blank_seqs_otus.txt | awk '$1=" "' |awk 'BEGIN {OFS="\n"}{$1=$1;print}' 	>$PWD/Blank_Xlargestclusters_readsIDs.txt

	filter_fasta.py 
		-f $PWD/Blank_seqs.fna 
		-o $PWD/Blank_seqs_XlargestclustersAll.fna 
		-s $PWD/Blank_Xlargestclusters_readsIDs.txt

	grep '^[^>]' $PWD/Blank_seqs_XlargestclustersAll.fna |sort|uniq >$PWD/Blank_seqs_XlargestclustersAll_uniq.fna


# 5. Filter now your samples from your demultiplexed fasta file.

	filter_fasta.py 
		-f $PWD/seqs.fna 
		-p Samples 
		-o $PWD/Samples_seqs.fna


# 6. Filter your Blank "selected sequences" from your samples 

	grep -vFf $PWD/Blank_seqs_XlargestclustersAll_uniq.fna $PWD/Samples_seqs.fna >$PWD/Samples.Blanklargestclusters_fil.fna

	grep '^[^>]' -B 1 $PWD/Samples.Blanklargestclusters_fil.fna |sed '/--/d' >$PWD/Samples.BlanklargestCl_filt_seqsFinal.fna
	

--> With this last fasta file you can now keep on performig your taxonomic analyses with your standard pipeline.

--> Replace $PWD with your path
