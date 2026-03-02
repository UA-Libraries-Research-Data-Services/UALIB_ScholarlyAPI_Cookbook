# Contributing

## How to Contribute

The UA Libraries Scholarly API Cookbook is an open source resource. Contributions from students, faculty, staff, and the broader community are welcome.

### Reporting Errors or Suggesting Improvements

If you find an error, outdated code, broken link, or unclear explanation, please open an issue in the [GitHub Issues](https://github.com/UA-Libraries-Research-Data-Services/UALIB_ScholarlyAPI_Cookbook/issues).

When creating an issue, it is helpful to include:

- A link to the specific tutorial page
- A description of the issue
- Any relevant error messages
- Suggested corrections (if available)

We review issues regularly and will respond as soon as possible.

### Requesting New APIs or Tutorials

If you would like to see support for a specific scholarly API, please open a GitHub issue describing:

- The name of the API
- A link to its official documentation
- Why it would be useful for research or teaching
- Whether the API is open access or requires institutional credentials

### Submitting Code Contributions

Before submitting a pull request, please open an issue to discuss proposed changes. This helps ensure alignment with project goals and avoids duplicated work.

For tutorial contributions:

- Follow existing Python or R tutorial structure and formatting, particularly as it relates to licensing, external packages, and API authentication.
- Ensure all code runs without errors.
- Include clear explanations and comments.
- Avoid hard-coded credentials or private API keys.

All contributions are reviewed for accuracy, clarity, reproducibility, and consistency with project standards.

## Project Scope

As of 2025, the Cookbook focuses on maintaining and standardizing Python and R tutorials. Other language have been archived and are no longer actively maintained.

### Project Infrastructure

The Scholarly API Cookbook is built using the following tools:

- Python tutorials are written in [Jupyter Notebooks](https://jupyter.org/).
- R tutorials are written in RMarkdown and exported to Markdown.
- Narrative content is written in [reStructuredText](https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html).
- The site is compiled using [Jupyter Book](https://jupyterbook.org/).
- An automated GitHub workflow builds the site and deploys it using [GitHub Actions](https://docs.github.com/en/actions).

Contributors should ensure that all notebooks and Markdown files build successfully before submitting changes.

## Questions

If you are unsure whether an idea fits the project scope, please open an issue to start a discussion.
