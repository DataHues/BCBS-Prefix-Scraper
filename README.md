# BCBS Prefix Scraper Report

## 1. Introduction
This project ensures a dependable source for Blue Cross Blue Shield (BCBS) prefixes and insurer names. The official BCBS Prefix List website is often cluttered with ads and loads slowly, making it unreliable for quick reference. To provide consistent, ad-free access to this information, the scraper:

1. Visits every alphabetical range (A–Z) of BCBS prefixes.  
2. Handles pagination on each subpage.  
3. Outputs a consolidated CSV file that can be referenced internally without relying on the site.

## 2. Data Source
- **Website**: https://mypayerdirectory.com/bcbs-prefix-list/  
- **Content**:  
  - A series of subpages, each covering a three-letter prefix range (e.g., `faa-to-fzz`, `gaa-to-gzz`, `taa-to-tzz`).  
  - Each subpage has a paginated HTML table with two columns:
    1. **Prefix** (BCBS code)  
    2. **Name** (insurer name)

## 3. Methods

1. **Launch Headless Browser**  
   - Used Playwright’s async API to open a headless Chromium instance.

2. **Locate Alphabetical Subpages**  
   - Navigated to `/bcbs-prefix-list/`.  
   - Waited for JavaScript to inject all A–Z links (any `<a>` whose `href` contains `"-to-"`).  
   - Collected every link containing `/bcbs-prefix-list/` and filtered for `"-to-"`, deduped, and sorted alphabetically.

3. **Scrape Each Subpage**  
   - For each A–Z URL:  
     1. Loaded the page and waited for the table to render.  
     2. Queried all `<tr>` rows in `<table><tbody>`.  
     3. Extracted text from the first two `<td>` cells (`td:nth-child(1)` for Prefix; `td:nth-child(2)` for Name).  
     4. Appended each nonempty `[prefix, name]` pair to a list.  
     5. Checked for a “Next” button (`a.paginate_button.next`). If present and not disabled, clicked it and repeated.

4. **Close Browser & Export**  
   - Closed the browser once all subpages and pagination pages were visited.  
   - Converted the collected list into a pandas DataFrame with columns `["Prefix", "Name"]`.  
   - Wrote the DataFrame to a CSV file.

5. **Post-Processing (Sorting)**  
   - Loaded the CSV in pandas, sorted by the “Prefix” column, and saved the sorted version so prefixes appear in true A–Z order.

## 4. Results

- **Total Prefix-Name Pairs Scraped**: 18,801  
- **Output File**: [BCBS_Data.csv](./BCBS-Prefix-Scraper/BCBS_Data.csv) 
  - Two columns: `Prefix` (BCBS code) and `Name` (insurer name).  
  - Alphabetically sorted from A to Z.
- **Repository (Code)**: [BCBS_Code.ipynb](./BCBS-Prefix-Scraper/BCBS_Code.ipynb)

## 5. Discussion

- **Coverage**  
  All alphabetical subpages (A–Z) were discovered and scraped, including dynamically injected links. Pagination on each page was handled to ensure no rows were missed.

- **Reliability**  
  The public Blue Cross Blue Shield Prefix List often experiences downtime. Wrapping navigation and selectors in `try/except` blocks prevented a single failure from aborting the entire process.

- **Challenges**  
  1. **Dynamic Content**: Alphabetical links load via JavaScript, requiring an explicit wait before scraping.  
  2. **Pagination Variation**: Some subpages had multiple pages, so detecting and clicking the “Next” button until it became disabled was necessary.  
  3. **Selector Changes**: If the site’s HTML structure changes (e.g., new CSS classes), the scraper’s selectors must be updated.

## 6. Evaluation and Conclusion

- **Key Learnings**  
  - Playwright’s async API can reliably scrape JavaScript-driven sites.  
  - Robust error handling is essential for unstable web sources.

- **Next Steps**  
  - Automate scraping on a regular schedule to keep the CSV up to date.  
  - Monitor for site changes and update selectors as needed.  
  - Consider adding incremental update logic to detect new prefixes without re-scraping everything.

## 7. References

- BCBS Prefix List website (https://mypayerdirectory.com/bcbs-prefix-list/)
- Playwrite Documentation ([https://playwright.dev/docs/intro](https://playwright.dev/python/docs/api/class-playwright))


