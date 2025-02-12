---
title: "wiley-tdm"
output: 
  html_document:
    keep_md: true
---

# Wiley Text and Data Mining (TDM) in R

by Michael T. Moen

**Wiley TDM:** https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining

**Wiley TDM Terms of Use:** Please check with your institution to see their Text and Data Mining Agreement

The Wiley Text and Data Mining (TDM) API allows users to retrieve the full-text articles of Wiley content in PDF form. This tutorial content is intended to help facilitate academic research.

*These recipe examples were tested on February 12, 2025.*

**_NOTE:_** The Wiley TDM API limits requests to a maximum of 3 requests per second.

## Setup

### Import Libraries

This tutorial uses the following libraries:


``` r
library(httr)
```

### Text and Data Mining Token

A token is required to access the Wiley TDM API. Sign up can be found [here](https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining#accordionHeader-2). Import your token below:


``` r
wiley_token <- Sys.getenv("wiley_token")

# The token will be sent as a header in the API calls
headers <- add_headers("Wiley-TDM-Client-Token" = wiley_token)
```

## 1. Retrieve full-text of an article

The Wiley TDM API returns the full-text of an article as a PDF when given the article's DOI.

In the first example, we download the full-text of the article with the DOI "10.1002/net.22207". This article was found on the Wiley Online Library.


``` r
# DOI to download
doi <- "10.1002/net.22207"
url <- paste0("https://api.wiley.com/onlinelibrary/tdm/v1/articles/", doi)

response <- GET(url, headers)

if (status_code(response) == 200) {
  # Download if status code indicates success
  filename <- paste0(gsub("/", "_", doi), ".pdf")
  writeBin(content(response, "raw"), filename)
  cat(paste0(filename, " downloaded successfully\n"))
  
} else {
  # Print status code if unsuccessful
  cat(paste0("Failed to download PDF. Status code: ", status_code(response), "\n"))
}
```

```
## 10.1002_net.22207.pdf downloaded successfully
```

## 2. Retrieve full-text of multiple articles

In this example, we download 5 articles found in the Wiley Online Library:


``` r
# DOIs of articles to download
dois <- c(
  "10.1111/j.1467-8624.2010.01564.x",
  "10.1111/1467-8624.00164",
  "10.1111/cdev.12864",
  "10.1111/j.1467-8624.2007.00995.x",
  "10.1111/j.1467-8624.2010.01499.x",
  "10.1111/j.1467-8624.2010.0149.x"  # Invalid DOI, will throw error
)

# Loop through DOIs and download each article
for (doi in dois) {
  url <- paste0("https://api.wiley.com/onlinelibrary/tdm/v1/articles/", doi)
  response <- GET(url, headers)
  
  if (status_code(response) == 200) {
    # Download if status code indicates success
    filename <- paste0(gsub("/", "_", doi), ".pdf")
    writeBin(content(response, "raw"), filename)
    cat(paste0(filename, " downloaded successfully\n"))
    
  } else {
    # Print status code if unsuccessful
    cat(paste0("Failed to download PDF. Status code: ", status_code(response), "\n"))
  }
  
  # Wait 1 second to be nice to Wiley's servers
  Sys.sleep(1)
}
```

```
## 10.1111_j.1467-8624.2010.01564.x.pdf downloaded successfully
## 10.1111_1467-8624.00164.pdf downloaded successfully
## 10.1111_cdev.12864.pdf downloaded successfully
## 10.1111_j.1467-8624.2007.00995.x.pdf downloaded successfully
## 10.1111_j.1467-8624.2010.01499.x.pdf downloaded successfully
## Failed to download PDF. Status code: 404
```

## R Session Info


``` r
sessionInfo()
```

```
## R version 4.4.2 (2024-10-31 ucrt)
## Platform: x86_64-w64-mingw32/x64
## Running under: Windows 11 x64 (build 22631)
## 
## Matrix products: default
## 
## 
## locale:
## [1] LC_COLLATE=English_United States.utf8 
## [2] LC_CTYPE=English_United States.utf8   
## [3] LC_MONETARY=English_United States.utf8
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.utf8    
## 
## time zone: America/Chicago
## tzcode source: internal
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] httr_1.4.7
## 
## loaded via a namespace (and not attached):
##  [1] digest_0.6.37     R6_2.5.1          fastmap_1.2.0     xfun_0.50        
##  [5] cachem_1.1.0      knitr_1.49        htmltools_0.5.8.1 rmarkdown_2.29   
##  [9] lifecycle_1.0.4   cli_3.6.3         sass_0.4.9        jquerylib_0.1.4  
## [13] compiler_4.4.2    tools_4.4.2       curl_6.1.0        evaluate_1.0.3   
## [17] bslib_0.8.0       yaml_2.3.10       rlang_1.1.4       jsonlite_1.8.9
```
