ScienceDirect API in Bash
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

by Avery Fernandez

**ScienceDirect**: https://www.sciencedirect.com/

**Elsevier Developer Portal:** https://dev.elsevier.com/

**ScienceDirect APIs Specification:** https://dev.elsevier.com/sd_api_spec.html

**Elsevier How to Guide: Text Mining:** https://dev.elsevier.com/tecdoc_text_mining.html

Please check with your institution for their Text and Data Mining Agreement with Elsevier.

These recipe examples use the Elsevier ScienceDirect Article (Full-Text) API. This tutorial content is intended to help facillitate academic research. Before continuing or reusing any of this code, please be aware of Elsevier's API policies and appropiate use-cases, as for example, Elsevier has detailed policies regarding `text and data mining of Elsevier full-text content <https://dev.elsevier.com/text_mining.html>`_. If you have copyright or other related text and data mining questions, please contact The University of Alabama Libraries.

*These recipe examples were tested on November 22, 2022.*

Setup
======

Program requirements
--------------------

In order to run this code, you will need to first install `curl`_. curl is used to request the data from the API.

.. _curl: https://github.com/curl/curl

Create a variable for API Key
---------------------------------

Save your API key to a separate text file, then create a variable for your key. Avoid displaying your API key in your terminal (to prevent accidental sharing). 

.. code-block:: shell

   apiKey=$(cat "apikey.txt")

Identifier Note
-----------------

We will use DOIs as the article identifiers. See our Crossref and Scopus API tutorials for workflows on how to create lists of DOIs and identifiers for specific searches and journals. The Elsevier ScienceDirect Article (Full-Text) API also accepts other identifiers like Scopus IDs and PubMed IDs (see API specification documents linked above).

1. Retrieve full-text XML of an article
=======================================

.. code-block:: shell

   elsevier_url="https://api.elsevier.com/content/article/doi/"
   doi1='10.1016/j.tetlet.2017.07.080' # example Tetrahedron Letters article
   curl "$elsevier_url""$doi1"$"?APIKey=""$apiKey"$"&httpAccept=text/xml" > fulltext1.xml # save to file

2. Retrieve plain text of an article
====================================

.. code-block:: shell

   elsevier_url="https://api.elsevier.com/content/article/doi/"
   doi2='10.1016/j.tetlet.2022.153680' # example Tetrahedron Letters article
   curl "$elsevier_url""$doi2"$"?APIKey=""$apiKey"$"&httpAccept=text/plain" > fulltext2.txt # save to file

3. Retrieve full-text in a loop
===============================

Create an array of 5 DOIs for testing.

.. code-block:: shell

   declare -a dois=("10.1016/j.tetlet.2018.10.031" "10.1016/j.tetlet.2018.10.033" "10.1016/j.tetlet.2018.10.034" "10.1016/j.tetlet.2018.10.038" "10.1016/j.tetlet.2018.10.041")

Retrieve article full text for each DOI in a loop and save each article to a separate file. Example shown for plain text, XML also works (replace 'plain' with 'xml')

.. code-block:: shell

   elsevier_url="https://api.elsevier.com/content/article/doi/"
   for doi in "${dois[@]}"
   do
     doi_name=$(echo "$doi" | sed 's/\//-/') # can't save files with a '/' character on Linux
     echo "$doi_name"
     curl "$elsevier_url""$doi"$"?APIKey=""$apiKey"$"&httpAccept=text/plain" > "$doi_name"$"_plain_text.txt"
     sleep 1 # pause for 1 second between API calls
   done

.. code-block:: shell

   ls

**Output:**

.. code-block:: shell

   10.1016-j.tetlet.2018.10.031_plain_text.txt
   10.1016-j.tetlet.2018.10.033_plain_text.txt
   10.1016-j.tetlet.2018.10.034_plain_text.txt
   10.1016-j.tetlet.2018.10.038_plain_text.txt
   10.1016-j.tetlet.2018.10.041_plain_text.txt

