=========================================================
 Introduction to the Patric Command Line Interface (CLI)
=========================================================

.. highlight:: shell

In addition to the PATRIC web interface, we provide a command-line
interface to the data in the PATRIC database. The command-line
interface (or CLI) allows one to integrate PATRIC data with existing
applications, to casually browse the data in text form, and to process
batches of data queries or computational analyses using the PATRIC
services.

The CLI consists of two primary components: a set of executable
command-line scripts and a Perl API module. The command-line scripts are
typically written in Perl and make calls to the API module. In this
chapter we will primarly focus on the scripting interface.



Most of the PATRIC command line tools take as input a file containing
a single column set or a tab-separated table and they output a
modified table. The most common modification is the addition of one of
more columns. We create "pipelines" of these tools to implement fairly
complex transformations leading to the final table containing the
desired output. 

Accessing Genome Information
============================

We begin with accessing information about genomes::

 $ p3-all-genomes

 genome.genome_id
 1390.176
 1398.26
 1345597.3
 282669.3
 .
 .
 .

The program ``p3-all-genomes`` takes no input and is what we call a
generator; it returns the set of all genome ids in PATRIC.  Notice
that the first line is a header identifying the columns in the
table. Most p3 commands expect this header. If you were interested in
certain data about the genomes, you would use the command
``p3-get-genome-data``, which takes as input a set of genome ids, like
this::

 $ p3-all-genomes | p3-get-genome-data

 genome.genome_id	genome.genome_id	genome.genome_name	genome.taxon_id	genome.genome_status	genome.gc_content
 1390.176	1390.176	Bacillus amyloliquefaciens strain B425	1390	WGS	45.7
 1398.26	1398.26	Bacillus coagulans strain B4098	1398	WGS	47.39
 1345597.3	1345597.3	Helicobacter pylori SA216A	1345597	WGS	39.02
 282669.3	282669.3	Psychrobacter cibarius strain W1	282669	WGS	70.43
 .
 .
 .

Here we use ``p3-all-genomes`` to generate a set of input ids that we
pipe into p3-get-genome-data, which returns a 6 column table with data
about the genome; the genome_id, genome_name, taxon_id, genome_status
and gc_content.  You can control which of these fields is returned
with the ``–a`` argument::

 $ p3-all-genomes |p3-get-genome-data -a genome_name

 genome.genome_id	genome.genome_name
 1390.176	Bacillus amyloliquefaciens strain B425
 1398.26	Bacillus coagulans strain B4098
 1345597.3	Helicobacter pylori SA216A
 282669.3	Psychrobacter cibarius strain W1
 1349753.3	Caldimonas taiwanensis NBRC 104434
 1285191.3	Desulfotomaculum intricatum strain NBRC 109411
 .
 .
 .

If you were interested in only Streptococcus genomes, you could use the match command like this::

 $ p3-all-genomes  | p3-get-genome-data -a genome_name | p3-match -c2 Streptococcus

 genome.genome_id	genome.genome_name
 1313.7195	Streptococcus pneumoniae strain 2842STDY5753638
 1313.7189	Streptococcus pneumoniae strain 2842STDY5643920
 1313.7203	Streptococcus pneumoniae strain 2842STDY5643723
 1313.7208	Streptococcus pneumoniae strain 2842STDY5643999
 1313.7199	Streptococcus pneumoniae strain 2842STDY5644588
 1313.7207	Streptococcus pneumoniae strain 2842STDY5643980
 .
 .
 .


Here, we retrieved the id of all genomes in PATRIC, piped that output
to get the name and produce a two column table, and piped that table
to a command to match for the string Streptococcus in column 2, thus
filtering the table to contain only Streptococcus genomes.

There are other ways to accomplish this, but the example serves to
demonstrate what we mean by creating pipelines of commands and
producing tables of information.

Accessing Features
==================

If you want to look into the features of a genome, you would use the
``p3-get-genome-features`` command. This command takes as input a
set of genome ids. The following example uses the :any:`p3-echo` command to
generate input for ``p3-get-genome-features``::

 $ p3-echo -t genome.genome_id 282669.3 | p3-get-genome-features | head 

 genome.genome_id	feature.patric_id	feature.feature_type	feature.location	feature.product
 282669.3	fig|282669.3.repeat.1	repeat_region	1..127	repeat region
 282669.3	fig|282669.3.repeat.2	repeat_region	586..712	repeat region
 282669.3	fig|282669.3.peg.4	CDS	complement(1..909)	Aspartyl-tRNA(Asn) amidotransferase subunit A (EC 6.3.5.6) @ Glutamyl-tRNA(Gln) amidotransferase subunit A (EC 6.3.5.7)
 282669.3	fig|282669.3.repeat.3	repeat_region	1..127	repeat region
 282669.3	fig|282669.3.repeat.4	repeat_region	805..931	repeat region
 282669.3	fig|282669.3.repeat.5	repeat_region	869..1006	repeat region
 282669.3	fig|282669.3.repeat.6	repeat_region	1..127	repeat region
 282669.3	fig|282669.3.repeat.7	repeat_region	1110..1236	repeat region
 282669.3	fig|282669.3.repeat.8	repeat_region	1..127	repeat region

Notice that the command returns all information about features by
default. If you were only interested in the feature ids, you would
specify that with the ``-a`` option::

 $ p3-echo -t genome.genome_id 282669.3 | p3-get-genome-features -a patric_id | head 
 genome.genome_id	feature.patric_id

 282669.3	fig|282669.3.repeat.1
 282669.3	fig|282669.3.repeat.2
 282669.3	fig|282669.3.peg.4
 282669.3	fig|282669.3.repeat.3
 282669.3	fig|282669.3.repeat.4
 282669.3	fig|282669.3.repeat.5
 282669.3	fig|282669.3.repeat.6
 282669.3	fig|282669.3.repeat.7
 282669.3	fig|282669.3.repeat.8

Since this returns all feature types, it might be desirable to limit
the features returned to a specific type. Here, we return the ids of
only the pegs in a Genome by using the ``--equal`` option::

 $ p3-echo -t genome.genome_id 282669.3 | p3-get-genome-features --equal feature_type,CDS -a patric_id| head 
 genome.genome_id	feature.patric_id
 282669.3	fig|282669.3.peg.4
 282669.3	fig|282669.3.peg.43
 282669.3	fig|282669.3.peg.72
 282669.3	fig|282669.3.peg.83
 282669.3	fig|282669.3.peg.90
 282669.3	fig|282669.3.peg.117
 282669.3	fig|282669.3.peg.179
 282669.3	fig|282669.3.peg.207
 282669.3	fig|282669.3.peg.214

In this tutorial we have introduced the basics of using the PATRIC
Command Line Interface (CLI) and how to access data relating to
genomes and features.

In the following tutorials, you will learn how to install the Patric
CLI, what all the commands are and how to use them to explore the
PATRIC website, to build collections of data and to apply
bioinformatic tools against your data.

