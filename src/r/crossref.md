---
title: "CrossRef API in R"
output:
  html_document:
    keep_md: TRUE
---



# Crossref API in R

By Michael T. Moen, Cyrus Gomes, Vincent F. Scalfani, and Adam M. Nguyen

The Crossref API provides metadata about publications, including articles, books, and conference proceedings. This metadata spans items such as author details, journal details, references, and DOIs (Digital Object Identifiers). Working with Crossref allows for programmatic access to bibliographic information and can streamline large-scale metadata retrieval.

Please see the following resources for more information on API usage:
- Documentation
    - <a href="https://api.crossref.org/swagger-ui/index.html" target="_blank">Crossref API Documentation</a>
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/a-non-technical-introduction-to-our-api/" target="_blank">Crossref API Introduction</a>
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/text-and-data-mining/" target="_blank">Crossref Data Mining</a>
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/text-and-data-mining-for-members/" target="_blank">Crossref Data Mining for Members</a>
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/text-and-data-mining-for-researchers/" target="_blank">Crossref Data Mining for Researchers</a>
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/providing-full-text-links-to-tdm-tools/" target="_blank">Crossref Full-Text Links</a>
- Terms
    - <a href="https://www.crossref.org/membership/terms/" target="_blank">Crossref Terms of Use</a>
- Data Reuse
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/rest-api-metadata-license-information/" target="_blank">Crossref Metadata Reuse</a>
    - <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/providing-licensing-information-to-tdm-tools/" target="_blank">Crossref TDM Licensing</a>

**_NOTE:_** The <a href="https://api.crossref.org/swagger-ui/index.html" target="_blank">Crossref API</a> limits requests to a maximum of 50 per second.

*These recipe examples were tested on March 23, 2026.*

_**Note:**_ From our testing, we have found that the Crossref metadata across publishers and even journals can vary considerably. As a result, it can be easier to work with one journal at a time when using the Crossref API (particularly when trying to extract selected data from records).

## Setup

### Load Libraries

The following packages need to be installed into your environment to run the code examples in this tutorial. These packages can be installed with `install.packages()`.

- <a href="https://cran.r-project.org/web/packages/httr/index.html" target="_blank">httr: Tools for Working with URLs and HTTP</a>
- <a href="https://cran.r-project.org/web/packages/jsonlite/index.html" target="_blank">jsonlite: A Simple and Robust JSON Parser and Generator for R</a>

We load the libraries used in this tutorial below:


``` r
library(httr)
library(jsonlite)
```

### Import API Key

It is important to provide an email address when making requests to the Crossref API. This is used to contact you in case of any issues with your requests.

We keep our token in a `.Renviron` file that is stored in the working directory and use `Sys.getenv()` to access it. The `.Renviron` should have an entry like the one below.

```text
CROSSREF_EMAIL="PUT_YOUR_EMAIL_HERE"
```

Below, we can test to whether the email was successfully imported.


``` r
if (nzchar(Sys.getenv("CROSSREF_EMAIL"))) {
  print("Email successfully loaded.")
} else {
  warning("Email not found or is empty.")
}
```

```
## [1] "Email successfully loaded."
```

## 1. Basic Crossref API Call

In this section, we perform a basic API call to the Crossref service to retrieve metadata for a single DOI.

We will:
1. Build the Crossref endpoint using our base URL, DOI, and the `mailto` parameter.
2. Retrieve the response.
3. Examine and parse the JSON data.


``` r
# Set base URL to Crossref API
WORKS_URL <- "https://api.crossref.org/works/"

doi <- "10.1186/1758-2946-4-12"  # Example DOI
params <- list(
  mailto = Sys.getenv("CROSSREF_EMAIL")
)

response <- GET(paste0(WORKS_URL, doi), query = params)

# Status code 200 indicates success
response$status_code
```

```
## [1] 200
```
This calls the Crossref API to retrieve metadata for a single DOI, but the data is in a hexadecimal format. We can extract the information we need from the call using `fromJSON()` with `rawToChar()`.


``` r
data <- fromJSON(rawToChar(response$content))
data$status
```

```
## [1] "ok"
```

### Extract Data from API Response

In the snippet below, we parse and extract some key fields from the response:
1. **Journal title** via the `container-title` key.
2. **Article title** via the `title` key.
3. **Author names** via the `author` key.
4. **Bibliographic references** via the `reference` key.


``` r
# Extract journal title
data$message$`container-title`
```

```
## [1] "Journal of Cheminformatics"
```


``` r
# Extract article title
data$message$title
```

```
## [1] "The Molecule Cloud - compact visualization of large collections of molecules"
```


``` r
# Extract author names
authors_data <- data$message$author
authors <- paste(authors_data$given, authors_data$family)
authors
```

```
## [1] "Peter Ertl"     "Bernhard Rohde"
```


``` r
# Extract bibliography references
bib_refs <- data$message$reference$unstructured

# Print first 3 references
bib_refs[1:3]
```

```
## [1] "Martin E, Ertl P, Hunt P, Duca J, Lewis R: Gazing into the crystal ball; the future of computer-aided drug design. J Comp-Aided Mol Des. 2011, 26: 77-79."                         
## [2] "Langdon SR, Brown N, Blagg J: Scaffold diversity of exemplified medicinal chemistry space. J Chem Inf Model. 2011, 26: 2174-2185."                                                 
## [3] "Blum LC, Reymond J-C: 970 Million druglike small molecules for virtual screening in the chemical universe database GDB-13. J Am Chem Soc. 2009, 131: 8732-8733. 10.1021/ja902302h."
```

## 2. Crossref API Call with a Loop

In this section, we want to request metadata from multiple DOIs at once. We will:
1. Create a list of several DOIs.
2. Loop through that list, calling the Crossref API for each DOI.
3. Store each response in a new list.
4. Parse specific data, such as article titles and affiliations.

> **Note**: We include a one-second sleep (`Sys.sleep(1)`) between requests to respect Crossref's <a href="https://api.crossref.org/swagger-ui/index.html" target="_blank">policies</a>. Crossref has usage guidelines that discourage extremely rapid repeated requests. Please also check out Crossref's <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/tips-for-using-public-data-files-and-plus-snapshots/" target="_blank">public data file</a> for bulk downloads.


``` r
dois <- c(
  '10.1021/acsomega.1c03250',                          
  '10.1021/acsomega.1c05512',
  '10.1021/acsomega.8b01647',
  '10.1021/acsomega.1c04287',
  '10.1021/acsomega.8b01834'
)

doi_metadata <- list(list()) # A list of lists is used to hold the data from the 5 DOIs
i <- 1
for (doi in dois) {
  response <- GET(paste0(WORKS_URL, doi), query = params)
  doi_metadata[[i]] <- fromJSON(rawToChar(response$content))
  i <- i + 1
  Sys.sleep(1)  # Important to add a delay between API calls
}
```


``` r
# Extract article titles
titles <- sapply(doi_metadata, function(x) x$message$title)
titles
```

```
## [1] "Navigating into the Chemical Space of Monoamine Oxidase Inhibitors by Artificial Intelligence and Cheminformatics Approach"
## [2] "Impact of Artificial Intelligence on Compound Discovery, Design, and Synthesis"                                            
## [3] "How Precise Are Our Quantitative Structure–Activity Relationship Derived Predictions for New Query Chemicals?"             
## [4] "Applying Neuromorphic Computing Simulation in Band Gap Prediction and Chemical Reaction Classification"                    
## [5] "QSPR Modeling of the Refractive Index for Diverse Polymers Using 2D Descriptors"
```


``` r
i <- 1
for (article in doi_metadata) {

  cat(paste("DOI", i, "\n"))
  authors <- article$message$author

  for (k in seq_len(NROW(authors))) {
    affiliations <- authors$affiliation[[k]]
    if (is.null(affiliations) || length(affiliations) == 0) {
      cat(" - No affiliation provided\n")
    } else {
      cat(" - ", affiliations$name[1], "\n", sep = "")
    }
  }

  i <- i + 1
  cat("\n")
}
```

```
## DOI 1 
##  - Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India
##  - Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India
##  - Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India
##  - Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India
##  - Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India
##  - Department of Pharmaceutics and Industrial Pharmacy, College of Pharmacy, Taif University, P.O. Box 11099, Taif 21944, Saudi Arabia
##  - Department of Pharmaceutical Chemistry, College of Pharmacy, Jouf University, Sakaka, Al Jouf 72341, Saudi Arabia
##  - Department of Pharmaceutical Chemistry and Analysis, Amrita School of Pharmacy, Amrita Vishwa Vidyapeetham, AIMS Health Sciences Campus, Kochi 682041, India
## 
## DOI 2 
##  - Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany
##  - Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany
##  - Department of Life Science Informatics and Data Science, B-IT, LIMES Program Unit Chemical Biology and Medicinal Chemistry, Rheinische Friedrich-Wilhelms-Universität, Friedrich-Hirzebruch-Allee 6, D-53115 Bonn, Germany
## 
## DOI 3 
##  - Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India
##  - Drug Theoretics and Cheminformatics Laboratory, Department of Pharmaceutical Technology, Jadavpur University, Kolkata 700 032, India
##  - Interdisciplinary Center for Nanotoxicity, Department of Chemistry, Physics and Atmospheric Sciences, Jackson State University, Jackson, Mississippi 39217, United States
## 
## DOI 4 
##  - Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States
##  - Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States
##  - Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States
##  - Department of Chemical and Biomolecular Engineering, The Ohio State University, Columbus, Ohio 43210, United States
## 
## DOI 5 
##  - Department of Pharmacoinformatics, National Institute of Pharmaceutical Educational and Research (NIPER), Chunilal Bhawan, 168, Manikata Main Road, 700054 Kolkata, India
##  - Department of Coatings and Polymeric Materials, North Dakota State University, Fargo, North Dakota 58108-6050, United States
##  - Drug Theoretics and Cheminformatics Laboratory, Division of Medicinal and Pharmaceutical Chemistry, Department of Pharmaceutical Technology, Jadavpur University, 700032 Kolkata, India
```

## 3. Retrieve Journal Information

Crossref also provides an endpoint to query journal metadata using the **ISSN**. In this section, we:
1. Use the `journals` endpoint.
2. Provide an ISSN.
3. Inspect the returned JSON data.


``` r
JOURNALS_URL <- "https://api.crossref.org/journals/"
params <- list(
  mailto = Sys.getenv("CROSSREF_EMAIL")
)

# ISSN for the journal BMC Bioinformatics
issn <- "1471-2105"

response <- GET(paste0(JOURNALS_URL, issn), query = params)
data <- fromJSON(rawToChar(response$content))

# Print top-level structure of the response message
str(data$message, max.level = 1)
```

```
## List of 11
##  $ last-status-check-time: num 1.76e+12
##  $ counts                :List of 3
##  $ breakdowns            :List of 1
##  $ publisher             : chr "Springer (Biomed Central Ltd.)"
##  $ coverage              :List of 24
##  $ title                 : chr "BMC Bioinformatics"
##  $ subjects              : list()
##  $ coverage-type         :List of 3
##  $ flags                 :List of 26
##  $ ISSN                  : chr [1:2] "1471-2105" "1471-2105"
##  $ issn-type             :'data.frame':	2 obs. of  2 variables:
```


``` r
# Extract total number of articles from the journal in Crossref
data$message$counts$`total-dois`
```

```
## [1] 12831
```


``` r
# Extract percentage of articles from the journal with abstracts in Crossref
data$message$coverage$`abstracts-current`
```

```
## [1] 0.7874225
```

## 4. Get Article DOIs for a Journal

We can get all article DOIs for a given journal and year range by combining the **journals** endpoint with **filters**.
For example, to retrieve all DOIs for BMC Bioinformatics published in **2014**, we filter between the start date (`from-pub-date`) and end date (`until-pub-date`) of 2014.

> **Note**: By default, the API only returns the first 20 results. We can specify `rows` to increase this up to **1000**. If the total number of results is **greater** than 1000, we can use the `offset` parameter to page through the results in multiple calls.

Below, we demonstrate:
1. Filtering to get only DOIs from 2014.
2. Increasing the `rows` to 700.
3. Pushing beyond the 1000-row limit by using `offset`.

### Retrieve and Display First 20 DOIs


``` r
params <- list(
  select = "DOI",
  filter = "from-pub-date:2014,until-pub-date:2014",
  mailto = Sys.getenv("CROSSREF_EMAIL")
)
issn <- "1471-2105"  # ISSN for the journal BMC Bioinformatics

response <- GET(paste0(JOURNALS_URL, issn, "/works"), query = params)
doi_data = fromJSON(rawToChar(response$content))

# Print DOIs from the response
doi_data$message$items$DOI
```

```
##  [1] "10.1186/1471-2105-15-38"      "10.1186/1471-2105-15-s10-p35"
##  [3] "10.1186/1471-2105-15-s10-p24" "10.1186/1471-2105-15-122"    
##  [5] "10.1186/1471-2105-15-24"      "10.1186/s12859-014-0397-8"   
##  [7] "10.1186/1471-2105-15-16"      "10.1186/s12859-014-0411-1"   
##  [9] "10.1186/1471-2105-15-268"     "10.1186/1471-2105-15-119"    
## [11] "10.1186/1471-2105-15-s6-s3"   "10.1186/s12859-014-0376-0"   
## [13] "10.1186/1471-2105-15-310"     "10.1186/1471-2105-15-335"    
## [15] "10.1186/1471-2105-15-192"     "10.1186/1471-2105-15-95"     
## [17] "10.1186/1471-2105-15-s9-s12"  "10.1186/1471-2105-15-254"    
## [19] "10.1186/1471-2105-15-152"     "10.1186/1471-2105-15-333"
```

### Increase Rows to Retrieve More Than 20 DOIs


``` r
# Add the rows parameter to increase the number of results
params <- list(
  select = "DOI",
  filter = "from-pub-date:2014,until-pub-date:2014",
  rows = 700,
  mailto = Sys.getenv("CROSSREF_EMAIL")
)
issn <- "1471-2105"  # ISSN for the journal BMC Bioinformatics

response <- GET(paste0(JOURNALS_URL, issn, "/works"), query = params)
doi_data = fromJSON(rawToChar(response$content))

# Print number of DOIs in the response
length(doi_data$message$items$DOI)
```

```
## [1] 619
```

### Paged Retrieval with Offsets

If we need more than 1000 records, we can combine `rows=1000` with the `offset` parameter. We:
1. Determine the total number of results (`total-results`).
2. Calculate how many loops we need based on 1000 items per page.
3. For each page, we adjust the `offset` by `1000 * n`.
4. Collect all DOIs into one large list.


``` r
# First, get total number of results to see if we exceed 1000
params <- list(
  filter = "from-pub-date:2014,until-pub-date:2016",
  select = "DOI",
  rows = 1000,
  mailto = Sys.getenv("CROSSREF_EMAIL")
)
response <- GET(paste0(JOURNALS_URL, issn, "/works"), query = params)
initial_data <- fromJSON(rawToChar(response$content))

num_results <- initial_data$message$`total-results`
cat("Total results for 2014-2016:", num_results)
```

```
## Total results for 2014-2016: 1772
```


``` r
# Page through results if more than 1000
journal_dois <- c()

# Calculate how many pages we need
pages_needed <- as.integer(num_results/1000) + 1

for (n in 0:pages_needed) {
  # Build URL using offset
  params <- list(
    filter = "from-pub-date:2014,until-pub-date:2016",
    select = "DOI",
    rows = 1000,
    offset = as.character(1000 * n),
    mailto = Sys.getenv("CROSSREF_EMAIL")
  )
  response <- GET(paste0(JOURNALS_URL, issn, "/works"), query = params)
  page_data <- fromJSON(rawToChar(response$content))
  journal_dois <- c(journal_dois, page_data$message$items$DOI)
  Sys.sleep(1)  # Important to respect Crossref usage guidelines
}

# Print number of DOIs extracted
length(journal_dois)
```

```
## [1] 1772
```


``` r
# Show results 1000 through 1010
print(journal_dois[1000:1010])
```

```
##  [1] "10.1186/1471-2105-16-s1-s7"  "10.1186/s12859-015-0593-1"  
##  [3] "10.1186/s12859-016-1133-3"   "10.1186/1471-2105-15-46"    
##  [5] "10.1186/1471-2105-15-s6-s6"  "10.1186/1471-2105-16-s1-s7" 
##  [7] "10.1186/s12859-015-0622-0"   "10.1186/s12859-016-1193-4"  
##  [9] "10.1186/1471-2105-15-116"    "10.1186/s12859-016-1178-3"  
## [11] "10.1186/1471-2105-15-s12-s9"
```
