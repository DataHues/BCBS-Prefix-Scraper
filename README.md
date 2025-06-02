# Blue-Cross-Blue-Shield-Scraper
# 1. Introduction
The BCBS Prefix Scraper is a Python script built with Playwright (async API) and pandas. Its purpose is to collect every Blue Cross Blue Shield (BCBS) prefix and the corresponding insurer name from the BCBS Prefix List website. By scraping all alphabetical subpages (A–Z) and handling pagination, it generates a single, consolidated CSV file. This eliminates the need for manual lookups when the public site is unreliable.

## 2. Data Source
- **Website**: https://mypayerdirectory.com/bcbs-prefix-list/  
- **Content**:  
  - A series of subpages, each covering a three-letter prefix range (e.g., “faa-to-fzz,” “gaa-to-gzz,” “taa-to-tzz,” etc.).  
  - Each subpage contains a paginated HTML table with two columns:  
    1. **Prefix** (three-letter BCBS code)  
    2. **Name** (insurer name)

## 3. Methodology

1. **Launch Headless Browser**  
   - Uses Playwright’s async API to open a headless Chromium instance.

2. **Locate Alphabetical Subpages**  
   - Navigate to `/bcbs-prefix-list/`.  
   - Wait for JavaScript to inject all “*-to-*” links that correspond to A–Z ranges.  
   - Collect every link containing `/bcbs-prefix-list/` and `"-to-"`, dedupe, and sort alphabetically.

3. **Scrape Each Subpage**  
   - For each A–Z URL:  
     1. Load the page and wait for the table to appear.  
     2. Iterate through all `<tr>` rows in the table body.  
     3. For each row, extract the first two `<td>` cells (`td:nth-child(1)` for Prefix; `td:nth-child(2)` for Name).  
     4. Append nonempty `[prefix, name]` pairs to an in-memory list.  
     5. Check for a “Next” button (`a.paginate_button.next`); if present and not disabled, click it and repeat.

4. **Close Browser & Export**  
   - After visiting all subpages, close Chromium.  
   - Convert the collected list into a pandas DataFrame with columns `["Prefix", "Name"]`.  
   - Write the DataFrame to `BCBS_Prefix_Data.csv` (UTF-8, no index).

5. **Post-Processing (Sorting)**  
   - Load `BCBS_Prefix_Data.csv` in pandas, sort by the “Prefix” column, and save as `BCBS_Prefix_Data_Cleaned.csv` (or overwrite the original file).

## 4. Challenges Faced

1. **Dynamic Link Injection**  
   - The site loads some alphabetical subpage links via JavaScript after the initial HTML render.  
   - Initial attempts using a static CSS selector (e.g., `a[href*="bcbs-prefixes"]`) missed several ranges.  
   - Solution: Wait for any `a[href*="/bcbs-prefix-list/"][href*="-to-"]` element to appear, then use `eval_on_selector_all` to gather all links whose `href` contains `"-to-"`.

2. **Pagination Variability**  
   - Different subpages had varying numbers of pagination pages (some single-page, some multi-page).  
   - The “Next” button text and disabled state could change depending on CSS classes.  
   - Solution: In each loop, check for `a.paginate_button.next`; break if it’s missing or has `"disabled"` in its class.

3. **Site Unreliability**  
   - The public BCBS Prefix List occasionally experiences downtime or partial content loading.  
   - Relying on a single HTTP GET could yield incomplete data.  
   - Solution: Wrap every navigation and selector call in `try/except` blocks. Log errors and continue scraping remaining subpages rather than aborting completely.

4. **Selector Robustness**  
   - If BCBS modified their table structure (e.g., adding extra columns or changing class names), the scraper would break.  
   - Solution: Use `td:nth-child(1)` and `td:nth-child(2)` rather than relying on specific CSS classes like `column-1` or `column-2`.

---

## 5. Usage Instructions

1. **Clone the Repository**  
   ```bash
   git clone https://github.com/your-username/BCBS-Prefix-Scraper.git
   cd BCBS-Prefix-Scraper
2. **Set Up Environment**
   python -m venv venv
   source venv/bin/activate    # macOS/Linux
   venv\Scripts\activate       # Windows
   pip install -r requirements.txt
   playwright install

