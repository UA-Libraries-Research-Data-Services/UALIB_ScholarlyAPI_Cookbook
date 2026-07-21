---
title: "Open Science Framework (OSF) API in R"
output: 
  html_document:
    keep_md: true
---

# Open Science Framework API in R

by Nick Peinitz, Avery Fernandez and Michael T. Moen

<div class="rmd-btn-wrapper"> <a class="rmd-btn" href="https://github.com/UA-Libraries-Research-Data-Services/UALIB_ScholarlyAPI_Cookbook/blob/main/rmarkdown/osf.Rmd" target="\_blank" rel="noreferrer"> View RMarkdown File </a> </div>

The OSF API allows users to fetch metadata and files from the OSF platform. This cookbook will guide you through the setup and usage of the API, including fetching metadata for preprints and downloading PDFs.

Please see the following resources for more information on API usage:

- Documentation

  - <a href="https://developer.osf.io/" target="\_blank">OSF API Documentation</a>

- Terms of Use

  - <a href="https://github.com/CenterForOpenScience/cos.io/blob/master/TERMS_OF_USE.md" target="\_blank">OSF API Terms of Use</a>

***NOTE:*** Please see access details and rate limit requests for this API in the official documentation.

*These recipe examples were tested on July 17, 2026.*

## Setup

### Import Libraries

The following external libraries need to be installed into your environment to run the code examples in this tutorial:

- <a href="https://github.com/r-lib/httr" target="\_blank">httr</a>

- <a href="https://github.com/gaborcsardi/dotenv" target="\_blank">dotenv</a>

- <a href="https://github.com/jeroen/jsonlite" target="\_blank">jsonlite</a>

We import the libraries used in this tutorial below:

``` r
library(httr)
library(dotenv)
library(jsonlite)
```

### Import Access Token

Authentication is not required to access the OSF API, but will increase your rate limit. You can sign up for one <a href="https://osf.io/settings/tokens/" target="\_blank">here</a>.

We keep our API key in a `.env` file and use the `dotenv` library to access it. If you would like to use this method, create a file named `.env` in the same directory as this file and add the following line to it:

```text
OSF_API_TOKEN=add-your-api-token-here
```

``` r
load_dot_env()
API_TOKEN <- Sys.getenv("OSF_API_TOKEN")
if (API_TOKEN == "") {
    message("API key not found. Please set 'OSF_API_TOKEN' in your .env file.")
}
```

The OSF API requires the API token to be passed as a header:

``` r
HEADERS <- add_headers(
  Authorization = paste("Bearer", API_TOKEN)
)
```

## 1\. Fetching CC-BY 4.0 License Info

Using the `licenses` endpoint, we can find data relating to various licenses. In this example, we limit our search to CC-BY 4.0 licenses.

``` r
url <- "https://api.osf.io/v2/licenses?filter[name]=cc-by&filter[name]=4.0"
response <- GET(url, HEADERS)
data <- fromJSON(content(response, "text", encoding = "UTF-8"))
licenses <- data$data$attributes

for(i in seq_len(nrow(licenses))) {
    cat(licenses$name[i], "\n")
    cat(licenses$url[i], "\n\n")
    }
```

```text
CC-By Attribution 4.0 International
https://creativecommons.org/licenses/by/4.0/legalcode 

CC-BY Attribution-No Derivatives 4.0 International
https://creativecommons.org/licenses/by-nd/4.0/legalcode 

CC-BY Attribution-NonCommercial 4.0 International
https://creativecommons.org/licenses/by-nc/4.0/legalcode 

CC-BY Attribution-NonCommercial-ShareAlike 4.0 International
https://creativecommons.org/licenses/by-nc-sa/4.0/legalcode 
```

From the data returned, we can also retrieve the full-text of the licenses.

``` r
# Output limited to the first 264 characters for demonstration purposes
print(substr(data$data$attributes$text[1], 1, 264))
```

```
[1] "Creative Commons Attribution 4.0 International Public License\n\nBy exercising the Licensed Rights (defined below), You accept and agree to be bound by the terms and conditions of this Creative Commons Attribution 4.0 International Public License (\"Public License\")."
```

For the next example, we will create a named list named `ccby4_ids` that maps OSF license IDs to license names.

``` r
ccby4_ids <- list()
for (i in seq_along(data$data$id)) {
  ccby4_ids[[data$data$id[i]]] <- data$data$attributes$name[i]
}

ccby4_ids
```

```text
$`563c1cf88c5e4a3877f9e96a`
[1] "CC-By Attribution 4.0 International"

$`60bf983b58510b0009a5a9a4`
[1] "CC-BY Attribution-No Derivatives 4.0 International"

$`60bf992258510b0009a5a9a6`
[1] "CC-BY Attribution-NonCommercial 4.0 International"

$`60bf99e058510b0009a5a9a9`
[1] "CC-BY Attribution-NonCommercial-ShareAlike 4.0 International"
```

## 2\. Fetching Preprint Metadata and PDFs

In this use case, we will fetch the metadata for preprints that fall under a specified subject and are licensed under CC-BY 4.0 using the `preprints` endpoint. The metadata includes titles, publication dates, DOIs, authors, and PDF URLs.

### Function to Fetch Preprints Metadata

This function retrieves the metadata of CC-BY 4.0 preprints for a given subject, using the `ccby4_ids` obtained in the previous example to determine whether a preprint is CC-BY 4.0. For the sake of demonstration, only the first 100 preprints returned by the API are examined in this example.

``` r
# Function for fetching the metadata of preprints of a subject,
# keeping only CC-BY 4.0 preprints

fetch_preprints_metadata <- function(subject, limit = 1) {
  stopifnot(is.character(subject))
  stopifnot(is.numeric(limit))

  base_url <- "https://api.osf.io/v2/preprints"
  params <- list(
    "filter[subjects]" = subject,
    "page[size]" = 100
  )

  preprints <- list()
  url <- base_url
  iteration <- 0
  while (!is.null(url) && iteration < limit) {
    iteration <- iteration + 1
    if (url == base_url) {
      response <- GET(url, query = params, HEADERS)
    }
    else {
      response <- GET(url, HEADERS)
    }
    Sys.sleep(1)

    data <- fromJSON(
      content(response, "text", encoding = "UTF-8"),
      simplifyVector = FALSE
    )

    # Check if the preprint licenses are CC-BY 4.0
    for (preprint in data$data) {

      # Skip preprints that do not have license information
      if (is.null(preprint$relationships$license)) {
        next
      }

      licenseid <- preprint$relationships$license$data$id

      if (licenseid %in% names(ccby4_ids)) {
        preprints[[length(preprints) + 1]] <- preprint
      }
    }

    url <- data$links[["next"]]
  }

  return(preprints)
}

# Retrieve CC-BY 4.0 preprints from the first page of OSF preprint results (up to 100 records)
ccby4_metadata <- fetch_preprints_metadata(
  subject = "Education",
  limit = 1
)

# Print number of CC-BY 4.0 preprints found
length(ccby4_metadata)
```

```text
[1] 72
```

### Function to Get Contributors

This function will be used by the `process_preprints` function below to find the contributors from the preprint metadata.

``` r
get_contributors <- function(contributors_url) {
  if (is.null(contributors_url)) {
    return(list())
  }

  response <- GET(contributors_url, HEADERS)

  data <- fromJSON(
    content(response, "text", encoding = "UTF-8"),
    simplifyVector = FALSE
  )

  contributors <- character()

  for (contributor in data$data) {
    contributors <- c(contributors, contributor$embeds$users$data$attributes$full_name)
  }

  return(contributors)
}
```

### Processing Preprints

The following function processes the preprints metadata and downloads the PDFs for preprints that have a CC-BY 4.0 license.

``` r
process_preprints <- function(preprints, subject) {
  dir.create(paste0(subject, "_pdfs"), showWarnings = FALSE, recursive = TRUE)

  metadata_list <- list()

  for (preprint in preprints) {
    title <- preprint$attributes$title
    date <- preprint$attributes$date_published
    doi <- preprint$attributes$doi
    reviewed_doi <- preprint$links$preprint_doi

    contributors_url <- preprint$relationships$contributors$links$related$href
    authors <- get_contributors(contributors_url)

    pdf_url <- preprint$relationships$primary_file$links$related$href

    licenseid <- preprint$relationships$license$data$id
    license <- ccby4_ids[[licenseid]]

    metadata <- list (
      title = title,
      date = date,
      doi = ifelse(is.null(doi), NA_character_, doi),
      peer_reviewed_doi = reviewed_doi,
      authors = paste(authors, collapse = "; "),
      pdf_url = pdf_url,
      license = license
    )
    metadata_list[[length(metadata_list) + 1]] <- metadata

    # Don't download the PDF if no URL is available or the license isn't regular CC-BY 4.0
    if (is.null(metadata$pdf_url) || metadata$license != "CC-By Attribution 4.0 International") {
      next
    }

    # Download PDF
    pdf_response <- GET(metadata$pdf_url, HEADERS)
    if (!is.null(doi) && nzchar(doi)) {
      pdf_filename <- paste0(subject, "_pdfs/", gsub("/", "_", gsub("\\?", "", doi)), ".pdf")
    }
    else {
      parts <- strsplit(pdf_url, "/")[[1]]
      pdf_filename <- paste0(subject, "_pdfs/", parts[length(parts) - 1], ".pdf")
    }
    writeBin(
      content(pdf_response, "raw"),
      pdf_filename
    )
  }
  return(metadata_list)
}
```

### Example Usage

Fetch metadata and download PDFs for the preprints of the subject "Education".

``` r
# Note that this code block might take a few minutes to fully run
metadata_list <- process_preprints(ccby4_metadata, "Education")
df <- do.call(rbind, lapply(metadata_list, as.data.frame))
write.csv(
  df,
  "preprints_metadata.csv",
  row.names = FALSE
)
head(df)
```

```
##   title                                                        date                   doi
##   <chr>                                                        <chr>                  <chr>
## 1 El oficio sin ley: el verdadero estatuto jurídico del corre… 2026-07-17T09:42:34…  10.5281/zenodo.21402788
## 2 Los herederos del vivo: la expectativa sucesoria y el inte…  2026-07-17T09:48:38…  10.5281/zenodo.21402249
## 3 Early Warning System for At-Risk Students using SHAP value…  2026-07-15T22:37:05…  NA
## 4 A Global School Attendance Crisis: Rising Absenteeism and …  2026-07-15T22:37:38…  NA
## 5 Dimensional Consensus and Operationalisation Gaps in K–12 …  2026-07-15T22:39:33…  NA
## 6 From Digital Literacy to AI Literacy: New Directions for L…  2026-07-15T22:41:38…  NA
##
##   peer_reviewed_doi                      authors                                pdf_url
##   <chr>                                  <chr>                                  <chr>
## 1 https://doi.org/10.31235/osf.io/nqzs5_v1 Andrés Gabriel Varas Quijón          https://api.osf.io/v2/files/6a59701c55eafd50b0025e38/
## 2 https://doi.org/10.31235/osf.io/68fpv_v1 Andrés Gabriel Varas Quijón          https://api.osf.io/v2/files/6a595c1d73227e6090d6db9f/
## 3 https://doi.org/10.35542/osf.io/ga46q_v1 Lokesh Gundoju                       https://api.osf.io/v2/files/6a57a34b87a7fd78c67afbc4/
## 4 https://doi.org/10.35542/osf.io/mfxag_v1 Alec I. Kennedy; Rolf Strietholt     https://api.osf.io/v2/files/6a574d6334f0e2b617de6ef1/
## 5 https://doi.org/10.35542/osf.io/s5re4_v1 Thomas Leitgeb                       https://api.osf.io/v2/files/6a573d7834b5ab154ade6edc/
## 6 https://doi.org/10.35542/osf.io/d8eg5_v1 Mohamed Ouhejjou                     https://api.osf.io/v2/files/6a56f9149136620b07de6e4e/
##
##   license
##   <chr>
## 1 CC-By Attribution 4.0 International
## 2 CC-By Attribution 4.0 International
## 3 CC-By Attribution 4.0 International
## 4 CC-By Attribution 4.0 International
## 5 CC-By Attribution 4.0 International
## 6 CC-By Attribution 4.0 International
##
## 6 rows | 1-6 of 7 columns
``
```

## 3\. Batch Processing for Multiple Subjects

This example demonstrates how the functions above can be used to retrieve the data and PDFs for multiple subjects.

``` r
subjects <- list(
  "Education",
  "Social and Behavioral Sciences"
)
```

``` r
for (subject in subjects) {
  preprints <- fetch_preprints_metadata(subject)
  metadata_list <- process_preprints(preprints, subject)
  df <- dplyr::bind_rows(metadata_list)
  write.csv(
    df,
    paste0(subject, ".csv"),
    row.names = FALSE
  )
  cat(
    "Saved",
    length(metadata_list),
    "preprints for",
    subject,
    "\n"
  )
}
```

```text
Saved 72 preprints for Education
Saved 86 preprints for Social and Behavioral Sciences
```