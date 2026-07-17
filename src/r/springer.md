---
title: "Springer Nature API in R"
output: 
  html_document:
    keep_md: true
---



# Springer Nature API in R

by Nick Peinitz, Avery Fernandez and Vincent F. Scalfani

<div class="rmd-btn-wrapper">
  <a class="rmd-btn"
     href="https://github.com/UA-Libraries-Research-Data-Services/UALIB_ScholarlyAPI_Cookbook/blob/main/rmarkdown/springer.Rmd"
     target="_blank"
     rel="noreferrer">
    View RMarkdown File
  </a>
</div>

These recipe examples use the Springer Nature Open Access API to retrieve metadata and full-text content. The Springer Nature Open Access API includes about 1.5 million full-text records.

*There is also a Full-Text API for subscription content. Please check with your institution for their Text and Data Mining or related License Agreement with Springer Nature.*

Please see the following resources for more information on API usage:
- Documentation
    - <a href="https://dev.springernature.com/" target="_blank">Springer Nature API</a>
    - <a href="https://dev.springernature.com/docs/api-endpoints/open-access/" target="_blank">Springer Nature API Documentation</a>
    - <a href="https://dev.springernature.com/docs/quick-start/api-access/" target="_blank">Springer Nature API Access Quick Start</a>
    - <a href="https://dev.springernature.com/docs/live-documentation/" target="_blank">Springer API Playground</a>
- Terms
    - <a href="https://www.springernature.com/gp/legal/general-terms-of-use/15067848" target="_blank">Springer Nature General Terms of Use</a>
    - <a href="https://dev.springernature.com/terms-conditions" target="_blank">Springer Nature API Terms and Conditions</a>
- Data Reuse
    - <a href="https://www.springernature.com/gp/researchers/text-and-data-mining" target="_blank">Springer Nature Text and Data Mining Policies</a>
    - <a href="https://dev.springernature.com/tdm-reservation-policy/" target="_blank">Springer Nature TDM Reservation Policy</a>


_**NOTE:**_ Check with your institution to determine your <a href="https://dev.springernature.com/docs/rate-limit-details/rate-limits/" target="_blank">API rate limit with Springer Nature</a>.

*If you have copyright or other related text and data mining questions, please contact The University of Alabama Libraries or your respective library/institution.*

*These recipe examples were tested on July 17, 2026.*


## Setup

### Import Libraries

The following external libraries need to be installed into your environment to run the code examples in this tutorial:

- <a href="https://github.com/r-lib/httr2" target="_blank">httr2</a>
- <a href="https://github.com/gaborcsardi/dotenv" target="_blank">dotenv</a>
- <a href="https://github.com/r-lib/xml2" target="_blank">xml2</a>

We import the libraries used in this tutorial below:


```r
library(httr2)
library(dotenv)
library(xml2)
```

### Import API Key

Authentication is required to access the Springer Nature API. You can sign up for one at the <a href="https://dev.springernature.com/" target="_blank">Springer Nature Developer Portal</a>.

We keep our API key in a separate file, a `.env` file, and use the `dotenv` library to access it. If you use this method, create a file named `.env` in the same directory as this notebook and add the following line to it:

```text
SPRINGER_API_KEY=PUT_YOUR_API_KEY_HERE
```


``` r
load_dot_env()
API_KEY <- Sys.getenv("SPRINGER_API_KEY")
if (API_KEY == "") {
    message("API key not found. Please set 'SPRINGER_API_KEY' in your .env file.")
}
```

## 1. Retrieve Full-Text JATS XML of an Article

In this section, we demonstrate how to retrieve the JATS XML content for a specific article based on its DOI.

The <a href="https://jats.nlm.nih.gov/archiving/" target="_blank">JATS</a> XML format is a standard intended for tagging, archiving, and exchanging journal articles. The Springer Nature Open Access API allows us to retrieve articles in JATS XML format.

Key parameters:
* `base_url`: The base URL for the Springer Nature API (Open Access JATS endpoint).
* `q=(doi:DOI)`: The query parameter used to search for an article based on its DOI.
* `api_key`: The query parameter used to pass our valid API key.

More details about the parameters can be found at <a href="https://dev.springernature.com/restfuloperations" target="_blank">Springer Nature Developer Portal</a>.

You can also play around with the API using the <a href="https://dev.springernature.com/docs/live-documentation/" target="_blank">Springer API Playground</a>.

```r
# Example article from SpringerOpen Brain Informatics
# This article is under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
# https://doi.org/10.1186/s40708-025-00250-5
doi <- '10.1186/s40708-025-00250-5'

tryCatch (
    {
    response <- request("https://api.springernature.com/openaccess/jats") |>
                    req_url_query(
                        q = sprintf('doi:"%s")', doi),
                        api_key = API_KEY
                ) |>
                req_perform()

                writeLines(
                    resp_body_string(response),
                    "fulltext.jats"
                )

                message(
                    sprintf(
                        "JATS XML successfully retrieved for DOI %s. Saved to fulltext.jats",
                        doi
                    )
                )
    },
    error = function(e) {
        message (
            sprintf (
                "Error retrieving JATS XML for DOI %s: %s",
                doi,
                e$message
            )
        )
    }
)
```

```text
JATS XML successfully retrieved for DOI 10.1186/s40708-025-00250-5. Saved to fulltext.jats
```

## 2. Retrieve Full-Text in a Loop

In many cases, you may have a list of DOIs and want to retrieve the full-text for each of them. Below, we loop over a set of DOIs, retrieve the JATS XML, and store each one in a separate file.
A short delay (`Sys.sleep(1)`) is used to avoid hitting rate limits.

``` r
dois <- list(
    # Licensed under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
    # https://doi.org/10.1186/s40708-025-00250-5
    '10.1186/s40708-025-00250-5', 
    # Licensed under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
    # https://doi.org/10.1186/s40708-024-00247-6
    '10.1186/s40708-024-00247-6', 
    # Licensed under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
    # https://doi.org/10.1186/s40708-024-00243-w
    '10.1186/s40708-024-00243-w', 
    # Licensed under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
    # https://doi.org/10.1186/s40708-023-00202-x
    '10.1186/s40708-023-00202-x', 
    # Licensed under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
    # https://doi.org/10.1186/s40708-023-00204-9
    '10.1186/s40708-023-00204-9' 
)

for (i in seq_along(dois)) {
  doi <- dois[[i]]
  print(
    paste0(
      "Retrieving JATS XML for DOI ",
      doi,
      "(",
      i,
      "/",
      length(dois),
      ")..."
    )
  )

  tryCatch (
    {
    response <- request("https://api.springernature.com/openaccess/jats") |>
                    req_url_query(
                        q = sprintf('doi:"%s")', doi),
                        api_key = API_KEY
                ) |>
                req_perform()

                doi_name <- gsub("/", "_", doi, fixed = TRUE)
                doi_name <- gsub('"', "", doi_name, fixed = TRUE)

                output_file = paste0(doi_name, "_jats_text.jats")

                writeLines(
                    resp_body_string(response),
                    output_file
                )

                message(
                        paste0("JATS XML retrieved for DOI",
                        doi,
                        ". Saved to ",
                        output_file,
                        "."
                      )
                )
    },
    error = function(e) {
        message (
            sprintf (
                "Error retrieving JATS XML for DOI %s: %s",
                doi,
                e$message
            )
        )   
    }
  )

  # Delay to avoid hitting rate limits
  Sys.sleep(1)
}
```

```text
[1] "Retrieving JATS XML for DOI 10.1186/s40708-025-00250-5(1/5)..."
JATS XML retrieved for DOI10.1186/s40708-025-00250-5. Saved to 10.1186_s40708-025-00250-5_jats_text.jats.

[1] "Retrieving JATS XML for DOI 10.1186/s40708-024-00247-6(2/5)..."
JATS XML retrieved for DOI10.1186/s40708-024-00247-6. Saved to 10.1186_s40708-024-00247-6_jats_text.jats.

[1] "Retrieving JATS XML for DOI 10.1186/s40708-024-00243-w(3/5)..."
JATS XML retrieved for DOI10.1186/s40708-024-00243-w. Saved to 10.1186_s40708-024-00243-w_jats_text.jats.

[1] "Retrieving JATS XML for DOI 10.1186/s40708-023-00202-x(4/5)..."
JATS XML retrieved for DOI10.1186/s40708-023-00202-x. Saved to 10.1186_s40708-023-00202-x_jats_text.jats.

[1] "Retrieving JATS XML for DOI 10.1186/s40708-023-00204-9(5/5)..."
JATS XML retrieved for DOI10.1186/s40708-023-00204-9. Saved to 10.1186_s40708-023-00204-9_jats_text.jats.
```

## 3. Acquire and Parse Metadata (JSON)

Alternatively, you can retrieve only the metadata in JSON format by switching the base URL to the `json` endpoint. Then, you can parse relevant fields (e.g., abstract, publication date, etc.).

``` r
# Example article from SpringerOpen Brain Informatics
# This article is under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
# https://doi.org/10.1186/s40708-025-00250-5
# Harnessing the synergy of statistics and \
# deep learning for BCI competition 4 dataset 4: a novel approach
# Gauttam Jangir, Nisheeth Joshi & Gaurav Purohit 
doi <- '10.1186/s40708-025-00250-5'

metadata_response <- tryCatch (
    {
    response <- request("https://api.springernature.com/openaccess/json") |>
                    req_url_query(
                        q = sprintf('doi:"%s")', doi),
                        api_key = API_KEY
                ) |>
                req_perform()
    
    resp_body_json(response)

    },
    error = function(e) {
        message (
            sprintf (
                "Error retrieving JSON metadata for DOI %s: %s",
                doi,
                e$message
            )
        )
        list() 
    }
)

names(metadata_response)
```



```text
[1] "apiMessage" "query"      "result"     "records"    "facets" 
 ```


Below is an example of how to retrieve specific fields from the metadata, such as the article's **abstract**, **DOI**, **publication date**, **publication name**, and **title**.


``` r
# metadata_response usually contains the named elements: c("apiMessage", "query", "records")

api_message <- metadata_response[["apiMessage"]]
query_info <- metadata_response[["query"]]
records <- metadata_response[["records"]]
if (is.null(records))
{
  records <- list()
}

print(paste0("API Message:", api_message))
print(paste0("Query:", query_info))

if (length(records) > 0)
{
  # Take the first record if available
  first_record <- records[[1]]
  abstract <- first_record[["abstract"]]
  if (is.null(abstract))
  {
    abstract <- list()
  }
  p <- abstract[["p"]]
  if (is.null(p))
  {
    p <- ""
  }
  cat("Abstract:", p, "\n")

  print(paste0("DOI:", first_record[["doi"]]))
  print(paste0("Online Date:", first_record[["onlineDate"]]))
  print(paste0("Print Date:", first_record[["printDate"]]))
  print(paste0("Publication Name:", first_record[["publicationName"]]))
  print(paste0("Title:", first_record[["title"]]))

  # Get the authors
  creators <- first_record[["creators"]]
  if (is.null(creators))
  {
    creators <- list()
  }

  authors <- c()

  for (author in creators)
  {
    authors <- c(
      authors,
      author[["creator"]]
    )
  }
  print(paste0("Authors:", paste(authors, collapse = ", ")))
} else {
  print("No 'records' were returned in the JSON response.")
}
```


```text
[1] "API Message:This JSON was provided by Springer Nature"
[1] "Query:doi:\"10.1186/s40708-025-00250-5\")"
Abstract: Human brain signal processing and finger’s movement coordination is a complex mechanism. In this mechanism finger’s movement is mostly performed for every day’s task. It is well known that to capture such movement EEG or ECoG signals are used. In this order to find the patterns from these signals is important. The BCI competition 4 dataset 4 is one such standard dataset of ECoG signals for individual finger movement provided by University of Washington, USA. In this work, this dataset is, statistically analyzed to understand the nature of data and outliers in it. Effectiveness of pre-processing algorithm is then visualized. The cleaned dataset has dual polarity and gaussian distribution nature which makes Tanh activation function suitable for the neural network BC4D4 model. BC4D4 uses Convolutional neural network for feature extraction, dense neural network for pattern identification and incorporating dropout & regularization making the proposed model more resilient. Our model outperforms the state of the art work on the dataset 4 achieving 0.85 correlation value that is 1.85X (Winner of BCI competition 4, 2012) & 1.25X (Finger Flex model, 2022). 
[1] "DOI:10.1186/s40708-025-00250-5"
[1] "Online Date:2025-02-15"
[1] "Print Date:"
[1] "Publication Name:Brain Informatics"
[1] "Title:Harnessing the synergy of statistics and deep learning for BCI competition 4 dataset 4: a novel approach"
[1] "Authors:Jangir, Gauttam, Joshi, Nisheeth, Purohit, Gaurav"
```

## 4. Parsing XML for Metadata

Sometimes you may want to extract specific pieces of data (e.g., *title*, *abstract*, *authors*, *subjects*) directly from the **JATS XML** instead of the JSON. In this example, we use the R `xml2` package to parse the XML.

The XML structure has a `<records>` tag containing one or more `<article>` tags. Each `<article>` has a `<front>` section for metadata, a `<body>` for main text, and possibly `<back>` for references, etc.

``` r
# example article from SpringerOpen Brain Informatics
# This article is under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
# https://doi.org/10.1186/s40708-025-00250-5
# Harnessing the synergy of statistics and \
# deep learning for BCI competition 4 dataset 4: a novel approach
# Gauttam Jangir, Nisheeth Joshi & Gaurav Purohit 
doi <- '10.1186/s40708-025-00250-5'

xml_data <- NULL
tryCatch (
    {
    response <- request("https://api.springernature.com/openaccess/jats") |>
                    req_url_query(
                        q = sprintf('doi:"%s")', doi),
                        api_key = API_KEY
                ) |>
                req_perform()

      xml_data <- resp_body_string(response)

                message(
                    sprintf(
                        "JATS XML successfully retrieved for DOI %s.",
                        doi
                    )
                )
    },
    error = function(e) {
        message (
            sprintf (
                "Error retrieving JATS XML for DOI %s: %s",
                doi,
                e$message
            )
        )
        NULL
    }
)
```


```text
JATS XML successfully retrieved for DOI 10.1186/s40708-025-00250-5.
```

``` r
root = NULL

if (!is.null(xml_data))
{
  tryCatch (
    {
      root <- read_xml(xml_data)
      print("XML data successfully parsed.")
    },
    error = function(e) {
        message (
            sprintf (
                "Error parsing XML data: %s",
                e$message
            )
        )   
    }
  )
}
```

```text
[1] "XML data successfully parsed."
```

``` r
article_data <- list (
  title = NULL,
  abstract = NULL,
  authors = character(),
  subjects = character()
)

if (!is.null(root))
{
  # Assume there's at least one article under records.
  first_article <- xml_find_first(root, ".//records/article")
  if (!inherits(first_article, "xml_missing"))
  {
    # Title
    title_elem <- xml_find_first(first_article, ".//front/article-meta/title-group/article-title")
    if (!inherits(title_elem, "xml_missing"))
    {
      article_data[["title"]] <- xml_text(title_elem)
    }

    # Abstract
    abstract_elem <- xml_find_first(first_article, ".//front/article-meta/abstract/p")
    if (!inherits(abstract_elem, "xml_missing"))
    {
      article_data[["abstract"]] <- xml_text(abstract_elem)
    }

    # Authors
    authors <- xml_find_all(first_article, ".//front/article-meta/contrib-group/contrib/name")
    for (author in authors)
    {
      # Each author element may have multiple child tags (given, surname, etc.).
      full_name <- paste(xml_text(xml_children(author)), collapse = " ")

      article_data$authors <- c(
        article_data$authors,
        full_name
      )
    }

    # Subjects (keywords)
    subjects <- xml_find_all(first_article, ".//front/article-meta/kwd-group/kwd")
    for (subject in subjects)
    {
      article_data$subjects <- c(
        article_data$subjects,
        xml_text(subject)
      )
    }
  } else {
    print("No article data found in the XML.")
  }
} else {
  print("No valid XML data to parse.")
}

str(article_data)
```


```text
List of 4
 $ title   : chr "Harnessing the synergy of statistics and deep learning for BCI competition 4 dataset 4: a novel approach"
 $ abstract: chr "Human brain signal processing and finger’s movement coordination is a complex mechanism. In this mechanism fing"| __truncated__
 $ authors : chr [1:3] "Jangir Gauttam" "Joshi Nisheeth" "Purohit Gaurav"
 $ subjects: chr [1:6] "BCI (Brain Computer Interface)" "EEG (electroencephalogram)" "Electrocorticography (ECoG)" "Event Related Potential (ERP)" ...
```

## 5. Parsing XML for Figure Captions

Figure captions often appear under `<fig>` tags inside the `<body>` element. Each figure may have a `<label>` tag for the figure number and a `<caption>` tag for the figure's description.

``` r
# Example article from SpringerOpen Brain Informatics
# This article is under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
# https://doi.org/10.1186/s40708-025-00250-5
# Harnessing the synergy of statistics and \
# deep learning for BCI competition 4 dataset 4: a novel approach
# Gauttam Jangir, Nisheeth Joshi & Gaurav Purohit 
doi <- '10.1186/s40708-025-00250-5'

xml_data <- NULL
tryCatch (
    {
    response <- request("https://api.springernature.com/openaccess/jats") |>
                    req_url_query(
                        q = sprintf('doi:"%s")', doi),
                        api_key = API_KEY
                ) |>
                req_perform()

      xml_data <- resp_body_string(response)

                message(
                    sprintf(
                        "JATS XML successfully retrieved for DOI %s.",
                        doi
                    )
                )
    },
    error = function(e) {
        message (
            sprintf (
                "Error retrieving JATS XML for DOI %s: %s",
                doi,
                e$message
            )
        )   
    }
)
```


```text
JATS XML successfully retrieved for DOI 10.1186/s40708-025-00250-5.
```

``` r
root = NULL

if (!is.null(xml_data))
{
  tryCatch (
    {
      root <- read_xml(xml_data)
      print("XML data successfully parsed.")
    },
    error = function(e) {
        message (
            sprintf (
                "Error parsing XML data: %s",
                e$message
            )
        )   
    }
  )
} else {
  print("No valid XML data to parse.")
}
```
```text
[1] "XML data successfully parsed."
```

```r
figures_data <- list()

if (!inherits(root, "xml_missing") && !is.null(root))
{
  # Find all <fig> elements within the <body> of the XML
  figures <- xml_find_all(root, ".//body//fig")
  for (fig in figures)
  {
    # Extract the <label> element (e.g., "Figure 1") if it exists
    label <- xml_find_first(fig, "label")
    # Extract the <caption> element (e.g., description of the figure) if it exists
    caption <- xml_find_first(fig, "caption")

    # Get the text content of the label, or use an empty string if not present
    if (!inherits(label, "xml_missing"))
    {
      label_text <- xml_text(label)
    } else {
      label_text <- ""
    }

    # Get the text content of the caption, joining all inner text, or use an empty string if not present
    if(!inherits(caption, "xml_missing"))
    {
      caption_text <- xml_text(caption)
    } else {
      caption_text <- ""
    }

    # Append the figure's label and caption as a named list
    figures_data[[length(figures_data) + 1]] <- list(
      label = label_text,
      caption = trimws(caption_text)
    )
  }
} else {
  # If the XML root is not valid, print an error message
  print("No valid XML data to parse.")
}

# Check if any figures were found and processed
if (length(figures_data) > 0)
{
  print("Figures data:")
  # Iterate through the collected figures and print their details
  for (i in seq_along(figures_data))
  {
    fig_data <- figures_data[[i]]
    print(paste0("Figure ", i, ":"))
    print(paste0("Label: ", fig_data[["label"]]))
    print(paste0("Caption: ", fig_data[["caption"]]))
    print("\n")
  }
} else {
  # If no figures were found, print a message indicating this
  print("No figures data found in the XML.")
}
```


```text
[1] "Figures data:"
[1] "Figure 1:"
[1] "Label: Fig. 1"
[1] "Caption: Capturing individual finger flexion [57, 58]"
[1] "\n"
[1] "Figure 2:"
[1] "Label: Fig. 2"
[1] "Caption: Subject 1 fingers"
[1] "\n"
[1] "Figure 3:"
[1] "Label: Fig. 3"
[1] "Caption: Box Plot (five-point summary)"
[1] "\n"
[1] "Figure 4:"
[1] "Label: Fig. 4"
[1] "Caption: Subject 2 fingers"
[1] "\n"
[1] "Figure 5:"
[1] "Label: Fig. 5"
[1] "Caption: Subject 3 fingers"
[1] "\n"
[1] "Figure 6:"
[1] "Label: Fig. 6"
[1] "Caption: Unusual data point (Outlier) in dataset"
[1] "\n"
[1] "Figure 7:"
[1] "Label: Fig. 7"
[1] "Caption: Isolation forest tree"
[1] "\n"
[1] "Figure 8:"
[1] "Label: Fig. 8"
[1] "Caption: Histogram of subject 1"
[1] "\n"
[1] "Figure 9:"
[1] "Label: Fig. 9"
[1] "Caption: Subject 1 fingers after isolation forest"
[1] "\n"
[1] "Figure 10:"
[1] "Label: Fig. 10"
[1] "Caption: Histogram of subject 2"
[1] "\n"
[1] "Figure 11:"
[1] "Label: Fig. 11"
[1] "Caption: Subject 2 fingers after isolation forest"
[1] "\n"
[1] "Figure 12:"
[1] "Label: Fig. 12"
[1] "Caption: Histogram of subject 3"
[1] "\n"
[1] "Figure 13:"
[1] "Label: Fig. 13"
[1] "Caption: Subject 3 fingers after isolation forest"
[1] "\n"
[1] "Figure 14:"
[1] "Label: Fig. 14"
[1] "Caption: BC4D4 model architecture"
[1] "\n"
[1] "Figure 15:"
[1] "Label: Fig. 15"
[1] "Caption: Activation functions"
[1] "\n"
[1] "Figure 16:"
[1] "Label: Fig. 16"
[1] "Caption: BC4D4 model layered architecture"
[1] "\n"
[1] "Figure 17:"
[1] "Label: Fig. 17"
[1] "Caption: Correlation value of BC4D4 with softsign & tanh"
[1] "\n"
[1] "Figure 18:"
[1] "Label: Fig. 18"
[1] "Caption: Models comparison"
[1] "\n"
```

## 6. Extracting Full-Text from the Body

Finally, we can extract a rough "plain text" version of the article body by iterating through each element (e.g., `<sec>`, `<p>`), capturing the text, and joining it into a single string. This can help with quick text-based analyses.

```r
# Example article from SpringerOpen Brain Informatics
# This article is under CC-BY-4.0 license https://creativecommons.org/licenses/by/4.0/
# https://doi.org/10.1186/s40708-025-00250-5
# Harnessing the synergy of statistics and \
# deep learning for BCI competition 4 dataset 4: a novel approach
# Gauttam Jangir, Nisheeth Joshi & Gaurav Purohit 
doi <- '10.1186/s40708-025-00250-5' 

xml_data <- NULL
tryCatch (
    {
    response <- request("https://api.springernature.com/openaccess/jats") |>
                    req_url_query(
                        q = sprintf('doi:"%s")', doi),
                        api_key = API_KEY
                ) |>
                req_perform()

      xml_data <- resp_body_string(response)

                message(
                    sprintf(
                        "JATS XML successfully retrieved for DOI %s.",
                        doi
                    )
                )
    },
    error = function(e) {
        message (
            sprintf (
                "Error retrieving JATS XML for DOI %s: %s",
                doi,
                e$message
            )
        )   
    }
)
```

```text
JATS XML successfully retrieved for DOI 10.1186/s40708-025-00250-5.
```

```r
root = NULL

if (!is.null(xml_data))
{
  tryCatch (
    {
      root <- read_xml(xml_data)
      print("XML data successfully parsed.")
    },
    error = function(e) {
        message (
            sprintf (
                "Error parsing XML data: %s",
                e$message
            )
        )   
    }
  )
} else {
  print("No valid XML data to parse.")
}
```

```text
[1] "XML data successfully parsed."
```

```r
full_text <- ""
if (!is.null(root) && !inherits(root, "xml_missing"))
{
  # Find the body element
  body <- xml_find_first(root, ".//body")
  if (!is.null(body) && !inherits(body, "xml_missing"))
  {
    # We'll store text in a list, then join them.
    text_parts <- list()

    # We can iterate over each top-level child in the body.
    # Typically <sec> tags hold paragraphs, etc.
    sections <- xml_children(body)
    for (section in sections)
    {
      # For each subsection, gather all text.
      sub_sections <- xml_children(section)
      for (sub_section in sub_sections)
      {
        text_parts <- c(
          text_parts,
          xml_text(sub_section)
        )
      }
      # Combine everything
      full_text <- paste(text_parts, collapse = "\n")
      full_text <- trimws(full_text)
    }
  } else {
    print("No body content found in the XML.")
  }
} else {
  print("No valid XML data to parse.")
}

if (length(full_text) > 0 && full_text != "")
{
  writeLines(
    full_text,
    "fulltext.txt"
    )
  print("Full text extracted and saved to 'fulltext.txt'.")  
} else {
  print("No full text content found in the XML.")
}
```

```text
[1] "Full text extracted and saved to 'fulltext.txt'."
```

```r
# Output a portion of the full text to the console
print(substr(full_text, 1, 395))
```
```text
[1] "Introduction\nTHe brain is the most active organ of the human body that takes input, processes them, and gives output. Fingers play an important role in human life that is why one of the active fields for rehabilitation is the Brain-Computer Interface (BCI) where fingers and EEG signals are studied together for the normal routine return of a physically challenged or locomotive disabled person."
```