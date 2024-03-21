World Bank API in Unix Shell
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Avery Fernandez

See the `World Bank API documentation`_.

These recipe examples were tested on March 25, 2022 using GNOME Terminal in Ubuntu 18.04.

.. _World Bank API documentation: https://datahelpdesk.worldbank.org/knowledgebase/articles/889392-about-the-indicators-api-documentation

Program requirements
=========================

In order to run this code, you will need to first install `curl`_, `jq`_, and `gnuplot`_. curl is used to request the data from the API, jq is used to parse the JSON data, and gnuplot is used to plot the data.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/
.. _gnuplot: http://www.gnuplot.info/

1. Get list of country iso2Codes and names
===========================================

For obtaining data from the World Bank API, it is helpful to first obtain a list 
of country codes and names.

First, define the root World Bank API and the API URL for obtaining country code data:

.. code-block:: shell

   api="https://api.worldbank.org/v2/"; country_url=$api$"country/?format=json&per_page=500" 

.. note::
   
   The ``;`` allows us to enter multiple variable assignments on one line and the ``$`` allows for variable expansion.

Next, request and save the JSON data from the World Bank API:

.. code-block:: shell

   country_data=$(curl $country_url)

.. note::

   If you want to instead quickly view a formatted output of the data, try the silent option (``-s``) in curl piped to jq as follows: ``curl -s $country_url | jq '.'``

Get the length of the data:

.. code-block:: shell

   echo $country_data | jq '.[1] | length'

**Output:**

.. code-block:: shell

   299

View the first element:

.. code-block:: shell

   echo $country_data | jq '.[1][0]'

**Output:**

.. code-block:: shell

   {
     "id": "ABW",
     "iso2Code": "AW",
     "name": "Aruba",
     "region": {
       "id": "LCN",
       "iso2code": "ZJ",
       "value": "Latin America & Caribbean "
     },
     "adminregion": {
       "id": "",
       "iso2code": "",
       "value": ""
     },
     "incomeLevel": {
       "id": "HIC",
       "iso2code": "XD",
       "value": "High income"
     },
     "lendingType": {
       "id": "LNX",
       "iso2code": "XX",
       "value": "Not classified"
     },
     "capitalCity": "Oranjestad",
     "longitude": "-70.0167",
     "latitude": "12.5167"
   }

Next, extract out the iso2codes from the country_data

.. code-block:: shell

   declare -A country_iso2Code
   for (( i = 0; i < $(echo $country_data | jq '.[1] | length'); i++ ))
   do
     country=$(echo $country_data | jq ".[1][$i].name");
     iso=$(echo $country_data | jq ".[1][$i].iso2Code");
     echo $iso$" : "$country;
     country_iso2Code["$iso"]="$country";
   done;

**Output:**

.. code-block:: shell

   "AW" : "Aruba"
   "ZH" : "Africa Eastern and Southern"
   "AF" : "Afghanistan"
   "A9" : "Africa"
   "ZI" : "Africa Western and Central"
   "AO" : "Angola"
   "AL" : "Albania"
   "AD" : "Andorra"
   "1A" : "Arab World"
   "AE" : "United Arab Emirates"
   ...
   ...
   ...

.. note::

  ``declare -A`` creates an associative array; ``country_iso2Code["$iso"]="$country"`` stores the iso variable and corresponding country name. 

Since we saved the iso2codes and country names in the associative array, ``country_iso2code``, it is also possible to loop through and display the data as follows:

.. code-block:: shell

   for isos in "${!country_iso2Code[@]}"; do
     echo "$isos - ${country_iso2Code[$isos]}";
   done

*Output not shown here*

.. note::

   ``!`` selects individual indices of the associative array; ``@`` specifies all elements in the array.


2. Compile a Custom Indicator Dataset
======================================

There are many available indicators: https://data.worldbank.org/indicator

We will select three indicators for this example:

1. Scientific and Technical Journal Article Data = `IP.JRN.ARTC.SC`_

2. Patent Applications, residents = `IP.PAT.RESD`_

3. GDP per capita (current US$) Code = `NY.GDP.PCAP.CD`_

Note that these three selected indicators have a `CC-BY 4.0 license`_.

We will compile this indicator data for the United States (US) and United Kingdom (GB).

.. _IP.JRN.ARTC.SC: https://data.worldbank.org/indicator/IP.JRN.ARTC.SC?view=chart
.. _IP.PAT.RESD: https://data.worldbank.org/indicator/IP.PAT.RESD?view=chart
.. _NY.GDP.PCAP.CD: https://data.worldbank.org/indicator/NY.GDP.PCAP.CD?view=chart
.. _CC-BY 4.0 license: https://datacatalog.worldbank.org/public-licenses#cc-by

.. code-block:: shell

   indicators=('IP.JRN.ARTC.SC' 'IP.PAT.RESD' 'NY.GDP.PCAP.CD')

Generate the web API URLs we need for U.S. and U.K. and retrieve the data.

.. code-block:: shell

   api="https://api.worldbank.org/v2/"

.. code-block:: shell

   declare -A US_indicator_data
   for indic in "${indicators[@]}"
   do
       US_indicator_data[$indic]=$(curl $api$"country/US/indicator/"$indic$"/?format=json&per_page=500")
       sleep 1;
   done

.. code-block:: shell

   declare -A UK_indicator_data
   for indic in "${indicators[@]}"
   do
       UK_indicator_data[$indic]=$(curl $api$"country/GB/indicator/"$indic$"/?format=json&per_page=500")
       sleep 1;
   done

Now we need to extract the data and compile for analysis.

column 1: year

column 2: Scientific and Technical Journal Article Data = ``IP.JRN.ARTC.SC``

column 3: Patent Applications, residents = ``IP.PAT.RESD``

column 4: GDP per capita (current US$) Code = ``NY.GDP.PCAP.CD``

U.S. data extraction:

.. code-block:: shell

   declare -A US_data_JRN
   declare -A US_data_PAT
   declare -A US_data_NY
   for (( years = 0; years < $(echo ${US_indicator_data['IP.JRN.ARTC.SC']} | jq '.[1] | length'); years++ ))
   do
     year=$(echo ${US_indicator_data['IP.JRN.ARTC.SC']} | jq ".[1][$years].date" | tr -d '"')
     US_data_JRN[$year]=$(echo ${US_indicator_data['IP.JRN.ARTC.SC']} | jq ".[1][$years].value")
     US_data_PAT[$year]=$(echo ${US_indicator_data['IP.PAT.RESD']} | jq ".[1][$years].value")
     US_data_NY[$year]=$(echo ${US_indicator_data['NY.GDP.PCAP.CD']} | jq ".[1][$years].value")
   done;
   echo $'"year","IP.JRN.ARTC.SC","IP.PAT.RESD","NY.GDP.PCAP.CD"' >> US_data.csv
   for years in "${!US_data_JRN[@]}"; do
     echo $years$","${US_data_JRN[$years]}$","${US_data_PAT[$years]}$","${US_data_NY[$years]} | sed 's/null/NaN/g' >> US_data.csv
   done

.. note::

   ``sed 's/null/NaN/g'`` is used to replace missing data with NaN.

.. code-block:: shell

   head US_data.csv

**Output:**

.. code-block:: shell

   "year","IP.JRN.ARTC.SC","IP.PAT.RESD","NY.GDP.PCAP.CD"
   1979,NaN,NaN,11674.1818666548
   1978,NaN,NaN,10564.9482220275
   1973,NaN,NaN,6726.35895596695
   1972,NaN,NaN,6094.01798986165
   1971,NaN,NaN,5609.38259952519
   1970,NaN,NaN,5234.2966662115
   1977,NaN,NaN,9452.57651914511
   1976,NaN,NaN,8592.25353727612
   1975,NaN,NaN,7801.45666356443

U.K. Data extraction:

column 1: year

column 2: Scientific and Technical Journal Article Data = ``IP.JRN.ARTC.SC``

column 3: Patent Applications, residents = ``IP.PAT.RESD``

column 4: GDP per capita (current US$) Code = ``NY.GDP.PCAP.CD``

.. code-block:: shell

   declare -A UK_data_JRN
   declare -A UK_data_PAT
   declare -A UK_data_NY
   for (( years = 0; years < $(echo ${UK_indicator_data['IP.JRN.ARTC.SC']} | jq '.[1] | length'); years++ ))
   do
     year=$(echo ${UK_indicator_data['IP.JRN.ARTC.SC']} | jq ".[1][$years].date" | tr -d '"')
     UK_data_JRN[$year]=$(echo ${UK_indicator_data['IP.JRN.ARTC.SC']} | jq ".[1][$years].value")
     UK_data_PAT[$year]=$(echo ${UK_indicator_data['IP.PAT.RESD']} | jq ".[1][$years].value")
     UK_data_NY[$year]=$(echo ${UK_indicator_data['NY.GDP.PCAP.CD']} | jq ".[1][$years].value")
   done;
   echo $'"year","IP.JRN.ARTC.SC","IP.PAT.RESD","NY.GDP.PCAP.CD"' >> UK_data.csv
   for years in "${!UK_data_JRN[@]}"; do
     echo "$years"$","${UK_data_JRN[$years]}$","${UK_data_PAT[$years]}$","${UK_data_NY[$years]} | sed 's/null/NaN/g' >> UK_data.csv
   done


.. note::

   ``sed 's/null/NaN/g'`` is used to replace missing data with NaN.

.. code-block:: shell

   tail UK_data.csv

**Output:**

.. code-block:: shell

   2003,75564.08,20426,34487.4675722539
   1984,NaN,19093,8179.19444064991
   2000,77244.9,22050,28223.0675706515
   1985,NaN,19672,8652.21654247593
   2001,73779.92,21423,27806.4488245133
   1988,NaN,20536,15987.1680775688
   1989,NaN,19732,16239.2821960944
   2008,91357.74,16523,47549.3486286006
   2009,93803.37,15985,38952.2110262455
   2020,NaN,NaN,41059.1688090547

3. Plot Indicator data
=======================

Create a line plot of US/UK Number of Scientific and Technical Journal Articles and Patents by year.

.. code-block:: shell

   awk -F',' '{ print $1","$2+$3","$4; }' US_data.csv | sort -t"," -k1n,1 > US_sorted.csv
   awk -F',' '{ print $1","$2+$3","$4; }' UK_data.csv | sort -t"," -k1n,1 > UK_sorted.csv
   sed -i "1s/.*/'year','US Articles and Patents','US GDP'/" US_sorted.csv
   sed -i "1s/.*/'year','UK Articles and Patents','UK GDP'/" UK_sorted.csv

.. note::

   ``awk`` is combining the second column and third column into a single column; ``sort`` is to sort the data by the year; ``sed`` is to change the first row to accurately name the columns.

.. code-block:: shell

   head US_sorted.csv

**Output:**

.. code-block:: shell

   'year','US Articles and Patents','US GDP'
   1960,nan,3007.12344537862
   1961,nan,3066.56286916615
   1962,nan,3243.84307754988
   1963,nan,3374.51517105082
   1964,nan,3573.94118474743
   1965,nan,3827.52710972039
   1966,nan,4146.31664631665
   1967,nan,4336.42658722171
   1968,nan,4695.92339043178

Plot the data as an ascii plot:

.. code-block:: shell

   gnuplot -e "set datafile separator ','; \
   set datafile missing NaN; \
   set key outside; \
   set key autotitle columnhead; \
   set term dumb size 130, 30; \
   set xrange [2000:2018]; \
   set ylabel 'First Y Units'; \
   set xlabel 'Time'; \
   set title 'US and UK data'; \
   set y2tics nomirror; \
   set ytics nomirror; \
   set size 1,1; \
   plot 'US_sorted.csv' using 1:2 with lines axis x1y1, '' using 1:3 with lines axis x1y2, \
   'UK_sorted.csv' using 1:2 with lines axis x1y1, '' using 1:3 with lines axis x1y2"

**Output:**

.. code-block:: shell

                                             US and UK data                                                                      
                                                                                                                                  
               800000 +--------------------------------------------------------------+ 65000                                      
                      |      +      +      +      +      +      +      +      +      |         'US Articles and Patents' *******  
                      |                                           *****************##|                          'US GDP' #######  
               700000 |-+                                      ***              ###+-| 60000   'UK Articles and Patents' $$$$$$$  
                      |                                 *******              ###     |                          'UK GDP' %%%%%%%  
                      |                      ***********                 ####        |                                            
               600000 |-+             *******                         ###          +-| 55000                                      
                      |            ***                            ####               |                                            
                      |        ****            %%%             ###                   |                                            
               500000 |********               %            ####                    +-| 50000                                      
                      |                      %####%########            %%%%          |                                            
               400000 |-+                 ##%      %                 %%    %       +-| 45000                                      
                      |               ####%%        %             %%%       %        |                                            
                      |             ##%%%%          %       %%%%%%           %     %%|                                            
               300000 |-+         ##%%               %    %%                  %%%%%+-| 40000                                      
                      |        ### %                  %%%%                           |                                            
                      |########    %                                                 |                                            
               200000 |-+         %                                                +-| 35000                                      
                      |         %%                                                   |                                            
                      |       %%                                                     |                                            
               100000 |$$$$$%%$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$| 30000                                      
                      |%%%%%                                                         |                                            
                      |      +      +      +      +      +      +      +      +      |                                            
                    0 +--------------------------------------------------------------+ 25000                                      
                     2000   2002   2004   2006   2008   2010   2012   2014   2016   2018                                          
                                                   Time                                                



