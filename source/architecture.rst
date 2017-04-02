=================
Architecture
=================

The PATRIC system is comprised of three primary components:

 * A database which stores the core PATRIC data (genomics,
   transcriptomics, protein–protein interactions, 3D protein
   structures and sequence typing data),

 * A web application which exposes the database to online users, and

 * A collection of back-end services which support the user data
   workspace and the data analysis applications.

Database
========

The PATRIC database is implemented as a set of Solr cores. Each core
stores a denormalized form of one aspect of the PATRIC data. The
cores' schemas are defined to allow efficient search and faceting for
fields common to and useful for bioinformatic analyses. For instance,
the :any:`solr-genome_feature` core (storing information about features on
genomes, such as protein encoding genes and RNAs) contains the genome
identifier and genome name. This allows the user to make use of the
fulltext Solr index to search for strings like "dnaK Escherichia" to
limit the results of a search to a given set of organisms.

The cores are defined using the Solr schema language; we have
incorporated a translation of the cores' schemas in this book in the
:any:`schema-reference` appendix. The following cores store the
primary data used in PATRIC:

 * :any:`solr-genome`: The genome core has an entry for every genome
   in PATRIC.

 * :any:`solr-genome_feature`: The genome feature core has an entry
   for every feature in every genome in PATRIC. It includes
   information about the annotations on the features along with the
   features' sequence data and location on the chromosome.

 * :any:`solr-genome_sequence`: The genome sequence core stores
   information about the contigs in each genome, along with the contig
   DNA sequences.


