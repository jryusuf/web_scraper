# Configuration Management (Django)

Effective management of scraping targets, keywords, and settings is crucial for controlling the system's behavior and adapting to new requirements.

## Role & Purpose

*   Acts as the central repository for all operational parameters guiding the scraping process.
*   Provides an interface for users/administrators to define *what* to scrape, *where* to scrape it from, and *how*.
*   Stores metadata about target websites needed for successful scraping (e.g., requires Playwright, specific selectors, date filter parameters).

## Technology Choice: Django + PostgreSQL

*   **Django Framework:** Chosen primarily for its outstanding **built-in Admin Interface**. This automatically generates a web UI for managing database models, drastically reducing the effort needed to build a configuration tool.
*   **PostgreSQL Database:** Serves as the backend database storing the configuration data. Chosen for reliability, robustness, and compatibility with Django.

## Key Data Models (Conceptual)

The Django application would define models roughly corresponding to:

### `Website` Model
*   `name`: Human-readable name (e.g., "LinkedIn Jobs").
*   `base_url`: Base URL of the site.
*   `search_url_template`: URL structure for performing searches (with placeholders for keywords, location, etc.).
*   `robots_txt_status`: (e.g., Allowed, Disallowed, Not Checked).
*   `requires_playwright`: Boolean flag indicating if browser automation is needed.
*   `pagination_type`: (e.g., 'next_link', 'load_more_button', 'infinite_scroll', 'api').
*   `date_filter_param`: Specific URL parameter or interaction logic for date filtering.
*   `selectors`: (Potentially JSON/TextField) Storing CSS/XPath selectors for key elements (job links, title, description - though parsing logic might be separate).
*   `is_active`: Boolean flag to easily enable/disable scraping for this site.
*   `base_delay`: Default download delay for this site.
*   ... other site-specific metadata.

### `Keyword` Model
*   `text`: The keyword/phrase to search for (e.g., "Data Engineer").
*   `type`: (Optional) Category like 'Role', 'Skill', 'Tool'.

### `Location` Model
*   `text`: The location to search within (e.g., "London, UK", "Remote").

### `ScrapeTarget` Model (Linking Table)
*   `website`: Foreign Key to `Website`.
*   `keyword`: Foreign Key to `Keyword`.
*   `location`: Foreign Key to `Location`.
*   `is_active`: Boolean flag for this specific combination.
*   `frequency`: How often to schedule this target (e.g., 'daily', 'hourly').
*   `last_scheduled`: Timestamp of the last dispatch.

## Benefits of this Approach

*   **Centralized Control:** All configuration in one place.
*   **User-Friendly Management:** Easy updates via the Django Admin UI without direct database access or code changes for simple config updates.
*   **Structured & Relational:** Database enforces relationships (e.g., which keywords apply to which sites).
*   **Version Control:** Django migrations allow schema changes to be version controlled.
*   **Programmatic Access:** The Dispatcher service can easily query these models using Django's ORM.