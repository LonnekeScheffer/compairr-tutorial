# CompAIRR Tutorial

The original CompAIRR repository can be found [here](https://github.com/uio-bmi/compairr). 

Test your CompAIRR installation by running the following command:

`compairr --version`

Your version of CompAIRR should be 1.11.0 or higher. With earlier versions, some of the commands in the tutorials below may not work.


## Tutorial part 1: repertoire vs repertoire comparison

The input file [emerson.tsv](example_data%2Frepertoire_vs_repertoire%2Femerson.tsv) 
contains a subset of [the data published by Emerson et al](https://doi.org/10.1038/ng.3822). 
The file contains 5 repertoires separated by the following identifiers: HIP02811, HIP08337, HIP13168, HIP13929, HIP14240.

We can compute an overlap matrix between these 5 repertoires by calling: 

`compairr --matrix emerson.tsv --out out.txt`

The repertoire overlap matrix can be found in out.txt. 
By default, the values in the output matrix represent the sum of product of duplicate counts of matching sequences.
We can change this by choosing a different `--score` parameter value. 
For example, we can instead compute the sum of minimum duplicate counts (take the minimum count between
two matching sequences) by setting `--score min`: 

`compairr --matrix emerson.tsv --out out.txt --score min`

Alternatively, if we are just interested in knowing the number of matching clonotypes while ignoring
their counts, this can be done by specifying `--ignore-counts`. When this is specified, it does not
matter if `--score` is set to `min`, `max`, `mean`, `product` or `ratio`; it will always yield the same result.

`compairr --matrix emerson.tsv --out out.txt --ignore-counts`

If you are interested in knowing *which* sequences are overlapping, add the `--pairs` argument:

`compairr --matrix --out out.txt --pairs pairs.txt emerson.tsv`

If you don't like the matrix format, you can specify `--alternative` to get the output in a long format:

`compairr --matrix --out out.txt --alternative emerson.tsv`

Instead of computing the raw number of overlapping sequences, we can also compute a similarity score: Morisita-Horn (`--score MH`) or Jaccard (`--score Jaccard`):

`compairr --matrix --out out_mh.txt --score MH emerson.tsv`

However, the Morisita-Horn and Jaccard scores should not be able to exceed 1, and 
the repertoires compared to themselves have a score greater than 1. 
This is because the repertoires contain duplicates, which may for example happen if 
different nucleotide sequences resolve to the same amino acid sequence. 

CompAIRR has a function to remove duplicates. The duplicate_count values for duplicates are summed.
This can be done by running the `--deduplicate` functionality:

`compairr --deduplicate --out emerson_deduplicated.tsv emerson.tsv`

Now we can try to compute the Morisita-Horn distance matrix again, but with the deduplicated input file, and the MH score no longer exceed 1:

`compairr --matrix --out out_mh_dedup.txt --score MH  emerson_deduplicated.tsv`

So far, a match has been counted if both the CDR3 sequence and V and J genes are matching. 
But in some cases we are not interested in the genes. In that case, the `--ignore-genes` argument can be added: 

`compairr --matrix --out out_mh_dedup.txt --score MH --ignore-genes emerson_deduplicated.tsv`

However, this will result in the same issue we saw before: some of the Morisita-Horn scores exceed 1. 
This is because the V and J genes were not removed during deduplication.
When running an analysis with `--ignore-genes`, this flag should also be specified during deduplication:

`compairr --deduplicate --ignore-genes --out emerson_deduplicated_ignore_genes.tsv emerson.tsv`

The resulting deduplicated file does not contain columns for V and J genes. 
Computing the Morisita-Horn distance matrix again will now not contain values greater than 1:

`compairr --matrix --out out_mh_dedup.txt --score MH --ignore-genes emerson_deduplicated_ignore_genes.tsv`



## Tutorial part 2: sequence vs repertoire comparison

The input file [IEDB_data.tsv](example_data%2Frepertoire_vs_sequences%2FIEDB_data.tsv)
contains a version of the IEDB TCR beta sequences ([original source](https://github.com/IEDB/TCRMatch/tree/master/data)).
The original column name `trimmed_seq` has been renamed to `junction_aa` for compatibility with CompAIRR. 

Additionally, a small set of CDR3 sequences can be found in the file [cdr3s.tsv](example_data%2Frepertoire_vs_sequences%2Fcdr3s.tsv).

To see which of the cdr3s exist in the IEDB file, we can run compairr in `--existence` mode: 

`compairr --existence cdr3s.tsv IEDB_data.tsv --out out.txt`

The command above will fail with an error: Missing essential column(s) in header of AIRR TSV input file: duplicate_count v_call j_call
We do not have duplicate_count and V and J information available, so we must add the correct 
arguments to ignore this information: 

`compairr --existence cdr3s.tsv IEDB_data.tsv --out out.txt --ignore-genes --ignore-counts`

The `--pairs` argument can again be added to observe which sequences were matching: 

`compairr cdr3s.tsv IEDB_data_junction.tsv --existence --out out.txt --ignore-genes --ignore-counts --pairs pairs.out`

Most sequences do not have a match, but we can increase the number of matches by allowing a small number of
differences (differing amino acids) between the cdr3s and the IEDB data:

`compairr cdr3s.tsv IEDB_data_junction.tsv --existence --out out.txt --ignore-genes --ignore-counts --pairs pairs.out --differences 1`
`compairr cdr3s.tsv IEDB_data_junction.tsv --existence --out out.txt --ignore-genes --ignore-counts --pairs pairs.out --differences 2`

With `--differences 1`, it is also possible to allow 1 insertion or deletion: 

`compairr cdr3s.tsv IEDB_data.tsv --existence --out out.txt --ignore-genes --ignore-counts --pairs pairs.out --differences 1 --indels`

See for example the cdr3 with sequence_id 14 in the pairs output file. 
The original sequence and the matching IEDB sequence are of a different length. 

We can also allow a large number of differences, and see what their distance to the matching IEDB sequence was: 

`compairr cdr3s.tsv IEDB_data_junction.tsv --existence --out out.txt --ignore-genes --ignore-counts --pairs pairs.out --differences 5 --distance`

Note that for large datasets, the running time may increase a lot when setting a large number of differences. 
Multithreading (specified with the `--threads` parameter) can help reduce the running time. 

Lastly, it may be of interest to keep certain additional columns from the input file(s) in the resulting 'pairs' file. 
The parameter `--keep-column` can be used to specify the (comma-separated) column names to keep. For example, keeping
the 'epitopes' and 'source_organisms' columns from the IEDB file:

`compairr cdr3s.tsv IEDB_data_junction.tsv --existence --out out.txt --ignore-genes --ignore-counts --pairs pairs.out --differences 1 --keep-columns epitopes,source_organisms`
