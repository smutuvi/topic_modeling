topic-stability
===============

### Summary
Despite the many [topic modeling algorithms](http://en.wikipedia.org/wiki/Topic_model) that have been proposed for text mining, a common challenge is selecting an appropriate number of topics for a particular data set. Choosing too few topics will produce results that are overly broad, while choosing too many will result in the over-clustering of a data into many redundant, highly-similar topics. We have developed a *stability analysis* approach to address this problem, the idea being that a model with an appropriate number of topics will be more robust to perturbations in the data. Details of this approach are described in the following paper:

	How Many Topics? Stability Analysis for Topic Models (2014)
	Derek Greene, Derek O'Callaghan, Pádraig Cunningham
	http://arxiv.org/abs/1404.4606	
	
This repository contains a Python reference implementation of the above approach.

### Dependencies
Tested with Python 2.7 and Python 3.5, and requiring the following packages, which are available via PIP:

* Required: [numpy >= 1.8.0](http://www.numpy.org/)
* Required: [scikit-learn >= 0.14](http://scikit-learn.org/stable/)
* Required for NMF: [nimfa >= 1.2.x](http://nimfa.biolab.si/)
* Required for LDA: [scipy >= 0.13](http://www.scipy.org/)
* Required for utility tools: [prettytable >= 0.7.2](https://code.google.com/p/prettytable/)

The following dependency is bundled with this project:
- [hungarian-algorithm 2013-11-03](https://github.com/tdedecko/hungarian-algorithm)
 
To run the LDA tools, an installation of Mallet 2.0 is required, which is available [here](http://mallet.cs.umass.edu/). The current code has been tested with Mallet version 2.0.8-RC3

### Basic Usage
Before applying topic modeling to a corpus, the first step is to pre-process the corpus and store it in a suitable format. The script 'parse-text.py' can be used to parse a directory of plain text documents. Here, we parse all .txt files in the directory or sub-directories of 'data/sample'. The output files will also have the prefix 'sample'.

	python parse-text.py data/sample/ -o sample

The output will be a number of Joblib binary files, with the main corpus file being named 'sample.pkl'.

If we are interested in applying topic modelling based on Non-negative Matrix Factorization (NMF), we next generate a *reference* set of topics on the pre-processed corpus by using the script 'reference-nmf.py'.  Our initial estimate for a range for the number of topics (*k*) for our corpus is between 2 and 8.

	python reference-nmf.py sample.pkl --kmin 2 --kmax 8 -o reference-nmf/

The output of the process above will be 7 sub-directories of 'reference-nmf', each containing a reference topic model for a different value of *k*.

Next, we generate a set of 50 "standard" topic models for each value of *k* using 'generate-nmf.py'. Again we specify the range of candidates values for *k* to be [2,8].
	
	python generate-nmf.py sample.pkl --kmin 2 --kmax 8 -r 50 -o topic-nmf/
	
The output of this process will be 7 sub-directories of 'topic-nmf', each containing 50 topic modeling results for a different value of *k* (e.g. 'topic-nmf/nmf_k08/' contains results for *k=8*). The term rankings for each result are stored in separate files in these sub-directories (e.g. 'topic-nmf/nmf_k08/ranks_1000_050.pkl').

Once all topic models have been generated, to evaluate the stability of a specific value of *k*, use the 'topic-stability.py' tool. The required arguments for the tool are the reference ranks file, followed by the list of topic model rank files for the same value of *k*. For instance, to evaluate the stability for *k=2* using the top 20 terms from the rankings generated as per above, run:

	python topic-stability.py -t 20 reference-nmf/nmf_k02/ranks_reference.pkl topic-nmf/nmf_k02/ranks*

### Other Algorithms

This package also includes tools to apply stability analysis for other topic modeling approaches. Stability model selection is performed in an analogous way to that described for NMF above.

##### Latent Dirichlet Allocation (LDA)

A wrapper is included for the LDA implementation from the Mallet toolkit. Note that the path to the main Mallet script file needs to be passed as a parameter when running LDA. Specify the option '--rerank' to re-rank terms using the method proposed by Blei and Lafferty (2009). 

	python reference-lda.py sample.pkl --kmin 2 --kmax 8 --rerank -o reference-lda -p ./mallet/bin/mallet
	python generate-lda.py sample.pkl --kmin 2 --kmax 8 -r 50 --rerank -o topic-lda/ -p ./mallet/bin/mallet

As for NMF, to evaluate the stability of LDA for *k=2* using the top 20 terms from the rankings generated as per above, run:

	python topic-stability.py -t 20 reference-lda/lda_k02/ranks_reference.pkl topic-lda/lda_k02/ranks*

### Utility Tools

A number of other utility tools are included in the package:

* 'display-topics.py': Simple tool to display term rankings stored in one or more PKL
files.
* 'validate-topics.py': Compare term rankings, with "gold standard" term rankings coming from a set of ground truth classes associated with a given corpus.
* 'convert-pkl2mtx.py': Convert a previously pre-processed corpus, stored in binary Joblib (PKL) format, into a plain text format for use with other tools.

