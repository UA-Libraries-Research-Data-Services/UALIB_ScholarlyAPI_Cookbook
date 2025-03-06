CAS Common Chemistry API in Bash
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

by Avery Fernandez and Vincent F. Scalfani

These recipe examples were tested on April 1, 2022 using GNOME Terminal (with Bash 4.4.20) in Ubuntu 18.04.

**CAS Common Chemistry API Documentation (requires registration):** https://www.cas.org/services/commonchemistry-api

**Attribution:** This tutorial uses the `CAS Common Chemistry API`_. Example data shown is licensed under the `CC BY-NC 4.0 license`_.

.. _CAS Common Chemistry API: https://commonchemistry.cas.org/
.. _CC BY-NC 4.0 license: https://creativecommons.org/licenses/by-nc/4.0/

Program requirements
=========================

In order to run this code, you will need to first install `curl`_, `jq`_, and `gnuplot`_. curl is used to request the data from the API, jq is used to parse the JSON data, and gnuplot is used to plot the data. In addition, if you want to be able to print the molecules as ASCII characters in your terminal, you will need to install `RDKit`_ and download the `print_mols`_ Python script.

.. _curl: https://github.com/curl/curl
.. _jq: https://stedolan.github.io/jq/
.. _gnuplot: http://www.gnuplot.info/
.. _RDKit: https://www.rdkit.org/
.. _print_mols: https://github.com/vfscalfani/teletype_mols

1. Common Chemistry Record Detail Retrieval
============================================

Information about substances in CAS Common Chemistry can be retrieved using the ``/detail`` API and a CAS RN identifier:

Setup API parameters
---------------------

Create variables for the CAS Common Chemistry detail base URL and an example CAS RN (10094-36-7, ethyl cyclohexanepropionate):

.. code-block:: shell

   detail_base_url="https://commonchemistry.cas.org/api/detail?"
   casrn1="10094-36-7"

Request data from CAS Common Chemistry Detail API
--------------------------------------------------

.. code-block:: shell

   casrn1_data=$(curl $detail_base_url$"cas_rn="$casrn1)

View data
----------

.. code-block:: shell

   echo "$casrn1_data" | jq '.'

**Output:**

.. code-block:: shell

   {
     "uri": "substance/pt/10094367",
     "rn": "10094-36-7",
     "name": "Ethyl cyclohexanepropionate",
     "image": "<svg width=\"228.6\" viewBox=\"0 0 7620 3716\" text-rendering=\"auto\" stroke-width=\"1\" stroke-opacity=\"1\" stroke-miterlimit=\"10\" stroke-linejoin=\"miter\" stroke-linecap=\"square\" stroke-dashoffset=\"0\" stroke-dasharray=\"none\" stroke=\"black\" shape-rendering=\"auto\" image-rendering=\"auto\" height=\"111.48\" font-weight=\"normal\" font-style=\"normal\" font-size=\"12\" font-family=\"'Dialog'\" fill-opacity=\"1\" fill=\"black\" color-rendering=\"auto\" color-interpolation=\"auto\" xmlns=\"http://www.w3.org/2000/svg\"><g><g stroke=\"white\" fill=\"white\"><rect y=\"0\" x=\"0\" width=\"7620\" stroke=\"none\" height=\"3716\"/></g><g transform=\"translate(32866,32758)\" text-rendering=\"geometricPrecision\" stroke-width=\"44\" stroke-linejoin=\"round\" stroke-linecap=\"round\"><line y2=\"-30850\" y1=\"-31419\" x2=\"-30792\" x1=\"-31777\" fill=\"none\"/><line y2=\"-29715\" y1=\"-30850\" x2=\"-30792\" x1=\"-30792\" fill=\"none\"/><line y2=\"-31419\" y1=\"-30850\" x2=\"-31777\" x1=\"-32762\" fill=\"none\"/><line y2=\"-29146\" y1=\"-29715\" x2=\"-31777\" x1=\"-30792\" fill=\"none\"/><line y2=\"-30850\" y1=\"-29715\" x2=\"-32762\" x1=\"-32762\" fill=\"none\"/><line y2=\"-29715\" y1=\"-29146\" x2=\"-32762\" x1=\"-31777\" fill=\"none\"/><line y2=\"-31376\" y1=\"-30850\" x2=\"-29885\" x1=\"-30792\" fill=\"none\"/><line y2=\"-30850\" y1=\"-31376\" x2=\"-28978\" x1=\"-29885\" fill=\"none\"/><line y2=\"-31376\" y1=\"-30850\" x2=\"-28071\" x1=\"-28978\" fill=\"none\"/><line y2=\"-30960\" y1=\"-31376\" x2=\"-27352\" x1=\"-28071\" fill=\"none\"/><line y2=\"-31376\" y1=\"-30960\" x2=\"-26257\" x1=\"-26976\" fill=\"none\"/><line y2=\"-30850\" y1=\"-31376\" x2=\"-25350\" x1=\"-26257\" fill=\"none\"/><line y2=\"-32202\" y1=\"-31376\" x2=\"-28140\" x1=\"-28140\" fill=\"none\"/><line y2=\"-32202\" y1=\"-31376\" x2=\"-28002\" x1=\"-28002\" fill=\"none\"/><text y=\"-30671\" xml:space=\"preserve\" x=\"-27317\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">O</text><text y=\"-32242\" xml:space=\"preserve\" x=\"-28224\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">O</text></g></g></svg>",
     "inchi": "InChI=1S/C11H20O2/c1-2-13-11(12)9-8-10-6-4-3-5-7-10/h10H,2-9H2,1H3",
     "inchiKey": "InChIKey=NRVPMFHPHGBQLP-UHFFFAOYSA-N",
     "smile": "C(CC(OCC)=O)C1CCCCC1",
     "canonicalSmile": "O=C(OCC)CCC1CCCCC1",
     "molecularFormula": "C<sub>11</sub>H<sub>20</sub>O<sub>2</sub>",
     "molecularMass": "184.28",
     "experimentalProperties": [
       {
         "name": "Boiling Point",
         "property": "105-113 °C @ Press: 17 Torr",
         "sourceNumber": 1
       }
     ],
     "propertyCitations": [
       {
         "docUri": "document/pt/document/22252593",
         "sourceNumber": 1,
         "source": "De Benneville, Peter L.; Journal of the American Chemical Society, (1940), 62, 283-7, CAplus"
       }
     ],
     "synonyms": [
       "Cyclohexanepropanoic acid, ethyl ester",
       "Cyclohexanepropionic acid, ethyl ester",
       "Ethyl cyclohexanepropionate",
       "Ethyl cyclohexylpropanoate",
       "Ethyl 3-cyclohexylpropionate",
       "Ethyl 3-cyclohexylpropanoate",
       "3-Cyclohexylpropionic acid ethyl ester",
       "NSC 71463",
       "Ethyl 3-cyclohexanepropionate"
     ],
     "replacedRns": [],
     "hasMolfile": true
   }

Display a Molecule Drawing
---------------------------

For displaying the molecule drawing, we could extract out the SVG image string and display the SVG in an image viewer program, however since we are working within a terminal without graphics, we will instead extract out the SMILES and pipe these to a `print_mols`_ Python script, which uses the cheminformatics program RDKit to parse the SMILES, compute drawing coordinates, and then print the molecule as ASCII characters:

.. code-block:: shell

   echo "$casrn1_data" | jq '.["smile"]' | tr -d '"' | python3 print_mols.py -


**Output:**

.. code-block:: shell

                           O                                                    
                                                                                 
                           *                                                    
                                                                                
           C               C                   C               C                
                                                                                
       *       *       *       *         *         *       *       *            
                                                                                
   C               O               C                   C               C        
                                                                                
                                                       *               *        
                                                                                
                                                       C               C        
                                                           *       *            
                                                               C               

.. note::

   ``jq '.["smile"]'`` extracts out the SMILES string in the smile field; ``tr -d '"'`` removes the quotes; ``python3 print_mols.py -`` prints the molecule.


Select some specific data
---------------------------

Get Experimental Properties:

.. code-block:: shell

   echo $casrn1_data | jq '.["experimentalProperties"][0]'

**Output:**

.. code-block:: shell

   {
     "name": "Boiling Point",
     "property": "105-113 °C @ Press: 17 Torr",
     "sourceNumber": 1
   }


Get Boiling Point property:

.. code-block:: shell

   echo $casrn1_data | jq '.["experimentalProperties"][0]["property"]'

**Output:**

.. code-block:: shell

   "105-113 °C @ Press: 17 Torr"

Get InChIKey:

.. code-block:: shell

   echo $casrn1_data | jq '.["inchiKey"]'

**Output:**

.. code-block:: shell

   "InChIKey=NRVPMFHPHGBQLP-UHFFFAOYSA-N"

Get Canonical SMILES:

.. code-block:: shell

   echo $casrn1_data | jq '.["canonicalSmile"]'

**Output:**

.. code-block:: shell

   "O=C(OCC)CCC1CCCCC1"


2.  Common Chemistry API record detail retrieval in a loop
===========================================================

Setup API parameters
--------------------

.. code-block:: shell

   detail_base_url="https://commonchemistry.cas.org/api/detail?"
   declare -a casrn_list=("10094-36-7" "10031-92-2" "10199-61-8" "10036-21-2" "1019020-13-3")
   echo "${casrn_list[@]}"

**Output:**

.. code-block:: shell

   10094-36-7 10031-92-2 10199-61-8 10036-21-2 1019020-13-3

Request data for each CAS RN and save to an array
--------------------------------------------------

.. code-block:: shell

   declare -a casrn_data
   for casrn in "${casrn_list[@]}"
   do
     data=$(curl $detail_base_url$"cas_rn="$casrn)
     casrn_data+=("$data")
     sleep 1
   done

View the first record:

.. code-block:: shell

   echo "${casrn_data[0]}" | jq '.'

**Output:**

.. code-block:: shell

   {
     "uri": "substance/pt/10094367",
     "rn": "10094-36-7",
     "name": "Ethyl cyclohexanepropionate",
     "image": "<svg width=\"228.6\" viewBox=\"0 0 7620 3716\" text-rendering=\"auto\" stroke-width=\"1\" stroke-opacity=\"1\" stroke-miterlimit=\"10\" stroke-linejoin=\"miter\" stroke-linecap=\"square\" stroke-dashoffset=\"0\" stroke-dasharray=\"none\" stroke=\"black\" shape-rendering=\"auto\" image-rendering=\"auto\" height=\"111.48\" font-weight=\"normal\" font-style=\"normal\" font-size=\"12\" font-family=\"'Dialog'\" fill-opacity=\"1\" fill=\"black\" color-rendering=\"auto\" color-interpolation=\"auto\" xmlns=\"http://www.w3.org/2000/svg\"><g><g stroke=\"white\" fill=\"white\"><rect y=\"0\" x=\"0\" width=\"7620\" stroke=\"none\" height=\"3716\"/></g><g transform=\"translate(32866,32758)\" text-rendering=\"geometricPrecision\" stroke-width=\"44\" stroke-linejoin=\"round\" stroke-linecap=\"round\"><line y2=\"-30850\" y1=\"-31419\" x2=\"-30792\" x1=\"-31777\" fill=\"none\"/><line y2=\"-29715\" y1=\"-30850\" x2=\"-30792\" x1=\"-30792\" fill=\"none\"/><line y2=\"-31419\" y1=\"-30850\" x2=\"-31777\" x1=\"-32762\" fill=\"none\"/><line y2=\"-29146\" y1=\"-29715\" x2=\"-31777\" x1=\"-30792\" fill=\"none\"/><line y2=\"-30850\" y1=\"-29715\" x2=\"-32762\" x1=\"-32762\" fill=\"none\"/><line y2=\"-29715\" y1=\"-29146\" x2=\"-32762\" x1=\"-31777\" fill=\"none\"/><line y2=\"-31376\" y1=\"-30850\" x2=\"-29885\" x1=\"-30792\" fill=\"none\"/><line y2=\"-30850\" y1=\"-31376\" x2=\"-28978\" x1=\"-29885\" fill=\"none\"/><line y2=\"-31376\" y1=\"-30850\" x2=\"-28071\" x1=\"-28978\" fill=\"none\"/><line y2=\"-30960\" y1=\"-31376\" x2=\"-27352\" x1=\"-28071\" fill=\"none\"/><line y2=\"-31376\" y1=\"-30960\" x2=\"-26257\" x1=\"-26976\" fill=\"none\"/><line y2=\"-30850\" y1=\"-31376\" x2=\"-25350\" x1=\"-26257\" fill=\"none\"/><line y2=\"-32202\" y1=\"-31376\" x2=\"-28140\" x1=\"-28140\" fill=\"none\"/><line y2=\"-32202\" y1=\"-31376\" x2=\"-28002\" x1=\"-28002\" fill=\"none\"/><text y=\"-30671\" xml:space=\"preserve\" x=\"-27317\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">O</text><text y=\"-32242\" xml:space=\"preserve\" x=\"-28224\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">O</text></g></g></svg>",
     "inchi": "InChI=1S/C11H20O2/c1-2-13-11(12)9-8-10-6-4-3-5-7-10/h10H,2-9H2,1H3",
     "inchiKey": "InChIKey=NRVPMFHPHGBQLP-UHFFFAOYSA-N",
     "smile": "C(CC(OCC)=O)C1CCCCC1",
     "canonicalSmile": "O=C(OCC)CCC1CCCCC1",
     "molecularFormula": "C<sub>11</sub>H<sub>20</sub>O<sub>2</sub>",
     "molecularMass": "184.28",
     "experimentalProperties": [
       {
         "name": "Boiling Point",
         "property": "105-113 °C @ Press: 17 Torr",
         "sourceNumber": 1
       }
     ],
     "propertyCitations": [
       {
         "docUri": "document/pt/document/22252593",
         "sourceNumber": 1,
         "source": "De Benneville, Peter L.; Journal of the American Chemical Society, (1940), 62, 283-7, CAplus"
       }
     ],
     "synonyms": [
       "Cyclohexanepropanoic acid, ethyl ester",
       "Cyclohexanepropionic acid, ethyl ester",
       "Ethyl cyclohexanepropionate",
       "Ethyl cyclohexylpropanoate",
       "Ethyl 3-cyclohexylpropionate",
       "Ethyl 3-cyclohexylpropanoate",
       "3-Cyclohexylpropionic acid ethyl ester",
       "NSC 71463",
       "Ethyl 3-cyclohexanepropionate"
     ],
     "replacedRns": [],
     "hasMolfile": true
   }

Display Molecule Drawings
-------------------------------

We can use a similar technique to display the molecules as shown above. We will first extract out the SMILES strings then print them as ASCII characters using the `print_mols`_ Python script.

.. code-block:: shell

   for data in "${!casrn_data[@]}"
   do
     echo "${casrn_data[$data]}" | jq '.["smile"]' | tr -d '"' | python3 print_mols.py -
   done

**Output:**

.. code-block:: shell

                                                                                
                           O                                                    
                                                                                
                           *                                                    
                                                                                
           C               C                   C               C                
                                                                                
       *       *       *       *         *         *       *       *            
                                                                                
   C               O               C                   C               C        
                                                                                
                                                       *               *        
                                                                                
                                                       C               C        
                                                           *       *            
                                                               C                
                                                                                
                                                                                
                                                                                
                                                                                
                                                                                
                                                   O                            
                                                                                
                                                   *                            
                                                                                
                                                   C           C                
                                                 *     *     *     *            
                                               C           O           C        
                                           *                                    
   C           C           C           C                                        
     *     *       *     *     *     *                                          
       C               C           C                                            
                                                                                
                                                                                
                                                                            
                                                                            
                                                                            
       C                           O                                        
             *                                                              
     *             C                                                        
                                   *                                        
   C               *                                                        
                                                                            
       *           N               C                   C                    
               *       *       *       *         *         *                
           N               C               O                   C            
                                                                            
                                                                            
   
                                                                                
                                                                                
                                                                                
                   O                                   O                        
                                                                                
                   *                                   *                        
                                                                                
       C           C           C           C           C       C                
     *     *     *     *     *   *     *     *     *     *   *     *            
   C           O           C       C           C           O           C        
                                   *           *                                
                                   C           C                                
                                       *     *                                  
                                           C                                    
                                                                                
                                                                                
                                                                        
                           N                                            
                                                                        
                           *                                            
                                                                        
           C               C                   C                        
                                                                        
       *       *       *         *         *       *                    
                                                                        
   C               O                   C               C                
                                                                        
                                       *               *                
                                                                        
                                       C               C                
                                           *       *                    
                                               C                        
                                                                        
                                                                     
Select some specific data
---------------------------                                                             
                                                                     
Get canonical SMILES:

.. code-block:: shell                                                                 
                                                                  
   declare -a cansmiles
   for data in "${!casrn_data[@]}"
   do
     cansmiles+=("$(echo "${casrn_data[$data]}" | jq '.["canonicalSmile"]')")
   done
   echo "${cansmiles[@]}"

**Output:**

.. code-block:: shell                                                        
                                                                     
   "O=C(OCC)CCC1CCCCC1" "O=C(C#CCCCCCC)OCC" "O=C(OCC)CN1N=CC=C1" "O=C(OCC)C1=CC=CC(=C1)CCC(=O)OCC" "N=C(OCC)C1=CCCCC1"

Get synonyms:

.. code-block:: shell 

   declare -a synonyms_list
   for data in "${!casrn_data[@]}"
   do
     synonyms_list+=("$(echo "${casrn_data[$data]}" | jq '.["synonyms"]')")
   done
   echo "${synonyms_list[@]}"

**Output:**

.. code-block:: shell

   [
     "Cyclohexanepropanoic acid, ethyl ester",
     "Cyclohexanepropionic acid, ethyl ester",
     "Ethyl cyclohexanepropionate",
     "Ethyl cyclohexylpropanoate",
     "Ethyl 3-cyclohexylpropionate",
     "Ethyl 3-cyclohexylpropanoate",
     "3-Cyclohexylpropionic acid ethyl ester",
     "NSC 71463",
     "Ethyl 3-cyclohexanepropionate"
   ] [
     "2-Nonynoic acid, ethyl ester",
     "Ethyl 2-nonynoate",
     "NSC 190985"
   ] [
     "1<em>H</em>-Pyrazole-1-acetic acid, ethyl ester",
     "Pyrazole-1-acetic acid, ethyl ester",
     "Ethyl 1<em>H</em>-pyrazole-1-acetate",
     "Ethyl 1-pyrazoleacetate",
     "Ethyl 2-(1<em>H</em>-pyrazol-1-yl)acetate"
   ] [
     "Benzenepropanoic acid, 3-(ethoxycarbonyl)-, ethyl ester",
     "Hydrocinnamic acid, <em>m</em>-carboxy-, diethyl ester",
     "Ethyl 3-(ethoxycarbonyl)benzenepropanoate"
   ] [
     "1-Cyclohexene-1-carboximidic acid, ethyl ester",
     "Ethyl 1-cyclohexene-1-carboximidate"
   ]

Transform synonym array of lists to a flat structure:

.. code-block:: shell

   declare -a synonyms_flat
   for data in "${!casrn_data[@]}"
   do
     # loops through each list and grabs their data
     for (( i = 0 ; i < $(echo "${casrn_data[$data]}" | jq '.["synonyms"] | length') ; i++))
     do
       synonyms_flat+=("$(echo "${casrn_data[$data]}" | jq ".synonyms[$i]")")
     done
   done
   echo "${synonyms_flat[@]}"

**Output:**

.. code-block:: shell

   "Cyclohexanepropanoic acid, ethyl ester" "Cyclohexanepropionic acid, ethyl ester" "Ethyl cyclohexanepropionate" "Ethyl cyclohexylpropanoate" "Ethyl 3-cyclohexylpropionate" "Ethyl 3-cyclohexylpropanoate" "3-Cyclohexylpropionic acid ethyl ester" "NSC 71463" "Ethyl 3-cyclohexanepropionate" "2-Nonynoic acid, ethyl ester" "Ethyl 2-nonynoate" "NSC 190985" "1<em>H</em>-Pyrazole-1-acetic acid, ethyl ester" "Pyrazole-1-acetic acid, ethyl ester" "Ethyl 1<em>H</em>-pyrazole-1-acetate" "Ethyl 1-pyrazoleacetate" "Ethyl 2-(1<em>H</em>-pyrazol-1-yl)acetate" "Benzenepropanoic acid, 3-(ethoxycarbonyl)-, ethyl ester" "Hydrocinnamic acid, <em>m</em>-carboxy-, diethyl ester" "Ethyl 3-(ethoxycarbonyl)benzenepropanoate" "1-Cyclohexene-1-carboximidic acid, ethyl ester" "Ethyl 1-cyclohexene-1-carboximidate"

3. Common Chemistry Search
============================

In addition to the ``/detail`` API, the CAS Common Chemistry API has a ``/search`` method that allows searching by CAS RN, SMILES, InChI/InChIKey, and name.

Setup API Parameters
----------------------

The InChIKey is an example and is Quinine:

.. code-block:: shell

   search_base_url="https://commonchemistry.cas.org/api/search?q="
   IK="InChIKey=LOUPRKONTZGTKE-WZBLMQSHSA-N"

Request data from CAS Common Chemistry Search API
---------------------------------------------------

Search query:

.. code-block:: shell

   quinine_search_data=$(curl $search_base_url$IK)
   echo "$quinine_search_data" | jq '.'

**Output:**

.. code-block:: shell

   {
     "count": 1,
     "results": [
       {
         "rn": "130-95-0",
         "name": "Quinine",
         "image": "<svg width=\"309.3\" viewBox=\"0 0 10310 5592\" text-rendering=\"auto\" stroke-width=\"1\" stroke-opacity=\"1\" stroke-miterlimit=\"10\" stroke-linejoin=\"miter\" stroke-linecap=\"square\" stroke-dashoffset=\"0\" stroke-dasharray=\"none\" stroke=\"black\" shape-rendering=\"auto\" image-rendering=\"auto\" height=\"167.76\" font-weight=\"normal\" font-style=\"normal\" font-size=\"12\" font-family=\"'Dialog'\" fill-opacity=\"1\" fill=\"black\" color-rendering=\"auto\" color-interpolation=\"auto\" xmlns=\"http://www.w3.org/2000/svg\"><g><g stroke=\"white\" fill=\"white\"><rect y=\"0\" x=\"0\" width=\"10310\" stroke=\"none\" height=\"5592\"/></g><g transform=\"translate(32866,32758)\" text-rendering=\"geometricPrecision\" stroke-width=\"44\" stroke-linejoin=\"round\" stroke-linecap=\"round\"><line y2=\"-28559\" y1=\"-28036\" x2=\"-26635\" x1=\"-25742\" fill=\"none\"/><line y2=\"-29819\" y1=\"-28559\" x2=\"-26635\" x1=\"-26635\" fill=\"none\"/><line y2=\"-28036\" y1=\"-28559\" x2=\"-25367\" x1=\"-24474\" fill=\"none\"/><line y2=\"-30451\" y1=\"-29819\" x2=\"-25555\" x1=\"-26635\" fill=\"none\"/><line y2=\"-28559\" y1=\"-29819\" x2=\"-24474\" x1=\"-24474\" fill=\"none\"/><line y2=\"-29504\" y1=\"-28828\" x2=\"-25194\" x1=\"-26005\" fill=\"none\"/><line y2=\"-29819\" y1=\"-30451\" x2=\"-24474\" x1=\"-25555\" fill=\"none\"/><line y2=\"-29082\" y1=\"-28559\" x2=\"-27542\" x1=\"-26635\" fill=\"none\"/><line y2=\"-29819\" y1=\"-30344\" x2=\"-22660\" x1=\"-23567\" fill=\"none\"/><line y2=\"-29700\" y1=\"-30223\" x2=\"-22729\" x1=\"-23636\" fill=\"none\"/><line y2=\"-28779\" y1=\"-29082\" x2=\"-28071\" x1=\"-27542\" fill=\"none\"/><line y2=\"-30703\" y1=\"-30131\" x2=\"-28524\" x1=\"-27542\" fill=\"none\"/><line y2=\"-31850\" y1=\"-30703\" x2=\"-28524\" x1=\"-28524\" fill=\"none\"/><line y2=\"-31705\" y1=\"-30847\" x2=\"-28354\" x1=\"-28354\" fill=\"none\"/><line y2=\"-30131\" y1=\"-30703\" x2=\"-29507\" x1=\"-28524\" fill=\"none\"/><line y2=\"-30131\" y1=\"-30703\" x2=\"-27542\" x1=\"-26560\" fill=\"none\"/><line y2=\"-30347\" y1=\"-30778\" x2=\"-27505\" x1=\"-26768\" fill=\"none\"/><line y2=\"-31850\" y1=\"-32422\" x2=\"-28524\" x1=\"-29507\" fill=\"none\"/><line y2=\"-32312\" y1=\"-31850\" x2=\"-27730\" x1=\"-28524\" fill=\"none\"/><line y2=\"-30703\" y1=\"-30131\" x2=\"-30489\" x1=\"-29507\" fill=\"none\"/><line y2=\"-30778\" y1=\"-30347\" x2=\"-30281\" x1=\"-29544\" fill=\"none\"/><line y2=\"-30703\" y1=\"-31850\" x2=\"-26560\" x1=\"-26560\" fill=\"none\"/><line y2=\"-32422\" y1=\"-31850\" x2=\"-29507\" x1=\"-30489\" fill=\"none\"/><line y2=\"-32205\" y1=\"-31774\" x2=\"-29544\" x1=\"-30281\" fill=\"none\"/><line y2=\"-31850\" y1=\"-32312\" x2=\"-26560\" x1=\"-27354\" fill=\"none\"/><line y2=\"-31760\" y1=\"-32107\" x2=\"-26745\" x1=\"-27340\" fill=\"none\"/><line y2=\"-31850\" y1=\"-30703\" x2=\"-30489\" x1=\"-30489\" fill=\"none\"/><line y2=\"-30275\" y1=\"-30703\" x2=\"-31200\" x1=\"-30489\" fill=\"none\"/><line y2=\"-30541\" y1=\"-30272\" x2=\"-32040\" x1=\"-31575\" fill=\"none\"/><polygon stroke-width=\"1\" stroke=\"none\" points=\" -24474 -29819 -23602 -30402 -23532 -30284\"/><polygon stroke-width=\"1\" points=\" -24474 -29819 -23602 -30402 -23532 -30284\" fill=\"none\"/><polygon stroke-width=\"1\" stroke=\"none\" points=\" -26635 -28559 -26973 -27837 -27092 -27903\"/><polygon stroke-width=\"1\" points=\" -26635 -28559 -26973 -27837 -27092 -27903\" fill=\"none\"/><line y2=\"-28860\" y1=\"-28796\" x2=\"-25945\" x1=\"-26066\" fill=\"none\"/><line y2=\"-28657\" y1=\"-28611\" x2=\"-25865\" x1=\"-25952\" fill=\"none\"/><line y2=\"-28454\" y1=\"-28427\" x2=\"-25785\" x1=\"-25838\" fill=\"none\"/><line y2=\"-28252\" y1=\"-28242\" x2=\"-25706\" x1=\"-25723\" fill=\"none\"/><line y2=\"-29478\" y1=\"-29530\" x2=\"-25257\" x1=\"-25130\" fill=\"none\"/><line y2=\"-29686\" y1=\"-29727\" x2=\"-25321\" x1=\"-25221\" fill=\"none\"/><line y2=\"-29894\" y1=\"-29924\" x2=\"-25384\" x1=\"-25312\" fill=\"none\"/><line y2=\"-30102\" y1=\"-30121\" x2=\"-25448\" x1=\"-25403\" fill=\"none\"/><line y2=\"-30310\" y1=\"-30317\" x2=\"-25512\" x1=\"-25493\" fill=\"none\"/><line y2=\"-30131\" y1=\"-30128\" x2=\"-27473\" x1=\"-27612\" fill=\"none\"/><line y2=\"-29914\" y1=\"-29912\" x2=\"-27487\" x1=\"-27598\" fill=\"none\"/><line y2=\"-29697\" y1=\"-29695\" x2=\"-27502\" x1=\"-27583\" fill=\"none\"/><line y2=\"-29480\" y1=\"-29479\" x2=\"-27516\" x1=\"-27569\" fill=\"none\"/><line y2=\"-29263\" y1=\"-29263\" x2=\"-27530\" x1=\"-27554\" fill=\"none\"/><text y=\"-28380\" xml:space=\"preserve\" x=\"-28602\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">OH</text><text y=\"-29983\" xml:space=\"preserve\" x=\"-31540\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">O</text><text y=\"-30691\" xml:space=\"preserve\" x=\"-32762\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">CH</text><text y=\"-30602\" xml:space=\"preserve\" x=\"-32185\" stroke=\"none\" font-size=\"313.3333\" font-family=\"sans-serif\">3</text><text y=\"-32242\" xml:space=\"preserve\" x=\"-27695\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">N</text><text y=\"-27747\" xml:space=\"preserve\" x=\"-25708\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">N</text><text y=\"-27473\" xml:space=\"preserve\" x=\"-27311\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">H</text><text y=\"-28600\" xml:space=\"preserve\" x=\"-27695\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">R</text><text y=\"-28522\" xml:space=\"preserve\" x=\"-26540\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">S</text><text y=\"-27337\" xml:space=\"preserve\" x=\"-25818\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">S</text><text y=\"-30573\" xml:space=\"preserve\" x=\"-25708\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">S</text><text y=\"-29495\" xml:space=\"preserve\" x=\"-24876\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">R</text></g></g></svg>"
       }
     ]
   }

Note that with the CAS Common Chemistry Search API, only the image data, name, and CAS RN is returned. In order to retrieve the full record, we can combine our search with the related detail API:

Extract CAS RN:

.. code-block:: shell

   quinine_rn=$(echo "$quinine_search_data" | jq '.["results"][0]["rn"]' | tr -d '"')
   echo "$quinine_rn"

**Output:**

.. code-block:: shell

   130-95-0

Get detailed record for quinine:

.. code-block:: shell

   detail_base_url="https://commonchemistry.cas.org/api/detail?"
   quinine_detail_data=$(curl $detail_base_url$"cas_rn="$quinine_rn)
   echo "$quinine_detail_data" | jq '.'

**Output:**

.. code-block:: shell

   {
     "uri": "substance/pt/130950",
     "rn": "130-95-0",
     "name": "Quinine",
     "image": "<svg width=\"309.3\" viewBox=\"0 0 10310 5592\" text-rendering=\"auto\" stroke-width=\"1\" stroke-opacity=\"1\" stroke-miterlimit=\"10\" stroke-linejoin=\"miter\" stroke-linecap=\"square\" stroke-dashoffset=\"0\" stroke-dasharray=\"none\" stroke=\"black\" shape-rendering=\"auto\" image-rendering=\"auto\" height=\"167.76\" font-weight=\"normal\" font-style=\"normal\" font-size=\"12\" font-family=\"'Dialog'\" fill-opacity=\"1\" fill=\"black\" color-rendering=\"auto\" color-interpolation=\"auto\" xmlns=\"http://www.w3.org/2000/svg\"><g><g stroke=\"white\" fill=\"white\"><rect y=\"0\" x=\"0\" width=\"10310\" stroke=\"none\" height=\"5592\"/></g><g transform=\"translate(32866,32758)\" text-rendering=\"geometricPrecision\" stroke-width=\"44\" stroke-linejoin=\"round\" stroke-linecap=\"round\"><line y2=\"-28559\" y1=\"-28036\" x2=\"-26635\" x1=\"-25742\" fill=\"none\"/><line y2=\"-29819\" y1=\"-28559\" x2=\"-26635\" x1=\"-26635\" fill=\"none\"/><line y2=\"-28036\" y1=\"-28559\" x2=\"-25367\" x1=\"-24474\" fill=\"none\"/><line y2=\"-30451\" y1=\"-29819\" x2=\"-25555\" x1=\"-26635\" fill=\"none\"/><line y2=\"-28559\" y1=\"-29819\" x2=\"-24474\" x1=\"-24474\" fill=\"none\"/><line y2=\"-29504\" y1=\"-28828\" x2=\"-25194\" x1=\"-26005\" fill=\"none\"/><line y2=\"-29819\" y1=\"-30451\" x2=\"-24474\" x1=\"-25555\" fill=\"none\"/><line y2=\"-29082\" y1=\"-28559\" x2=\"-27542\" x1=\"-26635\" fill=\"none\"/><line y2=\"-29819\" y1=\"-30344\" x2=\"-22660\" x1=\"-23567\" fill=\"none\"/><line y2=\"-29700\" y1=\"-30223\" x2=\"-22729\" x1=\"-23636\" fill=\"none\"/><line y2=\"-28779\" y1=\"-29082\" x2=\"-28071\" x1=\"-27542\" fill=\"none\"/><line y2=\"-30703\" y1=\"-30131\" x2=\"-28524\" x1=\"-27542\" fill=\"none\"/><line y2=\"-31850\" y1=\"-30703\" x2=\"-28524\" x1=\"-28524\" fill=\"none\"/><line y2=\"-31705\" y1=\"-30847\" x2=\"-28354\" x1=\"-28354\" fill=\"none\"/><line y2=\"-30131\" y1=\"-30703\" x2=\"-29507\" x1=\"-28524\" fill=\"none\"/><line y2=\"-30131\" y1=\"-30703\" x2=\"-27542\" x1=\"-26560\" fill=\"none\"/><line y2=\"-30347\" y1=\"-30778\" x2=\"-27505\" x1=\"-26768\" fill=\"none\"/><line y2=\"-31850\" y1=\"-32422\" x2=\"-28524\" x1=\"-29507\" fill=\"none\"/><line y2=\"-32312\" y1=\"-31850\" x2=\"-27730\" x1=\"-28524\" fill=\"none\"/><line y2=\"-30703\" y1=\"-30131\" x2=\"-30489\" x1=\"-29507\" fill=\"none\"/><line y2=\"-30778\" y1=\"-30347\" x2=\"-30281\" x1=\"-29544\" fill=\"none\"/><line y2=\"-30703\" y1=\"-31850\" x2=\"-26560\" x1=\"-26560\" fill=\"none\"/><line y2=\"-32422\" y1=\"-31850\" x2=\"-29507\" x1=\"-30489\" fill=\"none\"/><line y2=\"-32205\" y1=\"-31774\" x2=\"-29544\" x1=\"-30281\" fill=\"none\"/><line y2=\"-31850\" y1=\"-32312\" x2=\"-26560\" x1=\"-27354\" fill=\"none\"/><line y2=\"-31760\" y1=\"-32107\" x2=\"-26745\" x1=\"-27340\" fill=\"none\"/><line y2=\"-31850\" y1=\"-30703\" x2=\"-30489\" x1=\"-30489\" fill=\"none\"/><line y2=\"-30275\" y1=\"-30703\" x2=\"-31200\" x1=\"-30489\" fill=\"none\"/><line y2=\"-30541\" y1=\"-30272\" x2=\"-32040\" x1=\"-31575\" fill=\"none\"/><polygon stroke-width=\"1\" stroke=\"none\" points=\" -24474 -29819 -23602 -30402 -23532 -30284\"/><polygon stroke-width=\"1\" points=\" -24474 -29819 -23602 -30402 -23532 -30284\" fill=\"none\"/><polygon stroke-width=\"1\" stroke=\"none\" points=\" -26635 -28559 -26973 -27837 -27092 -27903\"/><polygon stroke-width=\"1\" points=\" -26635 -28559 -26973 -27837 -27092 -27903\" fill=\"none\"/><line y2=\"-28860\" y1=\"-28796\" x2=\"-25945\" x1=\"-26066\" fill=\"none\"/><line y2=\"-28657\" y1=\"-28611\" x2=\"-25865\" x1=\"-25952\" fill=\"none\"/><line y2=\"-28454\" y1=\"-28427\" x2=\"-25785\" x1=\"-25838\" fill=\"none\"/><line y2=\"-28252\" y1=\"-28242\" x2=\"-25706\" x1=\"-25723\" fill=\"none\"/><line y2=\"-29478\" y1=\"-29530\" x2=\"-25257\" x1=\"-25130\" fill=\"none\"/><line y2=\"-29686\" y1=\"-29727\" x2=\"-25321\" x1=\"-25221\" fill=\"none\"/><line y2=\"-29894\" y1=\"-29924\" x2=\"-25384\" x1=\"-25312\" fill=\"none\"/><line y2=\"-30102\" y1=\"-30121\" x2=\"-25448\" x1=\"-25403\" fill=\"none\"/><line y2=\"-30310\" y1=\"-30317\" x2=\"-25512\" x1=\"-25493\" fill=\"none\"/><line y2=\"-30131\" y1=\"-30128\" x2=\"-27473\" x1=\"-27612\" fill=\"none\"/><line y2=\"-29914\" y1=\"-29912\" x2=\"-27487\" x1=\"-27598\" fill=\"none\"/><line y2=\"-29697\" y1=\"-29695\" x2=\"-27502\" x1=\"-27583\" fill=\"none\"/><line y2=\"-29480\" y1=\"-29479\" x2=\"-27516\" x1=\"-27569\" fill=\"none\"/><line y2=\"-29263\" y1=\"-29263\" x2=\"-27530\" x1=\"-27554\" fill=\"none\"/><text y=\"-28380\" xml:space=\"preserve\" x=\"-28602\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">OH</text><text y=\"-29983\" xml:space=\"preserve\" x=\"-31540\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">O</text><text y=\"-30691\" xml:space=\"preserve\" x=\"-32762\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">CH</text><text y=\"-30602\" xml:space=\"preserve\" x=\"-32185\" stroke=\"none\" font-size=\"313.3333\" font-family=\"sans-serif\">3</text><text y=\"-32242\" xml:space=\"preserve\" x=\"-27695\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">N</text><text y=\"-27747\" xml:space=\"preserve\" x=\"-25708\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">N</text><text y=\"-27473\" xml:space=\"preserve\" x=\"-27311\" stroke=\"none\" font-size=\"433.3333\" font-family=\"sans-serif\">H</text><text y=\"-28600\" xml:space=\"preserve\" x=\"-27695\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">R</text><text y=\"-28522\" xml:space=\"preserve\" x=\"-26540\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">S</text><text y=\"-27337\" xml:space=\"preserve\" x=\"-25818\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">S</text><text y=\"-30573\" xml:space=\"preserve\" x=\"-25708\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">S</text><text y=\"-29495\" xml:space=\"preserve\" x=\"-24876\" stroke=\"none\" font-style=\"italic\" font-size=\"313.3333\" font-family=\"sans-serif\">R</text></g></g></svg>",
     "inchi": "InChI=1S/C20H24N2O2/c1-3-13-12-22-9-7-14(13)10-19(22)20(23)16-6-8-21-18-5-4-15(24-2)11-17(16)18/h3-6,8,11,13-14,19-20,23H,1,7,9-10,12H2,2H3/t13-,14-,19-,20+/m0/s1",
     "inchiKey": "InChIKey=LOUPRKONTZGTKE-WZBLMQSHSA-N",
     "smile": "[C@@H](O)(C=1C2=C(C=CC(OC)=C2)N=CC1)[C@]3([N@@]4C[C@H](C=C)[C@H](C3)CC4)[H]",
     "canonicalSmile": "OC(C=1C=CN=C2C=CC(OC)=CC21)C3N4CCC(C3)C(C=C)C4",
     "molecularFormula": "C<sub>20</sub>H<sub>24</sub>N<sub>2</sub>O<sub>2</sub>",
     "molecularMass": "324.42",
     "experimentalProperties": [
       {
         "name": "Melting Point",
         "property": "57 °C",
         "sourceNumber": 1
       }
     ],
     "propertyCitations": [
       {
         "docUri": "",
         "sourceNumber": 1,
         "source": "PhysProp data were obtained from Syracuse Research Corporation of Syracuse, New York (US)"
       }
     ],
     "synonyms": [
       "Cinchonan-9-ol, 6′-methoxy-, (8α,9<em>R</em>)-",
       "Quinine",
       "(8α,9<em>R</em>)-6′-Methoxycinchonan-9-ol",
       "6′-Methoxycinchonidine",
       "(-)-Quinine",
       "(8<em>S</em>,9<em>R</em>)-Quinine",
       "(<em>R</em>)-(-)-Quinine",
       "NSC 192949",
       "WR297608",
       "Qualaquin",
       "Mosgard",
       "Quinlup",
       "Quine 9",
       "Cinkona",
       "Quinex",
       "Quinlex",
       "Rezquin",
       "QSM",
       "SW 85833",
       "(<em>R</em>)-(6-Methoxy-4-quinolyl)[(2<em>S</em>)-5-vinylquinuclidin-2-yl]methanol"
     ],
     "replacedRns": [
       "6912-57-8",
       "12239-42-8",
       "21480-31-9",
       "55980-20-6",
       "72646-90-3",
       "95650-40-1",
       "128544-03-6",
       "767303-40-2",
       "840482-04-4",
       "857212-53-4",
       "864908-93-0",
       "875538-34-4",
       "888714-03-2",
       "890027-24-4",
       "894767-09-0",
       "898813-59-7",
       "898814-28-3",
       "899813-83-3",
       "900786-66-5",
       "900789-95-9",
       "906550-97-8",
       "909263-47-4",
       "909767-48-2",
       "909882-78-6",
       "910878-25-0",
       "910880-97-6",
       "911445-75-5",
       "918778-04-8",
       "1071756-51-8",
       "1267651-57-9",
       "1628705-47-4",
       "2244812-93-7",
       "2244812-97-1",
       "2409557-51-1",
       "2566761-34-8"
     ],
     "hasMolfile": true
   }

Handle multiple results
-------------------------

Setup search query parameters with SMILES for butadiene as an example:

.. code-block:: shell

   search_base_url="https://commonchemistry.cas.org/api/search?q="
   smi_bd="C=CC=C"

Request data from CAS Common Chemistry Search API:

.. code-block:: shell

   smi_search_data=$(curl $search_base_url$smi_bd)

Get results count:

.. code-block:: shell

   echo $smi_search_data | jq '.["count"]'

**Output:**

.. code-block:: shell

   7

Extract out CAS RNs:

.. code-block:: shell

   declare -a smi_casrn_list
   for (( i = 0 ; i < $(echo "$smi_search_data" | jq '.["count"]') ; i++ ))
   do
     smi_casrn_list+=( "$(echo "$smi_search_data" | jq ".results[$i].rn" | tr -d '"')" )
   done
   echo "${smi_casrn_list[@]}"

**Output:**

.. code-block:: shell

   106-99-0 16422-75-6 26952-74-9 29406-96-0 29989-19-3 31567-90-5 9003-17-2

Now use the detail API to retrieve the full records:

.. code-block:: shell

   detail_base_url="https://commonchemistry.cas.org/api/detail?"
   declare -a smi_detail_data
   for casrn in "${smi_casrn_list[@]}"
   do
     smi_detail_data+=( "$(curl "$detail_base_url"$"cas_rn=""$casrn")" )
     sleep 1
   done

.. note::

   You can use ``echo`` and ``jq`` to view the data. For example, the first record: ``echo "${smi_detail_data[0]}" | jq '.'``

Get some specific data such as name from the detail records:

.. code-block:: shell

   declare -a names
   for name_idx in "${smi_detail_data[@]}"
   do
     names+=( "$(echo $name_idx | jq '.["name"]')" )
   done
   echo "${names[@]}"

**Output:**

.. code-block:: shell

   "1,3-Butadiene" "Butadiene trimer" "Butadiene dimer" "1,3-Butadiene, homopolymer, isotactic" "1,3-Butadiene-<em>1</em>,<em>1</em>,<em>2</em>,<em>3</em>,<em>4</em>,<em>4</em>-<em>d</em><sub>6</sub>, homopolymer" "Syndiotactic polybutadiene" "Polybutadiene"
                                                     
Handle multiple page results
------------------------------

The CAS Common Chemistry API returns 50 results per page, and only the first page is returned by default. If the search returns more than 50 results, the offset option can be added to page through and obtain all results.

Setup search query parameters:

.. code-block:: shell

   search_base_url="https://commonchemistry.cas.org/api/search?q="
   n="selen*"

Get results count for CAS Common Chemistry Search:

.. code-block:: shell

   num_Results=$(curl "$search_base_url""$n" | jq '.["count"]')
   echo "$num_Results"

**Output:**

.. code-block:: shell

   191

Request data and save to an array in a loop for each page:

.. code-block:: shell

   declare -a n_search_data
   for (( i = 0 ; i < "$num_Results" ; i+=50 ))
   do
     n_search_data+=( "$(curl "$search_base_url""$n"$"&offset=""$i")" )
     sleep 1
   done

Length of search data includes a top level list for each query:

.. code-block:: shell

   echo "${#n_search_data[@]}"

**Output:**

.. code-block:: shell

   4

Data within the array contain the results:

.. code-block:: shell

   for data in "${n_search_data[@]}"
   do
     echo "$data" | jq '.["results"] | length'
   done

**Output:**

.. code-block:: shell

   50
   50
   50
   41

We can index and extract out the first CAS RN like this:

.. code-block:: shell

   echo "${n_search_data[0]}" | jq '.["results"][0]["rn"]' | tr -d '"'

**Output:**

.. code-block:: shell

   10025-68-0

Extract out all CAS RNs from the array:

.. code-block:: shell

   declare -a n_casrn_list
   for n_idx in "${n_search_data[@]}"
   do
     for (( i = 0 ; i < $(echo "$n_idx" | jq '.["results"] | length') ; i++ ))
     do
       n_casrn_list+=("$(echo "$n_idx" | jq ".results[$i].rn" | tr -d '"')")
     done
   done

Get length of casrn_list:

.. code-block:: shell

   echo "${#n_casrn_list[@]}"

**Output:**

.. code-block:: shell

   191

Show first 10 values:

.. code-block:: shell

   echo "${n_casrn_list[@]:0:10}"

**Output:**

.. code-block:: shell

   10025-68-0 10026-03-6 10026-23-0 10101-96-9 10102-18-8 10102-23-5 10112-94-4 10161-84-9 10214-40-1 10236-58-5

Now we can loop through each CAS RN and use the detail API to obtain the entire record. This will query CAS Common Chem 191 times and take ~5 min. The silent option (``-s``) for curl was used here to hide the progress outputs.

.. code-block:: shell

   detail_base_url="https://commonchemistry.cas.org/api/detail?"
   declare -a n_detail_data
   for casrn in "${n_casrn_list[@]}"
   do
     n_detail_data+=("$(curl -s "$detail_base_url"$"cas_rn=""$casrn")")
     sleep 1
   done

Extract out some data such as molecularMass and save the data to a file: ``mms.csv``:

.. code-block:: shell

   declare -a mms
   for mm_idx in "${n_detail_data[@]}"
   do
     mm=$(echo "$mm_idx" | jq '.["molecularMass"]')
     echo "$mm" | sed 's/\"\"/NaN/g' | tr -d '"' >> mms.csv
     mms+=("$mm")
   done

.. note::

   ``sed 's/\"\"/NaN/g'`` is used here to replace empty numbers with NaN.

View the mms.csv file:

.. code-block:: shell

   head mms.csv

**Output:**

.. code-block:: shell

   228.83
   220.77
   NaN
   NaN
   NaN
   NaN
   NaN
   300.24
   NaN
   168.05

Get number of lines in file:

.. code-block:: shell

   wc -l mms.csv

**Output:**

.. code-block:: shell

   191 mms.csv

Finally, we can create a simple visualization from the extracted molecularMass values (from the selen* search) using gnuplot. See the `gnuplot documentation`_ for more information about the smooth frequency histogram.

.. _gnuplot documentation: http://www.gnuplot.info/documentation.html

.. code-block:: shell

   gnuplot -e "set datafile separator ','; \
   set datafile missing NaN; \
   set title 'Histogram of available molecularMass values for selen* search'; \
   set xlabel 'molecularMass'; \
   set term dumb; \
   set yrange [0:35]; \
   set xrange [0:1000]; \
   binwidth=50; \
   bin(val)=binwidth*floor(val/binwidth); \
   plot 'mms.csv' using (bin(column(1))):(1.0) smooth frequency with boxes notitle"

**Output:**

.. code-block:: shell
                                                                               
             Histogram of available molecularMass values for selen* search        
                                                                                  
     35 +---------------------------------------------------------------------+   
        |             +             +             +             +             |   
     30 |-+      ****                                                       +-|   
        |        *  *                                                         |   
        |        *  *                                                         |   
     25 |-+      *  *                                                       +-|   
        |        *  *                                                         |   
     20 |-+      *  *                                                       +-|   
        |    *****  *                                                         |   
        |    *   *  *****                                                     |   
     15 |-+  *   *  *   *                                                   +-|   
        |    *   *  *   *                                                     |   
     10 |-+  *   *  *   *  *****                                            +-|   
        |    *   *  *   ****   *                                              |   
        |    *   *  *   *  *   ****                                           |   
      5 |-+  *   *  *   *  *   *  *                                         +-|   
        |*****   *  * + *  *   *  * + ****   **** +             +             |   
      0 +---------------------------------------------------------------------+   
        0            200           400           600           800           1000 
                                     molecularMass                                

