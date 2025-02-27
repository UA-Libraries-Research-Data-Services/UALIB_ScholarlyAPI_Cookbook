Chronicling America API in Bash
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Avery Fernandez

**LOC Chronicling America API Documentation:** https://chroniclingamerica.loc.gov/about/api/

These recipe examples were tested on December 7, 2022 using GNOME Terminal in Ubuntu 18.04.

**Attribution:** We thank **Professor Jessica Kincaid** (UA Libraries, Hoole Special Collections)
for the use-cases. All data was collected from the Library of Congress, Chronicling America: Historic
American Newspapers site, using the API.

Note that the data from the *Alabama state intelligencer*, *The age-herald*, and the 
*Birmingham age-herald* were contributed to Chronicling America by The University of 
Alabama Libraries: https://chroniclingamerica.loc.gov/awardees/au/

Program requirements
=========================

In order to run this code, you will need to first install `curl`_, `jq`_, and `gnuplot`_.
curl is used to request the data from the API, jq is used to parse the JSON data,
and gnuplot is used to plot the data.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/
.. _gnuplot: http://www.gnuplot.info/

1. Basic API request
=============================

The Chronicling America API identifies newspapers and other records using LCCNs.
We can query the API once we have the LCCN for the newspaper and even ask for
particular issues and editions. For example, the following link lists newspapers
published in the state of Alabama, from which the LCCN can be obtained:
https://chroniclingamerica.loc.gov/newspapers/?state=Alabama

Here is an example with the Alabama State Intelligencer:

.. code-block:: shell
   
   api="https://chroniclingamerica.loc.gov/"
   request=$(curl -s $api$"lccn/sn84023600.json")
   echo $request | jq '.'

**Output:**

.. code-block:: shell

   {
   "place_of_publication": "Tuskaloosa [sic], Ala.",
   "lccn": "sn84023600",
   "start_year": "183?",
   "place": [
      "Alabama--Tuscaloosa--Tuscaloosa"
   ],
   "name": "Alabama State intelligencer. [volume]",
   "publisher": "T.M. Bradford",
   "url": "https://chroniclingamerica.loc.gov/lccn/sn84023600.json",
   "end_year": "18??",
   "issues": [],
   "subject": []
   }

Indexing into the json output allows data to be extracted using key names as demonstrated below:

.. code-block:: shell

   echo $request | jq '.["name"]'

**Output:**

.. code-block:: shell

   "Alabama State intelligencer. [volume]"

.. code-block:: shell

   echo $request | jq '.["publisher"]'

**Output:**

.. code-block:: shell

   "T.M. Bradford"

Moving on to another publication, we can get the 182nd page (seq-182) of the Evening Star
newspaper published on November 19, 1961.

.. code-block:: shell

   request=$(curl -s $api$"lccn/sn83045462/1961-11-19/ed-1/seq-182.json")
   echo $request | jq '.'

**Output:**

.. code-block:: shell

   {
   "jp2": "https://chroniclingamerica.loc.gov/lccn/sn83045462/1961-11-19/ed-1/seq-182.jp2",
   "sequence": 182,
   "text": "https://chroniclingamerica.loc.gov/lccn/sn83045462/1961-11-19/ed-1/seq-182/ocr.txt",
   "title": {
      "url": "https://chroniclingamerica.loc.gov/lccn/sn83045462.json",
      "name": "Evening star. [volume]"
   },
   "pdf": "https://chroniclingamerica.loc.gov/lccn/sn83045462/1961-11-19/ed-1/seq-182.pdf",
   "ocr": "https://chroniclingamerica.loc.gov/lccn/sn83045462/1961-11-19/ed-1/seq-182/ocr.xml",
   "issue": {
      "url": "https://chroniclingamerica.loc.gov/lccn/sn83045462/1961-11-19/ed-1.json",
      "date_issued": "1961-11-19"
   }
   }

Next, extract the URL for the PDF file and open it from the terminal. The `-L`
option in curl allows for redirection to load the PDF:

.. code-block:: shell

   url=$(echo $request | jq '.["pdf"]' | tr -d '"')
   curl $url -L --output outfile.pdf
   xdg-open outfile.pdf

2. Frequency of “University of Alabama” mentions
=====================================================

The URL below limits to searching newspapers in the state of Alabama and provides 500 results of 
“University of Alabama” mentions. Note that phrases can be searched by putting them inside parentheses for the query.

.. code-block:: shell

   api="https://chroniclingamerica.loc.gov/"
   request=$(curl -s $api$"search/pages/results/?state=Alabama&proxtext=(University%20of%20Alabama)&rows=500&format=json" | jq .'["items"]')

Get the length of returned data:

.. code-block:: shell

   length=$(echo "$request" | jq '. | length')
   echo "$length"

**Output:**

.. code-block:: shell

   500

Next, display the first record

.. code-block:: shell

   echo "$request" | jq '.[0]'

**Output:**

.. code-block:: shell

   {
   "sequence": 48,
   "county": [
      "Jefferson"
   ],
   "edition": null,
   "frequency": "Daily",
   "id": "/lccn/sn85038485/1924-07-13/ed-1/seq-48/",
   "subject": [
      "Alabama--Birmingham.--fast--(OCoLC)fst01204958",
      "Birmingham (Ala.)--Newspapers."
   ],
   "city": [
      "Birmingham"
   ],
   "date": "19240713",
   "title": "The Birmingham age-herald. [volume]",
   "end_year": 1950,
   "note": [
      "Also issued on microfilm from Bell & Howell, Micro Photo Div.; the Library of Congress, Photoduplication Service.",
      "Also published in a weekly ed.",
      "Archived issues are available in digital format from the Library of Congress Chronicling America online collection.",
      "Publication suspended with July 12, 1945 issue due to a printers' strike; resumed publication with Aug. 17, 1945 issue."
   ],
   "state": [
      "Alabama"
   ],
   "section_label": "Tuscaloosa Section",
   "type": "page",
   "place_of_publication": "Birmingham, Ala.",
   "start_year": 1902,
   "edition_label": "",
   "publisher": "Age-Herald Co.",
   "language": [
      "English"
   ],
   "alt_title": [
      "Age-herald",
      "Birmingham news, the Birmingham age-herald"
   ],
   "lccn": "sn85038485",
   "country": "Alabama",
   "ocr_eng": "canes at the University .of Alabama\nMORGAN HALL -\nSMITH HALL\n' hi i ..mil w i 1»..IIgylUjAiU. '. n\njjiIi\n(ARCHITECTS* MODEL)\nCOMER. HALli\nMINING\n••tSgSB?\n* i v' y -4\n■Si ' 3>\nA GLIMP9E OF FRATERNITY ROW\nTHE GYMNASIUM\nTuscaloosa, Alabama\nADV.",
   "batch": "au_foster_ver01",
   "title_normal": "birmingham age-herald.",
   "url": "https://chroniclingamerica.loc.gov/lccn/sn85038485/1924-07-13/ed-1/seq-48.json",
   "place": [
      "Alabama--Jefferson--Birmingham"
   ],
   "page": "8"
   }

Loop through the records and extract the dates:

.. code-block:: shell

   declare -a dates
   for (( i = 0 ; i < "$length" ; i++ ));
   do
     dates+=("$(echo "$request" | jq ".[$i].date" | tr -d '"')")
   done

Check the length of dates:

.. code-block:: shell

   echo "${#dates[@]}"

**Output:**

.. code-block:: shell

   500

Display the first 10 dates:

.. code-block:: shell

   echo "${dates[@]:0:10}"

**Output:**

.. code-block:: shell

   19240713 19180818 19240224 19160806 19130618 19240217 19140602 19120714 19220917 19170513

We'll do a bit of data transformation on the dates before plotting:

.. code-block:: shell

   declare -a formattedDates
   for date in "${dates[@]}";
   do
     year=$(echo "$date" | cut -c1-4)
     month=$(echo "$date" | cut -c5-6)
     day=$(echo "$date" | cut -c7-8)
     formatted=$year$"/"$month$"/"$day
     echo $'"'"$formatted"$'"' >> dates.csv
     formattedDates+=("$formatted")
   done
   echo "${formattedDates[@]:0:10}"

**Output:**

.. code-block:: shell

   1924/07/13 1918/08/18 1924/02/24 1916/08/06 1913/06/18 1924/02/17 1914/06/02 1912/07/14 1922/09/17 1917/05/13

Next, plot the data using gnuplot. 
See the `gnuplot documentation`_ for more information about the smooth frequency histogram.

.. _gnuplot documentation: http://www.gnuplot.info/documentation.html

.. code-block:: shell

   head dates.csv

**Output:**

.. code-block:: shell

   "1924/07/13"
   "1918/08/18"
   "1924/02/24"
   "1916/08/06"
   "1913/06/18"
   "1924/02/17"
   "1914/06/02"
   "1912/07/14"
   "1922/09/17"
   "1917/05/13"

.. code-block:: shell

   cat graph.gnuplot

**Output:**

.. code-block:: Shell

   set datafile separator ','
   binwidth=4
   set term dumb
   bin(x,width)=width*floor(x/width)
   plot 'dates.csv' using (bin($1,binwidth)):(1.0) smooth freq with boxes notitle

.. code-block:: shell

   gnuplot -p graph.gnuplot

**Output:**

.. code-block:: Shell

   120 +--------------------------------------------------------------------+   
      |         +         +         +        +         +         +         |   
      |                                           ***                      |   
   100 |-+                                         * * ***                +-|   
      |                                           * * * *                  |   
      |                                           * * * *                  |   
      |                                           * * * *                  |   
   80 |-+                                         * * * *                +-|   
      |                                           * *** *                  |   
      |                                       *** * * * *                  |   
   60 |-+                                     * *** * * *                +-|   
      |                                       * * * * * *                  |   
      |                                     *** * * * * *                  |   
   40 |-+                                   * * * * * * *                +-|   
      |                                     * * * * * * *                  |   
      |                                     * * * * * * *                  |   
      |                                     * * * * * * **********         |   
   20 |-+                                   * * * * * * *        *       +-|   
      |                                   *** * * * * * *        *         |   
      |         +         ***************** *+* * * * *+*        *         |   
    0 +--------------------------------------------------------------------+   
     1820      1840      1860      1880     1900      1920      1940      1960 


3. Industrialization keywords frequency in the Birmingham Age-Herald
==========================================================================

We will try to obtain the frequency of “Iron” on the front pages of the Birmingham Age- herald newspapers from
the year 1903 to 1949 (limited to first 500 rows for testing here).

.. code-block:: shell

   api="https://chroniclingamerica.loc.gov/"
   request=$(curl "$api"$"search/pages/results/?state=Alabama&lccn=sn85038485&dateFilterType=yearRange&date1=1903&date2=1949&sequence=1&andtext=Iron&rows=500&searchType=advanced&format=json" | jq '.["items"]')
   
.. code-block:: shell

   echo "$request" | jq '. | length'

**Output:**

.. code-block:: shell

   500

Extract the dates and do some formatting as shown before:

.. code-block:: shell

   declare -a dates
   length=$(echo "$request" | jq '. | length')
   for (( i = 0 ; i < "$length" ; i++ ));
   do
     dates+=("$(echo "$request" | jq ".[$i].date" | tr -d '"')")
   done

   declare -a formattedDates
   for date in "${dates[@]}";
   do
     year=$(echo "$date" | cut -c1-4)
     month=$(echo "$date" | cut -c5-6)
     day=$(echo "$date" | cut -c7-8)
     formatted=$year$"/"$month$"/"$day
     echo $'"'"$formatted"$'"' >> dates2.csv
     formattedDates+=("$formatted")
   done

Check to make sure we have 500 dates:

.. code-block:: shell

   cat dates2.csv | wc -l

**Output:**

.. code-block:: shell

   500

And plot the data:

.. code-block:: shell

   cat graph.gnuplot 

**Output:**

.. code-block:: shell

   set datafile separator ','
   binwidth=2
   set term dumb
   bin(x,width)=width*floor(x/width)
   plot 'dates2.csv' using (bin($1,binwidth)):(1.0) smooth freq with boxes notitle

.. code-block:: shell

   gnuplot -p graph.gnuplot

**Output:**

.. code-block:: shell

   90 +---------------------------------------------------------------------+   
      |             +             +             +             +             |   
   80 |-+                      *******                                    +-|   
      |                        *     *                                      |   
   70 |-+     *******          *     *                                    +-|   
      |       *     *          *     *                                      |   
      |       *     ************     *                                      |   
   60 |-+     *     *     *    *     ******                               +-|   
      |       *     *     *    *     *    *                                 |   
   50 |-+     *     *     *    *     *    *                      ******   +-|   
      |       *     *     *    *     *    *                      *    *     |   
   40 |-+     *     *     *    *     *    *                      *    *   +-|   
      |       *     *     *    *     *    *                      *    *     |   
   30 |-+     *     *     *    *     *    *                      *    *   +-|   
      |  ******     *     *    *     *    *     *******          *    *     |   
      |  *    *     *     *    *     *    *     *     *    *******    *     |   
   20 |-+*    *     *     *    *     *    *******     *    *     *    *   +-|   
      |  *    *     *     *    *     *    *     *     *    *     *    *     |   
   10 |-+*    *     *     *    *     *    *     *     ******     *    ****+-|   
      |  *    *     *     *    *  +  *    *     *     *    *  +  *    *  *  |   
      0 +---------------------------------------------------------------------+   
      1900          1905          1910          1915          1920          1925 


