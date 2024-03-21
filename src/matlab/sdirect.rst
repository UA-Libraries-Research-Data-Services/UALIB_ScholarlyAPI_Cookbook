ScienceDirect API in Matlab
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Anastasia Ramig

These recipe examples use the Elsevier ScienceDirect Article (Full-Text) API. Code was tested with MATLAB R2022b and sample data downloaded from the ScienceDirect API on November 22, 2022 via http://api.elsevier.com and https://www.sciencedirect.com/.

You will need to register for an API key from the Elsevier Developer portal in order to use the ScienceDirect API. This tutorial content is intended to help facilitate academic research. Before continuing or reusing any of this code, please be aware of Elsevier’s API policies and appropriate use-cases, as for example, Elsevier has detailed policies regarding `text and data mining of Elsevier full-text content <https://dev.elsevier.com/text_mining.html>`_. If you have copyright or other related text and data mining questions, please contact The University of Alabama Libraries.

**ScienceDirect APIs Specification:** https://dev.elsevier.com/sd_api_spec.html

**Elsevier How to Guide: Text Mining:** https://dev.elsevier.com/tecdoc_text_mining.html

Setup
======

Import API Key
---------------------------------

As a good practice, do not display your API key in your computational notebook (to prevent accidental sharing). Save your API key to a separate text file, then import your key.

.. code-block:: matlab

   %% import API key from file
   myAPIKey = importdata("apikey.txt");

Identifier Note
-----------------

We will use DOIs as the article identifiers. See our Crossref and Scopus API tutorials for workflows on how to create lists of DOIs and identifiers for specific searches and journals. The Elsevier ScienceDirect Article (Full-Text) API also accepts other identifiers like Scopus IDs and PubMed IDs (see API specification documents linked above).

1. Retrieve full-text XML of an article
=======================================

.. code-block:: matlab

   %% download XML text
   elsevier_url = "https://api.elsevier.com/content/article/doi/";
   doi1 = '10.1016/j.tetlet.2017.07.080'; %% example Tetrahedron Letters article
   fulltext1 = {webread(elsevier_url + doi1 + "?APIKey=" + myAPIKey + "&httpAccept=text/xml")};
 
   %% save to file
   writecell(fulltext1, "fulltext1.txt");  %% can change to .xml after writing
   

2. Retrieve plain text of an article
====================================

.. code-block:: matlab

   %% download simplified text
   elsevier_url = "https://api.elsevier.com/content/article/doi/";
   doi2 = "10.1016/j.tetlet.2022.153680";
   fulltext2 = webread(elsevier_url + doi2 + "?APIKey=" + myAPIKey + "&httpAccept=text/plain");
 
   %% save to file
   writematrix(fulltext2, "fulltext2.txt", "Delimiter", "\t");

3. Retrieve full-text in a loop
===============================

Create an array of 5 DOIs for testing.

.. code-block:: matlab

   %% make a list of 5 DOIs for testing
   dois = ["10.1016/j.tetlet.2018.10.031","10.1016/j.tetlet.2018.10.033",...
    "10.1016/j.tetlet.2018.10.034","10.1016/j.tetlet.2018.10.038",...
    "10.1016/j.tetlet.2018.10.041"];

Retrieve article full text for each DOI in a loop and save each article to a separate file. Example shown for plain text, XML also works (replace 'plain' with 'xml')

.. code-block:: matlab

   for i = 1:length(dois)
       article = webread(elsevier_url + dois(i) + "?APIKey=" + myAPIKey + "&httpAccept=text/plain");
    
       %% replace '/' with '_' since you can't save files with an '/' character on Matlab
       old = "/";
       new = "_";
       doi_name = replace(dois(i), old, new);
       writematrix(article, (doi_name + "_plain_text.txt"), "Delimiter", "\t");
    
       %% pause for 1 second between API calls
       pause(1)
   end

