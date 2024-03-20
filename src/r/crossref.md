---
title: \...in R
output: html_document
editor_options: 
  chunk_output_type: console
---

# Crossref API in R

by Cyrus Gomes, Vincent F. Scalfani, and Adam M. Nguyen

**Documentation:**

Crossref API Documentation: https://api.crossref.org/swagger-ui/index.html

These recipe examples were tested on March 24 2023.

See the bottom of the document for information on R and package versions.

From our testing, we have found that the crossref metadata across publishers and even journals can vary considerably. As a result, it can be easier to work with one journal at a time when using the crossref API (e.g., particulary when trying to extract selected data from records).

## Setup

Importing the necessary libraries and setting up the base api:

If you haven’t already, run “install.packages('Package’)” in your R Console to install package you need for this tutorial, substituting Package in the code for the name of the package you need, e.g. tidyverse, purrr, etc..


```r
library(tidyverse)  #essential packages
library(purrr)      #character manipulation 
library(httr)       #GET() API requests
library(jsonlite)   #converting to JSON

# Set base URL to Crossref API
base_url <- "https://api.crossref.org/works/"
```

## 1. Basic CrossRef API call

### Setup basic parameters


```r
email <- "your_email@ua.edu" #Change this to your email
mailto <- "?mailto="
doi <- "10.1186/1758-2946-4-12" #example

complete_data <- paste(base_url, doi, mailto, email, sep="") #combines it all in one line
complete_data
```

```
## [1] "https://api.crossref.org/works/10.1186/1758-2946-4-12?mailto=your_email@ua.edu"
```

### Request data from CrossRef


```r
raw_api_data <- GET(complete_data)$content # fetches content from the data
head(raw_api_data,n=100)
```

```
##   [1] 7b 22 73 74 61 74 75 73 22 3a 22 6f 6b 22 2c 22 6d 65 73 73 61 67 65 2d 74
##  [26] 79 70 65 22 3a 22 77 6f 72 6b 22 2c 22 6d 65 73 73 61 67 65 2d 76 65 72 73
##  [51] 69 6f 6e 22 3a 22 31 2e 30 2e 30 22 2c 22 6d 65 73 73 61 67 65 22 3a 7b 22
##  [76] 69 6e 64 65 78 65 64 22 3a 7b 22 64 61 74 65 2d 70 61 72 74 73 22 3a 5b 5b
```
The raw data is in hexadecimal and is hard to read.

### Converting the data to JSON format (which is more readable)

```r
api_data <- fromJSON(rawToChar(raw_api_data), flatten = TRUE)
print(api_data)
```

```
## $status
## [1] "ok"
## 
## $`message-type`
## [1] "work"
## 
## $`message-version`
## [1] "1.0.0"
## 
## $message
## $message$indexed
## $message$indexed$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2023    3    7
## 
## $message$indexed$`date-time`
## [1] "2023-03-07T09:15:56Z"
## 
## $message$indexed$timestamp
## [1] 1.678181e+12
## 
## 
## $message$`reference-count`
## [1] 16
## 
## $message$publisher
## [1] "Springer Science and Business Media LLC"
## 
## $message$issue
## [1] "1"
## 
## $message$license
##   content-version delay-in-days                                        URL
## 1             tdm             0 http://creativecommons.org/licenses/by/2.0
##   start.date-parts      start.date-time start.timestamp
## 1       2012, 7, 6 2012-07-06T00:00:00Z    1.341533e+12
## 
## $message$`content-domain`
## $message$`content-domain`$domain
## list()
## 
## $message$`content-domain`$`crossmark-restriction`
## [1] FALSE
## 
## 
## $message$`short-container-title`
## [1] "J Cheminform"
## 
## $message$`published-print`
## $message$`published-print`$`date-parts`
##      [,1] [,2]
## [1,] 2012   12
## 
## 
## $message$DOI
## [1] "10.1186/1758-2946-4-12"
## 
## $message$type
## [1] "journal-article"
## 
## $message$created
## $message$created$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## $message$created$`date-time`
## [1] "2012-07-06T12:14:34Z"
## 
## $message$created$timestamp
## [1] 1.341577e+12
## 
## 
## $message$source
## [1] "Crossref"
## 
## $message$`is-referenced-by-count`
## [1] 36
## 
## $message$title
## [1] "The Molecule Cloud - compact visualization of large collections of molecules"
## 
## $message$prefix
## [1] "10.1186"
## 
## $message$volume
## [1] "4"
## 
## $message$author
##      given family   sequence affiliation
## 1    Peter   Ertl      first        NULL
## 2 Bernhard  Rohde additional        NULL
## 
## $message$member
## [1] "297"
## 
## $message$`published-online`
## $message$`published-online`$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## 
## $message$reference
##         key doi-asserted-by first-page                        DOI volume
## 1   336_CR1       publisher         77  10.1007/s10822-011-9487-0     26
## 2   336_CR2       publisher       2174          10.1021/ci2001428     26
## 3   336_CR3       publisher       8732          10.1021/ja902302h    131
## 4   336_CR4       publisher        156 10.2174/157340908785747410      4
## 5   336_CR5       publisher        322 10.2174/157340908786786010      4
## 6   336_CR6       publisher         47          10.1021/ci600338x     47
## 7   336_CR7       publisher        366     10.1002/minf.201000019     29
## 8   336_CR8       publisher       4443          10.1021/jo8001276     73
## 9   336_CR9            <NA>       <NA>                       <NA>   <NA>
## 10 336_CR10            <NA>       <NA>                       <NA>   <NA>
## 11 336_CR11       publisher       D255         10.1093/nar/gkp965     38
## 12 336_CR12       publisher        177          10.1021/ci049714+     45
## 13 336_CR13       publisher      D1100         10.1093/nar/gkr777     40
## 14 336_CR14       publisher        347 10.1016/j.cbpa.2010.02.018     14
## 15 336_CR15       publisher        374          10.1021/ci0255782     43
## 16 336_CR16            <NA>       <NA>                       <NA>   <NA>
##              author year
## 1          E Martin 2011
## 2        SR Langdon 2011
## 3           LC Blum 2009
## 4          J Dubois 2008
## 5  JL Medina-Franco 2008
## 6   A Schuffenhauer 2007
## 7         S Langdon 2010
## 8         AH Lipkus 2008
## 9              <NA> <NA>
## 10             <NA> <NA>
## 11           Y Wang 2009
## 12         JJ Irwin 2004
## 13        A Gaulton 2012
## 14        ME Welsch 2010
## 15           P Ertl 2003
## 16             <NA> <NA>
##                                                                                                                                                                                                                                                                  unstructured
## 1                                                                                                                   Martin E, Ertl P, Hunt P, Duca J, Lewis R: Gazing into the crystal ball; the future of computer-aided drug design. J Comp-Aided Mol Des. 2011, 26: 77-79.
## 2                                                                                                                                           Langdon SR, Brown N, Blagg J: Scaffold diversity of exemplified medicinal chemistry space. J Chem Inf Model. 2011, 26: 2174-2185.
## 3                                                                                          Blum LC, Reymond J-C: 970 Million druglike small molecules for virtual screening in the chemical universe database GDB-13. J Am Chem Soc. 2009, 131: 8732-8733. 10.1021/ja902302h.
## 4                                                                                                       Dubois J, Bourg S, Vrain C, Morin-Allory L: Collections of compounds - how to deal with them?. Cur Comp-Aided Drug Des. 2008, 4: 156-168. 10.2174/157340908785747410.
## 5                                                                 Medina-Franco JL, Martinez-Mayorga K, Giulianotti MA, Houghten RA, Pinilla C: Visualization of the chemical space in drug discovery. Cur Comp-Aided Drug Des. 2008, 4: 322-333. 10.2174/157340908786786010.
## 6                                                  Schuffenhauer A, Ertl P, Roggo S, Wetzel S, Koch MA, Waldmann H: The Scaffold Tree - visualization of the scaffold universe by hierarchical scaffold classification. J Chem Inf Model. 2007, 47: 47-58. 10.1021/ci600338x.
## 7                                                                                                          Langdon S, Ertl P, Brown N: Bioisosteric replacement and scaffold hopping in lead generation and optimization. Mol Inf. 2010, 29: 366-385. 10.1002/minf.201000019.
## 8                                                            Lipkus AH, Yuan Q, Lucas KA, Funk SA, Bartelt WF, Schenck RJ, Trippe AJ: Structural diversity of organic chemistry. A scaffold analysis of the CAS Registry. J Org Chem. 2008, 73: 4443-4451. 10.1021/jo8001276.
## 9                                                                                                                                 mib 2010.10, Molinspiration Cheminformatics: \n                    http://www.molinspiration.com\n                    \n                  ,
## 10                                                                                                                Bernhard R: Avalon Cheminformatics Toolkit. \n                    http://sourceforge.net/projects/avalontoolkit/\n                    \n                  ,
## 11                                                                              Wang Y, Bolton E, Dracheva S, Karapetyan K, Shoemaker BA, Suzek TO, Wang J, Xiao J, Zhang J, Bryant SH: An overview of the PubChem BioAssay resource. Nucleic Acids Res. 2009, 38: D255-D266.
## 12                                                                                                                              Irwin JJ, Shoichet BK: ZINC − a free database of commercially available compounds for virtual screening. J Chem Inf Model. 2004, 45: 177-182.
## 13           Gaulton A, Bellis LJ, Bento AP, Chambers J, Davies M, Hersey A, Light Y, McGlinchey S, Michalovich D, Al-Lazikani B, Overington JP: ChEMBL: a large-scale bioactivity database for drug discovery. Nucleic Acids Res. 2012, 40: D1100-D1107. 10.1093/nar/gkr777.
## 14                                                                                                        Welsch ME, Snyder SA, Stockwell BR: Privileged scaffolds for library design and drug discovery. Curr Opin Chem Biol. 2010, 14: 347-361. 10.1016/j.cbpa.2010.02.018.
## 15 Ertl P: Cheminformatics analysis of organic substituents: Identification of the most common substituents, calculation of substituent properties, and automatic identification of drug-like bioisosteric groups. J Chem Inf Comp Sci. 2003, 43: 374-380. 10.1021/ci0255782.
## 16                                                                                                                                                                                                                        TagCrowd: \n                    http://tagcrowd.com
##              journal-title
## 1     J Comp-Aided Mol Des
## 2         J Chem Inf Model
## 3            J Am Chem Soc
## 4  Cur Comp-Aided Drug Des
## 5  Cur Comp-Aided Drug Des
## 6         J Chem Inf Model
## 7                  Mol Inf
## 8               J Org Chem
## 9                     <NA>
## 10                    <NA>
## 11       Nucleic Acids Res
## 12        J Chem Inf Model
## 13       Nucleic Acids Res
## 14     Curr Opin Chem Biol
## 15     J Chem Inf Comp Sci
## 16                    <NA>
## 
## $message$`container-title`
## [1] "Journal of Cheminformatics"
## 
## $message$`original-title`
## list()
## 
## $message$language
## [1] "en"
## 
## $message$link
##                                                                     URL
## 1       http://link.springer.com/content/pdf/10.1186/1758-2946-4-12.pdf
## 2 http://link.springer.com/article/10.1186/1758-2946-4-12/fulltext.html
## 3       http://link.springer.com/content/pdf/10.1186/1758-2946-4-12.pdf
##      content-type content-version intended-application
## 1 application/pdf             vor          text-mining
## 2       text/html             vor          text-mining
## 3 application/pdf             vor  similarity-checking
## 
## $message$deposited
## $message$deposited$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2019    6   24
## 
## $message$deposited$`date-time`
## [1] "2019-06-24T14:22:07Z"
## 
## $message$deposited$timestamp
## [1] 1.561386e+12
## 
## 
## $message$score
## [1] 1
## 
## $message$resource
## $message$resource$primary
## $message$resource$primary$URL
## [1] "https://jcheminf.biomedcentral.com/articles/10.1186/1758-2946-4-12"
## 
## 
## 
## $message$subtitle
## list()
## 
## $message$`short-title`
## list()
## 
## $message$issued
## $message$issued$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## 
## $message$`references-count`
## [1] 16
## 
## $message$`journal-issue`
## $message$`journal-issue`$issue
## [1] "1"
## 
## $message$`journal-issue`$`published-print`
## $message$`journal-issue`$`published-print`$`date-parts`
##      [,1] [,2]
## [1,] 2012   12
## 
## 
## 
## $message$`alternative-id`
## [1] "336"
## 
## $message$URL
## [1] "http://dx.doi.org/10.1186/1758-2946-4-12"
## 
## $message$relation
## named list()
## 
## $message$ISSN
## [1] "1758-2946"
## 
## $message$`issn-type`
##       value       type
## 1 1758-2946 electronic
## 
## $message$subject
## [1] "Library and Information Sciences"           
## [2] "Computer Graphics and Computer-Aided Design"
## [3] "Physical and Theoretical Chemistry"         
## [4] "Computer Science Applications"              
## 
## $message$published
## $message$published$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## 
## $message$`article-number`
## [1] "12"
```

Replace the above DOI value with a different DOI to customize.

### Let's see the reference part of CrossRef in a convenient tabular way


```r
api_data$message$`reference`[1:7,1:7]    # prints the table of data
```

```
##       key doi-asserted-by first-page                        DOI volume
## 1 336_CR1       publisher         77  10.1007/s10822-011-9487-0     26
## 2 336_CR2       publisher       2174          10.1021/ci2001428     26
## 3 336_CR3       publisher       8732          10.1021/ja902302h    131
## 4 336_CR4       publisher        156 10.2174/157340908785747410      4
## 5 336_CR5       publisher        322 10.2174/157340908786786010      4
## 6 336_CR6       publisher         47          10.1021/ci600338x     47
## 7 336_CR7       publisher        366     10.1002/minf.201000019     29
##             author year
## 1         E Martin 2011
## 2       SR Langdon 2011
## 3          LC Blum 2009
## 4         J Dubois 2008
## 5 JL Medina-Franco 2008
## 6  A Schuffenhauer 2007
## 7        S Langdon 2010
```

### Selecting specific data

Getting the Journal Title


```r
#Get Journal Title
api_data$message$`container-title` # prints the title
```

```
## [1] "Journal of Cheminformatics"
```

Getting the Article Title


```r
api_data$message$`title` 
```

```
## [1] "The Molecule Cloud - compact visualization of large collections of molecules"
```

Getting the Article Author Names


```r
for (au in api_data$message$author){
  article_authors <- paste(api_data$message$author$given, api_data$message$author$family)
}
article_authors
```

```
## [1] "Peter Ertl"     "Bernhard Rohde"
```

Getting the Bibliography References and saving them to a list


```r
#Get bibliography references and save to list
bib_refs <- list()  #declare a list
for (ref in api_data$message$reference){
  bib_refs <- append(x = bib_refs, api_data$message$reference$unstructured)
}
bib_refs[0:5]
```

```
## [[1]]
## [1] "Martin E, Ertl P, Hunt P, Duca J, Lewis R: Gazing into the crystal ball; the future of computer-aided drug design. J Comp-Aided Mol Des. 2011, 26: 77-79."
## 
## [[2]]
## [1] "Langdon SR, Brown N, Blagg J: Scaffold diversity of exemplified medicinal chemistry space. J Chem Inf Model. 2011, 26: 2174-2185."
## 
## [[3]]
## [1] "Blum LC, Reymond J-C: 970 Million druglike small molecules for virtual screening in the chemical universe database GDB-13. J Am Chem Soc. 2009, 131: 8732-8733. 10.1021/ja902302h."
## 
## [[4]]
## [1] "Dubois J, Bourg S, Vrain C, Morin-Allory L: Collections of compounds - how to deal with them?. Cur Comp-Aided Drug Des. 2008, 4: 156-168. 10.2174/157340908785747410."
## 
## [[5]]
## [1] "Medina-Franco JL, Martinez-Mayorga K, Giulianotti MA, Houghten RA, Pinilla C: Visualization of the chemical space in drug discovery. Cur Comp-Aided Drug Des. 2008, 4: 322-333. 10.2174/157340908786786010."
```

### Save JSON data to a file

This is particularly useful for downstream testing or returning to results in the future (e.g., no need to keep requesting the data from crossref, save the results to a file)


```r
#Save JSON data to a file
jsonData <- toJSON(api_data)
write (jsonData, "output.json")
```

### Load JSON data from a file


```r
loaded_data <- fromJSON("output.json")
loaded_data                       #Load Json data from file 
```

```
## $status
## [1] "ok"
## 
## $`message-type`
## [1] "work"
## 
## $`message-version`
## [1] "1.0.0"
## 
## $message
## $message$indexed
## $message$indexed$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2023    3    7
## 
## $message$indexed$`date-time`
## [1] "2023-03-07T09:15:56Z"
## 
## $message$indexed$timestamp
## [1] 1.678181e+12
## 
## 
## $message$`reference-count`
## [1] 16
## 
## $message$publisher
## [1] "Springer Science and Business Media LLC"
## 
## $message$issue
## [1] "1"
## 
## $message$license
##   content-version delay-in-days                                        URL
## 1             tdm             0 http://creativecommons.org/licenses/by/2.0
##   start.date-parts      start.date-time start.timestamp
## 1       2012, 7, 6 2012-07-06T00:00:00Z    1.341533e+12
## 
## $message$`content-domain`
## $message$`content-domain`$domain
## list()
## 
## $message$`content-domain`$`crossmark-restriction`
## [1] FALSE
## 
## 
## $message$`short-container-title`
## [1] "J Cheminform"
## 
## $message$`published-print`
## $message$`published-print`$`date-parts`
##      [,1] [,2]
## [1,] 2012   12
## 
## 
## $message$DOI
## [1] "10.1186/1758-2946-4-12"
## 
## $message$type
## [1] "journal-article"
## 
## $message$created
## $message$created$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## $message$created$`date-time`
## [1] "2012-07-06T12:14:34Z"
## 
## $message$created$timestamp
## [1] 1.341577e+12
## 
## 
## $message$source
## [1] "Crossref"
## 
## $message$`is-referenced-by-count`
## [1] 36
## 
## $message$title
## [1] "The Molecule Cloud - compact visualization of large collections of molecules"
## 
## $message$prefix
## [1] "10.1186"
## 
## $message$volume
## [1] "4"
## 
## $message$author
##      given family   sequence affiliation
## 1    Peter   Ertl      first        NULL
## 2 Bernhard  Rohde additional        NULL
## 
## $message$member
## [1] "297"
## 
## $message$`published-online`
## $message$`published-online`$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## 
## $message$reference
##         key doi-asserted-by first-page                        DOI volume
## 1   336_CR1       publisher         77  10.1007/s10822-011-9487-0     26
## 2   336_CR2       publisher       2174          10.1021/ci2001428     26
## 3   336_CR3       publisher       8732          10.1021/ja902302h    131
## 4   336_CR4       publisher        156 10.2174/157340908785747410      4
## 5   336_CR5       publisher        322 10.2174/157340908786786010      4
## 6   336_CR6       publisher         47          10.1021/ci600338x     47
## 7   336_CR7       publisher        366     10.1002/minf.201000019     29
## 8   336_CR8       publisher       4443          10.1021/jo8001276     73
## 9   336_CR9            <NA>       <NA>                       <NA>   <NA>
## 10 336_CR10            <NA>       <NA>                       <NA>   <NA>
## 11 336_CR11       publisher       D255         10.1093/nar/gkp965     38
## 12 336_CR12       publisher        177          10.1021/ci049714+     45
## 13 336_CR13       publisher      D1100         10.1093/nar/gkr777     40
## 14 336_CR14       publisher        347 10.1016/j.cbpa.2010.02.018     14
## 15 336_CR15       publisher        374          10.1021/ci0255782     43
## 16 336_CR16            <NA>       <NA>                       <NA>   <NA>
##              author year
## 1          E Martin 2011
## 2        SR Langdon 2011
## 3           LC Blum 2009
## 4          J Dubois 2008
## 5  JL Medina-Franco 2008
## 6   A Schuffenhauer 2007
## 7         S Langdon 2010
## 8         AH Lipkus 2008
## 9              <NA> <NA>
## 10             <NA> <NA>
## 11           Y Wang 2009
## 12         JJ Irwin 2004
## 13        A Gaulton 2012
## 14        ME Welsch 2010
## 15           P Ertl 2003
## 16             <NA> <NA>
##                                                                                                                                                                                                                                                                  unstructured
## 1                                                                                                                   Martin E, Ertl P, Hunt P, Duca J, Lewis R: Gazing into the crystal ball; the future of computer-aided drug design. J Comp-Aided Mol Des. 2011, 26: 77-79.
## 2                                                                                                                                           Langdon SR, Brown N, Blagg J: Scaffold diversity of exemplified medicinal chemistry space. J Chem Inf Model. 2011, 26: 2174-2185.
## 3                                                                                          Blum LC, Reymond J-C: 970 Million druglike small molecules for virtual screening in the chemical universe database GDB-13. J Am Chem Soc. 2009, 131: 8732-8733. 10.1021/ja902302h.
## 4                                                                                                       Dubois J, Bourg S, Vrain C, Morin-Allory L: Collections of compounds - how to deal with them?. Cur Comp-Aided Drug Des. 2008, 4: 156-168. 10.2174/157340908785747410.
## 5                                                                 Medina-Franco JL, Martinez-Mayorga K, Giulianotti MA, Houghten RA, Pinilla C: Visualization of the chemical space in drug discovery. Cur Comp-Aided Drug Des. 2008, 4: 322-333. 10.2174/157340908786786010.
## 6                                                  Schuffenhauer A, Ertl P, Roggo S, Wetzel S, Koch MA, Waldmann H: The Scaffold Tree - visualization of the scaffold universe by hierarchical scaffold classification. J Chem Inf Model. 2007, 47: 47-58. 10.1021/ci600338x.
## 7                                                                                                          Langdon S, Ertl P, Brown N: Bioisosteric replacement and scaffold hopping in lead generation and optimization. Mol Inf. 2010, 29: 366-385. 10.1002/minf.201000019.
## 8                                                            Lipkus AH, Yuan Q, Lucas KA, Funk SA, Bartelt WF, Schenck RJ, Trippe AJ: Structural diversity of organic chemistry. A scaffold analysis of the CAS Registry. J Org Chem. 2008, 73: 4443-4451. 10.1021/jo8001276.
## 9                                                                                                                                 mib 2010.10, Molinspiration Cheminformatics: \n                    http://www.molinspiration.com\n                    \n                  ,
## 10                                                                                                                Bernhard R: Avalon Cheminformatics Toolkit. \n                    http://sourceforge.net/projects/avalontoolkit/\n                    \n                  ,
## 11                                                                              Wang Y, Bolton E, Dracheva S, Karapetyan K, Shoemaker BA, Suzek TO, Wang J, Xiao J, Zhang J, Bryant SH: An overview of the PubChem BioAssay resource. Nucleic Acids Res. 2009, 38: D255-D266.
## 12                                                                                                                              Irwin JJ, Shoichet BK: ZINC − a free database of commercially available compounds for virtual screening. J Chem Inf Model. 2004, 45: 177-182.
## 13           Gaulton A, Bellis LJ, Bento AP, Chambers J, Davies M, Hersey A, Light Y, McGlinchey S, Michalovich D, Al-Lazikani B, Overington JP: ChEMBL: a large-scale bioactivity database for drug discovery. Nucleic Acids Res. 2012, 40: D1100-D1107. 10.1093/nar/gkr777.
## 14                                                                                                        Welsch ME, Snyder SA, Stockwell BR: Privileged scaffolds for library design and drug discovery. Curr Opin Chem Biol. 2010, 14: 347-361. 10.1016/j.cbpa.2010.02.018.
## 15 Ertl P: Cheminformatics analysis of organic substituents: Identification of the most common substituents, calculation of substituent properties, and automatic identification of drug-like bioisosteric groups. J Chem Inf Comp Sci. 2003, 43: 374-380. 10.1021/ci0255782.
## 16                                                                                                                                                                                                                        TagCrowd: \n                    http://tagcrowd.com
##              journal-title
## 1     J Comp-Aided Mol Des
## 2         J Chem Inf Model
## 3            J Am Chem Soc
## 4  Cur Comp-Aided Drug Des
## 5  Cur Comp-Aided Drug Des
## 6         J Chem Inf Model
## 7                  Mol Inf
## 8               J Org Chem
## 9                     <NA>
## 10                    <NA>
## 11       Nucleic Acids Res
## 12        J Chem Inf Model
## 13       Nucleic Acids Res
## 14     Curr Opin Chem Biol
## 15     J Chem Inf Comp Sci
## 16                    <NA>
## 
## $message$`container-title`
## [1] "Journal of Cheminformatics"
## 
## $message$`original-title`
## list()
## 
## $message$language
## [1] "en"
## 
## $message$link
##                                                                     URL
## 1       http://link.springer.com/content/pdf/10.1186/1758-2946-4-12.pdf
## 2 http://link.springer.com/article/10.1186/1758-2946-4-12/fulltext.html
## 3       http://link.springer.com/content/pdf/10.1186/1758-2946-4-12.pdf
##      content-type content-version intended-application
## 1 application/pdf             vor          text-mining
## 2       text/html             vor          text-mining
## 3 application/pdf             vor  similarity-checking
## 
## $message$deposited
## $message$deposited$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2019    6   24
## 
## $message$deposited$`date-time`
## [1] "2019-06-24T14:22:07Z"
## 
## $message$deposited$timestamp
## [1] 1.561386e+12
## 
## 
## $message$score
## [1] 1
## 
## $message$resource
## $message$resource$primary
## $message$resource$primary$URL
## [1] "https://jcheminf.biomedcentral.com/articles/10.1186/1758-2946-4-12"
## 
## 
## 
## $message$subtitle
## list()
## 
## $message$`short-title`
## list()
## 
## $message$issued
## $message$issued$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## 
## $message$`references-count`
## [1] 16
## 
## $message$`journal-issue`
## $message$`journal-issue`$issue
## [1] "1"
## 
## $message$`journal-issue`$`published-print`
## $message$`journal-issue`$`published-print`$`date-parts`
##      [,1] [,2]
## [1,] 2012   12
## 
## 
## 
## $message$`alternative-id`
## [1] "336"
## 
## $message$URL
## [1] "http://dx.doi.org/10.1186/1758-2946-4-12"
## 
## $message$relation
## named list()
## 
## $message$ISSN
## [1] "1758-2946"
## 
## $message$`issn-type`
##       value       type
## 1 1758-2946 electronic
## 
## $message$subject
## [1] "Library and Information Sciences"           
## [2] "Computer Graphics and Computer-Aided Design"
## [3] "Physical and Theoretical Chemistry"         
## [4] "Computer Science Applications"              
## 
## $message$published
## $message$published$`date-parts`
##      [,1] [,2] [,3]
## [1,] 2012    7    6
## 
## 
## $message$`article-number`
## [1] "12"
```

## 2. CrossRef API call with a Loop

### Setup API parameters


```r
base_url <- "https://api.crossref.org/works/"
email <- "your_email@ua.edu" #Change this to your email
mailto <- "?mailto="
doi <- "10.1186/1758-2946-4-12" #example
```

### Create a List of DOIs


```r
#Create List of DOIs

doi_list <- list('10.1021/acsomega.1c03250',                          
             '10.1021/acsomega.1c05512',
             '10.1021/acsomega.8b01647',
             '10.1021/acsomega.1c04287',
             '10.1021/acsomega.8b01834')
```

### Request metadata for each DOI from CrossRef API and save to a list


```r
doi_metadata <- list(list()) # a list of lists is used to hold the data from the 5 DOIs
i <- 1
for (doi in doi_list){
  doi_metadata[[i]] <-fromJSON(rawToChar(GET(paste(base_url, doi, mailto, email, sep=""))$content), flatten = TRUE)
  i <- i+1
  Sys.sleep(1)  # important to add a delay between API calls
}
# doi_metadata ## not shown here as it is long.
```

Let's select some specific data


```r
# Get Article titles
for (item in 1:length(doi_metadata)){
  print(doi_metadata[[item]]$message$title)
}
```

```
## [1] "Navigating into the Chemical Space of Monoamine Oxidase Inhibitors by Artificial Intelligence and Cheminformatics Approach"
## [1] "Impact of Artificial Intelligence on Compound Discovery, Design, and Synthesis"
## [1] "How Precise Are Our Quantitative Structure–Activity Relationship Derived Predictions for New Query Chemicals?"
## [1] "Applying Neuromorphic Computing Simulation in Band Gap Prediction and Chemical Reaction Classification"
## [1] "QSPR Modeling of the Refractive Index for Diverse Polymers Using 2D Descriptors"
```


```r
#Get Author affiliations for each Article
for (item in 1:length(doi_metadata)){
  for (au in 1:length(doi_metadata[[item]]$message$author)){
    print(doi_metadata[[item]]$message$author$affiliation[[1]]$name)
  }
}
```

```
## [1] "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
## [1] "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
## [1] "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
## [1] "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
## [1] "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
## [1] "Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India"
## [1] "Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany"
## [2] "Data Science and AI, Imaging and Data Analytics, Clinical Pharmacology & Safety Sciences, R&D, AstraZeneca, SE-431 83 Gothenburg, Sweden"                                                                                  
## [1] "Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany"
## [2] "Data Science and AI, Imaging and Data Analytics, Clinical Pharmacology & Safety Sciences, R&D, AstraZeneca, SE-431 83 Gothenburg, Sweden"                                                                                  
## [1] "Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany"
## [2] "Data Science and AI, Imaging and Data Analytics, Clinical Pharmacology & Safety Sciences, R&D, AstraZeneca, SE-431 83 Gothenburg, Sweden"                                                                                  
## [1] "Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany"
## [2] "Data Science and AI, Imaging and Data Analytics, Clinical Pharmacology & Safety Sciences, R&D, AstraZeneca, SE-431 83 Gothenburg, Sweden"                                                                                  
## [1] "Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany"
## [2] "Data Science and AI, Imaging and Data Analytics, Clinical Pharmacology & Safety Sciences, R&D, AstraZeneca, SE-431 83 Gothenburg, Sweden"                                                                                  
## [1] "Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany"
## [2] "Data Science and AI, Imaging and Data Analytics, Clinical Pharmacology & Safety Sciences, R&D, AstraZeneca, SE-431 83 Gothenburg, Sweden"                                                                                  
## [1] "Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India"
## [1] "Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India"
## [1] "Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India"
## [1] "Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India"
## [1] "Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India"
## [1] "Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India"
## [1] "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
## [1] "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
## [1] "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
## [1] "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
## [1] "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
## [1] "Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States"
## [1] "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
## [1] "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
## [1] "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
## [1] "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
## [1] "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
## [1] "Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India"
```

## 3. Crossref API call for journal information

### Setup API parameters


```r
jbase_url <- "https://api.crossref.org/journals/" # the base url for api calls
email <- "your_email@ua.edu" # Change this to be your email
mailto <- "?mailto="
issn <- "1471-2105"  # issn for the journal BMC Bioinformatics
```

### Request journal data from crossref API


```r
jour_data <- fromJSON(rawToChar(GET(paste(jbase_url, issn, mailto, email, sep=""))$content), flatten = TRUE) 
jour_data
```

```
:tags: ["output_scroll"]
## $status
## [1] "ok"
## 
## $`message-type`
## [1] "journal"
## 
## $`message-version`
## [1] "1.0.0"
## 
## $message
## $message$`last-status-check-time`
## [1] 1.679633e+12
## 
## $message$counts
## $message$counts$`current-dois`
## [1] 1306
## 
## $message$counts$`backfile-dois`
## [1] 10498
## 
## $message$counts$`total-dois`
## [1] 11804
## 
## 
## $message$breakdowns
## $message$breakdowns$`dois-by-issued-year`
##       [,1] [,2]
##  [1,] 2010  861
##  [2,] 2019  762
##  [3,] 2008  745
##  [4,] 2009  729
##  [5,] 2011  722
##  [6,] 2006  633
##  [7,] 2021  622
##  [8,] 2014  619
##  [9,] 2007  613
## [10,] 2012  609
## [11,] 2013  607
## [12,] 2015  603
## [13,] 2020  585
## [14,] 2017  585
## [15,] 2022  582
## [16,] 2016  550
## [17,] 2018  536
## [18,] 2005  414
## [19,] 2004  209
## [20,] 2023  102
## [21,] 2003   66
## [22,] 2002   40
## [23,] 2001    9
## [24,] 2000    1
## 
## 
## $message$publisher
## [1] "Springer (Biomed Central Ltd.)"
## 
## $message$coverage
## $message$coverage$`affiliations-current`
## [1] 0
## 
## $message$coverage$`similarity-checking-current`
## [1] 1
## 
## $message$coverage$`descriptions-current`
## [1] 0
## 
## $message$coverage$`ror-ids-current`
## [1] 0
## 
## $message$coverage$`funders-backfile`
## [1] 0.1704134
## 
## $message$coverage$`licenses-backfile`
## [1] 0.561726
## 
## $message$coverage$`funders-current`
## [1] 0.6906585
## 
## $message$coverage$`affiliations-backfile`
## [1] 0
## 
## $message$coverage$`resource-links-backfile`
## [1] 0.5611545
## 
## $message$coverage$`orcids-backfile`
## [1] 0.1299295
## 
## $message$coverage$`update-policies-current`
## [1] 1
## 
## $message$coverage$`ror-ids-backfile`
## [1] 0
## 
## $message$coverage$`orcids-current`
## [1] 0.4701378
## 
## $message$coverage$`similarity-checking-backfile`
## [1] 0.9561821
## 
## $message$coverage$`references-backfile`
## [1] 0.9400838
## 
## $message$coverage$`descriptions-backfile`
## [1] 0
## 
## $message$coverage$`award-numbers-backfile`
## [1] 0.1554582
## 
## $message$coverage$`update-policies-backfile`
## [1] 0.660983
## 
## $message$coverage$`licenses-current`
## [1] 1
## 
## $message$coverage$`award-numbers-current`
## [1] 0.6117917
## 
## $message$coverage$`abstracts-backfile`
## [1] 0.3347304
## 
## $message$coverage$`resource-links-current`
## [1] 1
## 
## $message$coverage$`abstracts-current`
## [1] 0.9785605
## 
## $message$coverage$`references-current`
## [1] 0.9984686
## 
## 
## $message$title
## [1] "BMC Bioinformatics"
## 
## $message$subjects
##   ASJC                          name
## 1 2604           Applied Mathematics
## 2 1706 Computer Science Applications
## 3 1312             Molecular Biology
## 4 1303                  Biochemistry
## 5 1315            Structural Biology
## 
## $message$`coverage-type`
## $message$`coverage-type`$all
## $message$`coverage-type`$all$`last-status-check-time`
## [1] 1.679633e+12
## 
## $message$`coverage-type`$all$affiliations
## [1] 0
## 
## $message$`coverage-type`$all$abstracts
## [1] 0.4059641
## 
## $message$`coverage-type`$all$orcids
## [1] 0.1675703
## 
## $message$`coverage-type`$all$licenses
## [1] 0.6102169
## 
## $message$`coverage-type`$all$references
## [1] 0.9465435
## 
## $message$`coverage-type`$all$funders
## [1] 0.2279736
## 
## $message$`coverage-type`$all$`similarity-checking`
## [1] 0.9610302
## 
## $message$`coverage-type`$all$`award-numbers`
## [1] 0.2059471
## 
## $message$`coverage-type`$all$`ror-ids`
## [1] 0
## 
## $message$`coverage-type`$all$`update-policies`
## [1] 0.698492
## 
## $message$`coverage-type`$all$`resource-links`
## [1] 0.6097086
## 
## $message$`coverage-type`$all$descriptions
## [1] 0
## 
## 
## $message$`coverage-type`$backfile
## $message$`coverage-type`$backfile$`last-status-check-time`
## [1] 1.679633e+12
## 
## $message$`coverage-type`$backfile$affiliations
## [1] 0
## 
## $message$`coverage-type`$backfile$abstracts
## [1] 0.3347304
## 
## $message$`coverage-type`$backfile$orcids
## [1] 0.1299295
## 
## $message$`coverage-type`$backfile$licenses
## [1] 0.561726
## 
## $message$`coverage-type`$backfile$references
## [1] 0.9400838
## 
## $message$`coverage-type`$backfile$funders
## [1] 0.1704134
## 
## $message$`coverage-type`$backfile$`similarity-checking`
## [1] 0.9561821
## 
## $message$`coverage-type`$backfile$`award-numbers`
## [1] 0.1554582
## 
## $message$`coverage-type`$backfile$`ror-ids`
## [1] 0
## 
## $message$`coverage-type`$backfile$`update-policies`
## [1] 0.660983
## 
## $message$`coverage-type`$backfile$`resource-links`
## [1] 0.5611545
## 
## $message$`coverage-type`$backfile$descriptions
## [1] 0
## 
## 
## $message$`coverage-type`$current
## $message$`coverage-type`$current$`last-status-check-time`
## [1] 1.679633e+12
## 
## $message$`coverage-type`$current$affiliations
## [1] 0
## 
## $message$`coverage-type`$current$abstracts
## [1] 0.9785605
## 
## $message$`coverage-type`$current$orcids
## [1] 0.4701378
## 
## $message$`coverage-type`$current$licenses
## [1] 1
## 
## $message$`coverage-type`$current$references
## [1] 0.9984686
## 
## $message$`coverage-type`$current$funders
## [1] 0.6906585
## 
## $message$`coverage-type`$current$`similarity-checking`
## [1] 1
## 
## $message$`coverage-type`$current$`award-numbers`
## [1] 0.6117917
## 
## $message$`coverage-type`$current$`ror-ids`
## [1] 0
## 
## $message$`coverage-type`$current$`update-policies`
## [1] 1
## 
## $message$`coverage-type`$current$`resource-links`
## [1] 1
## 
## $message$`coverage-type`$current$descriptions
## [1] 0
## 
## 
## 
## $message$flags
## $message$flags$`deposits-abstracts-current`
## [1] TRUE
## 
## $message$flags$`deposits-orcids-current`
## [1] TRUE
## 
## $message$flags$deposits
## [1] TRUE
## 
## $message$flags$`deposits-affiliations-backfile`
## [1] FALSE
## 
## $message$flags$`deposits-update-policies-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-similarity-checking-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-award-numbers-current`
## [1] TRUE
## 
## $message$flags$`deposits-resource-links-current`
## [1] TRUE
## 
## $message$flags$`deposits-ror-ids-current`
## [1] FALSE
## 
## $message$flags$`deposits-articles`
## [1] TRUE
## 
## $message$flags$`deposits-affiliations-current`
## [1] FALSE
## 
## $message$flags$`deposits-funders-current`
## [1] TRUE
## 
## $message$flags$`deposits-references-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-ror-ids-backfile`
## [1] FALSE
## 
## $message$flags$`deposits-abstracts-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-licenses-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-award-numbers-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-descriptions-current`
## [1] FALSE
## 
## $message$flags$`deposits-references-current`
## [1] TRUE
## 
## $message$flags$`deposits-resource-links-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-descriptions-backfile`
## [1] FALSE
## 
## $message$flags$`deposits-orcids-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-funders-backfile`
## [1] TRUE
## 
## $message$flags$`deposits-update-policies-current`
## [1] TRUE
## 
## $message$flags$`deposits-similarity-checking-current`
## [1] TRUE
## 
## $message$flags$`deposits-licenses-current`
## [1] TRUE
## 
## 
## $message$ISSN
## [1] "1471-2105" "1471-2105"
## 
## $message$`issn-type`
##       value       type
## 1 1471-2105      print
## 2 1471-2105 electronic
```

## 4. Crossref API - Get article DOIs for a journal

### Setup API parameters


```r
jbase_url <- "https://api.crossref.org/journals/" # the base url for api calls
email <- "your_email@ua.edu" # Change this to be your email
mailto = "&mailto="
issn <- "1471-2105"  # issn for the journal BMC Bioinformatics
journal_works2014 <- "/works?filter=from-pub-date:2014,until-pub-date:2014&select=DOI" # query to get DOIs for 2014
```

### Request DOI data from crossref API


```r
doi_data = fromJSON(rawToChar(GET(paste(jbase_url, issn, journal_works2014, mailto, email, sep=""))$content), flatten = TRUE)
doi_data
```

```
## $status
## [1] "ok"
## 
## $`message-type`
## [1] "work-list"
## 
## $`message-version`
## [1] "1.0.0"
## 
## $message
## $message$facets
## named list()
## 
## $message$`total-results`
## [1] 619
## 
## $message$items
##                             DOI
## 1   10.1186/1471-2105-15-s10-p5
## 2      10.1186/1471-2105-15-166
## 3       10.1186/1471-2105-15-83
## 4  10.1186/1471-2105-15-s10-p28
## 5   10.1186/1471-2105-15-s10-p6
## 6       10.1186/1471-2105-15-16
## 7   10.1186/1471-2105-15-s11-i1
## 8    10.1186/1471-2105-15-s1-s2
## 9     10.1186/s12859-014-0358-2
## 10 10.1186/1471-2105-15-s10-p35
## 11   10.1186/1471-2105-15-s7-s1
## 12 10.1186/1471-2105-15-s10-p32
## 13   10.1186/1471-2105-15-s4-s1
## 14  10.1186/1471-2105-15-s9-s12
## 15    10.1186/s12859-014-0415-x
## 16  10.1186/1471-2105-15-s13-s1
## 17     10.1186/1471-2105-15-305
## 18    10.1186/s12859-014-0411-1
## 19 10.1186/1471-2105-15-s10-p33
## 20   10.1186/1471-2105-15-s3-a2
## 
## $message$`items-per-page`
## [1] 20
## 
## $message$query
## $message$query$`start-index`
## [1] 0
## 
## $message$query$`search-terms`
## NULL
```

By default, 20 results are displayed. Crossref allows up to 1000 returned results using the rows parameter. To get all 619 results, we can increase the number of returned rows.


```r
rows <- "&rows=700"
doi_data_all <- fromJSON(rawToChar(GET(paste(jbase_url, issn, journal_works2014, rows, mailto, email, sep=""))$content), flatten = TRUE)
```

### Extract DOIs


```r
dois_list = list()
for (i in doi_data_all$message$items){
  dois_list <- append(x = dois_list, doi_data_all$message$items$DOI)
}
length(dois_list)
```

```
## [1] 619
```


```r
#display the first 20
print (dois_list[0:20])
```

```
## [[1]]
## [1] "10.1186/1471-2105-15-s7-s7"
## 
## [[2]]
## [1] "10.1186/1471-2105-15-84"
## 
## [[3]]
## [1] "10.1186/s12859-014-0411-1"
## 
## [[4]]
## [1] "10.1186/1471-2105-15-177"
## 
## [[5]]
## [1] "10.1186/1471-2105-15-s10-p24"
## 
## [[6]]
## [1] "10.1186/1471-2105-15-s1-s2"
## 
## [[7]]
## [1] "10.1186/1471-2105-15-s10-p32"
## 
## [[8]]
## [1] "10.1186/1471-2105-15-193"
## 
## [[9]]
## [1] "10.1186/1471-2105-15-192"
## 
## [[10]]
## [1] "10.1186/1471-2105-15-s11-i1"
## 
## [[11]]
## [1] "10.1186/1471-2105-15-88"
## 
## [[12]]
## [1] "10.1186/1471-2105-15-331"
## 
## [[13]]
## [1] "10.1186/1471-2105-15-s11-s10"
## 
## [[14]]
## [1] "10.1186/1471-2105-15-s1-s8"
## 
## [[15]]
## [1] "10.1186/1471-2105-15-s14-s1"
## 
## [[16]]
## [1] "10.1186/1471-2105-15-s10-p33"
## 
## [[17]]
## [1] "10.1186/1471-2105-15-s10-p35"
## 
## [[18]]
## [1] "10.1186/s12859-014-0358-2"
## 
## [[19]]
## [1] "10.1186/1471-2105-15-s10-p6"
## 
## [[20]]
## [1] "10.1186/1471-2105-15-115"
```

What if we have more than 1000 results in a single query?

For example, if we wanted the DOIs from BMC Bioinformatics for years 2014 through 2016?


```r
jbase_url <- "https://api.crossref.org/journals/" # the base url for api calls
email <- "your_email@ua.edu" # Change this to be your email
mailto <- "&mailto="
issn <- "1471-2105"  # issn for the journal BMC Bioinformatics
journal_works2014_2016 <- "/works?filter=from-pub-date:2014,until-pub-date:2016&select=DOI" # query to get DOIs for 2014-2016
doi_data2 <- fromJSON(paste(jbase_url, issn, journal_works2014_2016, mailto, email, sep=""), flatten = TRUE)
doi_data2
```

```
## $status
## [1] "ok"
## 
## $`message-type`
## [1] "work-list"
## 
## $`message-version`
## [1] "1.0.0"
## 
## $message
## $message$facets
## named list()
## 
## $message$`total-results`
## [1] 1772
## 
## $message$items
##                             DOI
## 1     10.1186/s12859-015-0538-8
## 2   10.1186/1471-2105-16-s18-s5
## 3     10.1186/s12859-016-1086-6
## 4     10.1186/s12859-016-1343-8
## 5    10.1186/1471-2105-15-s7-s7
## 6       10.1186/1471-2105-15-84
## 7     10.1186/s12859-014-0411-1
## 8      10.1186/1471-2105-15-177
## 9  10.1186/1471-2105-16-s15-p13
## 10    10.1186/s12859-015-0699-5
## 11 10.1186/1471-2105-15-s10-p24
## 12    10.1186/s12859-016-1297-x
## 13    10.1186/s12859-015-0791-x
## 14   10.1186/1471-2105-15-s1-s2
## 15    10.1186/s12859-015-0834-3
## 16    10.1186/s12859-016-1009-6
## 17 10.1186/1471-2105-15-s10-p32
## 18    10.1186/s12859-016-1129-z
## 19     10.1186/1471-2105-15-193
## 20     10.1186/1471-2105-15-192
## 
## $message$`items-per-page`
## [1] 20
## 
## $message$query
## $message$query$`start-index`
## [1] 0
## 
## $message$query$`search-terms`
## NULL
```

Here we see that the total results is over 1000 (total-results: 1772). An additional parameter that we can use with crossref API is called "offset". The offset option allows us to select sets of records and define a starting position (e.g., the first 1000, and then the second set of up to 1000.)


```r
doi_list2 <- list() #declare a list
rows <- "&rows=1000"

temp <- fromJSON(paste(jbase_url, issn, journal_works2014_2016, mailto, email, sep=""), flatten = TRUE)
numResults<- temp$message$`total-results`
Sys.sleep(1)

end <- as.integer(numResults/1000)+1

for (n in 0:end){ # list(range(int(numberOfResults/1000)+1)) = [0,1]
  query <- fromJSON(paste(jbase_url, issn, journal_works2014_2016, rows, "&offset=", as.character(1000*n), mailto, email, sep=""), flatten = TRUE)
  Sys.sleep(1)
  for (doi in query$message$items){
    doi_list2 <- append(x = doi_list2, query$message$items$DOI)
  }
}
```


```r
length(doi_list2)
```

```
## [1] 1772
```


```r
#how results 1000 through 1020
print(doi_list2[1000:1020])
```

```
## [[1]]
## [1] "10.1186/1471-2105-15-s11-s2"
## 
## [[2]]
## [1] "10.1186/1471-2105-15-228"
## 
## [[3]]
## [1] "10.1186/1471-2105-15-s2-s7"
## 
## [[4]]
## [1] "10.1186/s12859-014-0349-3"
## 
## [[5]]
## [1] "10.1186/1471-2105-15-327"
## 
## [[6]]
## [1] "10.1186/1471-2105-15-s1-s14"
## 
## [[7]]
## [1] "10.1186/s12859-015-0768-9"
## 
## [[8]]
## [1] "10.1186/1471-2105-15-s1-s4"
## 
## [[9]]
## [1] "10.1186/s12859-016-1253-9"
## 
## [[10]]
## [1] "10.1186/1471-2105-15-314"
## 
## [[11]]
## [1] "10.1186/s12859-016-1147-x"
## 
## [[12]]
## [1] "10.1186/s12859-016-1386-x"
## 
## [[13]]
## [1] "10.1186/1471-2105-15-s10-p13"
## 
## [[14]]
## [1] "10.1186/1471-2105-16-s15-p7"
## 
## [[15]]
## [1] "10.1186/s12859-014-0361-7"
## 
## [[16]]
## [1] "10.1186/s12859-014-0429-4"
## 
## [[17]]
## [1] "10.1186/1471-2105-15-s10-p8"
## 
## [[18]]
## [1] "10.1186/1471-2105-15-48"
## 
## [[19]]
## [1] "10.1186/s12859-016-1342-9"
## 
## [[20]]
## [1] "10.1186/s12859-015-0746-2"
## 
## [[21]]
## [1] "10.1186/s12859-016-1351-8"
```
## R Session Info

```r
sessionInfo()
```

```
## R version 4.2.1 (2022-06-23 ucrt)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 19042)
## 
## Matrix products: default
## 
## locale:
## [1] LC_COLLATE=English_United States.utf8 
## [2] LC_CTYPE=English_United States.utf8   
## [3] LC_MONETARY=English_United States.utf8
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.utf8    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
##  [1] jsonlite_1.8.4  httr_1.4.5      lubridate_1.9.2 forcats_1.0.0  
##  [5] stringr_1.5.0   dplyr_1.1.0     purrr_1.0.1     readr_2.1.4    
##  [9] tidyr_1.3.0     tibble_3.1.8    ggplot2_3.4.1   tidyverse_2.0.0
## 
## loaded via a namespace (and not attached):
##  [1] bslib_0.4.2      compiler_4.2.1   pillar_1.8.1     jquerylib_0.1.4 
##  [5] tools_4.2.1      digest_0.6.31    timechange_0.2.0 evaluate_0.20   
##  [9] lifecycle_1.0.3  gtable_0.3.1     pkgconfig_2.0.3  rlang_1.0.6     
## [13] cli_3.6.0        rstudioapi_0.14  curl_5.0.0       yaml_2.3.7      
## [17] xfun_0.37        fastmap_1.1.0    withr_2.5.0      knitr_1.42      
## [21] hms_1.1.2        generics_0.1.3   vctrs_0.5.2      sass_0.4.5      
## [25] grid_4.2.1       tidyselect_1.2.0 glue_1.6.2       R6_2.5.1        
## [29] fansi_1.0.4      rmarkdown_2.20   tzdb_0.3.0       magrittr_2.0.3  
## [33] ellipsis_0.3.2   scales_1.2.1     htmltools_0.5.4  colorspace_2.1-0
## [37] utf8_1.2.3       stringi_1.7.12   munsell_0.5.0    cachem_1.0.7
```
