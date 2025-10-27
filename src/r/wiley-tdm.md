---
title: "Wiley Text and Data Mining (TDM) in R"
output: 
  html_document:
    keep_md: true
---

# Wiley Text and Data Mining (TDM) in R

by Michael T. Moen

The Wiley Text and Data Mining (TDM) API allows users to retrieve the full-text articles of subscribed Wiley content in PDF form. TDM use is for non-commercial scholarly research, see terms and restrictions in below links.

*This tutorial content is intended to help facilitate academic research. Please check your institution for their Text and Data Mining or related License Agreement with Wiley.*

Please see the following resources for more information on API usage:

- Documentation
    - <a href="https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining" target="_blank">Wiley Text and Data Mining</a>
- Terms
    - <a href="https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining#accordionHeader-3" target="_blank">Wiley Text and Data Mining Agreement</a>
- Data Reuse
    - <a href="https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining#accordionHeader-3" target="_blank">Wiley TDM Data Reuse</a> (see sections 4 and 5 of Text and Data Mining Agreement)

*These recipe examples were tested on October 27, 2025.*

**_NOTE:_** The Wiley TDM API limits requests to a maximum of 3 requests per second.

## Setup

### Import Libraries

The following packages need to be installed into your environment to run the code examples in this tutorial. These packages can be installed with `install.packages()`.

- <a href="https://cran.r-project.org/web/packages/httr/index.html" target="_blank">httr: Tools for Working with URLs and HTTP</a>

We load the libraries used in this tutorial below:


``` r
library(httr)
```

### Text and Data Mining Token

A token is required for text and data mining with Wiley. You can sign up for one on the <a href="https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining#accordionHeader-2" target="_blank">Wiley Text and Data Mining page</a>.

We keep our token in a `.Renviron` file that is stored in the working directory and use `Sys.getenv()` to access it. The `.Renviron` should have an entry like the one below.

```text
WILEY_TDM_TOKEN="PUT_YOUR_TOKEN_HERE"
```

Below, we can test to whether the key was successfully imported.


``` r
if (nzchar(Sys.getenv("WILEY_TDM_TOKEN"))) {
  print("API key successfully loaded.")
} else {
  warning("API key not found or is empty.")
}
```

```
## [1] "API key successfully loaded."
```

## 1. Retrieve full-text of an article

The Wiley TDM API returns the full-text of an article as a PDF when given the article's DOI.

In the first example, we download the full-text of the article with the DOI "10.1002/net.22207". This article was found on the Wiley Online Library.


``` r
# DOI to download
doi <- "10.1002/net.22207"
url <- paste0("https://api.wiley.com/onlinelibrary/tdm/v1/articles/", doi)

response <- GET(url, add_headers("Wiley-TDM-Client-Token" = Sys.getenv("WILEY_TDM_TOKEN")))

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
  response <- GET(url, add_headers("Wiley-TDM-Client-Token" = Sys.getenv("WILEY_TDM_TOKEN")))
  
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
