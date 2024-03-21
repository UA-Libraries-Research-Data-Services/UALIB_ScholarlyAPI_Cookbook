PubMed API in Unix Shell
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Avery Fernandez and Vincent F. Scalfani

These recipe examples were tested on November 23, 2022 using GNOME Terminal in Ubuntu 18.04.

**NCBI Entrez Programming Utilities documentation:** https://www.ncbi.nlm.nih.gov/books/NBK25501/

**Please see NCBI’s Data Usage Policies and Disclaimers:** https://www.ncbi.nlm.nih.gov/home/about/policies/

.. note::
  
   This tutorial uses ``curl`` and ``jq`` for interacting with the PubChem API. You may also be interested in using the `NCBI EDirect command line program <https://www.ncbi.nlm.nih.gov/books/NBK179288/>`_. We have workshop materials for EDirect with PubMed in our `UALIB Workshops repository <https://github.com/ualibweb/UALIB_Workshops>`_.

Program requirements
=========================

In order to run this code, you will need to first install `curl`_, `jq`_, and `gnuplot`_. curl is used to request the data from the API, jq is used to parse the JSON data, and gnuplot is used to plot the data.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/
.. _gnuplot: http://www.gnuplot.info/

1. Basic PubMed API call
=============================

For calling individual articles and publications, we will need to use this API url:

.. code-block:: shell

   summary='https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&'

Request data from PubMed API
-------------------------------

The article we are requesting has PubMed ID: 27933103. **retmode** in the web API URL specifies the file format, in this example, we will use json.

.. code-block:: shell

   url=$summary$'id=27933103&retmode=json'
   data_call=$(curl -s $url)
   echo $data_call


**Output:**

.. code-block:: shell

   {"header":{"type":"esummary","version":"0.3"},"result":{"uids":["27933103"],"27933103":{"uid":"27933103","pubdate":"2016","epubdate":"2016 Nov 23","source":"J Cheminform","authors":[{"name":"Scalfani VF","authtype":"Author","clusterid":""},{"name":"Williams AJ","authtype":"Author","clusterid":""},{"name":"Tkachenko V","authtype":"Author","clusterid":""},{"name":"Karapetyan K","authtype":"Author","clusterid":""},{"name":"Pshenichnov A","authtype":"Author","clusterid":""},{"name":"Hanson RM","authtype":"Author","clusterid":""},{"name":"Liddie JM","authtype":"Author","clusterid":""},{"name":"Bara JE","authtype":"Author","clusterid":""}],"lastauthor":"Bara JE","title":"Programmatic conversion of crystal structures into 3D printable files using Jmol.","sorttitle":"programmatic conversion of crystal structures into 3d printable files using jmol","volume":"8","issue":"","pages":"66","lang":["eng"],"nlmuniqueid":"101516718","issn":"1758-2946","essn":"1758-2946","pubtype":["Journal Article"],"recordstatus":"PubMed","pubstatus":"258","articleids":[{"idtype":"pubmed","idtypen":1,"value":"27933103"},{"idtype":"pmc","idtypen":8,"value":"PMC5122160"},{"idtype":"pmcid","idtypen":5,"value":"pmc-id: PMC5122160;"},{"idtype":"doi","idtypen":3,"value":"10.1186/s13321-016-0181-z"},{"idtype":"pii","idtypen":4,"value":"181"}],"history":[{"pubstatus":"received","date":"2016/08/15 00:00"},{"pubstatus":"accepted","date":"2016/11/16 00:00"},{"pubstatus":"entrez","date":"2016/12/10 06:00"},{"pubstatus":"pubmed","date":"2016/12/10 06:00"},{"pubstatus":"medline","date":"2016/12/10 06:01"}],"references":[],"attributes":["Has Abstract"],"pmcrefcount":33,"fulljournalname":"Journal of cheminformatics","elocationid":"","doctype":"citation","srccontriblist":[],"booktitle":"","medium":"","edition":"","publisherlocation":"","publishername":"","srcdate":"","reportnumber":"","availablefromurl":"","locationlabel":"","doccontriblist":[],"docdate":"","bookname":"","chapter":"","sortpubdate":"2016/11/23 00:00","sortfirstauthor":"Scalfani VF","vernaculartitle":""}}}


.. note::

   The silent option ``(-s)`` for curl was used to hide the progress outputs.

Let's extract the authors of the paper:

.. code-block:: shell

   echo $data_call | jq '.["result"]["27933103"]["authors"][]["name"]'


**Output:**

.. code-block:: shell

   "Scalfani VF"
   "Williams AJ"
   "Tkachenko V"
   "Karapetyan K"
   "Pshenichnov A"
   "Hanson RM"
   "Liddie JM"
   "Bara JE"

2. Request data using a Loop
============================

First, create an array of PubMed IDs:

.. code-block:: shell

   idList=('34813985' '34813932' '34813684' '34813661' '34813372' '34813140' '34813072')

We can loop through the ``idList`` as follows:

.. code-block:: shell

   for id in "${idList[@]}"
   do
       echo $id
   done

**Output:**

.. code-block:: shell

   34813985
   34813932
   34813684
   34813661
   34813372
   34813140
   34813072

For storing data when looping through the IDs, we can use associative arrays. For example:

.. code-block:: shell

   declare -A myarray
   myarray["34813985"]="data1"
   myarray["34813932"]="data2"
   echo ${myarray["34813985"]}
   echo ${myarray["34813932"]}

**Output:**

.. code-block:: shell

   data1
   data2

For extracting specific data from the returned PubMed data, we will use jq with the ``--arg`` option, which allows us to pass data into the jq environment, such as an ID variable:


.. code-block:: shell

   data=$(curl -s "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&id=34813072&retmode=json")


.. code-block:: shell

   echo $data | jq '.["result"]["34813072"]'

or, alternatively:

.. code-block:: shell

   id="34813072"
   echo $data | jq --arg location "$id" '.["result"][$location]'

**Output:**

.. code-block:: shell

   {
     "uid": "34813072",
     "pubdate": "2022",
     "epubdate": "",
     "source": "Methods Mol Biol",
     "authors": [
       {
         "name": "Liu S",
         "authtype": "Author",
         "clusterid": ""
       },
       {
         "name": "Narancic T",
         "authtype": "Author",
          "clusterid": ""
       },
       {
         "name": "Davis C",
         "authtype": "Author",
         "clusterid": ""
       },
       {
         "name": "O'Connor KE",
         "authtype": "Author",
         "clusterid": ""
       }
     ],
     "lastauthor": "O'Connor KE",
     "title": "CRISPR-Cas9 Editing of the Synthesis of Biodegradable Polyesters Polyhydroxyalkanaotes (PHA) in Pseudomonas putida KT2440.",
     "sorttitle": "crispr cas9 editing of the synthesis of biodegradable polyesters polyhydroxyalkanaotes pha in pseudomonas putida kt2440",
     "volume": "2397",
     "issue": "",
     "pages": "341-358",
     "lang": [
       "eng"
     ],
     "nlmuniqueid": "9214969",
     "issn": "1064-3745",
     "essn": "1940-6029",
     "pubtype": [
       "Journal Article"
     ],
     "recordstatus": "PubMed - indexed for MEDLINE",
     "pubstatus": "4",
     "articleids": [
       {
         "idtype": "pubmed",
         "idtypen": 1,
         "value": "34813072"
       },
       {
         "idtype": "doi",
         "idtypen": 3,
         "value": "10.1007/978-1-0716-1826-4_17"
       }
     ],
     "history": [
       {
         "pubstatus": "entrez",
         "date": "2021/11/23 12:28"
       },
       {
         "pubstatus": "pubmed",
         "date": "2021/11/24 06:00"
       },
       {
         "pubstatus": "medline",
         "date": "2022/01/27 06:00"
       }
     ],
     "references": [],
     "attributes": [
       "Has Abstract"
     ],
     "pmcrefcount": "",
     "fulljournalname": "Methods in molecular biology (Clifton, N.J.)",
     "elocationid": "doi: 10.1007/978-1-0716-1826-4_17",
     "doctype": "citation",
     "srccontriblist": [],
     "booktitle": "",
     "medium": "",
     "edition": "",
     "publisherlocation": "",
     "publishername": "",
     "srcdate": "",
     "reportnumber": "",
     "availablefromurl": "",
     "locationlabel": "",
     "doccontriblist": [],
     "docdate": "",
     "bookname": "",
     "chapter": "",
     "sortpubdate": "2022/01/01 00:00",
     "sortfirstauthor": "Liu S",
     "vernaculartitle": ""
   }

Finally, we can now extract out specific elements, such as the journal title (source).

.. code-block:: shell

   id="34813072"
   echo $data | jq --arg location "$id" '.["result"][$location]["source"]'


**Output:**

.. code-block:: shell

   "Methods Mol Biol"

Now, combine these steps to loop through the list of IDs and extract the journal titles:

.. code-block:: shell

   idList=('34813985' '34813932' '34813684' '34813661' '34813372' '34813140' '34813072')
   declare -A multiPapers
   for ids in "${idList[@]}"
   do
     multiPapers[$ids]=$(curl -s $summary$'id='$ids$'&retmode=json')
     sleep 1
   done
   for ids in "${idList[@]}"
   do
     echo ${multiPapers[$ids]} | jq --arg location "$ids" '.result[$location]["source"]'
   done

**Output:**

.. code-block:: shell

   "Cell Calcium"
   "Methods"
   "FEBS J"
   "Dev Growth Differ"
   "CRISPR J"
   "Chembiochem"
   "Methods Mol Biol"

3. PubMed API Calls with Requests and Parameters
=========================================================

For searching for articles, we will need to use this API url:

.. code-block:: shell

   search='https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&'

When searching through articles, we are given a few ways of filtering the data. A list of all the available parameters for these requests can be found in the official NCBI documentation:

https://www.ncbi.nlm.nih.gov/books/NBK25499/

We can specify the database by putting ``db=<database>`` into the URL. We will be using the PubMed database. We can also use term to search data by adding ``term=<searchQuery>``. Just be sure to replace spaces with a + instead. We can, for example, use a query to search PubMed, such as “neuroscience intervention learning”:

.. code-block:: shell

   url=$search$"term=neuroscience+intervention+learning&retmode=json"
   data=$(curl -s $url)

We can also use the query to search for an author.

we will add ```[au]``` after the name to specify it is an author:

.. code-block:: shell

   url=$search$"term=Darwin[au]&retmode=json"
   data=$(curl -s $url)
   echo $data

**Output:**

.. code-block:: shell

   {"header":{"type":"esearch","version":"0.3"},"esearchresult":{"count":"603","retmax":"20","retstart":"0","idlist":["36374290","36370080","36363931","36342372","36315101","36254119","36164491","36102812","36100038","36098658","36082519","35993699","35916364","35834740","35732810","35719898","35714393","35513308","35507730","35475719"],"translationset":[],"querytranslation":"Darwin[Author]"}}


The number of returned IDs can be adjusted with the ``retmax`` parameter:


.. code-block:: shell

   url=$search$"term=Darwin[au]&retmax=30&retmode=json"
   data=$(curl -s $url)
   echo $data | jq '.["esearchresult"]["idlist"]'

**Output:**

.. code-block:: shell

   [
   "36374290",
   "36370080",
   "36363931",
   "36342372",
   "36315101",
   "36254119",
   "36164491",
   "36102812",
   "36100038",
   "36098658",
   "36082519",
   "35993699",
   "35916364",
   "35834740",
   "35732810",
   "35719898",
   "35714393",
   "35513308",
   "35507730",
   "35475719",
   "35414258",
   "35301788",
   "35293777",
   "35122809",
   "35100046",
   "35073334",
   "35038915",
   "35034540",
   "34927345",
   "34923869"
   ]

We can get the number of IDs after a bit of cleanup with ``tr`` and ``awk``:

.. code-block:: shell

   echo $data | jq '.["esearchresult"]["idlist"]' | tr -d ' "[],' | awk 'NF' | wc -l

**Output:**

.. code-block:: shell

   30 

We can sort results using **usehistory=y**. This allows us to store the data for it to be sorted in the same API call. The addition of **sort=pub+date** will sort IDs by the publishing date.

.. code-block:: shell

   url=$search$"term=Coral+Reefs&retmode=json&usehistory=y&sort=pub+date"
   data=$(curl -s $url)


We can also search based on publication type by adding **AND** into the search in the term: **term=<searchQuery>+AND+filter[filterType]**.

**[pt]** specifies that the filter type is the publication type. More filters can be found at: https://pubmed.ncbi.nlm.nih.gov/help/.


.. code-block:: shell

   url=$search$"term=stem+cells+AND+clinical+trial[pt]&retmode=json"
   data=$(curl $url)
   sleep 1
   echo $data


4. PubMed API Metadata Visualization
===========================================

Frequency of topic sortpubdate field
-----------------------------------------

Extracting the sortpubdate field for a “hydrogel drug” search results, limited to publication type clinical trials:

.. code-block:: shell

   search='https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&'
   url=$search$"term=hydrogel+drug+AND+clinical+trial[pt]&sort=pub+date&retmax=500&retmode=json"
   data=$(curl -s $url)

Get the length of results:

.. code-block:: shell

   echo $data | jq '.["esearchresult"]["idlist"] | length'

**Output:**

.. code-block:: shell

   299

Next, loop through each ID and get the sortpubdate field. Note that this sortpubdate field may not necessarily be equivalent to a publication date:

.. code-block:: shell

   declare -a idList
   for (( id = 0; id < $(echo $data | jq '.["esearchresult"]["idlist"] | length'); id++ ))
   do
     idList+=($(echo $data | jq ".esearchresult.idlist[$id]" | tr -d '"'))
   done

Get the length of the array:

.. code-block:: shell

   echo ${#idList[@]}

**Output:**

.. code-block:: shell

   299

Show the first 10 IDs

.. code-block:: shell

   echo ${idList[@]:0:10}

**Output:**

.. code-block:: shell

   36203046 36261491 35830550 34653384 35556170 35413602 35041809 34915741 34695615 35062896

Now, loop through each ID, get the sortpubdate and save to a file. Note, this will take a few minutes:

.. code-block:: shell

   summary='https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&'
   for ids in ${idList[@]}
   do
     url=$summary$"id="$ids$"&retmode=json"
     data=$(curl -s $url)
     sleep 1
     echo $data | jq --arg location "$ids" '.["result"][$location]["sortpubdate"]' >> pubDates.csv
   done

Finally, plot the data using gnuplot.  See the `gnuplot documentation`_ for more information about the smooth frequency histogram.

.. _gnuplot documentation: http://www.gnuplot.info/documentation.html

.. code-block:: shell

   gnuplot -e "set datafile separator ','; \
   set title 'sortpubdate';
   set term dumb;
   binwidth=2; \
   bin(val)=binwidth*floor(val/binwidth); \
   plot 'pubDates.csv' using (bin(column(1))):(1.0) smooth frequency with boxes notitle"

**Output:**

.. code-block:: shell

                                    sortpubdate                                 
                                                                                
   35 +---------------------------------------------------------------------+   
      |       +       +      +       +       +       +****  +       +       |   
      |                                               *  *                  |   
   30 |-+                                             *  ****             +-|   
      |                                   ****        *  *  *****           |   
   25 |-+                                 *  ****     *  *  *   *         +-|   
      |                                   *  *  ****  *  *  *   *  ****     |   
      |                                   *  *  *  ****  *  *   ****  *     |   
   20 |-+                                 *  *  *  *  *  *  *   *  *  *   +-|   
      |                                   *  *  *  *  *  *  *   *  *  *     |   
   15 |-+                                 *  *  *  *  *  *  *   *  *  *   +-|   
      |                                   *  *  *  *  *  *  *   *  *  **    |   
      |                                   *  *  *  *  *  *  *   *  *  **    |   
   10 |-+                                 *  *  *  *  *  *  *   *  *  **  +-|   
      |             ****  ****      *******  *  *  *  *  *  *   *  *  **    |   
    5 |-+           *  ****  ********  *  *  *  *  *  *  *  *   *  *  **  +-|   
      |             *  *  *  *  *   *  *  *  *  *  *  *  *  *   *  *  **    |   
      |************** +*  *  *  *   *+ *  *  *  *  * +*  *  *   *  *+ **    |   
    0 +---------------------------------------------------------------------+   
     1980    1985    1990   1995    2000    2005    2010   2015    2020    2025 


Frequency of publication for an author search
-----------------------------------------------

.. code-block:: shell

   search='https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?db=pubmed&'
   url=$search$"term=Reed+LK[au]&sort=pub+date&retmax=500&retmode=json"
   data=$(curl -s $url)

Next, create the list of IDs:

.. code-block:: shell

   declare -a idList
   for (( id = 0; id < $(echo $data | jq '.["esearchresult"]["idlist"] | length'); id++ ))
   do
     idList+=($(echo $data | jq ".esearchresult.idlist[$id]" | tr -d '"'))
   done

Get the length of the array:

.. code-block:: shell

   echo ${#idList[@]}

**Output:**

.. code-block:: shell

   55

Next, collect the sortpubdate for each ID:

.. code-block:: shell

   summary='https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&'
   for ids in ${idList[@]}
   do
     url=$summary$"id="$ids$"&retmode=json"
     data=$(curl -s $url)
     sleep 1
     echo $data | jq --arg location "$ids" '.["result"][$location]["sortpubdate"]' >> pubDates2.csv
   done

Plot the data:

.. code-block:: shell

   gnuplot -e "set datafile separator ','; \
   set title 'sortpubdate';
   set term dumb;
   binwidth=3; \
   bin(val)=binwidth*floor(val/binwidth); \
   plot 'pubDates2.csv' using (bin(column(1))):(1.0) smooth frequency with boxes notitle"


**Output:**

.. code-block:: shell

                                    sortpubdate                                 
                                                                                
   16 +---------------------------------------------------------------------+   
      |       +       +      +       +       +       +      +   *** +       |   
   14 |-+                                                       * *       +-|   
      |                                                         * ****      |   
   12 |-+                                                       * *  *    +-|   
      |                                                         * *  *      |   
      |                                                         * *  *      |   
   10 |-+                                                       * *  *    +-|   
      |                                                         * *  *      |   
    8 |-+                                                ***    * *  *    +-|   
      |                                                  * *    * *  *      |   
    6 |-+                                                * *    * *  *    +-|   
      |                                              ***** *  *** *  *      |   
    4 |-+                                            *   * *  * * *  **   +-|   
      |                                              *   * *  * * *  **     |   
      |                                              *   * *  * * *  **     |   
    2 |-+                              ***************   * *  * * *  **   +-|   
      |*********************************     +       *   * **** * * +**     |   
    0 +---------------------------------------------------------------------+   
     1940    1950    1960   1970    1980    1990    2000   2010    2020    2030 


