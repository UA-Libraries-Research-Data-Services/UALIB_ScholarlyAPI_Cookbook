Scopus API in Unix Shell
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Avery Fernandez

These recipe examples use the Elsevier Scopus API. Code was tested and sample data downloaded from the Scopus
API on April 7, 2022 via http://api.elsevier.com and http://www.scopus.com. This tutorial content is intended to help 
facillitate academic research. Before continuing or reusing any of this code, please be aware of
Elsevier's `API policies and appropiate use-cases`_. You will also need to register for an API key
in order to use the Scopus API.

.. _API policies and appropiate use-cases: https://dev.elsevier.com/use_cases.html

Setup
==========

Program requirements
--------------------

In order to run this code, you will need to first install `curl`_, and `jq`_.
curl is used to request the data from the API and jq is used to parse the JSON data.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/

API Key Information
-------------------

We will start by setting up the API key. Save your key in a text file in
your current directory and import your key as follows:

.. code-block:: shell

   apiKey=$(cat "apikey.txt")

Set the url for the base API:

.. code-block:: shell

   api="https://api.elsevier.com/content/search/scopus"

1. Get Author data
======================

Records for Author
--------------------------------

.. code-block:: shell

   rawAuthorSearch=$(curl $api$"?query=AU-ID(55764087400)&apiKey=""$apiKey" | jq '.["search-results"]')
   
Here is how to view the returned data (`[0]` to show first record):

.. code-block:: shell

   echo "$rawAuthorSearch" | jq '.["entry"][0]'

The Raw JSON file is converted into a dictionary/associative array, which can be queried using the keys listed below:

.. code-block:: shell

   echo "$rawAuthorSearch" | jq '.["entry"][0] | keys'

**Output:**

.. code-block:: shell

   [
   "@_fa",
   "affiliation",
   "citedby-count",
   "dc:creator",
   "dc:identifier",
   "dc:title",
   "eid",
   "link",
   "openaccess",
   "openaccessFlag",
   "prism:aggregationType",
   "prism:coverDate",
   "prism:coverDisplayDate",
   "prism:doi",
   "prism:eIssn",
   "prism:issn",
   "prism:issueIdentifier",
   "prism:pageRange",
   "prism:publicationName",
   "prism:url",
   "prism:volume",
   "source-id",
   "subtype",
   "subtypeDescription"
   ]

Extracting all the DOIs from the author data:

.. code-block:: shell

   echo "$rawAuthorSearch" | jq '.["entry"][]["prism:doi"]'

**Output:**

.. code-block:: shell

   "10.1021/acs.jchemed.1c00904"
   "10.5860/crln.82.9.428"
   "10.1021/acs.iecr.8b02573"
   "10.1021/acs.jchemed.6b00602"
   "10.5062/F4TD9VBX"
   "10.1021/acs.macromol.6b02005"
   "10.1186/s13321-016-0181-z"
   "10.1021/acs.chemmater.5b04431"
   "10.1021/acs.jchemed.5b00512"
   "10.1021/acs.jchemed.5b00375"
   "10.5860/crln.76.9.9384"
   "10.5860/crln.76.2.9259"
   "10.1021/ed400887t"
   "10.1016/j.acalib.2014.03.015"
   "10.5062/F4XS5SB9"
   "10.1021/ma300328u"
   "10.1021/mz200108a"
   "10.1021/ma201170y"
   "10.1021/ma200184u"
   "10.1021/cm102374t"

Extract all titles:

.. code-block:: shell

   echo "$rawAuthorSearch" | jq '.["entry"][]["dc:title"]'

**Output:**

.. code-block:: shell

   "Using NCBI Entrez Direct (EDirect) for Small Molecule Chemical Information Searching in a Unix Terminal"
   "Using the linux operating system full-time tips and experiences from a subject liaison librarian"
   "Analysis of the Frequency and Diversity of 1,3-Dialkylimidazolium Ionic Liquids Appearing in the Literature"
   "Rapid Access to Multicolor Three-Dimensional Printed Chemistry and Biochemistry Models Using Visualization and Three-Dimensional Printing Software Programs"
   "Text analysis of chemistry thesis and dissertation titles"
   "Phototunable Thermoplastic Elastomer Hydrogel Networks"
   "Programmatic conversion of crystal structures into 3D printable files using Jmol"
   "Dangling-End Double Networks: Tapping Hidden Toughness in Highly Swollen Thermoplastic Elastomer Hydrogels"
   "Replacing the Traditional Graduate Chemistry Literature Seminar with a Chemical Research Literacy Course"
   "3D Printed Block Copolymer Nanostructures"
   "Hypotheses in librarianship: Applying the scientific method"
   "Recruiting students to campus: Creating tangible and digital products in the academic library"
   "3D printed molecules and extended solid models for teaching symmetry and point groups"
   "Repurposing Space in a Science and Engineering Library: Considerations for a Successful Outcome"
   "A model for managing 3D printing services in academic libraries"
   "Morphological phase behavior of poly(RTIL)-containing diblock copolymer melts"
   "Network formation in an orthogonally self-assembling system"
   "Access to nanostructured hydrogel networks through photocured body-centered cubic block copolymer melts"
   "Synthesis and ordered phase separation of imidazolium-based alkyl-ionic diblock copolymers made via ROMP"
   "Thermally stable photocuring chemistry for selective morphological trapping in block copolymer melt systems"

Citation information:

.. code-block:: shell

   echo "$rawAuthorSearch" | jq '.["entry"][]["citedby-count"]'

**Output:**

.. code-block:: shell

   "0"
   "0"
   "17"
   "24"
   "4"
   "11"
   "20"
   "6"
   "10"
   "25"
   "0"
   "0"
   "97"
   "6"
   "34"
   "40"
   "31"
   "18"
   "45"
   "11"

2. Author Data in a Loop
==========================

Number of Records for Author
---------------------------------

Setup an array of Authors and their Scopus IDs:

.. code-block:: shell

   declare -A names=( [36660678600]="Emy Decker" [57210944451]="Lindsey Lowry" [35783926100]="Karen Chapman" [56133961300]="Kevin Walker" [57194760730]="Sara Whitver" )

Find the number of records for each author:

.. code-block:: shell

   declare -A numRecords
   for ids in "${!names[@]}";
   do
     echo "$ids"
     AuthorData=$(curl $api"?query=AU-ID(""$ids"$")&apiKey=""$apiKey" | jq '.["search-results"]')
     echo "$AuthorData"
     numRecords[$ids]=$(echo "$AuthorData" | jq '.["opensearch:totalResults"]')
     sleep 1
   done

   for key in "${!numRecords[@]}";
   do
     echo "$key"$": ""${numRecords["$key"]}"
   done

**Output:**

.. code-block:: shell

   57210944451: "4"
   56133961300: "8"
   36660678600: "14"
   35783926100: "29"
   57194760730: "4"

Download Record Data
------------------------

Let's say we want the DOIs and cited by counts in a csv file

.. code-block:: shell

   truncate -s 0 authors.csv
   echo $"AuthorID,DOI,citedby" >> authors.csv
   for ids in "${!names[@]}";
   do
     AuthorData=$(curl $api"?query=AU-ID(""$ids"$")&apiKey=""$apiKey" | jq '.["search-results"]')
     sleep 1
     length=$(echo "$AuthorData" | jq '.["entry"] | length')
     for (( i = 0 ; i < length ; i++));
     do
       data=$(echo "$AuthorData" | jq ".entry[$i]")
       doi=$(echo "$data" | jq '.["prism:doi"]')
       cite=$(echo "$data" | jq '.["citedby-count"]')
       echo "${names["$ids"]}"$",""$doi"$",""$cite" >> authors.csv
     done
   done

**Output:**

.. code-block:: shell

   AuthorID,DOI,citedby
   Lindsey Lowry,"10.1080/1941126X.2021.1949153","1"
   Lindsey Lowry,"10.5860/lrts.65n1.4-13","0"
   Lindsey Lowry,"10.1080/00987913.2020.1733173","1"
   Lindsey Lowry,"10.1080/1941126X.2019.1634951","0"
   Kevin Walker,"10.1016/j.acalib.2021.102450","0"
   Kevin Walker,"10.1016/j.acalib.2020.102136","4"
   Kevin Walker,"10.1016/j.lisr.2019.100968","2"
   Kevin Walker,"10.1016/j.acalib.2019.02.013","10"
   Kevin Walker,"10.1027/1614-2241/a000166","2"
   ...
   ...

Get the article titles:

.. code-block:: shell

   for ids in "${!names[@]}";
   do
     echo $"Author: ""${names["$ids"]}"
     AuthorData=$(curl -s $api"?query=AU-ID(""$ids"$")&apiKey=""$apiKey" | jq '.["search-results"]') # -s makes the download silent
     sleep 1
     length=$(echo "$AuthorData" | jq '.["entry"] | length')
     for (( i = 0 ; i < length ; i++));
     do
       data=$(echo "$AuthorData" | jq ".entry[$i]")
       echo "$data" | jq '.["dc:title"]'
     done
   done

**Output:**

.. code-block:: shell

   Author: Lindsey Lowry
   "Exploring the evidence-base for electronic access troubleshooting: Where research meets practice"
   "Fighting an uphill battle: Troubleshooting assessment practices in academic libraries"
   "Where Do Our Problems Lie?: Comparing Rates of E-Access Problems Across Three Research Institutions"
   "Using LastPass to facilitate the gathering of usage statistics for e-resources: a case study"
   Author: Kevin Walker
   "Exploring adaptive boosting (AdaBoost) as a platform for the predictive modeling of tangible collection usage"
   "Assessing information literacy in first year writing"
   "Modeling time-to-trigger in library demand-driven acquisitions via survival analysis"
   "Application of adaptive boosting (AdaBoost) in demand-driven acquisition (DDA) prediction: A machine-learning approach"
   "Applying AdaBoost to Improve Diagnostic Accuracy: A Simulation Study"
   "Judging the Need for and Value of DDA in an Academic Research Library Setting"
   "Improving generalizability coefficient estimate accuracy: A way to incorporate auxiliary information"
   "Student Engagement in One-Shot Library Instruction"
   Author: Emy Decker
   "Launching chat service during the pandemic: inaugurating a new public service under emergency conditions"
   "Making Sense of the Lending Fill Rate in Interlibrary Loan: Investigating Causes for Low Fill Rates and Developing Potential Remedies"
   "Reaching academic library users during the COVID-19 pandemic: New and adapted approaches in access services"
   "Expediting the delivery of content to library users: When to buy versus when to borrow"
   ...
   ...

3. Get References via a Title Search
==========================================

Number of Title Match Records
---------------------------------

Search Scopus for all references containing' ChemSpider' in the record title

All the data will be stored into individual **entry** locations

.. code-block:: shell

   query=$(curl "$api"$"?query=TITLE(ChemSpider)&apiKey=""$apiKey" | jq '.["search-results"]')
   echo "$query" | jq '.["entry"][0]'
   length=$(echo "$query" | jq '.["entry"] | length')

Repeat this in a loop to get number of Scopus records for each title search:

.. code-block:: shell

   declare -a titles=("ChemSpider" "PubChem" "ChEMBL" "Reaxys" "SciFinder")
   declare -A storage
   for title in "${titles[@]}";
   do
     storage["$title"]=$(curl "$api"$"?query=TITLE(""$title"$")&apiKey=""$apiKey" | jq '.["search-results"]')
     sleep 1
   done

   for title in "${!storage[@]}";
   do
     search=$(echo "${storage["$title"]}" | jq '.["opensearch:totalResults"]')
     echo "$title"$": ""$search"
   done

**Output:**

.. code-block:: shell

   Reaxys: "8"
   PubChem: "83"
   SciFinder: "31"
   ChemSpider: "7"
   ChEMBL: "53"

Title Match Record Data
-----------------------------------

Create a csv of selected metadata:

.. code-block:: shell

   truncate -s 0 titles.csv
   echo $"Title,DOI,Article,Date" >> titles.csv
   for title in "${!storage[@]}";
   do
     length=$(echo "${storage["$title"]}" | jq '.["entry"] | length')
     for (( i = 0 ; i < "$length" ; i++));
     do
       data=$(echo "${storage["$title"]}" | jq ".entry[$i]" )
       doi=$(echo "$data" | jq '.["prism:doi"]')
       articleTitle=$(echo "$data" | jq '.["dc:title"]')
       date=$(echo "$data" | jq '.["prism:coverDate"]')
       echo "$title"$",""$doi"$",""$articleTitle"$",""$date" >> titles.csv
     done
   done

**Output:**

.. code-block:: shell

   Title,DOI,Article,Date
   Reaxys,null,"Store unit files for bundling activities - Reaxys","2018-04-06"
   Reaxys,null,"Hybrid Retrosynthesis: Organic Synthesis using Reaxys and SciFinder","2015-01-01"
   Reaxys,null,"Comparisons of the most important chemistry databases - Scifinder program and reaxys database system","2014-01-30"
   Reaxys,"10.1021/bk-2014-1164.ch008","The making of reaxys - Towards unobstructed access to relevant chemistry information","2014-01-01"
   Reaxys,null,"A chemistry searcher compares CAS'S SciFinder and elsevier's reaxys","2013-09-01"
   Reaxys,null,"Od beilsteina do reaxys","2012-04-30"
   Reaxys,null,"Store unit files for bundling activities - Reaxys","2011-11-07"
   Reaxys,"10.1002/nadc.201179450","Beilstein and Gmelin combined in Reaxys","2011-04-01"
   PubChem,"10.1016/j.bioorg.2022.105648","Structure-based discovery of a specific SHP2 inhibitor with enhanced blood–brain barrier penetration from PubChem database","2022-04-01"
