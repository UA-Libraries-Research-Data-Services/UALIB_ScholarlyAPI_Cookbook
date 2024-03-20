Crossref API in Unix Shell
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Avery Fernandez

**Crossref API documentation:** https://api.crossref.org/swagger-ui/index.html

These recipe examples were tested on March 7, 2022 using Ubuntu 18.04.

*From our testing, we have found that the crossref metadata across publishers and even journals can vary considerably. As a result, it can be easier to work with one journal at a time when using the crossref API (e.g., particulary when trying to extract selected data from records).*

Program requirements
=========================

In order to run this code, you will need to first install `curl`_ and `jq`_. curl is used to request the data from the API and jq is used to parse the JSON data.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/


1. Basic Crossref API call
===========================

Setup API parameters
---------------------

.. code-block:: shell

   base_url="https://api.crossref.org/works/"; email="your_email@ua.edu"; mailto="?mailto="$email; doi="10.1186/1758-2946-4-12"

.. note::
   
   The ``;`` allows us to enter multiple variable assignments on one line and the ``$`` allows for variable expansion.

Request data from Crossref API
-------------------------------

If you want to view the returned json data directly, you can pipe the curl -s (silent option) output to jq:

.. code-block:: shell

   curl -s $base_url$doi$mailto | jq '.'

*Output not shown here*

However, for our case, we will redirct the output to a file named storage.json

.. code-block:: shell

   curl $base_url$doi$mailto > storage.json

Select some specific data
---------------------------

For example, the container-title data, which contains the journal title:

.. code-block:: shell

   cat storage.json | jq '.["message"]["container-title"][0]'

**Output:**

.. code-block:: shell

   "Journal of Cheminformatics"

Get article title:

.. code-block:: shell

   cat storage.json | jq '.["message"]["title"][0]'

**Output:**

.. code-block:: shell

   "The Molecule Cloud - compact visualization of large collections of molecules"

Get article author names. First, check how many authors there are. One method is to use jq's builtin length function:

.. code-block:: shell

   cat storage.json | jq '.["message"]["author"] | length'

**Output:**

.. code-block:: shell

   2

Now we can incorporate the length into a for loop:

.. note:: 

   - The below for loop uses C syntax for looping range (e.g., ``for (( variable = 0; variable < range ; variable++ ))``).
   - The first name and last name of the authors are concatenated together using ``$`` variable expansiion.
   - The ``tr -d '"'`` command removes extra quotes around the names. 

.. code-block:: shell

   for (( i = 0; i < $(cat storage.json | jq '.["message"]["author"] | length'); i++ ))
   do
     name=$(cat storage.json | jq ".message.author[$i].given" | tr -d '"')$" "$(cat storage.json | jq ".message.author[$i].family" | tr -d '"');
     echo $name;
   done

**Output:**

.. code-block:: shell

   Peter Ertl
   Bernhard Rohde


Get bibliography references:

.. code-block:: shell

   cat storage.json | jq '.["message"]["reference"][].unstructured'

**Output:**

.. code-block:: shell

   "Martin E, Ertl P, Hunt P, Duca J, Lewis R: Gazing into the crystal ball; the future of computer-aided drug design. J Comp-Aided Mol Des. 2011, 26: 77-79."
   "Langdon SR, Brown N, Blagg J: Scaffold diversity of exemplified medicinal chemistry space. J Chem Inf Model. 2011, 26: 2174-2185."
   "Blum LC, Reymond J-C: 970 Million druglike small molecules for virtual screening in the chemical universe database GDB-13. J Am Chem Soc. 2009, 131: 8732-8733. 10.1021/ja902302h."


2. Crossref API call with a Loop
=================================

Setup API parameters
---------------------

.. code-block:: shell

   base_url="https://api.crossref.org/works/"; email="your_email@ua.edu"; mailto="?mailto="$email


Create a list of DOIs
----------------------

.. code-block:: shell
   
   doi_list=('10.1021/acsomega.1c03250' '10.1021/acsomega.1c05512' '10.1021/acsomega.8b01647' '10.1021/acsomega.1c04287' '10.1021/acsomega.8b01834')

Request metadata for each DOI from Crossref API and save to an array
---------------------------------------------------------------------

.. code-block:: shell

   declare -a my_array
   for (( i = 0 ; i < ${#doi_list[@]} ; i++ )); do
   my_array[$i]=$(curl $base_url${doi_list[$i]}$mailto)
   sleep 1;
   done

.. note::

  ``declare -a`` creates an array variable; ``${#doi_list[@]}`` returns length.

Select some specific data
---------------------------

Get article titles:

.. code-block:: shell

   for i in "${!my_array[@]}"
   do
   echo ${my_array[$i]} | jq '.["message"]["title"][0]'
   done

.. note::

   ``"${!my_array[@]}"`` returns array range.

**Output:**

.. code-block:: shell

   "Navigating into the Chemical Space of Monoamine Oxidase Inhibitors by Artificial Intelligence and Cheminformatics Approach"
   "Impact of Artificial Intelligence on Compound Discovery, Design, and Synthesis"
   "How Precise Are Our Quantitative Structure–Activity Relationship Derived Predictions for New Query Chemicals?"
   "Applying Neuromorphic Computing Simulation in Band Gap Prediction and Chemical Reaction Classification"
   "QSPR Modeling of the Refractive Index for Diverse Polymers Using 2D Descriptors"

Get all author affiliations for each article:

.. code-block:: shell

   for i in "${!my_array[@]}"
   do
   echo ${my_array[$i]} | jq '.["message"]["author"][].affiliation[0].name'
   done

.. code-block:: shell

   "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
   "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
   ...
   ...
   "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
   "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
   "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
   "Department of Coatings and Polymeric Materials, North Dakota State University, Fargo, North Dakota 58108-6050, United States"
   "Drug Theoretics and Cheminformatics Laboratory, Division of Medicinal and Pharmaceutical Chemistry, Department of Pharmaceutical Technology, Jadavpur University, 700032 Kolkata, India"


3. Crossref API call for Journal information
==============================================

Setup API parameters
---------------------

We will use the issn for the journal *BMC Bioinformatics* as an example:

.. code-block:: shell

   jbase_url="https://api.crossref.org/journals/"; email="your_email@ua.edu"; mailto="?mailto="$email; issn="1471-2105"


Request journal data from crossref API
---------------------------------------

.. code-block:: shell

   curl -s $jbase_url$issn$mailto | jq '.'

*Output not shown here*


4. Crossref API - Get article DOIs for a journal
=================================================

Setup API parameters
---------------------

We will use the issn for the journal *BMC Bioinformatics* and year 2014 as an example:

.. code-block:: shell

   jbase_url="https://api.crossref.org/journals/"; email="your_email@ua.edu"; mailto="&mailto="$email; issn="1471-2105"; journal_works2014="/works?filter=from-pub-date:2014,until-pub-date:2014&select=DOI"

Request DOI data from Crossref API
-----------------------------------

.. code-block:: shell

   curl -s $jbase_url$issn$journal_works2014$mailto | jq '.'

**Output:**

.. code-block:: shell

   {
     "status": "ok",
     "message-type": "work-list",
     "message-version": "1.0.0",
     "message": {
       "facets": {},
       "total-results": 619,
       "items": [
         {
           "DOI": "10.1186/1471-2105-15-84"
         },
         {
        "DOI": "10.1186/1471-2105-15-94"
         },
         {
           "DOI": "10.1186/1471-2105-15-172"
         },
         {
           "DOI": "10.1186/1471-2105-15-106"
         },
         {
           "DOI": "10.1186/1471-2105-15-s9-s12"

       ...
       ...

         },
         {
           "DOI": "10.1186/1471-2105-15-266"
         }
       ],
       "items-per-page": 20,
       "query": {
         "start-index": 0,
         "search-terms": null
       }
     }
   }

By default, 20 results are displayed. Crossref allows up to 1000 returned results using the rows parameter.
To get all 619 results, we can increase the number of returned rows and save the json output to a file:


.. code-block:: shell

   rows="&rows=700"
   curl $jbase_url$issn$journal_works2014$rows$mailto > dois_save.json

Extract DOIs
-----------------------------------

.. code-block:: shell

   cat dois_save.json | jq '.["message"]["items"][].DOI'

**Output:**

.. code-block:: shell

   "10.1186/1471-2105-15-84"
   "10.1186/1471-2105-15-94"
   "10.1186/1471-2105-15-172"
   "10.1186/1471-2105-15-106"
   "10.1186/1471-2105-15-s9-s12"
   "10.1186/1471-2105-15-33"
   "10.1186/1471-2105-15-s10-p33"
   "10.1186/1471-2105-15-161"
   "10.1186/1471-2105-15-278"
   "10.1186/1471-2105-15-147"
   "10.1186/1471-2105-15-s13-s3"
   "10.1186/1471-2105-15-254"
   "10.1186/1471-2105-15-s10-p24"
   "10.1186/1471-2105-15-s10-p6"
   "10.1186/s12859-014-0411-1"
   ...
   ...

.. code-block:: shell

   cat dois_save.json | jq '.["message"]["items"][].DOI' | wc -l

**Output:**

.. code-block:: shell

   619

**What if we have more than 1000 results in a single query?**

For example, if we wanted the DOIs from BMC Bioinformatics for years 2014 through 2016, we see that there are 1772 DOIs:

.. code-block:: shell

   journal_works2014_2016="/works?filter=from-pub-date:2014,until-pub-date:2016&select=DOI"
   curl -s $jbase_url$issn$journal_works2014_2016$mailto | jq '.["message"]["total-results"]'

**Output:**

.. code-block:: shell

   1772

An additional parameter that we can use with crossref API is called “offset”. The offset option allows us to select sets of records and define a starting position (e.g., the first 1000, and then the second set of up to 1000.)

.. code-block:: shell

   rows="&rows=1000"

.. code-block:: shell

   numResults=$(curl -s $jbase_url$issn$journal_works2014_2016$mailto | jq '.["message"]["total-results"]')
   echo $numResults

**Output:**

.. code-block:: shell

   1772

.. code-block:: shell

   for (( n = 0; n < numResults; n+=1000)); do
     curl -s $jbase_url$issn$journal_works2014_2016$rows$"&offset="$n$mailto | jq '.["message"]["items"][].DOI' >> dois_save2.txt
     sleep 1;
   done

.. code-block:: shell

   head dois_save2.txt

**Output:**

.. code-block:: shell

   "10.1186/1471-2105-15-84"
   "10.1186/1471-2105-15-94"
   "10.1186/1471-2105-16-s15-p11"
   "10.1186/s12859-016-1335-8"
   "10.1186/1471-2105-15-172"
   "10.1186/s12859-015-0538-8"
   "10.1186/1471-2105-15-106"
   "10.1186/1471-2105-16-s15-p20"
   "10.1186/1471-2105-15-s9-s12"
   "10.1186/s12859-016-1202-7"

.. code-block:: shell

   cat dois_save2.txt | wc -l

**Output:**

.. code-block:: shell

   1772

