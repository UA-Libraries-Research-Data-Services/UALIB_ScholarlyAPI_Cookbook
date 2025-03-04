# University of Alabama Libraries Scholarly API Cookbook

> [!IMPORTANT]
> March 2025 - We have decided to no longer maintain the Matlab, Mathematica, and C recipes and have removed them from the Scholarly API Cookbook. These archived recipes are still in the [UA Libraries Scholarly API Cookbook Archive](https://github.com/UA-Libraries-Research-Data-Services/Scholarly_API_Cookbook_Archive).
> November 2024 - Some R code tutorials that were originally MIT Licensed, are now licensed under the GPL-3 License to comply with the licensing terms of dependent R libraries.

The University of Alabama Libraries Scholarly API Cookbook is an open online book containing short scholarly API code examples (i.e., “recipes”) that demonstrate how to work with various scholarly web service APIs. It is part of the University of Alabama Libraries efforts to support Research Data Services. Read the book [here](https://ua-libraries-research-data-services.github.io/UALIB_ScholarlyAPI_Cookbook).

## License and Reuse

Most of the code in this repository is licensed under the [MIT License](https://github.com/UA-Libraries-Research-Data-Services/UALIB_ScholarlyAPI_Cookbook/blob/main/LICENSE).

The Python scripts in this repository are licensed under the MIT License. However, these scripts may rely on external libraries such as matplotlib, pandas, and others. These libraries are licensed under their own respective terms, and will need to be installed separately. Refer to the documentation of each library for installation instructions and licensing details.

The Bash tutorials are licensed under the MIT License (Bash itself is licensed under the GNU General Public License). Some of the included scripts may rely on external tools such as curl, jq, and gnuplot, each of which is licensed under its own terms. Users must obtain and install these tools separately. Refer to the documentation of each tool for installation instructions and licensing details.

Lastly, most of the R tutorial scripts are MIT licensed, but some are licensed under the [GPL-3 License](https://github.com/UA-Libraries-Research-Data-Services/UALIB_ScholarlyAPI_Cookbook/blob/main/LICENSE_selected_R_tutorials) because they depend on GPL-licensed R libraries (refer to the documentation of each R library for installation instructions and licensing details). The R tutorials with GPL-3 licenses are indicated at the top of the respective files and organized separately in the folder `src/r-gpl3/`.

We have endeavored to follow the appropriate terms and usage policies of each scholarly API, web service, and Z39.50 server. We have linked to the terms and policies where possible. Some database APIs may require a valid library subscription, institutional access, or individual account to use their services. Please be responsible when reusing these scripts and respect the API terms and usage policies (e.g., query limits, record downloads, data sharing restrictions). Data output snippets shown in this book are for demonstration purposes and are credited to the individual API or database service. The output generated from APIs or services remains subject to the terms and conditions of the respective provider. Some outputs (e.g., U.S. Government works) may be in the public domain, while others may require attribution or adherence to other conditions.

If you reuse the code, attribution would be appreciated. Please link to the Cookbook and cite our manuscript:

Link to Cookbook: https://ua-libraries-research-data-services.github.io/UALIB_ScholarlyAPI_Cookbook

Citation: Scalfani, V. F.; Walker, K. W.; Simpson, L.; Fernandez, A. M.; Patel, V. D.; Ramig, A.; Gomes, C.; Moen, M. T.; Nguyen, A. M. Creating a Scholarly API Cookbook: Supporting Library Users with Programmatic Access to Information. Issues in Science and Technology Librarianship, 2023, No. 104. https://doi.org/10.29173/istl2766.

```bibtex
@article{scalfani_creating_2023,
        title = {Creating a {Scholarly} {API} {Cookbook}: {Supporting} {Library} {Users} with {Programmatic} {Access} to {Information}},
        issn = {1092-1206},
        shorttitle = {Creating a {Scholarly} {API} {Cookbook}},
        url = {https://journals.library.ualberta.ca/istl/index.php/istl/article/view/2766},
        doi = {10.29173/istl2766},
        abstract = {Scholarly web-based application programming interfaces (APIs) allow users to interact with information and data programmatically. Interacting with information programmatically allows users to create advanced information query workflows and quickly access machine-readable data for downstream computations. With the growing availability of scholarly APIs from open and commercial library databases, supporting access to information via an API has become a key support area for research data services in libraries. This article describes our efforts with supporting API access through the development of an online Scholarly API Cookbook. The Cookbook contains code recipes (i.e., tutorials) for getting started with 10 different scholarly APIs, including for example, Scopus, World Bank, and PubMed. API tutorials are available in Python, Bash, Matlab, and Mathematica. A tutorial for interacting with library catalog data programmatically via Z39.50 is also included, as traditional library catalog metadata is rarely available via an API. In addition to describing the Scholarly API Cookbook content, we discuss our experiences building a student research data services programming team, challenges we encountered, and ideas to improve the Cookbook. The University of Alabama Libraries Scholarly API Cookbook is freely available and hosted on GitHub. All code within the API Cookbook is licensed with the permissive MIT license, and as a result, users are free to reuse and adapt the code in their teaching and research.},
        number = {104},
        urldate = {2023-10-13},
        journal = {Issues in Science and Technology Librarianship},
        author = {Scalfani, Vincent F. and Walker, Kevin W. and Simpson, Lance and Fernandez, Avery M. and Patel, Vishank D. and Ramig, Anastasia and Gomes, Cyrus and Moen, Michael T. and Nguyen, Adam M.},
        month = oct,
        year = {2023},
}
```

## Archived Recipes

We have decided to no longer maintain the Matlab, Mathematica, and C recipes and have removed them from the Scholarly API Cookbook. These archived recipes are still in the [UA Libraries Scholarly API Cookbook Archive](https://github.com/UA-Libraries-Research-Data-Services/Scholarly_API_Cookbook_Archive).

<!-- TODO: ADD MATLAB, MATHEMATICA, C ARCHIVE REUSE STUFF HERE -->
