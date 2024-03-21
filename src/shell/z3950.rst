Z39.50 in Unix Shell
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

.. sectionauthor:: Vincent F. Scalfani <vfscalfani@ua.edu>

by Vincent F. Scalfani and Cyrus Gomes

.. note::

   This an alpha version of a Z39.50 tutorial; we are still learning 
   about Z39.50 query construction and how best to apply a workflow for searching
   library catalogs programmatically. Comments are certainly welcome.

**UA Libraries Z39.50 Connection Details:** 
http://www.lib.ua.edu/research-tools/university-libraries-catalog/z39-50-connection/

These recipe examples were tested in February 2023 using Ubuntu 22.04 LTS and YAZ 5.31.1.

Program requirements
=========================

For **1.** through **3.** below, you will need to install Index Data's `yaz-client`_.

``yaz-client`` is a `Z39.50`_ and `SRU (Search/Retrieval via URL)`_ command-line program.
More broadly, the YAZ suite of software is an open source toolkit:

https://github.com/indexdata/yaz

On Debian-based Linux systems, you can install the ``yaz-client``
(and dependencies) via ``apt-get``:

.. code-block:: shell

   sudo apt-get install yaz yaz-doc

For other ways to install ``yaz`` and ``yaz-client``, see Index Data's documentation:

https://software.indexdata.com/yaz/doc/

.. _yaz-client: https://www.indexdata.com/resources/software/yaz/
.. _Z39.50: https://www.loc.gov/z3950/agency/
.. _SRU (Search/Retrieval via URL): https://www.loc.gov/standards/sru/

A very brief introduction to Z39.50
=======================================

What is Z39.50?
---------------------

Z39.50 is a standardized protocol that defines communication rules between a client computer
(e.g., your computer) and a server computer (e.g., a library database server) [#ref0]_.
The protocol also includes a query language, defines record syntax,
and other important parameters for exchanging information [#ref1]_.
For additional Z39.50 background and references, see [#ref2]_, [#ref3]_, 
and [#ref4]_. A copy of the Z39.50 standard is available from the Library of Congress [#ref5]_.

Z39.50 has been around for decades and is still widely used in libraries at the staff level, for example,
to retrieve metadata and search partner library catalogs.

What do end-users in 2023 use Z39.50 for?
---------------------------------------------

Today, the majority of end-users use a graphical website environment (via HTTP/HTTPS),
not Z39.50 for searching library catalogs. In our experience, end-users still use Z39.50 to configure
bibliographic software managers for searching the UA Libraries catalog.

What else can I use Z39.50 for?
-----------------------------------

You can use Z39.50 to search library catalogs programmtically with your own custom scripts and software.

Why would I use Z39.50 instead of a web service API?
------------------------------------------------------

If the information is available via a web service (e.g., REST) API, it's definitely easier to use an
API compared to Z39.50. However, to our knowledge, there is not wide availability
of API access for individual institution level library
catalog searches. As a result, Z39.50 may be your only choice.

How do I find Z39.50 connection details?
----------------------------------------------

You can find UA Libraries Z39.50 connection details here:

http://www.lib.ua.edu/research-tools/university-libraries-catalog/z39-50-connection/

For other Z39.50 connection details, see the Library of Congress Z39.50 Gateway:

https://www.loc.gov/z3950/

Index Data also maintains some Z39.50 accessible database indexes such as Project Gutenberg and Wikipedia:

https://www.indexdata.com/resources/open-content/

How do I construct Z39.50 queries?
------------------------------------

Constructing Z39.50 queries is likely different from what you are used to with modern database boolean queries.
Z39.50 allows for different query specifications (see p. 23 in [#ref5]_). A commonly implemented specification is
the Type-1 query, which is Reverse Polish Notation (RPN) with the bib-1 attribute set. Standard operators typically
accepted include: AND, OR, AND-NOT. Result sets are often limited to 10,000 maximum returned.

The entire bib-1 attribute set can be viewed in the manual pages on linux-baed systems:

.. code-block:: shell

   man bib1-attr

or online at: https://www.loc.gov/z3950/agency/defns/bib1.html

There are a variety of different attributes included in the bib-1 set including Use (1), Relation
(2), Position (3), Structure (4), Truncation (5) and Completeness (6).

Here are a few selected examples for each attribute type along with the corresponding value and name:

.. code-block:: shell
   
   # not a complete set, examples only.
   Use(1)
      1     Personal-name
      4     Title
      7     ISBN
      16    LC-call-number
      21    Subject-heading
      30    Date
      62    Abstract
      1001  Record-type
      1003  Author
      1016  Any
      1018  Publisher
      1023  Indexed-by
      1036  Author-Title-Subject

   RELATION (2)
      1 Less than
      2 Less than or equal
      3 Equal
      4 Greater or equal
      5 Greater than
      6 Not equal

   POSITION (3)
      1 First in field
      2 First in subfield
      3 Any position in field

   STRUCTURE (4)
      1 Phrase
      2 Word
      3 Key
      4 Year

   TRUNCATION (5)
      1 Right truncation
      2 Left truncation
      3 Left and right truncation
      100 Do not truncate

   COMPLETENESS (6)
      1 Incomplete subfield
      2 Complete subfield
      3 Complete field

.. hint::

   Something to be aware of is that Z39.50 implementations do not have to support all bib-1 attributes,
   so you will want to look at the Z39.50 connection details carefully for a list of supported attributes.
   For example, the UA Z39.50 implementation does not support relation attributes; all relations are considered equal.

To construct a query, you first define the operator (if needed), then the attribute(s), then the keyword(s).
Here are a few basic examples:

.. code-block:: shell
   
   # search for `cheminformatics` in the title field
   @attr 1=4 "cheminformatics"

   # search for `cheminformatics` in the title field at first position with truncation
   @attr 1=4 @attr 3=1 @attr 5=1 "cheminformatics"

   # search for `cheminformatics` in the title field and author `noordik`
   @and @attr 1=4 "cheminformatics" @attr 1=1003 "noordik"

   # search for `cheminformatics` in the title field but not "bioinformatics"
   @not @attr 1=4 "cheminformatics" @attr 1=4 "bioinformatics"

   # search for `drug discovery` in the abstract or title
   @or @attr @1=4 "drug discovery" @attr 1=62 "drug discovery"

1. Basic UA Libraries Catalog Searching
=========================================

We will use the ``yaz-client`` program for these search examples. First, start ``yaz-client`` in your terminal:

.. code-block:: shell

   yaz-client

After starting yaz-client, you should see a ``Z>`` prompt in the terminal. Next, open the connection to the
UA Libraries Catalog:

.. code-block:: shell

   open library.ua.edu:7090/voyager

If the connection is successful, you should get something like this:

**Output:**

.. code-block:: shell

   Connecting...OK.
   Sent initrequest.
   Connection accepted by v3 target.
   ID     : 34
   Name   : Voyager LMS - Z39.50 Server
   Version: 2010.3.0
   Options: search present
   Elapsed: 0.358596

Once connected to the UA Libraries Catalog, we can then search the catalog and retrieve records.

To exit ``yaz-client``, type ``quit``

.. code-block:: shell

   quit

**Output:**

.. code-block:: shell

   See you later, alligator.

Keyword, Title, and Author searches
---------------------------------------

Search for "dinosaur" as a keyword in any field (``1=1016``)

.. code-block:: shell

   find @attr 1=1016 "dinosaur"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 504
   records returned: 0
   Elapsed: 0.052500

Search for "dinosaur" in the title field (``1=4``) at first position (``3=1``) with truncation (``5=1``)

.. code-block:: shell
   
   find @attr 1=4 @attr 3=1 @attr 5=1 "dinosaur"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 180
   records returned: 0
   Elapsed: 0.076650

Search for "dinosaur" or "dinosauria" in the title field (``1=4``):

.. code-block:: shell

   find @or @attr 1=4 "dinosaur" @attr 1=4 "dinosauria"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 222
   records returned: 0
   Elapsed: 0.062672

Search for "dinosaur" in the title (``1=4``) or subject field (``1=21``):

.. code-block:: shell

   find @or @attr 1=4 "dinosaur" @attr 1=21 "dinosaur"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 235
   records returned: 0
   Elapsed: 0.059067

Search for "Arnold, Caroline" in the author field (``1=1003``):

.. code-block:: shell

   find @attr 1=1003 "Arnold, Caroline"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 35
   records returned: 0
   Elapsed: 0.038725

Search for "Arnold, Caroline" in the author field (``1=1003``) and "dinosaur" in the title field (``1=4``):

.. code-block:: shell

   find @and @attr 1=1003 "Arnold, Caroline" @attr 1=4 "dinosaur"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 3
   records returned: 0
   Elapsed: 0.008387

Identifier searches
-------------------------------

Search for the government document ``NAS 1.15:110209`` by GPO number (``1=50``):

.. code-block:: shell

   find @attr 1=50 "NAS 1.15:110209"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 1
   records returned: 0
   Elapsed: 0.024986

Find all LC call numbers (``1=16``) matches that start with ``TP145``:

.. code-block:: shell

   find @attr 1=16 "TP145"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 92
   records returned: 0
   Elapsed: 0.027160

2. Searching UA Libraries Catalog in a Loop
==============================================

Here are a few ways to run multiple searches with ``yaz-client``:

First, create a file with your queries. In this example we will search
for 5 books via their ISBN identifiers:

.. code-block:: shell

   cat mysearches

**Output:**

.. code-block:: shell

   open library.ua.edu:7090/voyager
   find @1=7 "1683925041"
   sleep 1
   find @1=7 "9780470183014"
   sleep 1
   find @1=7 "1565925858"
   sleep 1
   find @1=7 "9780136778851"
   sleep 1
   find @1=7 "1785284444"
   quit

Next, run ``yaz-client`` with the option ``-f``:

.. code-block:: shell

   yaz-client -f mysearches
   
**Output:**

.. code-block:: shell

   Connecting...OK.
   Sent initrequest.
   Connection accepted by v3 target.
   ID     : 34
   Name   : Voyager LMS - Z39.50 Server
   Version: 2010.3.0
   Options: search present
   Elapsed: 0.353889
   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 1
   records returned: 0
   Elapsed: 0.007999
   Done sleeping 1 seconds
   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 1
   records returned: 0
   Elapsed: 0.005176
   Done sleeping 1 seconds
   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 2
   records returned: 0
   Elapsed: 0.004862
   Done sleeping 1 seconds
   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 1
   records returned: 0
   Elapsed: 0.004774
   Done sleeping 1 seconds
   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 1
   records returned: 0
   Elapsed: 0.003902
   See you later, alligator.

Here is an alternative method with a bash loop:

.. code-block:: shell

   for isbn in \
      "1683925041" \
      "9780470183014" \
      "1565925858" \
      "9780136778851" \
      "1785284444"
   do
      printf "open library.ua.edu:7090/voyager\nfind @1=7 "$isbn"\nquit\n" |
      yaz-client -f /dev/stdin
      sleep 1
   done

.. note::

   ``/dev/stdin`` allows us to pass a string via stdin with the ``-f`` option, since ``yaz-client -f`` 
   expects a file [#ref6]_.

And here is a more efficient method suggested on GitHub which does not quit ``yaz-client`` on each loop [#ref7]_:

.. code-block:: shell

   for isbn in \
      "1683925041" \
      "9780470183014" \
      "1565925858" \
      "9780136778851" \
      "1785284444"
   do
      printf "open library.ua.edu:7090/voyager\nfind @1=7 "$isbn"\nsleep 1\n"
   done | yaz-client -f /dev/stdin

Finally, if you have a file with your search strings as one per line, use a while loop to avoid having to
write out your strings or declaring them as a bash variable:

.. code-block:: shell

   cat isbns.txt

**Output:**

.. code-block:: shell

   1683925041
   9780470183014
   1565925858
   9780136778851
   1785284444

.. code-block:: shell

   cat isbns.txt |
   while read isbn
   do
      printf "open library.ua.edu:7090/voyager\nfind @1=7 "$isbn"\nsleep 1\n"
   done | yaz-client -f /dev/stdin

3. Retrieve Record(s) Data
============================

USmarc
---------------

For catalog records at The University of Alabama, the default format returned within ``yaz-client`` 
is USmarc (MARC 21). The records are rendered as (mostly) human-readable within the terminal output.
If you are looking for "raw" MARC, that is, the complete machine-readable binary file, see the
below section on "Saving Raw MARC data".

To retrieve records in the terminal with ``yaz-client``, use the ``show`` command with a start
postion and optional number of records. For example, to get the first record:

.. code-block:: shell

   open library.ua.edu:7090/voyager

**Output:**

.. code-block:: shell

   Connecting...OK.
   Sent initrequest.
   Connection accepted by v3 target.
   ID     : 34
   Name   : Voyager LMS - Z39.50 Server
   Version: 2010.3.0
   Options: search present
   Elapsed: 0.514120

.. code-block:: shell

   find @or @attr 1=4 "dinosaur" @attr 1=4 "dinosauria"

**Output:**

.. code-block:: shell

   Sent searchRequest.
   Received SearchResponse.
   Search was a success.
   Number of hits: 222
   records returned: 0
   Elapsed: 0.087466

.. code-block:: shell

   show 1

**Output:**

.. code-block:: shell

   Sent presentRequest (1+1).
   Records: 1
   [VOYAGER]Record type: USmarc
   01239cam  2200325Ka 4500
   001 3444796
   005 20171110111851.0
   008 101221s2008    nyua   b      000 0 eng d
   020    $a 0760783950
   020    $a 9780760783955
   035    $a (OCoLC)ocn828688251
   035    $a (OCoLC)828688251
   035    $a 3444796
   040    $a ALM $c ALM $d UtOrBLW
   049    $a ALMM
   050  4 $a PZ7.H672 $b Adv 2008
   100 1  $a Hoff, Syd, $d 1912-2004. $0 http://id.loc.gov/authorities/names/n78086441
   245 10 $a Adventures of Danny and the dinosaur / $c Syd Hoff.
   264  1 $a New York : $b Barnes & Noble, $c 2008.
   300    $a 128 pages : $b color illustrations ; $c 24 cm.
   336    $a text $b txt $2 rdacontent
   337    $a unmediated $b n $2 rdamedia
   338    $a volume $b nc $2 rdacarrier
   490 1  $a I can read
   505 0  $a Danny and the dinosaur -- Happy birthday, Danny and the dinosaur! -- Danny and the dinosaur go to camp.
   520    $a Danny goes to a museum to see the dinosaurs and ends up spending the day outside with one.
   650  1 $a Dinosaurs $v Fiction.
   650  0 $a Dinosaurs $v Juvenile fiction. $0 http://id.loc.gov/authorities/subjects/sh2008102274
   830  0 $a I can read book. $0 http://id.loc.gov/authorities/names/n42013105
   994    $a C0 $b ALM

   nextResultSetPosition = 2
   Elapsed: 0.060194

To show the first 3 results, add a stop position ``show 1 + 4``:

.. code-block:: shell

   open library.ua.edu:7090/voyager
   find @or @attr 1=4 "dinosaur" @attr 1=4 "dinosauria"
   show 1 + 4
   quit

To quickly scan multiple records from a search, we can pipe the USMarc stdout to ``grep`` and display selected lines:

.. code-block:: shell

   printf "open library.ua.edu:7090/voyager\nfind @or @attr 1=4 "dinosaur" @attr 1=4 "dinosauria"\nshow 1+10\n" | \
   yaz-client -f /dev/stdin | grep "^245"

**Output:**

.. code-block:: shell

   245 10 $a Adventures of Danny and the dinosaur / $c Syd Hoff.
   245 10 $a Age of tephra beds at the Ocean Point dinosaur locality, North Slope, Alaska, based on K-Ar and 40Ar/39Ar analyses / $c by James E. Conrad, Edwin H. McKee, and Brent D. Turrin.
   245 10 $a Age of tephra beds at the Ocean Point dinosaur locality, North Slope, Alaska, based on K-Ar and 40Ar/39Ar analyses / $c by James E. Conrad, Edwin H. McKee, and Brent D. Turrin.
   245 10 $a American dinosaur abroad : $b a cultural history of Carnegie's plaster diplodocus / $c Ilja Nieuwland.
   245 10 $a American experience. $p Dinosaur wars $h [videorecording] / $c WGBH Boston ; produced by Mark Davis and Anna Saraceno ; written and directed by Mark Davis.
   245 14 $a The archaeology of Castle Park Dinosaur National Monument / $c by Robert F. Burgh and Charles R. Scoggin, with appendices by Edgar Anderson, Richard E. Pillmore [and] Volney H. Jones.
   245 10 $a Archeological investigations at two sites in Dinosaur National Monument $h [microform] : $b 42UN1724 and 5MF2645 / $c by James A. Truesdale.
   245 00 $a Artist With Dinosaur Model $h [electronic resource].
   245 10 $a Atlas of dinosaur adventures / $c illustrated by Lucy Letherland ; written by Emily Hawkins.
   245 10 $a Auks, rocks, and the odd dinosaur : $b inside stories from the Smithsonian's Museum of Natural History / $c Peggy Thomson.

How cool is that!

OPAC
------------------------------------

The University of Alabama Catalog also support the OPAC format, which can be useful for finding the
library location or checking if a book is available:

.. code-block:: shell

   open library.ua.edu:7090/voyager
   find @1=4 "core python programming"
   format opac
   show 1

**Output:**

.. code-block:: shell

   ...
   ...
   ...
   Data holdings 0
   typeOfRecord: x
   encodingLevel: 1
   receiptAcqStatus: 2
   generalRetention: 8
   completeness: 4
   dateOfReport: 000000
   nucCode: sel
   localLocation: Science & Engineering Library
   callNumber: QA76.73.P98 C48 2007
   circulation 0
   availableNow: 1
   itemId: 2359071
   renewable: 0
   onHold: 0
   nextResultSetPosition = 2
   Elapsed: 0.060914

.. note

   The ``availableNow: 1`` is equivalent to True. If the book is not available, this value will be 0 for False.

So here is a fun example, let's look at the availability of
print books in the C (Computer program language) subject heading:

.. code-block:: shell

   printf "open library.ua.edu:7090/voyager\nfind @not @attr 1=21 \"C (Computer program language)\" \
   @attr 1=1016 \"electronic resource\"\nformat opac\nshow 1+10\n" | \
   yaz-client -f /dev/stdin | grep --text -e "^245" -e "callNumber" -e "availableNow" -e "localLocation"

**Output:**

.. code-block:: shell

   245 10 $a Applications of numerical techniques with C / $c Suresh Chandra.
   localLocation: Archival Facility (use Request Item button for retrieval)
   callNumber: QA297 .C49 2006
   availableNow: 1
   localLocation: Science & Engineering Library
   callNumber: QA297 .C49 2006
   availableNow: 1
   245 10 $a Artificial intelligence using C / $c Herbert Schildt.
   localLocation: Science & Engineering Library
   callNumber: Q336 .S35 1987
   availableNow: 1
   245 12 $a A book on C : $b programming in C / $c Al Kelley, Ira Pohl.
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 K44 1998
   availableNow: 1
   245 10 $a C.
   localLocation: Gorgas Library Gov. Doc.
   callNumber: C 13.52:160
   availableNow: 1
   245 10 $a C & C++ code capsules : $b a guide for practitioners / $c Chuck Allison ; [foreword by Bruce Eckel].
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 A44 1998
   availableNow: 1
   245 10 $a C, an introduction to programming / $c Jim Keogh, Peter Aitken, Bradley L. Jones.
   localLocation: Gorgas Library
   callNumber: QA76.73.C15 K466; 1996
   availableNow: 1
   245 14 $a The C and UNIX dictionary : $b from absolute pathname to Zombie / $c Kaare Christian.
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 C49 1988
   availableNow: 1
   245 10 $a C/C++ programmers reference / $c Herbert Schildt.
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 S348; 1997
   availableNow: 0
   245 10 $a C for programmers : $b a complete tutorial based on the ANSI standard / $c Leendert Ammeraal.
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 A46; 1991
   availableNow: 1
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 A46; 1991
   availableNow: 1
   245 10 $a C in a nutshell / $c Peter Prinz and Tony Crawford.
   localLocation: Science & Engineering Library
   callNumber: QA76.73.C15 P74 2016
   availableNow: 1

Saving Raw MARC data
------------------------------

If you are looking to process or parse MARC records with software designed for MARC,
you probably want the Raw binary MARC. In that case, you can
use the ``yaz-client set_marcdump`` command to save the results to a named binary MARC file:

.. code-block:: shell

   open library.ua.edu:7090/voyager
   find @not @attr 1=21 "C (Computer program language)" @attr 1=1016 "electronic resource"
   set_marcdump C_books.marc
   show 1+10
   quit

If you have multiple queries and want to use a loop as shown in above to save MARC data, here
is one potential workflow that would print human-readable MARC to the terminal output and
save a file, isbn_records.marc, with the Raw binary MARC data:

.. code-block:: shell

   cat isbns.txt |
   while read isbn
   do
      printf "open library.ua.edu:7090/voyager\nfind @1=7 "$isbn"\nshow 1\nsleep 1\n"
   done | yaz-client -f /dev/stdin -m isbn_records.marc

4. z39-demo - A small Z39.50 C program using the YAZ toolkit
================================================================

Since Index Data YAZ is a complete toolkit, it's possible to write your own custom Z39.50 software.

As a result, we created a demonstration C program called ``z39-demo``:

https://github.com/ualibweb/z39-demo

``z39-demo`` is a small command line program written in C that can run Z39.50 searches via an input query
or an input file. By default ``z39-demo`` searches The University of Alabama library catalog, 
but it can accept different Z39.50 connections as an option. ``z39-demo`` is not as feature complete as ``yaz-client``,
but it offers a few conveniences and was a lot of fun to design and program.

.. warning::

   Consider ``z39.50-demo`` as an experiment program, it's not well-tested. 

Dependencies
----------------

This will vary depending on your operating system and environment,
however, here were the dependencies and related software we installed on Ubuntu 22.04 LTS:

.. code-block:: shell

   sudo apt-get install build-essential manpages-dev glibc-doc gcc-doc make-doc

Compiling from source
-------------------------

There are several possible workflows for compiling the ``z39-demo`` program,
here is one method that worked well for us on Ubuntu based linux:

1. Download latest yaz_5.**.orig.tar.gz from: https://ftp.indexdata.com/pub/yaz/ubuntu/jammy/
2. Unarchive folder, then:

.. code-block:: shell

   cd yaz_5.33.0
   ./configure
   make

Running z39-demo
-------------------

Shown below is the command to output the help file for the program in the terminal.

.. code-block:: shell

   ./z39-demo -h
   
**Output:**
   
.. code-block:: shell
   
   usage: myprogram [-h] [-z] [-o] -q/-i FILE

   z39-demo is a command line program that can run Z39.50 searches via an input query or an input file

   positional arguments:
      -q           required query for Z39.50 search
      -i FILE      input file with queries for Z39.50 with one per line
   optional arguments
      -h, -help    show help and exit
      -z           optional custom Z39.50 adress; default is University of Alabama Libraries Catalog
      -o FILE      optional to output binary MARC file (FILE 100 chars max.); default is to print to stdout in MARC ASCII format 
      -n number    optional to output specified number of results; default is to print the maximum number of results 


To search a query, use ``-q`` and specify a number of returned results with ``-n`` followed by a space and digit(s). 
By default, the UA Libraries Catalog is searched, and the returned MARC record(s) are sent to stdout:

.. code-block:: shell

   ./z39-demo -q "@attr 1=4 @attr 3=1 @attr 5=1 \"dinosaur\"" -n 1

**Output:**

.. code-block:: shell
   
   02574cem  2200529 i 4500
   001 7845907
   005 20181211165730.0
   007 aj canzn
   008 180604s2018    dcubg     a  f  0   eng c
   034 1  $a a $b 150000 $d W1092111 $e W1082735 $f N0444580 $g N0401450
   035    $a (marcive)tmp97451983
   035    $a (OCoLC)on1037101363
   035    $a 7845907
   040    $a GPO $b eng $e rda $c GPO $d MvI $d UtOrBLW
   042    $a pcc
   043    $a n-us-co $a n-us-ut
   049    $a GPBS
   052    $a 4311 $b D5
   052    $a 4341
   074    $a 0650
   086 0  $a I 29.21:D 61/2018
   110 1  $a United States. $b National Park Service, $e cartographer. $0 http://id.loc.gov/authorities/names/n79022809
   245 10 $a Dinosaur National Monument, Colorado/Utah / $c National Park Service, U.S. Department of the Interior.
   246 1  $i Alternative title: $a Dinosaur
   246 1  $i Title from verso: $a Visiting Dinosaur National Monument
   250    $a Last updated 2018.
   255    $a Scale approximately 1:150,000 $c (W 109�21'11"--W 108�27'35"/N 40�44'58"--N 40�14'50").
   264  1 $a [Washington, D.C.] : $b National Park Service, U.S. Department of the Interior, $c [2018]
   300    $a 1 map : $b color ; $c 58 x 43 cm, on sheet 60 x 43 cm, folded to 10 x 22 cm
   336    $a cartographic image $b cri $2 rdacontent
   337    $a unmediated $b n $2 rdamedia
   338    $a sheet $b nb $2 rdacarrier
   500    $a "*GPO: 2018--403-332/82048."
   500    $a Shipping list no.: 2018-0235-P.
   500    $a Title from panel.
   500    $a Relief shown by shading and spot heights.
   500    $a Includes text, timeline, area map, and color illustrations.
   500    $a Text, points of interest, and color illustrations on verso.
   650  0 $a National monuments $z Colorado $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85090029
   650  0 $a National monuments $z Utah $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85090039
   650  0 $a National parks and reserves $z Colorado $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85090065
   650  0 $a National parks and reserves $z Utah $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85090104
   650  0 $a Hiking $z Colorado $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85060793
   650  0 $a Hiking $z Utah $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85060793
   651  0 $a Dinosaur National Monument (Colo. and Utah) $v Maps. $0 http://id.loc.gov/authorities/subjects/sh85038092
   655  7 $a Maps. $2 lcgft $0 http://id.loc.gov/authorities/genreForms/gf2011026387
   655  7 $a Tourist maps. $2 lcgft $0 http://id.loc.gov/authorities/genreForms/gf2011026699

Here is how to output and save the binary MARC to a file:

.. code-block:: shell

   ./z39-demo -q "@attr 1=4 @attr 3=1 @attr 5=1 \"dinosaur\"" -o test.marc -n 1

To use a custom server address, add ``-z`` along with the address to search the query.

Here is an example with the National Library of Medicine:

https://support.nlm.nih.gov/knowledgebase/article/KA-04188/en-us

.. code-block:: shell
   
   ./z39-demo -z "na91.alma.exlibrisgroup.com:1921/01NLM_INST" -q "@1=4 dinosaur" -n 1

**Output:**

.. code-block:: shell
   
   00898cam a2200313 a 4500
   001 997189573406676
   005 20211203201556.0
   008 920813s1991    xxu||||  |||| 00||0|eng  
   010    $a 90-55948
   020    $a 9780060165383
   020    $a 0060165383
   035    $9 9212018
   035    $a (OCoLC)23141186
   040    $a DNLM $c DNLM
   041 0  $a eng
   044    $9 United States
   060 00 $a WM 203 $b B351d 1991
   100 1  $a Baur, Susan.
   245 14 $a The dinosaur man : $b tales of madness and enchantment from the back ward / $c Susan Baur.
   260    $a New York, N.Y. : $b Edward Burlingame Books, $c c1991.
   300    $a ix, 203 p. : $b ill.
   336    $a text $b txt $2 rdacontent
   337    $a unmediated $b n $2 rdamedia
   338    $a volume $b nc $2 rdacarrier
   650  2 $a Schizophrenia
   655  2 $a Popular Work
   935    $a (DNLM)718957-nlmdb
   995    $a AUTH $b 19920813 $c REV $d 20181116
   999    $a AUTH

To search multiple queries, put your search strings in a file with one per line:

.. code-block:: shell

   cat mysearches

**Output:**

.. code-block:: shell

   @1=7 1683925041
   @1=7 9780470183014
   @1=7 1565925858
   @1=7 9780136778851
   @1=7 1785284444

.. code-block:: shell

   ./z39-demo -i mysearches

.. rubric:: References

.. [#ref0] Ward, M. Expanding Access to Information with Z39.50. American Libraries 1994, 25 (7), 639-641. `<http://www.jstor.org/stable/25633315>`_

.. [#ref1] Lynch, C. A. The Z39. 50 Information Retrieval Standard. D-lib Magazine 1997, 3 (4). `<http://dlib.org/dlib/april97/04lynch.html>`_

.. [#ref2] Needleman, M. Z39.50 - a Review, Analysis and Some Thoughts on the Future. Library Hi Tech 2000, 18 (2), 158-165. `<https://doi.org/10.1108/07378830010333545>`_.

.. [#ref3] Z39.50 Implementation Experiences. NIST Special Publication 500-229. `<https://purl.fdlp.gov/GPO/gpo100304>`_.

.. [#ref4] `<https://www.loc.gov/z3950/agency/>`_.

.. [#ref5] `<https://www.loc.gov/z3950/agency/Z39-50-2003.pdf>`_

.. [#ref6] `<https://unix.stackexchange.com/questions/505828/how-to-pass-a-string-to-a-command-that-expects-a-file>`_

.. [#ref7] `<https://github.com/indexdata/yaz/issues/97>`_
