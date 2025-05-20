---
title: "wiley-tdm"
output: 
  html_document:
    keep_md: true
---

# Wiley Text and Data Mining (TDM) in R

by Michael T. Moen

This tutorial is designed to support academic research. Please consult your institution’s library or legal office regarding its Text and Data Mining license agreement with Wiley.

Documentation

Wiley Text and Data Mining Overview: https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining

Terms of Use

Wiley Text and Data Mining Agreement: https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining#accordionHeader-3

Data Reuse

[Service Name] Data Reuse: Link not provided — please update with correct URL.


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
