# Website Identification & robots.txt

Identifying relevant sources is the first step in the data collection process.

## Methods for Identification

A combination of methods will be used to discover potential sources for job listings:

*   **Search Engines:** Utilizing targeted search queries (e.g., "[Industry] job board", "jobs in [City] [Sector]", "careers at [Company Name]").
*   **Industry Directories & Lists:** Consulting known lists of major job boards and industry-specific career sites.
*   **Competitor Analysis:** Identifying where competitors or companies in target sectors post their jobs.
*   **Manual Curation:** Adding specific company career pages or niche job boards based on project requirements.

## Initial Vetting

Before adding a website to the scraping configuration, a brief manual review is necessary to:

*   Assess the relevance and volume of job listings.
*   Understand the basic site structure (static vs. dynamic, pagination type).
*   Check for clear terms of service regarding automated access.
*   **Crucially: Check the `robots.txt` file.**

## Respecting `robots.txt`

*   **Process:** Before any scraping attempt on a new domain, the system (or the initial setup process) **must** fetch and parse the `robots.txt` file located at the root of the domain (e.g., `https://www.example.com/robots.txt`).
*   **Adherence:** The rules defined in `robots.txt` (specifically `User-agent` and `Disallow` directives) will be strictly adhered to. If `robots.txt` disallows scraping specific paths relevant to job listings for user agents matching our scraper's identity, those paths will **not** be scraped.
*   **Configuration:** The configuration database might include a flag indicating if scraping is permitted based on `robots.txt`.
*   **Crawl-Delay:** If a `Crawl-delay` directive is present, it will be respected as a minimum delay between requests, potentially overriding default politeness settings if it's stricter.

Only websites deemed relevant, technically feasible to scrape (considering `robots.txt`), and aligned with project goals will be added to the configuration database for active scraping.