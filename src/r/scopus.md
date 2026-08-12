---

editor_options: 
  markdown: 
    wrap: 72
---

# Scopus API in Python

By Nick Peinitz, Vincent F. Scalfani, and Avery Fernandez

The Scopus API, provided by Elsevier, offers programmatic access to a comprehensive database of abstracts and citations from peer-reviewed literature. It supports advanced search capabilities, author and affiliation retrieval, and citation analysis, facilitating a wide range of academic and research applications.

*This tutorial content is intended to help facilitate academic research.*

Please see the following resources for more information on API usage: - Documentation - <a href="http://www.scopus.com" target="_blank">Scopus</a> - <a href="https://dev.elsevier.com/scopus.html" target="_blank">Scopus API</a> - <a href="https://dev.elsevier.com/api_docs.html" target="_blank">Elsevier API Documentation</a> - <a href="https://dev.elsevier.com/use_cases.html" target="_blank">Elsevier API Use Cases</a> - Terms - <a href="https://dev.elsevier.com/api_service_agreement.html" target="_blank">Elsevier API Service Agreement</a> - <a href="https://dev.elsevier.com/policy.html" target="_blank">Elsevier API Policy</a> - Data Reuse - <a href="https://www.elsevier.com/about/policies-and-standards/research-data" target="_blank">Elsevier Research Data Policy</a> - Scopus Platform

***NOTE:*** Please see access details and rate limit requests for this API in the official documentation.

*These recipe examples were tested on August 4, 2026.*

## Setup

### Import Libraries

The following external libraries need to be installed into your environment to run the code examples in this tutorial:

- <a href="https://github.com/r-lib/httr2" target="_blank">httr2</a>
- <a href="https://github.com/gaborcsardi/dotenv" target="_blank">dotenv</a>
- <a href="https://github.com/r-lib/xml2" target="_blank">xml2</a>
- <a href="https://github.com/tidyverse/dplyr" target="_blank">dplyr</a>
- <a href="https://github.com/tidyverse/purrr" target="_blank">purrr</a>
- <a href="https://github.com/tidyverse/tibble" target="_blank">tibble</a>

We import the libraries used in this tutorial below:

```r
library(httr2)
library(dotenv)
library(xml2)
library(dplyr)
library(purrr)
library(tibble)
```

### Import API Key

An API key is required to access the Scopus API. You can sign up for one at the <a href="https://dev.elsevier.com/apikey/manage" target="_blank">Scopus Developer Portal</a>.

We keep our API key in a separate file, a `.env` file, and use the `dotenv` library to access it. If you use this method, create a file named `.env` in the same directory as this notebook and add the following line to it:

``` text
SCOPUS_API_KEY=PUT_YOUR_API_KEY_HERE
```

```r
load_dot_env()
API_KEY <- Sys.getenv("SCOPUS_API_KEY")
if (API_KEY == "") {
    message("API key not found. Please set 'SCOPUS_API_KEY' in your .env file.")
} else {
  print("Environment and API key successfully loaded.")
}
```

``` text
[1] "Environment and API key successfully loaded."
```

## 1. Get Author Data

### Number of Records for Author

```r
BASE_URL <- "https://api.elsevier.com/content/search/scopus"
query <- "AU-ID(55764087400)"
httpAccept <- "application/json"
data <- tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()
  

          resp_body_json(response)

}, error = function(e) {
  message("An error ocurred: ", e$message)
  NULL
})
```

We can take a closer look at the data given back:

```r
str(data, max.level = 1)
```

``` text
List of 1
 $ search-results:List of 6
```

The JSON response is parsed into an R list. The top-level list contains an element named `search-results`.

```r
str(data[["search-results"]], max.level = 1)
```

``` text
List of 6
 $ opensearch:totalResults: chr "31"
 $ opensearch:startIndex  : chr "0"
 $ opensearch:itemsPerPage: chr "25"
 $ opensearch:Query       :List of 3
 $ link                   :List of 4
 $ entry                  :List of 25
```

Inside the `search-results` list, there are six keys: \* `entry` - the actual data we want \* `link` - a link to the API endpoint \* `opensearch:Query` - the query we used to get the data \* `opensearch:itemsPerPage` - the number of items per page \* `opensearch:startIndex` - the starting index of the items \* `opensearch:totalResults` - the total number of results

```r
str(data[["search-results"]]$entry, max.level = 1)
```

``` text
List of 25
 $ :List of 24
 $ :List of 22
 $ :List of 25
 $ :List of 24
 $ :List of 25
 $ :List of 26
 $ :List of 26
 $ :List of 24
 $ :List of 26
 $ :List of 24
 $ :List of 25
 $ :List of 26
 $ :List of 26
 $ :List of 24
 $ :List of 23
 $ :List of 24
 $ :List of 25
 $ :List of 24
 $ :List of 24
 $ :List of 26
 $ :List of 26
 $ :List of 26
 $ :List of 25
 $ :List of 26
 $ :List of 24
```

The `entry` key contains a list of publication records, we can see that the first publication record in the list like so:

```r
str(data[["search-results"]]$entry[[1]], max.level = 1)
```

``` text
List of 24
 $ @_fa                  : chr "true"
 $ link                  :List of 4
 $ prism:url             : chr "https://api.elsevier.com/content/abstract/scopus_id/105027280060"
 $ dc:identifier         : chr "SCOPUS_ID:105027280060"
 $ eid                   : chr "2-s2.0-105027280060"
 $ dc:title              : chr "Teaching Computer-Assisted Retrosynthesis Reaction Prediction with Open-Source Software"
 $ dc:creator            : chr "Scalfani V.F."
 $ prism:publicationName : chr "Journal of Chemical Education"
 $ prism:issn            : chr "00219584"
 $ prism:eIssn           : chr "19381328"
 $ prism:volume          : chr "103"
 $ prism:issueIdentifier : chr "1"
 $ prism:pageRange       : chr "358-369"
 $ prism:coverDate       : chr "2026-01-13"
 $ prism:coverDisplayDate: chr "13 January 2026"
 $ prism:doi             : chr "10.1021/acs.jchemed.5c00959"
 $ citedby-count         : chr "0"
 $ affiliation           :List of 1
 $ prism:aggregationType : chr "Journal"
 $ subtype               : chr "ar"
 $ subtypeDescription    : chr "Article"
 $ source-id             : chr "24169"
 $ openaccess            : chr "0"
 $ openaccessFlag        : logi FALSE
```

Each element of the `entry` list is itself a named list that contains metadata for a single Scopus record.

```r
entries <- data[["search-results"]][["entry"]]
df <- purrr::map_dfr(entries, function(x) {
  
  x[sapply(x, is.null)] <- NA
  
  x[sapply(x, is.list)] <- lapply(
    x[sapply(x, is.list)],
    function(z) as.character(jsonlite::toJSON(z, auto_unbox = TRUE))
  )
  
  as_tibble_row(x)
})

head(df)
```

```r
# See the columns of the data frame
colnames(df)
```

``` text
 [1] "@_fa"                   "link"                  
 [3] "prism:url"              "dc:identifier"         
 [5] "eid"                    "dc:title"              
 [7] "dc:creator"             "prism:publicationName" 
 [9] "prism:issn"             "prism:eIssn"           
[11] "prism:volume"           "prism:issueIdentifier" 
[13] "prism:pageRange"        "prism:coverDate"       
[15] "prism:coverDisplayDate" "prism:doi"             
[17] "citedby-count"          "affiliation"           
[19] "prism:aggregationType"  "subtype"               
[21] "subtypeDescription"     "source-id"             
[23] "openaccess"             "openaccessFlag"        
[25] "pii"                    "article-number"        
[27] "freetoread"             "freetoreadLabel"       
[29] "pubmed-id"
```

```r
# Number of rows
nrow(df)
```

``` text
[1] 25
```

```r
# We can index data from our new data frame, df.
# For example, create a vector of just the DOIs
dois <- df[["prism:doi"]]
dois
```

``` text
 [1] "10.1021/acs.jchemed.5c00959"   "10.1080/0194262X.2026.2655257"
 [3] "10.1016/j.acalib.2024.102984"  "10.1007/s10755-023-09648-7"   
 [5] "10.29173/istl2766"             "10.1515/pac-2022-1001"        
 [7] "10.1186/s13321-022-00664-x"    "10.1007/s10755-022-09636-3"   
 [9] "10.1515/pac-2022-2019"         "10.1021/acs.jchemed.1c00904"  
[11] "10.29173/istl2566"             "10.5860/crln.82.9.428"        
[13] "10.1021/acs.iecr.8b02573"      "10.1021/acs.jchemed.6b00602"  
[15] "10.5062/F4TD9VBX"              "10.1021/acs.macromol.6b02005" 
[17] "10.1186/s13321-016-0181-z"     "10.1021/acs.chemmater.5b04431"
[19] "10.1021/acs.jchemed.5b00512"   "10.1021/acs.jchemed.5b00375"  
[21] "10.5860/crln.76.9.9384"        "10.5860/crln.76.2.9259"       
[23] "10.1126/science.346.6214.1258" "10.1021/ed400887t"            
[25] "10.1016/j.acalib.2014.03.015"
```

```r
# Get a vector of article titles
titles = df[["dc:title"]]
titles
```

``` text
 [1] "Teaching Computer-Assisted Retrosynthesis Reaction Prediction with Open-Source Software"                                                                    
 [2] "Engineering Outreach Through Open Data: A Library-Led Model for Research Visibility and Engagement"                                                         
 [3] "Comparing impact of green open access and toll-access publication in the chemical sciences"                                                                 
 [4] "Citation Metrics and Boyer’s Model of Scholarship: How Do Bibliometrics and Altmetrics Respond to Research Impact?"                                         
 [5] "Creating a Scholarly API Cookbook: Supporting Library Users with Programmatic Access to Information"                                                        
 [6] "The current landscape of author guidelines in chemistry through the lens of research data sharing"                                                          
 [7] "Visualizing chemical space networks with RDKit and NetworkX"                                                                                                
 [8] "The Power Law and Emerging and Senior Scholar Publication Patterns"                                                                                         
 [9] "Cheminformatics: data and standards a Pure and Applied Chemistry special issue"                                                                             
[10] "Using NCBI Entrez Direct (EDirect) for Small Molecule Chemical Information Searching in a Unix Terminal"                                                    
[11] "Enhancing the Discovery of Chemistry Theses by Registering Substances and Depositing in PubChem"                                                            
[12] "Using the linux operating system full-time tips and experiences from a subject liaison librarian"                                                           
[13] "Analysis of the Frequency and Diversity of 1,3-Dialkylimidazolium Ionic Liquids Appearing in the Literature"                                                
[14] "Rapid Access to Multicolor Three-Dimensional Printed Chemistry and Biochemistry Models Using Visualization and Three-Dimensional Printing Software Programs"
[15] "Text analysis of chemistry thesis and dissertation titles"                                                                                                  
[16] "Phototunable Thermoplastic Elastomer Hydrogel Networks"                                                                                                     
[17] "Programmatic conversion of crystal structures into 3D printable files using Jmol"                                                                           
[18] "Dangling-End Double Networks: Tapping Hidden Toughness in Highly Swollen Thermoplastic Elastomer Hydrogels"                                                 
[19] "Replacing the Traditional Graduate Chemistry Literature Seminar with a Chemical Research Literacy Course"                                                   
[20] "3D Printed Block Copolymer Nanostructures"                                                                                                                  
[21] "Hypotheses in librarianship: Applying the scientific method"                                                                                                
[22] "Recruiting students to campus: Creating tangible and digital products in the academic library"                                                              
[23] "Finally free"                                                                                                                                               
[24] "3D printed molecules and extended solid models for teaching symmetry and point groups"                                                                      
[25] "Repurposing Space in a Science and Engineering Library: Considerations for a Successful Outcome"
```

```r
 # Now a vector of the cited by count
cited_by = df[["citedby-count"]]
cited_by
```

``` text
 [1] "0"   "0"   "5"   "8"   "3"   "4"   "89"  "6"   "0"   "3"   "0"  
[12] "0"   "26"  "30"  "9"   "14"  "29"  "8"   "15"  "28"  "1"   "1"  
[23] "0"   "123" "7" 
```

```r
# Get sum of cited_by
sum(as.numeric(cited_by))
```

``` text
[1] 409
```

## 2. Get Author Data in a Loop

### Number of Records for Author

```r
# Create text file to store author names and their Scopus AUIDs
writeLines(
  c(
    "Emy Decker\t36660678600",
    "Lindsey Lowry\t57210944451",
    "Karen Chapman\t35783926100",
    "Kevin Walker\t56133961300",
    "Sara Whitver\t57194760730"
  ),
"author.txt"
)
```

```r
# Load a list of author names and Scopus AUIDs
author_df <- read.delim(
  "author.txt",
  header = FALSE,
  sep = "\t"
)

author_list <- split(author_df, seq_len(nrow(author_df)))
str(author_list)
```

``` text
List of 5
 $ 1:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Emy Decker"
  ..$ V2: num 3.67e+10
 $ 2:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Lindsey Lowry"
  ..$ V2: num 5.72e+10
 $ 3:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Karen Chapman"
  ..$ V2: num 3.58e+10
 $ 4:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Kevin Walker"
  ..$ V2: num 5.61e+10
 $ 5:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Sara Whitver"
  ..$ V2: num 5.72e+10
```

```r
# Get number of Scopus records for each author
num_records <- list()

for (i in seq_along(author_list))
{
  author <- author_list[[i]][1, 1]
  authorID <- author_list[[i]][1, 2]

  query <- paste0("AU-ID(", authorID, ")")
  httpAccept <- "application/json"
  tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()
          Sys.sleep(1)
          
          data <- resp_body_json(response)

          number_of_records <- data[["search-results"]][["opensearch:totalResults"]]
          num_records[[length(num_records) + 1]] <- list(
              author = author,
              authorID = authorID,
              number_of_records = number_of_records)
  }, error = function(e) {
  message("An error ocurred: ", e$message)
  num_records[[length(num_records) + 1]] <<- list(
      author = author,
      authorID = authorID,
      number_of_records = NA)
})
}
```

```r
num_records
```

``` text
[[1]]
[[1]]$author
[1] "Emy Decker"

[[1]]$authorID
[1] 36660678600

[[1]]$number_of_records
[1] "24"


[[2]]
[[2]]$author
[1] "Lindsey Lowry"

[[2]]$authorID
[1] 57210944451

[[2]]$number_of_records
[1] "10"


[[3]]
[[3]]$author
[1] "Karen Chapman"

[[3]]$authorID
[1] 35783926100

[[3]]$number_of_records
[1] "23"


[[4]]
[[4]]$author
[1] "Kevin Walker"

[[4]]$authorID
[1] 56133961300

[[4]]$number_of_records
[1] "13"


[[5]]
[[5]]$author
[1] "Sara Whitver"

[[5]]$authorID
[1] 57194760730

[[5]]$number_of_records
[1] "8"
```

### Download Record Data

```r
# Let's say we want the DOIs and cited by counts in a list
cites <- list()

for (i in seq_along(author_list))
{
  author <- author_list[[i]][1, 1]
  authorID <- author_list[[i]][1, 2]
  
  query <- paste0("AU-ID(", authorID, ")")
  httpAccept <- "application/json"

  tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()
          Sys.sleep(1)

          data <- resp_body_json(response)
          
          entries <- data[["search-results"]][["entry"]]
          author_df <- purrr::map_dfr(entries, function(x) {
            
            x[sapply(x, is.null)] <- NA
            
            x[sapply(x, is.list)] <- lapply(
              x[sapply(x, is.list)],
              function(z) as.character(jsonlite::toJSON(z, auto_unbox = TRUE))
            )
            
            as_tibble_row(x)
          })
          
          # Get the DOIs and cited by counts
          dois <- author_df[["prism:doi"]]
          cited_by <- author_df[["citedby-count"]]
          
          # Create a list of lists with author, authorID, DOI, and cited by count
          for (i in seq_along(dois))
          {
            doi <- dois[i]
            cited <- cited_by[i]

            cites[[length(cites) + 1]] <- list(
              author = author,
              authorID = authorID,
              doi = doi,
              cited = cited
            )
          }
  }, error = function(e) {
  message("An error ocurred: ", e$message)
  num_records[[length(num_records) + 1]] <<- list(
      author = author,
      authorID = authorID,
      number_of_records = NA)
  })
}
```

```r
# The cites variable is a list of list with the data
# View data for first 5 records
cites[1:5]
```

``` text
[[1]]
[[1]]$author
[1] "Emy Decker"

[[1]]$authorID
[1] 36660678600

[[1]]$doi
[1] "10.1007/s10755-024-09698-5"

[[1]]$cited
[1] "1"


[[2]]
[[2]]$author
[1] "Emy Decker"

[[2]]$authorID
[1] 36660678600

[[2]]$doi
[1] "10.1016/j.acalib.2024.102858"

[[2]]$cited
[1] "3"


[[3]]
[[3]]$author
[1] "Emy Decker"

[[3]]$authorID
[1] 36660678600

[[3]]$doi
[1] NA

[[3]]$cited
[1] "0"


[[4]]
[[4]]$author
[1] "Emy Decker"

[[4]]$authorID
[1] 36660678600

[[4]]$doi
[1] "10.1016/j.acalib.2022.102648"

[[4]]$cited
[1] "0"


[[5]]
[[5]]$author
[1] "Emy Decker"

[[5]]$authorID
[1] 36660678600

[[5]]$doi
[1] "10.1016/j.acalib.2022.102634"

[[5]]$cited
[1] "4"
```

```r
# Add to data frame
cites_df <- bind_rows(cites)
cites_df
```


### Save Record Data to a File

Here is one method if you want to loop over author queries and save all Scopus document data to a file:

```r
# Load a list of author names and Scopus AUIDs
author_df <- read.delim(
  "author.txt",
  colClasses = "character",
  header = FALSE,
  sep = "\t"
)

author_list <- split(author_df, seq_len(nrow(author_df)))
str(author_list)
```

``` text
List of 5
 $ 1:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Emy Decker"
  ..$ V2: num 3.67e+10
 $ 2:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Lindsey Lowry"
  ..$ V2: num 5.72e+10
 $ 3:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Karen Chapman"
  ..$ V2: num 3.58e+10
 $ 4:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Kevin Walker"
  ..$ V2: num 5.61e+10
 $ 5:'data.frame':	1 obs. of  2 variables:
  ..$ V1: chr "Sara Whitver"
  ..$ V2: num 5.72e+10
```

```r
for (i in seq_along(author_list))
{
  authorName <- author_list[[i]][1, 1]
  authorID <- author_list[[i]][1, 2]

  query <- paste0("AU-ID(", authorID, ")")
  httpAccept <- "application/json"

  tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()
          Sys.sleep(2)
      
          data <- resp_body_json(response)
      
          # Extract the 'entry' data and convert it to a data frame
          if ("entry" %in% names(data[["search-results"]]))
            df <- bind_rows(data[["search-results"]][["entry"]])
              df[] <- lapply(df, function(col) {
                if (is.list(col)) {
                  sapply(col, jsonlite::toJSON, auto_unbox = TRUE)
                } else {
                    col
                }
              })
      
          # Save to file
          filename <- paste0(
            gsub(" ", "_", authorName),
            "_",
            authorID,
            "_ScopusData.tsv"
          )
          
          write.table(
            df,
            file = filename,
            sep = "\t",
            row.names = FALSE,
            quote = FALSE
          )
          print(paste0("Data for ", authorName, " saved to ", filename))
  }, error = function(e) {
      message("An error ocurred for ", authorName, ": ", e$message)
  })
}
```

``` text
[1] "Data for Emy Decker saved to Emy_Decker_36660678600_ScopusData.tsv"
[1] "Data for Lindsey Lowry saved to Lindsey_Lowry_57210944451_ScopusData.tsv"
[1] "Data for Karen Chapman saved to Karen_Chapman_35783926100_ScopusData.tsv"
[1] "Data for Kevin Walker saved to Kevin_Walker_56133961300_ScopusData.tsv"
[1] "Data for Sara Whitver saved to Sara_Whitver_57194760730_ScopusData.tsv"
```

```r
df_author <- read.delim(
  "Karen_Chapman_35783926100_ScopusData.tsv",
  sep = "\t"
)

head(df_author)
```

```r
# Get info about citedby_count
summary(as.numeric(df_author[["citedby.count"]]))
```

``` text
   Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
   0.00    4.00    6.00   13.88   12.00   71.00 
```

```r
# Get info about publication titles
x <- df_author[["prism.publicationName"]]

c(
  count <- sum(!is.na(x)),
  unique_vals <- length(unique(x[!is.na(x)])),
  top = names(sort(table(x), decreasing = TRUE))[1],
  freq = max(table(x))
)
```

``` text
                                      "97" 
                                           
                                      "11" 
                                       top 
"Behavioral and Social Sciences Librarian" 
                                      freq 
                                      "20" 
```


## 3. Get References via a Title Search

### Number of Title Match Records

```r
query <- "TITLE(ChemSpider)"
httpAccept <- "application/json"
tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()
        
          data <- resp_body_json(response)
          print(data[["search-results"]][["opensearch:totalResults"]])

}, error = function(e) {
  message("An error ocurred: ", e$message)
  data <- NULL
})
```

``` text
[1] "8"
```

```r
titleWord_list <- list(
  "ChemSpider", "PubChem", "ChEMBL", "Reaxys", "SciFinder"
)

# Get number of Scopus records for each title search
num_records_title <- list()

for (titleWord in titleWord_list)
{
  query <- paste0("TITLE(", titleWord, ")")
  httpAccept <- "application/json"
  tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()
      
          data <- resp_body_json(response)
          # Extract the total number of results
          numt = data[["search-results"]][["opensearch:totalResults"]]
      
          # Compile saved Scopus data into a list of lists
          num_records_title[[length(num_records_title) + 1]] <- list(
            titleWord,
            numt
          )
      
          # Delay 1 second between API calls to be nice to Elsevier servers
          Sys.sleep(1)
  }, error = function(e) {
    message("An error ocurred: ", e$message)
    num_records_title[[length(num_records_title) + 1]] <- list(
      titleWord,
      numt = NA
    )
  })
}
```

```r
num_records_title
```

``` text
[[1]]
[[1]][[1]]
[1] "ChemSpider"

[[1]][[2]]
[1] "8"


[[2]]
[[2]][[1]]
[1] "PubChem"

[[2]][[2]]
[1] "115"


[[3]]
[[3]][[1]]
[1] "ChEMBL"

[[3]][[2]]
[1] "72"


[[4]]
[[4]][[1]]
[1] "Reaxys"

[[4]][[2]]
[1] "9"


[[5]]
[[5]][[1]]
[1] "SciFinder"

[[5]][[2]]
[1] "35"
```


### Download Title Match Record Data

```r
titleWord_list <- list(
  "ChemSpider", "PubChem", "ChEMBL", "Reaxys", "SciFinder"
)

# Get number of Scopus records for each title search
scopus_title_data <- list()

for (titleWord in titleWord_list)
{
  query <- paste0("TITLE(", titleWord, ")")
  httpAccept <- "application/json"
  tryCatch({
          response <- request(BASE_URL) |>
                            req_headers(
                              "X-ELS-APIKey" = API_KEY,
                              "Accept" = httpAccept
                              ) |>
                            req_url_query(
                                query = query
                            ) |>
                            req_perform()

          # Delay 1 second between API calls to be nice to Elsevier servers
          Sys.sleep(1)
      
          data <- resp_body_json(response)
      
          # Extract the 'entry' data and convert it to a DataFrame
          entries <- data[["search-results"]][["entry"]]
          if (!is.null(entries))
          {
            for (entry in entries)
            {
              doi <- if ("prism:doi" %in% names(entry)) entry[["prism:doi"]] else NA
              title <- if ("dc:title" %in% names(entry)) entry[["dc:title"]] else NA
              coverDate <- if ("prism:coverDate" %in% names(entry)) entry[["prism:coverDate"]] else NA
      
          # Append to the list
              scopus_title_data[[length(scopus_title_data) + 1]] <- list(
                titleWord,
                doi,
                title,
                coverDate
              )
            }
          }

  }, error = function(e) {
    message("An error ocurred for ", titleWord, ": ", e$message)
    num_records_title[[length(num_records_title) + 1]] <- list(
      titleWord,
      doi = NA,
      title = NA,
      coverDate = NA
    )
  })
}
```

```r
scopus_title_data_df <- bind_rows(
  lapply(scopus_title_data, function(x) {
    tibble(
      titleWord = x[[1]],
      doi = x[[2]],
      title = x[[3]],
      coverDate = x[[4]],
    )
  })
)

scopus_title_data_df
```