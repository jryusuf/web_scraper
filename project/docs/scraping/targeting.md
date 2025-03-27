# Targeting Strategy (Keywords, Filters)

To maximize efficiency, reduce costs, and minimize load on target websites, scraping will be highly targeted rather than attempting full site crawls.

## Configuration-Driven Targeting

The core strategy relies on the configuration stored in the central database (managed via Django Admin):

*   **Target Entities:** Define specific combinations of:
    *   `Websites` (e.g., LinkedIn, Indeed, specific career page)
    *   `Keywords` (e.g., "Data Engineer", "Python", "React Developer")
    *   `Locations` (e.g., "London", "New York", "Remote")
*   **Search Execution:** The dispatcher service constructs specific search queries or navigates filter forms on the target website based on these configured combinations.

## Utilizing Website Filters

Whenever possible, the scraper will leverage built-in filtering mechanisms provided by the target websites.

### Date Filters

*   **Priority:** This is a crucial filter for managing incremental scraping and freshness.
*   **Implementation:** If a site allows filtering by "posted within last 24 hours," "posted last week," or similar, the scraper will be configured to use these options. The specific URL parameter or form interaction method for activating this filter will be stored in the website's configuration.
*   **Goal:** Primarily collect *new* or *recently updated* job listings.

### Other Filters

*   Keywords, location, job type (Full-time/Part-time), remote status filters provided by the site will be used directly during the search phase (Stage 1 scraping) as configured.

## Benefits of Targeting

*   **Reduced Load:** Significantly fewer pages are requested from target servers compared to broad crawling.
*   **Cost Savings:** Lower usage of bandwidth, proxies, and compute resources (especially when using browser automation).
*   **Data Relevance:** Focuses collection efforts on the most relevant job postings based on defined criteria.
*   **Scalability:** Allows adding more specific targets without linearly increasing the load for *existing* targets.