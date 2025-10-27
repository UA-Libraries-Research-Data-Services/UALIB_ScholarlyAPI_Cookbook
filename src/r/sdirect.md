---
title: "ScienceDirect API in R"
output: 
  html_document:
    keep_md: true
---

# ScienceDirect API in R

by Michael T. Moen

These recipe examples demonstrate how to use Elsevier’s <a href="https://dev.elsevier.com/" target="_blank">ScienceDirect API</a> to retrieve full-text articles in various formats (XML, text).

*This tutorial content is intended to help facilitate academic research. Please check your institution for their Text and Data Mining or related License Agreement with Elsevier.*

Please see the following resources for more information on API usage:

- Documentation
    - <a href="https://dev.elsevier.com/" target="_blank">ScienceDirect API</a>
    - <a href="https://dev.elsevier.com/sd_api_spec.html" target="_blank">ScienceDirect API Documentation</a>
- Terms
    - <a href="https://dev.elsevier.com/api_key_settings.html" target="_blank">ScienceDirect API Terms of Use</a>
- Data Reuse
    - <a href="https://dev.elsevier.com/tecdoc_text_mining.html" target="_blank">Elsevier Text & Data Mining</a>

_**NOTE:**_ See your institution's rate limit with <a href="https://dev.elsevier.com/api_key_settings.html" target="_blank">ScienceDirect API Terms of Use</a>.

*If you have copyright or other related text and data mining questions, please contact The University of Alabama Libraries or your respective library/institution.*

*These recipe examples were tested on October 27, 2025.*

## Setup

### Import Libraries

The following packages need to be installed into your environment to run the code examples in this tutorial. These packages can be installed with `install.packages()`.

- <a href="https://cran.r-project.org/web/packages/httr/index.html" target="_blank">httr: Tools for Working with URLs and HTTP</a>

We load the libraries used in this tutorial below:


``` r
library(httr)
```

### Import API Key

An API key is required for to access the ScienceDirect API. You can sign up for one at <a href="https://dev.elsevier.com/" target="_blank">Elsevier developer portal</a>.

We keep our token in a `.Renviron` file that is stored in the working directory and use `Sys.getenv()` to access it. The `.Renviron` should have an entry like the one below.

```text
SCIENCE_DIRECT_API_KEY="PUT_YOUR_API_KEY_HERE"
```

Below, we can test to whether the key was successfully imported.


``` r
if (nzchar(Sys.getenv("SCIENCE_DIRECT_API_KEY"))) {
  print("API key successfully loaded.")
} else {
  warning("API key not found or is empty.")
}
```

```
## [1] "API key successfully loaded."
```

### Identifier Note

We will use DOIs as the article identifiers. See our Crossref and Scopus API tutorials for workflows on how to create lists of DOIs and identfiers for specific searches and journals. The Elsevier ScienceDirect Article (Full-Text) API also accepts other identifiers like Scopus IDs and PubMed IDs (see API specification documents linked above).

## 1. Retrieve full-text XML of an article


``` r
# For XML download
elsevier_url <- "https://api.elsevier.com/content/article/doi/"
doi1 <- '10.1016/j.tetlet.2017.07.080' # Example Tetrahedron Letters article
fulltext1 <- GET(paste0(
  elsevier_url, doi1,
  "?APIKey=", Sys.getenv("SCIENCE_DIRECT_API_KEY"),
  "&httpAccept=text/xml"))

# Save to file
writeLines(content(fulltext1, "text"), "fulltext1.xml")
```

## 2. Retrieve plain text of an article


``` r
# For simplified text download
doi2 <- '10.1016/j.tetlet.2022.153680' # Example Tetrahedron Letters article
fulltext2 <- GET(paste0(
  elsevier_url, doi2,
  "?APIKey=", Sys.getenv("SCIENCE_DIRECT_API_KEY"),
  "&httpAccept=text/plain"))

# Save to file
writeLines(content(fulltext2, "text"), "fulltext2.txt")
```

## 3. Retrieve full-text in a loop


``` r
# Make a list of 5 DOIs for testing
dois <- c('10.1016/j.tetlet.2018.10.031',
          '10.1016/j.tetlet.2018.10.033',
          '10.1016/j.tetlet.2018.10.034',
          '10.1016/j.tetlet.2018.10.038',
          '10.1016/j.tetlet.2018.10.041')

for (doi in dois) {
  article <- GET(paste0(
    elsevier_url, doi,
    "?APIKey=", Sys.getenv("SCIENCE_DIRECT_API_KEY"),
    "&httpAccept=text/plain"))
  doi_name <- gsub("/", "_", doi)
  writeLines(content(article, "text"), paste0(doi_name, "_plain_text.txt"))
  Sys.sleep(1)
}
```
