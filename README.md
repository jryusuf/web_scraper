# Job Scraper System - Design Document

[![Documentation Status](https://img.shields.io/badge/docs-deployed-brightgreen)](https://jryusuf.github.io/web_scraper/) 

## Overview

This repository contains the design documentation for a scalable web scraping system focused on collecting publicly available job listing data. This project was undertaken as a response to a data engineering challenge.

The primary output is a detailed design document, built using MkDocs, outlining the proposed architecture, technology choices, data flow, operational considerations, and justifications.

## 📖 Accessing the Documentation

**The full, browseable design document is deployed via GitHub Pages and can be accessed here:**

➡️ **[https://jryusuf.github.io/web_scraper/](https://jryusuf.github.io/web_scraper/)** ⬅️


Navigate the documentation using the sidebar on the left, which follows the structure defined in the `mkdocs.yml` file.

## Project Goal

The designed system aims to:

*   Collect job postings from multiple job boards and company career pages.
*   Target specific roles, sectors, and locations based on configuration.
*   Handle common scraping challenges (dynamic content, pagination, rate limits, anti-scraping).
*   Process and store data in a structured format suitable for analysis and potential ML applications.
*   Be scalable, reliable, and maintainable.

## Documentation Stack

This documentation site itself is built using:

*   **MkDocs:** Static site generator for project documentation.
*   **MkDocs Material Theme:** For enhanced visuals and features.
*   **mkdocs-mermaid2-plugin:** For rendering embedded Mermaid diagrams.

## Running Locally

To build and view the documentation site locally:

1.  **Prerequisites:** Ensure you have Python 3.x and `pip` installed.
2.  **Clone Repository:**
    ```bash
    git clone https://github.com/jryusuf/web_scraper.git
    cd web_scraper
    ```
3.  **Set up Virtual Environment (Recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```
4.  **Install Dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

    ```bash
    mkdocs serve
    ```
6.  **View:** Open your web browser and navigate to `http://127.0.0.1:8000`. The site will automatically reload when you save changes to the Markdown files or `mkdocs.yml`.

## Deployment

This site is automatically deployed to GitHub Pages, typically via the `mkdocs gh-deploy` command or a GitHub Actions workflow configured to build and deploy the contents of the `docs/` directory on pushes to the main branch.