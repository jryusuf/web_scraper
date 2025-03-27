# Handling Pagination & Infinite Scroll

Job listings are often spread across multiple pages or loaded dynamically.

## Types of Pagination & Strategies

### 1. Next Page Links / Page Number Links

*   **Mechanism:** Standard HTML links (`<a>` tags) point to the next page or specific page numbers. URLs often change predictably (e.g., `?page=2`, `?p=3`).
*   **Strategy:**
    1.  Parse the current page to find the link to the next page (using CSS selectors or XPath).
    2.  If found, generate a request for the next page's URL.
    3.  Repeat until no "next page" link is found or a predefined page limit is reached.
    4.  This can be handled within a single worker task, looping through pages sequentially or yielding requests if using a Scrapy-like internal flow.

### 2. JavaScript-Driven Pagination ("Load More" Buttons)

*   **Mechanism:** Clicking a button executes JavaScript that fetches and inserts more results onto the current page, or triggers a background API call.
*   **Strategy:**
    *   **Option A (Browser Automation - Playwright):** Use Playwright to simulate clicking the "Load More" button repeatedly, waiting for new content to appear after each click, until the button disappears or no new content loads. Extract data from the fully loaded page.
    *   **Option B (API Reverse Engineering):** Monitor network traffic when clicking the button. Identify the underlying API request. If found, call this API directly (likely passing page numbers or cursors as parameters) using the lightweight `requests` library. This is strongly preferred if possible.

### 3. Infinite Scroll

*   **Mechanism:** New content automatically loads as the user scrolls down the page, often triggering background API calls.
*   **Strategy:**
    *   **Option A (Browser Automation - Playwright):** Use Playwright to repeatedly scroll down the page (`page.evaluate("window.scrollTo(0, document.body.scrollHeight)")`), waiting for new content elements or network activity to cease after each scroll, until no new content appears. Extract data from the fully loaded page.
    *   **Option B (API Reverse Engineering):** Monitor network traffic during scrolling. Identify the API calls fetching batches of results. Replicate these API calls directly using `requests`, passing appropriate parameters (page number, cursor, offset). This is strongly preferred if possible.

## Implementation Notes

*   The specific pagination method needs to be identified during initial website analysis and stored in the site's configuration.
*   The worker task logic will branch based on the configured pagination type for the target site.
*   Care must be taken to detect the end condition (no more pages/content) to avoid infinite loops.