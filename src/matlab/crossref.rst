Crossref API in Matlab
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Anastasia Ramig

**Crossref API documentation:** https://api.crossref.org/swagger-ui/index.html

These recipe examples were tested on April 21, 2022 in MATLAB R2021a.

*From our testing, we have found that the crossref metadata across publishers and even journals can vary considerably. As a result, it can be easier to work with one journal at a time when using the crossref API (e.g., particularly when trying to extract selected data from records).*

1. Basic Crossref API call
============================

Setup API parameters
----------------------

.. code-block:: matlab

   base_url = "https://api.crossref.org/works/"; %% the base url for api calls
   email = "your_email@ua.edu"; %% change this to your email
   mailto = "?mailto=" + email;
   options = weboptions('Timeout', 30);
   doi = "10.1186/1758-2946-4-12"; %% example

Request data from Crossref API
--------------------------------

.. code-block:: matlab

   api_data = webread(base_url + doi + mailto, options);
   disp(api_data.message)

**Output:**

.. code-block:: matlab

                   indexed: [1×1 struct]
           reference_count: 16
                 publisher: 'Springer Science and Business Media LLC'
                     issue: '1'
                   license: [1×1 struct]
            content_domain: [1×1 struct]
     short_container_title: {'J Cheminform'}
           published_print: [1×1 struct]
                       DOI: '10.1186/1758-2946-4-12'
                      type: 'journal-article'
                   created: [1×1 struct]
                    source: 'Crossref'
    is_referenced_by_count: 25
                     title: {'The Molecule Cloud - compact visualization of large collections of molecules'}
                    prefix: '10.1186'
                    volume: '4'
                    author: [2×1 struct]
                    member: '297'
          published_online: [1×1 struct]
                 reference: {16×1 cell}
           container_title: {'Journal of Cheminformatics'}
            original_title: []
                  language: 'en'
                      link: [3×1 struct]
                 deposited: [1×1 struct]
                     score: 1
                  resource: [1×1 struct]
                  subtitle: []
               short_title: []
                    issued: [1×1 struct]
          references_count: 16
             journal_issue: [1×1 struct]
            alternative_id: {'336'}
                       URL: 'http://dx.doi.org/10.1186/1758-2946-4-12'
                  relation: [1×1 struct]
                      ISSN: {'1758-2946'}
                 issn_type: [1×1 struct]
                   subject: {4×1 cell}
                 published: [1×1 struct]
            article_number: '12'


Select some specific data
-----------------------------

.. code-block:: matlab

   %% Get Journal title
   api_data.message.container_title

**Output:**

.. code-block:: matlab

   ans = 1×1 cell array
       {'Journal of Cheminformatics'}

.. code-block:: matlab

   %% Get article title
   api_data.message.title

**Output:**

.. code-block:: matlab

   ans = 1×1 cell array
       {'The Molecule Cloud - compact visualization of large collections of molecules'}

.. code-block:: matlab

   %% Get article author names
   names{1} = string(api_data.message.author(1).given) + " " + string(api_data.message.author(1).family);
   names{2} = string(api_data.message.author(2).given) + " " + string(api_data.message.author(2).family);
   disp(names)

**Output:**

.. code-block:: matlab

    {["Peter Ertl"]}    {["Bernhard Rohde"]}

.. code-block:: matlab

   %% get the bibliography references
   bib_refs = cell(1,length(api_data.message.reference)); % pre-allocate a cell array
   for ref = 1:length(api_data.message.reference)
       bib_refs{ref} = api_data.message.reference{ref}.unstructured;
   end
   %% display the first few references
   disp(bib_refs(1:5))

**Output:**

.. code-block:: matlab

   Column 1

    {'Martin E, Ertl P, Hunt P, Duca J, Lewis R: Gazing into the crystal ball; the future of com…'}

   Column 2

    {'Langdon SR, Brown N, Blagg J: Scaffold diversity of exemplified medicinal chemistry space.…'}

   Column 3

    {'Blum LC, Reymond J-C: 970 Million druglike small molecules for virtual screening in the ch…'}

   Column 4

    {'Dubois J, Bourg S, Vrain C, Morin-Allory L: Collections of compounds - how to deal with th…'}

   Column 5

    {'Medina-Franco JL, Martinez-Mayorga K, Giulianotti MA, Houghten RA, Pinilla C: Visualizatio…'}

2. Crossref API call with a Loop
=================================

Setup API parameters
---------------------

.. code-block:: matlab

   base_url = "https://api.crossref.org/works/"; %% the base url for api calls
   email = "your_email@ua.edu"; %% change this to your email
   mailto = "?mailto=" + email;

Create a list of DOIs
----------------------

.. code-block:: matlab

   %% Create a list of DOIs
   doi_list = ["10.1021/acsomega.1c03250",...
   "10.1021/acsomega.1c05512",...
   "10.1021/acsomega.8b01647",...
   "10.1021/acsomega.1c04287",...
   "10.1021/acsomega.8b01834"];

Request metadata for each DOI from Crossref API and save to a structure
--------------------------------------------------------------------------

.. code-block:: matlab

   %% get data for each of the dois in the list
   doi_metadata = struct;
   for doi = 1:length(doi_list)
       doi_metadata.doi{doi} =  webread(base_url + doi_list(doi) + mailto);
       pause(1)
   end
   doi_metadata

**Output:**

.. code-block:: matlab

   doi_metadata = struct with fields:
       doi: {[1×1 struct]  [1×1 struct]  [1×1 struct]  [1×1 struct]  [1×1 struct]}

Select some specific data
--------------------------

.. code-block:: matlab

   %% Create a table of information
   message_array = cell(1, length(doi_metadata.doi));
   for i = 1:length(doi_metadata.doi)
       message_array{i} = doi_metadata.doi{1, i};
   end
   message_table = cell2table(message_array);
   message_table = rows2vars(message_table);
   message_table.OriginalVariableNames = [];
   %% Get article titles
   titles = cell(1,height(message_table));
   for m = 1:height(message_table)
       message = [message_table.Var1(m, 1).message];
       titles(m) = message.title;
   end
   disp(titles)

**Output:**

.. code-block:: matlab

   Column 1

    {'Navigating into the Chemical Space of Monoamine Oxidase Inhibitors by Artificial Intellige…'}

   Column 2

    {'Impact of Artificial Intelligence on Compound Discovery, Design, and Synthesis'}

   Column 3

    {'How Precise Are Our Quantitative Structure–Activity Relationship Derived Predictions for N…'}

   Column 4

    {'Applying Neuromorphic Computing Simulation in Band Gap Prediction and Chemical Reaction Cl…'}

   Column 5

    {'QSPR Modeling of the Refractive Index for Diverse Polymers Using 2D Descriptors'}

3. Crossref API call for journal information
==============================================

Setup API parameters
---------------------

.. code-block:: matlab

   jbase_url = "https://api.crossref.org/journals/"; %% the base url for api calls
   email = "your_email@ua.edu"; %% change this to your email
   mailto = "?mailto=" + email;
   issn = "1471-2105"; %% issn for the journal BMC Bioinformatics

Request journal data from Crossref API
---------------------------------------

.. code-block:: matlab

   jour_data = webread(jbase_url + issn + mailto)

**Output:**

.. code-block:: matlab

   jour_data = struct with fields:
             status: 'ok'
       message_type: 'journal'
    message_version: '1.0.0'
            message: [1×1 struct]

.. code-block:: matlab

   % get subjects
   disp({jour_data.message.subjects.name})

**Output:**

.. code-block:: matlab

   Columns 1 through 3

    {'Applied Mathematics'}    {'Computer Science Applications'}    {'Molecular Biology'}

   Columns 4 through 5

    {'Biochemistry'}    {'Structural Biology'}

4. Crossref API - Get article DOIs for a journal
=================================================

Setup API parameters
----------------------

.. code-block:: matlab

   jbase_url = "https://api.crossref.org/journals/"; %% the base url for api calls
   email = "your_email@ua.edu"; %% Change this to be your email
   mailto = "&mailto=" + email;
   options = weboptions('Timeout', 60);
   issn = "1471-2105";  %% issn for the journal BMC Bioinformatics
   journal_works2014 = "/works?filter=from-pub-date:2014,until-pub-date:2014&select=DOI"; %% query to get DOIs for 2014

Request DOI data from Crossref API
-----------------------------------

.. code-block:: matlab

   doi_data = webread(jbase_url + issn + journal_works2014 + mailto, options)

**Output:**

.. code-block:: matlab

   doi_data = struct with fields:
             status: 'ok'
       message_type: 'work-list'
    message_version: '1.0.0'
            message: [1×1 struct]


.. code-block:: matlab

   doi_data.message.total_results

**Output:**

.. code-block:: matlab

   ans = 
      619

By default, 20 results are returned. Crossref allows up to 1000 returned results using the rows parameter. To get all 619 results, we can increase the number of returned rows.

.. code-block:: matlab

   rows = "&rows=700";
   weboptions('Timeout', 60);
   doi_data_all = webread(jbase_url + issn + journal_works2014 + rows + mailto, options);

Extract DOIs
--------------

.. code-block:: matlab

   dois_list = {doi_data_all.message.items.DOI}

**Output:**

.. code-block:: matlab

   dois_list = 1×619 cell
   '10.1186/1471-2105-15-158'    '10.1186/1471-2105-15-106'    '10.1186/1471-2105-15-268'    '10.1186/1471-2105-15-248' ...


What if we have more than 1000 results in a single query? For example, if we wanted the DOIs from BMC Bioinformatics for years 2014 through 2016?

.. code-block:: matlab

   jbase_url = "https://api.crossref.org/journals/"; %% the base url for api calls
   email = "your_email@ua.edu"; %% Change this to be your email
   mailto = "&mailto=" + email;
   options = weboptions('Timeout', 60);
   issn = "1471-2105";  %% issn for the journal BMC Bioinformatics
   journal_works2014_2016 = "/works?filter=from-pub-date:2014,until-pub-date:2016&select=DOI"; %% query to get DOIs for 2014-2016
   doi_data2 = webread(jbase_url + issn + journal_works2014_2016 + mailto, options);

.. code-block:: matlab

   doi_data2.message.total_results

**Output:**

.. code-block:: matlab

   ans = 
        1772

Here we see that the total results is over 1000 (total results: 1772). An additional parameter that we can use with crossref API is called "offset". The offset option allows us to select sets of records and define a starting position (e.g., the first 1000, and then the second set of up to 1000).

.. code-block:: matlab

   rows = "&rows=1000";
   numResults = doi_data.message.total_results;
   doi_list2 = cell(1,int16((numResults/1000)+1));
   for n = 1:(int16((numResults/1000)+1))
       query = webread(jbase_url + issn + journal_works2014_2016 + rows + "&offset=" + string((1000*(n-1))) + mailto, options);
       pause(1);
       doi_list2{n} = query;
   end

.. code-block:: matlab

   %% concatenate the results into a cell array
   doi_list3 = [doi_list2{1,1}.message.items; doi_list2{1, 2}.message.items];
   length(doi_list3)

**Output:**

.. code-block:: matlab

   ans = 
        1772

.. code-block:: matlab

   %  Show index results 1000-1020
   disp(struct2cell(doi_list3(1000:1020)))

**Output:**

.. code-block:: matlab

   Columns 1 through 2

    {'10.1186/1471-2105-15-139'}    {'10.1186/s12859-015-0768-9'}

   Columns 3 through 4

    {'10.1186/1471-2105-15-s6-s1'}    {'10.1186/1471-2105-15-157'}

   Columns 5 through 6

    {'10.1186/s12859-016-1246-8'}    {'10.1186/s12859-016-1155-x'}

   Columns 7 through 8

    {'10.1186/s12859-014-0381-3'}    {'10.1186/s12859-015-0725-7'}

   Columns 9 through 10

    {'10.1186/s12859-015-0465-8'}    {'10.1186/s12859-014-0426-7'}

   Columns 11 through 12

    {'10.1186/s12859-016-1326-9'}    {'10.1186/s12859-015-0636-7'}

   Columns 13 through 14

    {'10.1186/1471-2105-15-136'}    {'10.1186/s12859-015-0789-4'}

   Columns 15 through 16

    {'10.1186/1471-2105-15-164'}    {'10.1186/1471-2105-15-121'}

   Columns 17 through 18

    {'10.1186/s12859-016-1272-6'}    {'10.1186/1471-2105-15-s13-s2'}

   Columns 19 through 20

    {'10.1186/s12859-015-0451-1'}    {'10.1186/s12859-016-0929-5'}

   Column 21

    {'10.1186/s12859-016-1254-8'}

