U.S. Census API in Bash
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

by Avery Fernandez

**U.S. Census API documentation:** https://www.census.gov/data/developers/about.html

**U.S. Census Data Discovery Tool:** https://api.census.gov/data.html

These recipe examples were tested on December 13, 2022 using GNOME Terminal in Ubuntu 18.04.

See also the U.S. `Census API Terms of Service`_

.. _Census API Terms of Service: https://www.census.gov/data/developers/about/terms-of-service.html

**Attribution:** This tutorial uses the Census Bureau Data API but is not endorsed
or certified by the Census Bureau.

Setup
=====

Program requirements
--------------------

In order to run this code, you will need to first install `curl`_, `jq`_, and `gnuplot`_.
curl is used to request the data from the API, jq is used to parse the JSON data,
and gnuplot is used to plot the data.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/
.. _gnuplot: http://www.gnuplot.info/

API Key Information
-------------------

While an API key is not required to use the U.S. Census Data API, you may consider
registering for an API key as the API is limited to 500 calls a day without a key.
Sign up can be found here: https://api.census.gov/data/key_signup.html

If you want to add in your API Key, save the API key to a text file named "apiKey" and
import it using the following code:

.. code-block:: shell

   key=$(cat "apikey.txt");

Note that this tutorial does not use an API key.

1. Get population estimates of countries by state
=======================================================

Note: includes Washington, D.C. and Puerto Rico

We will use the population estimates from the 2019 dataset:
https://api.census.gov/data/2019/pep/population/examples.html

First, obtain a list of the state names and IDs from the API:

.. code-block:: shell

   api='https://api.census.gov/data/'
   state_ids=$(curl -s $api$"2019/pep/population?get=NAME&for=state:*"$key)
   echo $state_ids

**Output:**

.. code-block:: shell

   [["NAME","state"], ["Alabama","01"], ["Alaska","02"], ["Arizona","04"], ["Arkansas","05"], ["California","06"], ["Colorado","08"], ["Delaware","10"], ["District of Columbia","11"], ["Connecticut","09"], ["Florida","12"], ["Georgia","13"], ["Idaho","16"], ["Hawaii","15"], ["Illinois","17"], ["Indiana","18"], ["Iowa","19"], ["Kansas","20"], ["Kentucky","21"], ["Louisiana","22"], ["Maine","23"], ["Maryland","24"], ["Massachusetts","25"], ["Michigan","26"], ["Minnesota","27"], ["Mississippi","28"], ["Missouri","29"], ["Montana","30"], ["Nebraska","31"], ["Nevada","32"], ["New Hampshire","33"], ["New Jersey","34"], ["New Mexico","35"], ["New York","36"], ["North Carolina","37"], ["North Dakota","38"], ["Ohio","39"], ["Oklahoma","40"], ["Oregon","41"], ["Pennsylvania","42"], ["Rhode Island","44"], ["South Carolina","45"], ["South Dakota","46"], ["Tennessee","47"], ["Texas","48"], ["Vermont","50"], ["Utah","49"], ["Virginia","51"], ["Washington","53"], ["West Virginia","54"], ["Wisconsin","55"], ["Wyoming","56"], ["Puerto Rico","72"]]

Get the length:

.. code-block:: shell

   echo $state_ids | jq '. | length'

**Output:**

.. code-block:: shell

   53

Next, loop through each state and obtain population data:

.. code-block:: shell

   for (( i = 1; i < $(echo $state_ids | jq '. | length'); i++ ))
   do
      state=$(echo $state_ids | jq ".[$i][0]" | tr -d '"')
      stateID=$(echo $state_ids | jq ".[$i][1]" | tr -d '"')
      request=$(curl -s $api$"2019/pep/population?get=NAME,POP&for=county:*&in=state:"$stateID$key)
      sleep 1;
      for (( j = 1; j < $(echo $request | jq '. | length'); j++ ))
      do
         county=$(echo $request | jq ".[$j][0]" | tr -d '"' | cut -f1 -d",")
         population=$(echo $request | jq ".[$j][1]" | tr -d '"')
         echo $state$","$county$","$population >> state_populations.csv
      done
   done

View the first 25 lines

.. code-block:: shell

   head -n25 state_populations.csv

**Output:**

.. code-block:: shell

   Alabama,St. Clair County,89512
   Alabama,Cullman County,83768
   Alabama,Houston County,105882
   Alabama,Tuscaloosa County,209355
   Alabama,Coffee County,52342
   Alabama,Chilton County,44428
   Alabama,Coosa County,10663
   Alabama,Etowah County,102268
   Alabama,Lamar County,13805
   Alabama,Butler County,19448
   Alabama,Walker County,63521
   Alabama,Greene County,8111
   Alabama,Bullock County,10101
   Alabama,Chambers County,33254
   Alabama,Monroe County,20733
   Alabama,Lawrence County,32924
   Alabama,Lee County,164542
   Alabama,Marion County,29709
   Alabama,Pickens County,19930
   Alabama,Sumter County,12427
   Alabama,Jefferson County,658573
   Alabama,Choctaw County,12589
   Alabama,Franklin County,31362
   Alabama,Marengo County,18863
   Alabama,Russell County,57961

2. Get population estimates over a range of years
====================================================

We can use similar code as before, but now loop through different population estimate datasets by year.
Here are the specific APIs used:

Vintage 2015 Population Estimates: https://api.census.gov/data/2015/pep/population/examples.html

Vintage 2016 Population Estimates: https://api.census.gov/data/2016/pep/population/examples.html

Vintage 2017 Population Estimates: https://api.census.gov/data/2017/pep/population/examples.html

Note: includes Washington, D.C. and Puerto Rico.

.. code-block:: shell

   for year in {2015..2018}
   do
     for (( i = 1; i < $(echo $state_ids | jq '. | length'); i++ ))
     do
       state=$(echo $state_ids | jq ".[$i][0]" | tr -d '"')
       stateID=$(echo $state_ids | jq ".[$i][1]" | tr -d '"')
       request=$(curl -s $api$year$"/pep/population?get=GEONAME,POP&for=county:*&in=state:"$stateID$key)
       sleep 1;
       for (( j = 1; j < $(echo $request | jq '. | length'); j++ ))
       do
         county=$(echo $request | jq ".[$j][0]" | tr -d '"' | cut -f1 -d",")
         population=$(echo $request | jq ".[$j][1]" | tr -d '"')
         echo $year","$state$","$county$","$population >> state_populations_over_years.csv
       done
     done
   done

View the first 25 lines

.. code-block:: shell

   head -n25 state_populations_over_years.csv

**Output:**

.. code-block:: shell

   2015,Alabama,Baldwin County,203709
   2015,Alabama,Barbour County,26489
   2015,Alabama,Bibb County,22583
   2015,Alabama,Blount County,57673
   2015,Alabama,Bullock County,10696
   2015,Alabama,Butler County,20154
   2015,Alabama,Calhoun County,115620
   2015,Alabama,Chambers County,34123
   2015,Alabama,Cherokee County,25859
   2015,Alabama,Chilton County,43943
   2015,Alabama,Choctaw County,13170
   2015,Alabama,Clarke County,24675
   2015,Alabama,Clay County,13555
   2015,Alabama,Cleburne County,15018
   2015,Alabama,Coffee County,51211
   2015,Alabama,Colbert County,54354
   2015,Alabama,Conecuh County,12672
   2015,Alabama,Coosa County,10724
   2015,Alabama,Covington County,37835
   2015,Alabama,Autauga County,55347
   2015,Alabama,Lawrence County,33115
   2015,Alabama,Lee County,156993
   2015,Alabama,Limestone County,91663
   2015,Alabama,Lowndes County,10458
   2015,Alabama,Macon County,19105

3. Plot population change
================================

This data is based off the 2021 Population Estimates dataset:

https://api.census.gov/data/2021/pep/population/variables.html

The percentage change in population is from July 1, 2020 to July 1, 2021 for states 
(includes Washington, D.C. and Puerto Rico)

.. code-block:: shell

   request=$(curl -s $api$"2021/pep/population?get=NAME,POP_2021,DENSITY_2021,PPOPCHG_2021&for=state:*"$key)
   for (( i = 1; i < $(echo $request | jq '. | length'); i++ ))
   do
     state=$(echo $request | jq ".[$i][0]" | tr -d '"')
     population=$(echo $request | jq ".[$i][1]" | tr -d '"')
     density=$(echo $request | jq ".[$i][2]" | tr -d '"')
     populationChange=$(echo $request | jq ".[$i][3]" | tr -d '"')
     echo ${state}$","$population$","$density$","$populationChange >> state_change.csv
   done

Sort the data:

.. code-block:: shell

   sort state_change.csv > state_change.sorted

Create an associative array that replaces state name with abbreviation:

.. code-block:: shell

   declare -A abbreviation=( [Puerto Rico]=Pr [Alabama]=Al [Alaska]=Ak [Arizona]=Az [Arkansas]=Ar [California]=Ca [Colorado]=Co [Connecticut]=Ct [Delaware]=De [District of Columbia]=Dc [Florida]=Fl [Georgia]=Ga [Hawaii]=Hi [Idaho]=Id [Illinois]=Il [Indiana]=In [Iowa]=Ia [Kansas]=Ks [Kentucky]=Ky [Louisiana]=La [Maine]=Me [Maryland]=Md [Massachusetts]=Ma [Michigan]=Mi [Minnesota]=Mn [Mississippi]=Ms [Missouri]=Mo [Montana]=Mt [Nebraska]=Ne [Nevada]=Nv [New Hampshire]=Nh [New Jersey]=Nj [New Mexico]=Nm [New York]=Ny [North Carolina]=Nc [North Dakota]=Nd [Ohio]=Oh [Oklahoma]=Ok [Oregon]=Or [Pennsylvania]=Pa [Rhode Island]=Ri [South Carolina]=Sc [South Dakota]=Sd [Tennessee]=Tn [Texas]=Tx [Utah]=Ut [Vermont]=Vt [Virginia]=Va [Washington]=Wa [West Virginia]=Wv [Wisconsin]=Wi [Wyoming]=Wy )

Next, select only the population change and state abbreviation:

.. code-block:: shell
    
   while IFS=, read -r field1 field2 field3 field4
   do
     state_abbreviation=${abbreviation[$field1]}
       echo "$state_abbreviation,$field4" >> abbreviation_data.csv
   done < state_change.sorted

Next, plot the data:

.. code-block:: shell

   gnuplot -p popChange.gnuplot

**Output:**

.. code-block:: shell

                                                                                                                                                                
                                                            States Population Change from 2020 to 2021                                                           
                                                                                                                                                                 
   3 +-------------------------------------------------------------------------------------------------------------------------------------------------------+   
     |  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  + +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  |   
     |                                                                                                                                                       |   
     |                                                                                                                                                       |   
   2 |-+                                                                                                                                                   +-|   
     |                                                                            A                                                        A                 |   
     |                                                                                                                                                       |   
     |     A              A                                                                                                    A                             |   
   1 |-+                        A                                                       A              A                          A     A                  +-|   
     |                             A                          A                            A                                         A                       |   
     |        A     A                                                                                           A                             A              |   
     |                 A                       A  A                             A                                                                   A        |   
   0 |-+A                                            A  A        A        A          A           A           A     A        A                    A        A+-|   
     |                                                                 A     A                A                       A                                      |   
     |                                                     A        A                                     A              A                             A     |   
     |           A                    A                                                                                                                      |   
  -1 |-+                                    A                                                                                                              +-|   
     |                                                                                                                                                       |   
     |                                                                                              A                                                        |   
     |                                                                                                                                                       |   
  -2 |-+                                                                                                                                                   +-|   
     |                                                                                                                                                       |   
     |                                                                                                                                                       |   
     |  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  + +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  +  |   
  -3 +-------------------------------------------------------------------------------------------------------------------------------------------------------+   
    Al Ak Az Ar Ca Co Ct De Dc Fl Ga Hi Id Il In Ia Ks Ky La Me Md Ma Mi Mn Ms MoMt Ne Nv Nh Nj Nm Ny Nc Nd Oh Ok Or Pa Pr Ri Sc Sd Tn Tx Ut Vt Va Wa Wv Wi Wy   
                                                                                                                                                                

Here is the gnuplot file:

.. code-block:: shell

   cat popChange.gnuplot

**Output:**

.. code-block:: shell

   set datafile separator ','
   set title 'States Population Change from 2020 to 2021'
   set term dumb size 160,30
   plot 'abbreviation_data.csv' using 2:xtic(1) notitle
