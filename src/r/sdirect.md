# ScienceDirect API in R

by Michael T. Moen

**ScienceDirect**: https://www.sciencedirect.com/

**Elsevier Developer Portal:** https://dev.elsevier.com/

**ScienceDirect APIs Specification:** https://dev.elsevier.com/sd_api_spec.html

**Elsevier How to Guide: Text Mining:** https://dev.elsevier.com/tecdoc_text_mining.html

Please check with your institution for their Text and Data Mining Agreement with Elsevier.

These recipe examples use the Elsevier ScienceDirect Article (Full-Text) API. This tutorial content is intended to help facillitate academic research. Before continuing or reusing any of this code, please be aware of Elsevier’s API policies and appropiate use-cases, as for example, Elsevier has detailed policies regarding [text and data mining of Elsevier full-text content](https://dev.elsevier.com/text_mining.html). If you have copyright or other related text and data mining questions, please contact The University of Alabama Libraries.

*These recipe examples were tested on February 7, 2025.*

## Setup

### Import Libraries

```{r}
library(httr)
```

### Import API Key

An API key is required to access the ScienceDirect API. Registration is available on the [Elsevier developer portal](https://dev.elsevier.com/). The key is imported from an environment variable below:

```{r}
myAPIKey <- Sys.getenv("sciencedirect_key")
```

### Identifier Note

We will use DOIs as the article identifiers. See our Crossref and Scopus API tutorials for workflows on how to create lists of DOIs and identfiers for specific searches and journals. The Elsevier ScienceDirect Article (Full-Text) API also accepts other identifiers like Scopus IDs and PubMed IDs (see API specification documents linked above).

## 1. Retrieve full-text XML of an article

```{r}
# For XML download
elsevier_url <- "https://api.elsevier.com/content/article/doi/"
doi1 <- '10.1016/j.tetlet.2017.07.080' # Example Tetrahedron Letters article
fulltext1 <- GET(paste0(elsevier_url, doi1, "?APIKey=", myAPIKey, "&httpAccept=text/xml"))

# Save to file
writeLines(content(fulltext1, "text"), "fulltext1.xml")
```

## 2. Retrieve plain text of an article

```{r}
# For simplified text download
doi2 <- '10.1016/j.tetlet.2022.153680' # Example Tetrahedron Letters article
fulltext2 <- GET(paste0(elsevier_url, doi2, "?APIKey=", myAPIKey, "&httpAccept=text/plain"))

# Save to file
writeLines(content(fulltext2, "text"), "fulltext2.txt")
```

## 3. Retrieve full-text in a loop

```{r}
# Make a list of 5 DOIs for testing
dois <- c('10.1016/j.tetlet.2018.10.031',
          '10.1016/j.tetlet.2018.10.033',
          '10.1016/j.tetlet.2018.10.034',
          '10.1016/j.tetlet.2018.10.038',
          '10.1016/j.tetlet.2018.10.041')
```

```{r}
for (doi in dois) {
  article <- GET(paste0(elsevier_url, doi, "?APIKey=", myAPIKey, "&httpAccept=text/plain"))
  doi_name <- gsub("/", "_", doi)
  writeLines(content(article, "text"), paste0(doi_name, "_plain_text.txt"))
  Sys.sleep(1)
}
```

## R Session Info

```{r}
sessionInfo()
```

```
## R version 4.4.2 (2024-10-31 ucrt)
## Platform: x86_64-w64-mingw32/x64
## Running under: Windows 11 x64 (build 22631)
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
##  [1] compiler_4.4.2    R6_2.5.1          fastmap_1.2.0     cli_3.6.3
##  [5] htmltools_0.5.8.1 tools_4.4.2       curl_6.1.0        yaml_2.3.10
##  [9] rmarkdown_2.29    knitr_1.49        xfun_0.50         digest_0.6.37    
## [13] rlang_1.1.4       evaluate_1.0.3   
```
