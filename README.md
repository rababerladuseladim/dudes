# DUDes: a top-down taxonomic profiler for metagenomics

Vitor C. Piro (vitorpiro@gmail.com)

[![install with bioconda](https://img.shields.io/badge/install%20with-bioconda-brightgreen.svg?style=flat-square)](http://bioconda.github.io/recipes/dudes/README.html)

Piro, V. C., Lindner, M. S., & Renard, B. Y. (2016). DUDes: a top-down taxonomic profiler for metagenomics. Bioinformatics, 32(15), 2272–2280. http://doi.org/10.1093/bioinformatics/btw150

Requirements:
-------------
python3 and numpy (DUDes.py) and pandas (DUDesDB.py only)

Install:
--------

Local installation
	
	git clone https://github.com/pirovc/dudes.git
	cd dudes
	./DUDes.py -h

Global installation
	
	conda install -c bioconda dudes
	DUDes.py -h
	
	or
	
	git clone https://github.com/pirovc/dudes.git
	cd dudes
	python3 setup.py install
	DUDes.py -h


Usage:
------

- Download the pre-compiled index and database:
	
| Info 	| Date	| Size	| Link	|
| --- 	| --- 	| ---	| ---	|
| Archaea + Bacteria - RefSeq Complete Genomes | 2015-03 | 13.2 GB | https://zenodo.org/record/1036748/files/dudesdb_arc-bac_refseq-cg_201503.tar.gz |
| Archaea + Bacteria - RefSeq Complete Genomes | 2017-09 | 37.7 GB | https://zenodo.org/record/1037091/files/dudesdb_arc-bac_refseq-cg_201709.tar.gz |
| Fungal + Viral - RefSeq Complete Genomes | 2017-09 | 9.5 GB | https://zenodo.org/record/1037288/files/dudesdb_fun-vir_refseq-cg_201709.tar.gz |

Unpack:
	
	tar zxfv dudesdb_arc-bac_refseq-cg_201709.tar.gz

Map your reads (fastq) with bowtie2 (any other mapper/index can be used - check `-i` parameter on DUDes.py):
	
	bowtie2 -x dudesdb_arc-bac_refseq-cg_201709/arc-bac_refseq-cg_201709 --no-unal --very-fast -k 10 -1 reads.1.fq -2 reads.2.fq -S mapping_output.sam

Run DUDes

	DUDes.py -s mapping_output.sam -d dudesdb_arc-bac_refseq-cg_201709/arc-bac_refseq-cg_201709.npz -o output_prefix

Example with sample data:
-------------------------

	DUDes.py -s sampledata/hiseq_accuracy_k60.sam -d sampledata/arc-bac_refseq-cg_201503.npz -o sampledata/dudes_profile_output
	
- The sample data is based on a set of bacterial whole-genome shotgun reads comprising 10 organisms (HiSeq - 10000 reads [1]). The read set was mapped with Bowtie2 [2] against the set of complete genome sequences (dudesdb_arc-bac_refseq-cg_201503).

Custom index and dudes database:
--------------------------------

Index your reference file (.fasta) with bowtie2 (any other mapper/index can be used - check `-i` parameter on DUDes.py):
	
	bowtie2-build -f references.fasta custom_db
	
Create a dudes database based on the same set of references:

	[python3] DUDesDB.py -m 'av' -f references.fasta -n nodes.dmp -a names.dmp -g nucl_gb.accession2taxid -t 12 -o custom_db
	
Choose the parameter `-m` considering the format of the headers in your reference sequences:

	New NCBI header [>NC_009925.1 Acaryochloris marina MBIC11017, complete genome.]
		-m 'av'
	Old NCBI header [>gi|158333233|ref|NC_009925.1| Acaryochloris marina MBIC11017, complete genome.]
		-m 'gi'

`nodes.dmp` and `names.dmp` can be obtained from:
	
	ftp://ftp.ncbi.nih.gov/pub/taxonomy/taxdump.tar.gz
		
`nucl_gb.accession2taxid`, `nucl_wgs.accession2taxid` or `gi_taxid_nucl.dmp.gz`(depending on your reference origin) can be obtained from:
		
	ftp://ftp.ncbi.nih.gov/pub/taxonomy/accession2taxid/nucl_XX.accession2taxid
	ftp://ftp.ncbi.nih.gov/pub/taxonomy/gi_taxid_nucl.dmp.gz

Details:
--------

DUDes.py requires two main input files to perform the taxonomic analysis:
1) a sequence alignment/map file (.sam file)
2) a database generated by DUDesDB.py (.npz file)

DUDesDB.py links taxonomic information and reference sequences identifiers (GI or accession.version). The input to DUDesDB script should be the same set of reference sequences (or a subset with matching identifiers)** used for the index database of the mapping tool.

** It is possible to run DUDes with previously generated alignment/map files with a pre-compiled database (see above) or with a database generated from a different source/date/version from the mapping tool. DUDes' algorithm filters references (and matches) not found in DUDes database before performing the analysis. Notice that some information can be lost in this case.
	
Parameters:
-----------
 
	$ dudes --help

	usage: dudes [-h] (-s <sam_file> | -c <custom_blast_file>) -d <database_file>
             [-i <sam_format>] [-t <threads>] [-x <taxid_start>]
             [-m <max_read_matches>] [-a <min_reference_matches>]
             [-l <last_rank>] [-b <bin_size>] [--no-normalize]
             [-o <output_prefix>] [--debug]
             [--debug_plots_dir DEBUG_PLOTS_DIR] [-v]

    options:
      -h, --help            show this help message and exit
      -s <sam_file>         Alignment/mapping file in SAM format. DUDes does not
                            depend on any specific read mapper, but it requires
                            header information (@SQ
                            SN:gi|556555098|ref|NC_022650.1| LN:55956) and
                            mismatch information (check -i)
      -c <custom_blast_file>
                            Alignment/mapping file in custom BLAST format. The
                            required columns and their order are: 'qseqid',
                            'sseqid', 'slen', 'sstart', 'evalue'. Additional 
                            columns are ignored.
                            Example command for creating appropriate file with
                            diamond: 'diamond blastp -q {query_fasta} -d
                            {diamond_database} --outfmt 6 qseqid sseqid slen
                            sstart evalue'
      -d <database_file>    Database file (output from DUDesDB [.npz])
      -i <sam_format>       SAM file format, ignored for custom blast files
                            ['nm': sam file with standard cigar string plus NM
                            flag (NM:i:[0-9]*) for mismatches count | 'ex': just
                            the extended cigar string]. Default: 'nm'
      -t <threads>          # of threads. Default: 1
      -x <taxid_start>      Taxonomic Id used to start the analysis (1 = root).
                            Default: 1
      -m <max_read_matches>
                            Keep reads up to this number/percentile of matches (0:
                            off / 0-1: percentile / >=1: match count). Default: 0
      -a <min_reference_matches>
                            Minimum number/percentage of supporting matches to
                            consider the reference (0: off / 0-1: percentage /
                            >=1: read number). Default: 0.001
      -l <last_rank>        Last considered rank [superkingdom,phylum,class,order,
                            family,genus,species,strain]. Default: 'species'
      -b <bin_size>         Bin size (0-1: percentile from the lengths of all
                            references in the database / >=1: bp). Default: 0.25
      --no-normalize        Do not normalize by total sequence length of all
                            references belonging to an identified TaxID. The idea
                            of normalization is to quantify cell number rather
                            than total abundance.
      -o <output_prefix>    Output prefix. Default: STDOUT
      --debug               print debug info to STDERR
      --debug_plots_dir DEBUG_PLOTS_DIR
                            path to directory for writing debug plots to.
      -v                    show program's version number and exit

-----------

	$ dudesdb --help

	usage: dudesdb [-h] [-m <reference_mode>] -f [<fasta_files> ...] -g
               [<ref2tax_files> ...] -n <nodes_file> [-a <names_file>]
               [-o <output_prefix>] [-t <threads>] [-v]
    
    options:
      -h, --help            show this help message and exit
      -m <reference_mode>   'gi' uses the GI as the identifier (For headers like:
                            >gi|158333233|ref|NC_009925.1|) [NCBI is phasing out
                            sequence GI numbers in September 2016]. 'av' uses the
                            accession.version as the identifier (for headers like:
                            >NC_013791.2). 'up' uses the uniprot accession as
                            identifier (for headers like: >sp|Q197F8|... Default:
                            'av'
      -f [<fasta_files> ...]
                            Reference fasta file(s) for header extraction only,
                            plain or gzipped - the same file used to generate the
                            read mapping index. Each sequence header '>' should
                            contain a identifier as defined in the reference mode.
      -g [<ref2tax_files> ...]
                            reference id to taxid file(s):
                            'gi_taxid_nucl.dmp[.gz]' --> 'gi' mode,
                            '*.accession2taxid[.gz]' --> 'av' mode [from NCBI
                            taxonomy database https://ftp.ncbi.nlm.nih.gov/pub/tax
                            onomy/]'idmapping_selected.tab[.gz]' --> 'up' mode
                            [from https://ftp.expasy.org/databases/uniprot/current
                            _release/knowledgebase/idmapping/idmapping_selected.ta
                            b.gz
      -n <nodes_file>       nodes.dmp file [from NCBI taxonomy database
                            https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/]
      -a <names_file>       names.dmp file [from NCBI taxonomy database
                            https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/]
      -o <output_prefix>    Output prefix. Default: dudesdb
      -t <threads>          # of threads. Default: 1
      -v                    show program's version number and exit

	  
Change log:
-----------

2017-11-08 (v0.08):
- bug fixes on DUDesDB and multiple gzipped file suppport for fasta_files and ref2tax_files
- distutils installation

2016-11-03 (v0.07):
- code changed to python 3
- changed .ddb to a new and smaller database format -> .npz

2016-03-23 (v0.06):
- New database format supporing GI or accession.version as an identifier (DUDesDB.py parameter -m).
- Check for sam flags
- Faster code for identification matrix evaluation

References:
-----------

[1] Wood DE, Salzberg SL: Kraken: ultrafast metagenomic sequence classification using exact alignments. Genome Biology 2014, 15:R46.

[2] Langmead B, Salzberg SL. Fast gapped-read alignment with Bowtie 2. Nature Methods 2012, 9(4), 357–9.
