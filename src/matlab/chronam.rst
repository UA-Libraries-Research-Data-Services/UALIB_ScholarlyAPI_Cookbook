Chronicling America API in Matlab
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Anastasia Ramig

**LOC Chronicling America API Documentation:** https://chroniclingamerica.loc.gov/about/api/

These recipe examples were tested on December 6, 2022 in MATLAB R2022b.

**Attribution:** We thank **Professor Jessica Kincaid** (UA Libraries, Hoole Special Collections)
for the use-cases. All data was collected from the Library of Congress, Chronicling America: Historic
American Newspapers site, using the API.

Note that the data from the *Alabama state intelligencer*, *The age-herald*, and the 
*Birmingham age-herald* were contributed to Chronicling America by The University of 
Alabama Libraries: https://chroniclingamerica.loc.gov/awardees/au/

1. Basic API Request
============================

The Chronicling America API identifies newspapers and other records using LCCNs.
We can query the API once we have the LCCN for the newspaper and even ask for
particular issues and editions. For example, the following link lists newspapers 
published in the state of Alabama, from which the LCCN can be obtained:
https://chroniclingamerica.loc.gov/newspapers/?state=Alabama

Here is an example with the Alabama State Intelligencer:

.. code-block:: matlab

   %% set up the API parameters and pull json data for the given LCCN
   api_url = "https://chroniclingamerica.loc.gov/lccn/";
   lccn = "sn84023600";
   q = webread(api_url + lccn + ".json")

**Output:**

.. code-block:: matlab

   q = struct with fields:
    place_of_publication: 'Tuskaloosa [sic], Ala.'
                    lccn: 'sn84023600'
              start_year: '183?'
                   place: {'Alabama--Tuscaloosa--Tuscaloosa'}
                    name: 'Alabama State intelligencer. [volume]'
               publisher: 'T.M. Bradford'
                     url: 'https://chroniclingamerica.loc.gov/lccn/sn84023600.json'
                end_year: '18??'
                  issues: []
                 subject: []

.. code-block:: matlab

   %% extract the name from the search
   q.name

**Output:**

.. code-block:: matlab

   ans = 'Alabama State intelligencer. [volume]'

.. code-block:: matlab

   %% extract the publisher from the search
   q.publisher

**Output:**

.. code-block:: matlab

   ans = 'T.M. Bradford'

Moving on to another publication, we can get the 182nd page (seq-182) of the 
Evening Star newspaper published on November 19, 1961.

.. code-block:: matlab

   %% set up the API parameters and pull json data for the given LCCN
   lccn2 = "sn83045462/1961-11-19/ed-1/seq-182";
   q2 = webread(api_url + lccn2 + ".json");
   
   %% obtain the url for the pdf of the page
   q_url = q2.pdf

**Output:**

.. code-block:: matlab

   q_url = 'https://chroniclingamerica.loc.gov/lccn/sn83045462/1961-11-19/ed-1/seq-182.pdf'

.. code-block:: matlab

   %% view the PDF in web browser
   web(q_url)

2. Frequency of "University of Alabama" mentions
====================================================

The URL below limits to searching newspapers in the state of Alabama and provides
500 results (as a demo) of “University of Alabama” mentions. Note that phrases
can be searched by putting them inside parentheses for the query.

.. code-block:: matlab

   %% set up the API parameters and pull json data
   api_url = "https://chroniclingamerica.loc.gov/search/pages/results/?state=Alabama&proxtext=(University%20of%20Alabama)&rows=500&format=json";
   options = weboptions('Timeout', 30);
   alabamaInfo = webread(api_url, options);
   
   %% find the size of the data structure
   size(struct2table(alabamaInfo.items))

**Output:**

.. code-block:: matlab

   ans = 1x2
   500    28

.. code-block:: matlab

   %% extract the years from the dates given
   dates = {alabamaInfo.items.date};
   datesList = {ones(length(alabamaInfo.items), 1)};
   for i = 1:length(dates)
      datesList{i} = str2double(dates{i}(1:4));
   end
   %% plot a histogram of the mentions according to decade
   x = cell2mat(datesList);
   edges = [1890 1900 1910 1920 1930];
   xticks = ([1890, 1900, 1910, 1920]);
   histogram(x, edges)
   title("Mentions of University of Alabama by Decade");
   xlabel("Decade");
   ylabel("Mentions");

**Output:**

.. image:: imgs/matlab_chronam_im0.png

3. Industrialization keywords frequency in the Birmingham Age-Herald
=======================================================================

We will try to obtain the frequency of “Iron” on the front pages of the Birmingham Age- herald newspapers
from the year 1903 to 1949 (limited to first 500 rows for testing here).

.. code-block:: matlab

   %% set up the API parameters and pull json data for the given parameters
   api_url = "https://chroniclingamerica.loc.gov/search/pages/results/?state=Alabama&lccn=sn85038485&dateFilterType=yearRange&date1=1903&date2=1949&sequence=1&andtext=Iron&rows=500&searchType=advanced&format=json";
   ind = webread(api_url, options);

.. code-block:: matlab

   %% create a dataset of dates and format as datetimes
   dates2 = {ind.items.date};
   x2 = datetime(dates2, 'InputFormat', "yyyyMMdd");
   
   %% plot a histogram of mentions of iron by year
   histogram(x2.Year, 'BinMethod', 'integers')
   title("Iron Frequency in the Birmingham Age Herald");
   xlabel("Year");
   ylabel("Mentions");

.. image:: imgs/matlab_chronam_im1.png
