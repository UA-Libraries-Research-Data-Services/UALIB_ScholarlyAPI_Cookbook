Scopus API in Matlab
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Anastasia Ramig

These recipe examples use the Elsevier Scopus API. 
Code was tested with MATLAB R2021b and sample data downloaded from the Scopus API on April 26, 2022
via http://api.elsevier.com and http://www.scopus.com. This tutorial content is intended to help 
facillitate academic research. Before continuing or reusing any of this code, please be aware of
Elsevier's `API policies and appropiate use-cases`_. You will also need to register for an API key
in order to use the Scopus API.

.. _API policies and appropiate use-cases: https://dev.elsevier.com/use_cases.html

Setup
=========

We will start by setting up the API key. Save your key in a text file in the
same directory as the current Matlab folder and import your key as follows:

.. code-block:: matlab

   %% import API key from file
   myAPIKey = importdata("apiKey.txt");

1. Get Author Data
=====================

Number of Records for Author
-------------------------------

.. code-block:: matlab

   %% setup API information and pull data
   api_url = "https://api.elsevier.com/content/search/scopus?query=";
   author_id = "AU-ID(55764087400)&apiKey=";
   q = webread(api_url + author_id + myAPIKey)

**Output:**

.. code-block:: matlab

   q = struct with fields:
      search_results: [1x1 struct]

.. code-block:: matlab

   %% create an array of ones to pre-allocate doi_list
   doi_list = {ones(length(q.search_results.entry), 1)};

   %% create a list of dois from the data
   for i = 1:length(q.search_results.entry)
      doi_list{i} = q.search_results.entry{i,1}.prism_doi;
   end
   doi_list

**Output:**

.. code-block:: matlab

   doi_list = 1x20 cell
   '10.1021/acs.jchemed.1c00904''10.5860/crln.82.9.428''10.1021/acs.iecr.8b02573''10.1021/acs.jchemed.6b00602''10.5062/F4TD9VBX''10.1021/acs.macromol.6b02005''10.1186/s13321-016-0181-z''10.1021/acs.chemmater.5b04431''10.1021/acs.jchemed.5b00512''10.1021/acs.jchemed.5b00375''10.5860/crln.76.9.9384''10.5860/crln.76.2.9259''10.1021/ed400887t''10.1016/j.acalib.2014.03.015''10.5062/F4XS5SB9''10.1021/ma300328u''10.1021/mz200108a''10.1021/ma201170y''10.1021/ma200184u''10.1021/cm102374t'

.. code-block:: matlab

   %% create an array of ones to pre-allocate titles_list
   titles_list = {ones(length(q.search_results.entry), 1)};

   %% create a list of titles from the data
   for i = 1:length(q.search_results.entry)
      titles_list{i} = q.search_results.entry{i,1}.dc_title;
   end
   titles_list

**Output:**

.. code-block:: matlab

   titles_list = 1x20 cell
   'Using NCBI Entrez Direct (EDirect) for Small Molecule Chemical Informati…  'Using the linux operating system full-time tips and experiences from a s…  'Analysis of the Frequency and Diversity of 1,3-Dialkylimidazolium Ionic …  'Rapid Access to Multicolor Three-Dimensional Printed Chemistry and Bioch…  'Text analysis of chemistry thesis and dissertation titles''Phototunable Thermoplastic Elastomer Hydrogel Networks''Programmatic conversion of crystal structures into 3D printable files us…  'Dangling-End Double Networks: Tapping Hidden Toughness in Highly Swollen…  'Replacing the Traditional Graduate Chemistry Literature Seminar with a C…  '3D Printed Block Copolymer Nanostructures''Hypotheses in librarianship: Applying the scientific method''Recruiting students to campus: Creating tangible and digital products in…  '3D printed molecules and extended solid models for teaching symmetry and…  'Repurposing Space in a Science and Engineering Library: Considerations f…  'A model for managing 3D printing services in academic libraries''Morphological phase behavior of poly(RTIL)-containing diblock copolymer …  'Network formation in an orthogonally self-assembling system''Access to nanostructured hydrogel networks through photocured body-cente…  'Synthesis and ordered phase separation of imidazolium-based alkyl-ionic …  'Thermally stable photocuring chemistry for selective morphological trapp…  

.. code-block:: matlab

   %% create an array of ones to pre-allocate citedby_count
   citedby_count = {ones(length(q.search_results.entry), 1)};

   %% create a list of counts of how much each title was cited
   for i = 1:length(q.search_results.entry)
      citedby_count{i} = q.search_results.entry{i,1}.citedby_count;
   end
   citedby_count

**Output:**

.. code-block:: matlab

    citedby_count = 1x20 cell
    '0'          '0'          '17'         '25'         '5'          '11'         '20'         '6'          '10'         '25'         '0'          '0'          '98'         '6'          '34'         '40'         '31'         '18'         '45'         '11'         

.. code-block:: matlab

   %% find the total number of cites
   citesTotal = str2double(citedby_count);
   totalCites = sum(citesTotal)

**Output:**

.. code-block:: matlab

   totalCites = 402

2. Get Author Data in a Loop
==================================

Number of Records for Author
------------------------------

.. code-block:: matlab

   %% import author text data as a cell array
   authorList = importdata("authorData.txt")

**Output:**

.. code-block:: matlab

   authorList = 5x1 cell
   '{Emy Decker, 36660678600}'   
   '{Lindsey Lowry, 57210944451}'
   '{Karen Chapman, 35783926100}'
   '{Kevin Walker, 56133961300}' 
   '{Sara Whitver, 57194760730}' 


.. code-block:: matlab

   %% create a list of author names and delete the extra bracket from it
   authorList2 = cellfun(@(x) strsplit(x, ","), authorList, 'UniformOutput', false);
   for i = 1:length(authorList2)
      str = authorList2{i, 1}{1, 1};
      old = "{";
      new = "";
      authorList2{i, 1}{1, 1} = replace(str, old, new);
   end

   %% extract the author ids
   author_ids = {ones(length(authorList2), 1)};
   for i = 1:length(authorList2)
      pat = digitsPattern;
      author_ids{i} = extract(authorList2{i, 1}{1, 2}, pat);
   end

.. code-block:: matlab

   %% preallocate an array for the number of records
   numRecords = {ones(length(author_ids), 1)};

   %% find the number of records for each author and add it to the author list
   for i = 1:length(numRecords{1, 1})
      q1 = webread(api_url + "AU-ID(" + author_ids{1, i} + ")&apiKey=" + myAPIKey);
      numRecords{i} = length(q1.search_results.entry);
      pause(1)
      authorList2{i, 1}{1, 3} = numRecords{i};
   end
   disp(cell2table(authorList2))

**Output:**

.. code-block:: matlab

                         authorList2                   
    ________________________________________________

    {'Emy Decker'   }    {' 36660678600}'}    {[14]}
    {'Lindsey Lowry'}    {' 57210944451}'}    {[ 4]}
    {'Karen Chapman'}    {' 35783926100}'}    {[25]}
    {'Kevin Walker' }    {' 56133961300}'}    {[ 8]}
    {'Sara Whitver' }    {' 57194760730}'}    {[ 4]}

Get Record Data
-------------------

.. code-block:: matlab

   clear info 
   %% extract the dois and cites for each author
   for i = 1:length(author_ids)
      q_records = webread(api_url + "AU-ID(" + author_ids{1, i}+")&apiKey=" + myAPIKey);
      n = length(q_records.search_results.entry);
      
      %% preallocate cell array for the dois and cites
      doiList = cell(1, length(author_ids));
      citeList = cell(1, length(author_ids));
      for k = 1:n
         try
               doiList{1, i}{k, 1} = q_records.search_results.entry{k, 1}.prism_doi;
               citeList{1, i}{k, 1} = q_records.search_results.entry{k, 1}.citedby_count;
         catch
         end
      end
      pause(1)
      
      %% add the dois and cites to an overall information array
      info{1, 1}{1, i} = doiList{1, i};
      info{2, 1}{1, i} = citeList{1, i};
   end

   %% create arrays for the dois and cites
   dois = {};
   cites = {};
   for i = 1:width(info{1, 1})
      dois = vertcat(dois, info{1, 1}{1, i});
      cites = vertcat(cites, info{2, 1}{1, i});
   end

.. code-block:: matlab

   %% create a conclusive array
   authorArray = horzcat(dois, cites);
   nameArray = {};

   %% create an array of author names
   for i = 1:(length(numRecords))
      nameLength = int16(numRecords{i});
      authorName = cellstr(repmat(authorList2{i, 1}{1, 1}, nameLength, 1));
      nameArray = vertcat(nameArray, authorName);
   end

   %% add the author names to the informational array
   authorArray = horzcat(authorArray, nameArray)

**Output:**

.. code-block:: matlab

   authorArray = 55x3 cell
      1	2	3
   1	'10.1108/RSR-08-2021-0051'	'0'	'Emy Decker'
   2	'10.1080/1072303X.2021.1929642'	'0'	'Emy Decker'
   3	'10.1080/15367967.2021.1900740'	'8'	'Emy Decker'
   4	'10.1080/15367967.2020.1826951'	'0'	'Emy Decker'
   5	'10.1080/10691316.2020.1781725'	'0'	'Emy Decker'
   6	'10.1145/3347709.3347805'	'0'	'Emy Decker'
   7	'10.4018/978-1-5225-5631-2.ch09'	'3'	'Emy Decker'
   ...
   ...
   ...

Save Record Data to a file
-------------------------------

.. code-block:: matlab

   %% save the search for each author to a mat file
   for author = 1:length(author_ids)
      authorName = authorList2{author, 1}{1, 1};
      q2 = webread(api_url + "AU-ID" + "(" + author_ids{1, author} + ")&apiKey=" + myAPIKey);
      pause(1)
      filename = authorName + ".mat";
      save filename q2;
   end

.. code-block:: matlab

   %% save the author arrays to individual text files
   for i = 1:(length(numRecords))
      clear individualAuthorData;
      individualDois = info{1, 1}{1, i};
      individualCites = info{2, 1}{1, i};
      
      nameLength = int16(numRecords{i});
      authorName = cellstr(repmat(authorList2{i, 1}{1, 1}, nameLength, 1));
      
      individualAuthorData = horzcat(individualDois, individualCites);
      individualAuthorData = horzcat(individualAuthorData, authorName);
      
      writecell(individualAuthorData, (authorList2{i, 1}{1, 1} + ".txt"), "Delimiter", "\t");
   end

3. Get References via a Title Search
=====================================

Number of Title Match Records
----------------------------------

Search Scopus for all references containing 'ChemSpider" in the record title.

.. code-block:: matlab

   %% set up the API information
   api_url = "https://api.elsevier.com/content/search/scopus?query=";
   author_id = "TITLE(ChemSpider)&apiKey=";

   %% find the information for ChemSpider and get the total number of results
   q3 = webread(api_url + author_id + myAPIKey);
   q3.search_results.opensearch_totalResults

Repeat this in a loop to get number of Scopus records for each title search.

.. code-block:: matlab

   %% create a list of titles
   titleList = ["ChemSpider", "PubChem", "ChEMBL", "Reaxys", "SciFinder"];
   length(titleList)

   %% create an array of ones to pre-allocate numRecordsTitle
   clear numRecordsTitle
   numRecordsTitle = {ones(length(titleList), 1)};
   
   %% obtain the number of records for each title in the list and create an array
   for i = 1:length(titleList)
      qt = webread(api_url + "TITLE(" + titleList(i) + ")&apiKey=" + myAPIKey);
      numt = qt.search_results.opensearch_totalResults;
      numRecordsTitle{1, i}{1, 1} = titleList(i);
      numRecordsTitle{1, i}{1, 2} = numt;
      pause(1)
   end

Download Title Match Record Data
------------------------------------

Download records and create a list of selected metadata.

.. code-block:: matlab

   %% create a list of titles and preallocate an array
   titleList = ["ChemSpider", "PubChem", "ChEMBL", "Reaxys", "SciFinder"];
   scopusTitleData = {ones(length(titleList), 1)};
   %% find the dois, titles, and dates for each title in the list and put them into an array
   for t = 1:length(titleList)
      qt = webread(api_url + "TITLE(" + titleList(t) + ")&apiKey=" + myAPIKey);
      n = length(qt.search_results.entry);
      doiTitles = cell(1, length(titleList));
      titles = cell(1, length(titleList));
      dates = cell(1, length(titleList));
      for k = 1:n
         try
               doiTitles{1, t}{k, 1} = qt.search_results.entry{k, 1}.prism_doi;
               titles{1, t}{k, 1} = qt.search_results.entry{k, 1}.dc_title;
               dates{1, t}{k, 1} = qt.search_results.entry{k, 1}.prism_coverDate;
         catch
         end
      end
      pause(1)
      infoTitles{1, 1}{1, t} = doiTitles{1, t};
      infoTitles{2, 1}{1, t} = titles{1, t};
      infoTitles{3, 1}{1, t} = dates{1, t};
   end

.. code-block:: matlab

   %% create an overall array of the information found above
   titleDois = {};
   titlesFinal = {};
   datesFinal = {};
   for t = 1:width(info{1, 1})
      titleDois = vertcat(titleDois, infoTitles{1, 1}{1, t});
      titlesFinal = vertcat(titlesFinal, infoTitles{2, 1}{1, t});
      datesFinal = vertcat(datesFinal, infoTitles{3, 1}{1, t});
   end
   titleArray = horzcat(titleDois, titlesFinal);
   titleArray = horzcat(titleArray, datesFinal);
   %% create an array of names and add it to the overall array
   titlesNameArray = {};
   for t = 1:length(titleList)
      nameLength = length(infoTitles{1, 1}{1, t});
      titlesAuthorName = cellstr(repmat(titleList(t), nameLength, 1));
      titlesNameArray = vertcat(titlesNameArray, titlesAuthorName);
   end
   titleArray = horzcat(titleArray, titlesNameArray)

**Output:**

.. code-block:: matlab

      titleArray = 88x4 cell
      1	2	3	4
   1	'10.1039/c5np90022k'	'Editorial: ChemSpider-a tool for Natural Products research'	'2015-08-01'	'ChemSpider'
   2	'10.1021/bk-2013-1128.ch020'	'ChemSpider: How a free community resource of data can support the teaching of nmr spectroscopy'	'2013-01-01'	'ChemSpider'
   3	'10.1007/s13361-011-0265-y'	'Identification of "known unknowns" utilizing accurate mass data and chemspider'	'2012-01-01'	'ChemSpider'
   4	'10.1002/9781118026038.ch22'	'Chemspider: A Platform for Crowdsourced Collaboration to Curate Data Derived From Public Compound Databases'	'2011-05-03'	'ChemSpider'
   5	'10.1021/ed100697w'	'Chemspider: An online chemical information resource'	'2010-11-01'	'ChemSpider'
   6	'10.1016/j.bioorg.2022.105648'	'Structure-based discovery of a specific SHP2 inhibitor with enhanced blood–brain barrier penetration from PubChem database'	'2022-04-01'	'PubChem'
   7	'10.1016/j.jmb.2022.167514'	'PubChem Protein, Gene, Pathway, and Taxonomy Data Collections: Bridging Biology and Chemistry through Target-Centric Views of PubChem Data'	'2022-01-01'	'PubChem'
   8	'10.1007/s40011-021-01335-x'	'Identification a Novel Inhibitor for Aldo–Keto Reductase 1 C3 by Virtual Screening of PubChem Database'	'2022-01-01'	'PubChem'
   9	'10.1007/978-1-0716-2067-0_27'	'Plant Reactome and PubChem: The Plant Pathway and (Bio)Chemical Entity Knowledgebases'	'2022-01-01'	'PubChem'
   10	'10.1016/j.molstruc.2021.130968'	'3CL<sup>pro</sup> and PL<sup>pro</sup> affinity, a docking study to fight COVID19 based on 900 compounds from PubChem and literature. Are there new drugs to be found?'	'2021-12-05'	'PubChem'
   11	'10.1093/glycob/cwab078'	'Enhancing the interoperability of glycan data flow between ChEBI, PubChem and GlyGen'	'2021-11-01'	'PubChem'
   ...
   ...
   ...

