# Handling Dynamic Content (Playwright Decision)

Many modern job boards rely heavily on JavaScript to load content dynamically, render UI components, handle user interactions, and implement infinite scrolling.

## Challenges with Dynamic Content

*   **Data Not in Initial HTML:** Job listings, pagination controls, or filter results might not be present in the initial HTML source returned by a simple HTTP request.
*   **Client-Side Rendering:** Frameworks like React, Vue, Angular render content in the user's browser using JavaScript.
*   **User Interaction Required:** Content might only load after clicking buttons ("Load More"), scrolling down ("Infinite Scroll"), or interacting with forms.

## Technology Choice: Playwright

*   **Rationale:** When dynamic content loading or complex user interaction is necessary, a browser automation tool is required. Playwright is chosen for its modern API, good performance relative to other automation tools, and robust feature set (network interception, multiple browser support).
*   **Integration:** Playwright will be executed within the Celery workers for specific tasks identified as needing browser automation. The `scrapy-playwright` library could be leveraged if Scrapy were the primary driver, but in a direct Celery task, the `playwright` library would be used directly.

## Implementation Strategy

*   **Selective Use:** Playwright will **only** be used for websites or specific scraping stages explicitly configured as requiring it due to the high resource cost (CPU/RAM).
*   **Task Logic:** Celery tasks designated for Playwright usage will:
    1.  Launch a browser instance (e.g., Chromium).
    2.  Navigate to the target URL.
    3.  Wait for specific elements or network conditions indicating content has loaded (using Playwright's built-in waiting mechanisms).
    4.  Perform necessary interactions (clicking, scrolling).
    5.  Extract the fully rendered page source (`page.content()`) or specific data points directly using Playwright's locators.
    6.  Close the browser context/page gracefully to free up resources.
*   **Resource Management:** Worker concurrency and server resources must be carefully managed when running Playwright tasks. Kubernetes resource requests/limits and autoscaling based on load are crucial.

## Alternatives Considered

*   **Selenium:** Similar capabilities but often considered slightly less modern API-wise. Suffers from the same resource intensity.
*   **Splash:** A headless browser rendering service. Shifts resource load to a central Splash instance but adds operational complexity (managing Splash).
*   **Commercial Browser APIs:** Services like ScrapingBee, Browserless.io offer browser rendering as a service. Offloads resource management entirely but incurs direct costs per request/subscription.
*   **API Reverse Engineering:** Always the **preferred** alternative. If the underlying data can be accessed via an API call discovered through network analysis, Playwright should be avoided.

Playwright provides the necessary power for dynamic sites but will be employed judiciously due to its performance implications.