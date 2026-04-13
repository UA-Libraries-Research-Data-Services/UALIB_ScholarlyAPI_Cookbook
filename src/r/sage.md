---
title: "Sage Journals Text and Data Mining in R"
output: 
  html_document:
    keep_md: true
---

# Sage Journals Text and Data Mining in R

by Michael T. Moen

<div class="rmd-btn-wrapper">
  <a class="rmd-btn"
     href="https://github.com/UA-Libraries-Research-Data-Services/UALIB_ScholarlyAPI_Cookbook/blob/main/rmarkdown/sage.Rmd"
     target="_blank"
     rel="noreferrer">
    View RMarkdown File
  </a>
</div>

Sage Journals allow downloading of articles for which you have legitimate access (e.g. open access articles and those included in your institution's subscription) for non-commercial text and data mining (see restrictions in terms below). Access to text and data mining with Sage resources requires prior approval. Contact UA Libraries or your institution to check their agreement and enable access. Please see the following resources below for more information on Sage text and data mining, API usage, and policies/terms:

*This tutorial content is intended to help facilitate academic research. Please check your institution for their Text and Data Mining or related License Agreement with Sage Journals.*

- Documentation
    - <a href="https://journals.sagepub.com/" target="_blank">Sage Journals</a>
- Terms
    - <a href="https://journals.sagepub.com/page/policies/text-and-data-mining" target="_blank">Text and Data Mining on Sage Journals</a>
    - <a href="https://journals.sagepub.com/page/policies/text-and-data-mining-license" target="_blank">Sage Journals Text and Data Mining License</a>
    - <a href="https://journals.sagepub.com/page/policies/terms-of-use" target="_blank">Sage Journals Terms and Conditions of Use</a>
- Data Reuse
    - <a href="https://www.sagepub.com/tdm-ai-policy" target="_blank">Sage Policy on Text and Data Mining (TDM) and Artificial Intelligence (AI)</a>

**_NOTE:_** Please see access details and rate limit requests for this API in the official documentation.

*These recipe examples were tested on February 10, 2026.*

*This recipe uses the CrossRef API to obtain the full-text URLs of the articles, as recommended in <a href="https://journals.sagepub.com/page/policies/text-and-data-mining" target="_blank">Sage's Text and Data Mining overview</a>. For more information on usage for this API, please see our <a href="https://ua-libraries-research-data-services.github.io/UALIB_ScholarlyAPI_Cookbook/overview/crossref.html" target="_blank">CrossRef cookbook tutorials</a> and the <a href="https://www.crossref.org/documentation/retrieve-metadata/rest-api/text-and-data-mining-for-researchers/" target="_blank">text and data mining for researchers page of CrossRef's API documentation</a>.*

## Setup

### Load Libraries

The following external libraries need to be installed into your environment to run the code examples in this tutorial:

- <a href="https://cran.r-project.org/web/packages/httr/index.html" target="_blank">httr: Tools for Working with URLs and HTTP</a>
- <a href="https://cran.r-project.org/web/packages/jsonlite/index.html" target="_blank">jsonlite: A Simple and Robust JSON Parser and Generator for R</a>

We load the libraries used in this tutorial below:


``` r
library(httr)
library(jsonlite)
```

### Import Email

The CrossRef API requires users to provide an email address in API requests. 

We keep our email address in a `.Renviron` file and use the `Sys.getenv()` library to access it. If you would like to use this method, create a `.Renviron` file and add the following line to it:

```text
EMAIL="PUT_YOUR_EMAIL_HERE"
```


``` r
if (nzchar(Sys.getenv("EMAIL"))) {
  print("Email successfully loaded.")
} else {
  warning("Email not found or is empty.")
}
```

```
## [1] "Email successfully loaded."
```

### Enable Text and Data Mining with Sage

Access to text and data mining on Sage requires approval. Contact UA Libraries or your institution to check their agreement and enable access.

## 1. Retrieve a Full-Text Article as a PDF

To begin, let's consider a simple example where we retrieve the full text of an article.

For this example, we look at the following article licensed under <a href="https://creativecommons.org/licenses/by/4.0/" target="_blank">CC BY 4.0</a>:

<a href="https://doi.org/10.1177/14759217221075241" target="_blank">https://doi.org/10.1177/14759217221075241</a>

Sage permits non-commercial TDM for articles to those you have legitimate access to. If you can view the full text for the article of the DOI above in your browser, you should be able to access it programmatically below once you receive approval by Sage.


``` r
CROSSREF_URL <- "https://api.crossref.org/works/"

get_pdf_url <- function(doi) {
  # Query the Crossref API with a given DOI
  response <- GET(paste0(CROSSREF_URL, doi),
                  query = list(mailto = Sys.getenv("EMAIL")))
  
  # Extract links from the response
  data <- fromJSON(rawToChar(response$content))
  links_df <- data$message$link
  
  # Find row with the correct format and permissions
  row <- which(links_df$`content-type` == "application/pdf" &
               links_df$`intended-application` == "text-mining")
  return(links_df$URL[row])
}

doi <- "https://doi.org/10.1177/14759217221075241"
full_text_url <- get_pdf_url(doi)
full_text_url
```

```
## [1] "https://journals.sagepub.com/doi/pdf/10.1177/14759217221075241"
```

With the URL for the full-text article, we can now retrieve the data from Sage.


``` r
get_article_full_text <- function(url) {
  response <- GET(url)
  if (response$status_code == 200) {
    # Check whether the request redirected to the abstract page
    ft_url <- response$url
    if (grepl("https://journals.sagepub.com/doi/abs/", ft_url, fixed = TRUE)) {
      message("ERROR: You do not have access to this article's full text.")
      return(NULL)
    }
    return(response)
  } else if (response$status_code == 403) {
    message("ERROR: Access to TDM on Sage requires approval.")
    message("Contact UA Libraries or your institution for more guidance.")
  } else {
    message(paste("ERROR:", response$status_code))
  }
  return(NULL)
}

response <- get_article_full_text(full_text_url)
```

Since our query was successful, we download the full-text article as a PDF below:


``` r
download_full_text <- function(response, filename) {
  writeBin(content(response, "raw"), filename)
}

download_full_text(response, "article.pdf")
```

## 2. Retrieve Full-Text PDF Articles in a Loop

Using the functions defined in the previous example, we can retrieve the full text of several articles in a loop.


``` r
# Define a list of DOIs for CC BY 4.0 licensed articles
dois <- c(
  "https://doi.org/10.3233/NAI-240767",
  "https://doi.org/10.1177/20539517221145372",
  "https://doi.org/10.1177/09544062231164575",
  "https://doi.org/10.1177/2053951717743530",
  "https://doi.org/10.1177/00405175221145571"
)

# Download each article as a PDF
for (idx in seq_along(dois)) {
  doi <- dois[[idx]]
  url <- get_pdf_url(doi)
  response <- get_article_full_text(url)
  Sys.sleep(1)
  if (is.null(response)) {
    message(paste0("ERROR: Could not download ", url))
    next
  }
  filename <- paste0("article", idx, ".pdf")
  download_full_text(response, filename)
  message(paste(url, "downloaded as", filename))
}
```

```
## https://journals.sagepub.com/doi/pdf/10.3233/NAI-240767 downloaded as article1.pdf
```

```
## https://journals.sagepub.com/doi/pdf/10.1177/20539517221145372 downloaded as article2.pdf
```

```
## https://journals.sagepub.com/doi/pdf/10.1177/09544062231164575 downloaded as article3.pdf
```

```
## http://journals.sagepub.com/doi/pdf/10.1177/2053951717743530 downloaded as article4.pdf
```

```
## https://journals.sagepub.com/doi/pdf/10.1177/00405175221145571 downloaded as article5.pdf
```

## 3. Retrieve a Full-Text Article as a XML

This example uses the same article as section 1, retrieving the data as XML rather than a PDF.


``` r
get_xml_url <- function(doi) {
  # Query the Crossref API with a given DOI
  response <- GET(paste0(CROSSREF_URL, doi),
                  query = list(mailto = Sys.getenv("EMAIL")))
  
  # Extract links from the response
  data <- fromJSON(rawToChar(response$content))
  links_df <- data$message$link
  
  # Find row with the correct format and permissions
  row <- which(links_df$`content-type` == "application/xml" &
               links_df$`intended-application` == "text-mining")
  return(links_df$URL[row])
}

doi <- "https://doi.org/10.1177/14759217221075241"
full_text_url <- get_xml_url(doi)
full_text_url
```

```
## [1] "https://journals.sagepub.com/doi/full-xml/10.1177/14759217221075241"
```


``` r
response <- get_article_full_text(full_text_url)

download_full_text(response, "article.xml")
```

## 4. Retrieve Full-Text XML Articles in a Loop

This example uses the same articles from section 2, retrieving the data as XML rather than PDFs.


``` r
# Download the same list of DOIs from section 2 as XML
for (idx in seq_along(dois)) {
  doi <- dois[[idx]]
  url <- get_xml_url(doi)
  response <- get_article_full_text(url)
  Sys.sleep(1)
  if (is.null(response)) {
    message(paste0("ERROR: Could not download ", url))
    next
  }
  filename <- paste0("article", idx, ".xml")
  download_full_text(response, filename)
  message(paste(url, "downloaded as", filename))
}
```

```
## https://journals.sagepub.com/doi/full-xml/10.3233/NAI-240767 downloaded as article1.xml
```

```
## https://journals.sagepub.com/doi/full-xml/10.1177/20539517221145372 downloaded as article2.xml
```

```
## https://journals.sagepub.com/doi/full-xml/10.1177/09544062231164575 downloaded as article3.xml
```

```
## http://journals.sagepub.com/doi/full-xml/10.1177/2053951717743530 downloaded as article4.xml
```

```
## https://journals.sagepub.com/doi/full-xml/10.1177/00405175221145571 downloaded as article5.xml
```
