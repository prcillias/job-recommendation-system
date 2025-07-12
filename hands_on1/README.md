## 🌐 Data Source
Loker.id is one of Indonesia’s online job portals, connecting job seekers with companies across various industries and job categories. The platform features thousands of job vacancies with detailed information, including job titles, locations, company names, requirements, and employment types. In this project scrape structured job listing data from [Loker.id](https://www.loker.id/cari-lowongan-kerja) to collect comprehensive information from both the summary listings and the detailed job pages which can later be used for analysis and recommendation systems.

## 📂 Data Description
The data was extracted from 110 pages, resulting in approximately 2,200+ rows. The following fields were extracted:
| **Column**     | **Description** |
|----------------|------------------|
| `Job ID`       | Unique job identifier |
| `Title`        | Job title or position name |
| `Company`      | Name of the company posting the job |
| `Location`     | City or area where the job is based |
| `Salary`       | Salary range information, if available |
| `Category`     | Job main category and sub-category |
| `Posted Date`  | Posting date of the job listing |
| `Link`         | Direct link to the job’s detail page |
| `Role`         | Job role or function |
| `Type`         | Type of employment |
| `Education`    | Minimum education required |
| `Level`        | Job seniority level |
| `Description`  | Full job description from the detail page |

## 🧲 Scraping Process
### 🔧 Tools
- **Selenium** was used to automate browser actions because Loker.id displays job listings across multiple pages. To access all listings, the scraper needed to repeatedly click the next button to move through 110 paginated pages. Selenium also handled opening each job’s detail page, which loads additional content dynamically.
- **BeautifulSoup** was used to parse the HTML content from each loaded page and extract structured data such as job titles, companies, locations, salaries, and job descriptions.
### 🔄 Workflow
1. **Initialize Web Driver**: Launches Chrome in headless mode using Selenium.  
2. **Open Main Listings Page**: Navigates to the Loker.id job search page.  
3. **Handle Pagination**: Selenium clicks the next button to go through 110 pages of job listings.  
4. **Extract Summary Information**: Job title, company, location, salary, and post date are scraped from each listing. Fields that contain multiple values were separated using a semicolon (`;`)
5. **Open Job Detail Page**: Each listing is opened to extract job function, employment type, education, job level, and description.  
6. **Repeat for All Listings**: The scraper goes back to the main page and continues until all jobs are processed.  
7. **Store and Export Data**: The data is saved to a DataFrame and exported to a CSV file.  
8. **Close Browser**: The browser session is closed after scraping is complete.

## 🧹 Preprocessing
1. **Standardized Posted Date Format**:  
   Posting dates were formatted consistently for analysis.  
   Example: `7/9/2025 17:01` → `2025-07-09 17:01:00`

2. **Duplicate Handling**:  
   Duplicates were checked based on `Job ID` and some entries were found to have the same job posting but with different posted dates or descriptions.  
   - If duplicates had different posted dates, the most recent one was retained.  
   - If the only differences were in description formatting (spacing, alignment), one version was kept to avoid redundancy.

3. **Value Formatting and Standardization**:  
   Cleaned inconsistent formatting and spacing in key columns:
   - **Company Name**: Standardized capitalization for prefixes like "PT", "CV", and removed unnecessary punctuation (e.g., changed "Pt." to "PT").
   - **Type** and **Education**: Unified case style and spacing for consistency.

4. **Salary Parsing & Negotiation Flag**:  
   A new `Salary Negotiation` column was created:
   - If the salary field was "Negosiasi", both Min/Max salary were set to `null`, and `Salary Negotiation` was marked as `1`.
   - If the salary had a numeric value, it was cleaned and split into `Min Salary` and `Max Salary`.
   
   **Examples**:  
   - `Negosiasi` → Min: `<NA>`, Max: `<NA>`, Negotiation: `1`  
   - `Rp 3 – 4 Juta` → Min: `3000000`, Max: `4000000`, Negotiation: `0`  
   - `Rp 20 Juta Lebih` → Min: `20000000`, Max: `<NA>`, Negotiation: `0`

5. **Category Field Decomposition**:  
   The `Category` field contained mixed information like job type, education, experience, level, and category itself. It was split into structured columns:
   - **Min/Max Experience**: Extracted from phrases like `1-2 Tahun`, `3-5 Tahun`.
     - Example: `1-2 Tahun` → Min: `1`
     - `3-5 Tahun` → Max: `5`
   - **Fresh Graduate**: Binary flag (1/0) indicating whether the job accepts fresh graduates, extracted from `Category` or `Description`.
   - **Subcategory**: Extracted as a specialization within the main `Category`.

   **Examples**:
   - Raw: `Full Time;SMA / SMK / STM;1-2 Tahun;Marketing / Pemasaran;Content Marketing`  
     → Category: `Pemasaran`, Subcategory: `Content Marketing`  
   - Raw: `Kontrak;SMA / SMK / STM;Fresh Graduate;IT / Information Technology;Telekomunikasi`  
     → Category: `IT / Information Technology`, Subcategory: `Telekomunikasi`

6. **Education Binary Columns**:  
   Converted the `Education` field into multiple binary columns:
   - `SMA/SMK/STM`, `Diploma/D1/D2/D3`, `Sarjana/S1`, `Master/S2`, `Doctor/S3`
   - Example: If a listing included "SMA/SMK/STM;Diploma", those two columns were marked `1`, the rest `0`.

7. **Skills Extraction from Description**:
   - Cleaned bullet points and unnecessary symbols.
   - Formatted skill phrases into readable sentences.
   - Translated skills to English for consistency.
   - Applied stopwords removal and extracted noun chunks using **spaCy** for skill identification.
   - Cleaned `Skills` column.

### 🧽 Example Row: Before Preprocessing
| Field | Value |
|-------|-------|
| Job ID | 15758013 |
| Title | Staff Engineering |
| Company | Rusunami CityPark |
| Location | Jakarta Barat |
| Salary | Rp 3 – 4 Juta |
| Category | Kontrak;SMA / SMK / STM;1-2 Tahun;Staff / Officer;Teknik;Teknik Elektro / Elektronika |
| Posted Date | 7/9/2025 17:01 |
| Link | https://www.loker.id/teknik/elektro/staff-engineering-rusunami-citypark-jakarta-barat.html |
| Role | Teknisi Elektro |
| Type | Kontrak |
| Education | SMA / SMK / STM |
| Level | Staff / Officer |
| Description | (Long job description text, includes bullet points, symbols, skills, etc.) |

---

### 🧼 Example Row: After Preprocessing
| Field | Value |
|-------|-------|
| Job ID | 15758013 |
| Title | Staff Engineering |
| Company | Rusunami Citypark |
| Location | Jakarta Barat |
| Category | Teknik |
| Subcategory | Teknik Elektro / Elektronika |
| Role | Teknisi Elektro |
| Type | Contract |
| Level | Staff / Officer |
| Min Salary | 3000000 |
| Max Salary | 4000000 |
| Salary Negotiation | 0 |
| Fresh Graduate | 0 |
| Min Experience | 1 |
| Max Experience | 2 |
| SMA/SMK/STM | 1 |
| Diploma/D1/D2/D3 | 0 |
| Sarjana/S1 | 0 |
| Master/S2 | 0 |
| Doctor/S3 | 0 |
| Posted Date | 2025-07-09 17:01:00 |
| Description | (Cleaned and preserved full job description) |
| Skills | ['Plumbing', 'Troubleshooting', 'Mechanical Skiing'] |
| Link | https://www.loker.id/teknik/elektro/staff-engineering-rusunami-citypark-jakarta-barat.html |
  
## ✅ Final Columns:

| **Column**             | **Description** |
|------------------------|-----------------|
| `Job ID`               | Unique identifier for each job listing. |
| `Title`                | Job title or position name. |
| `Company`              | Name of the company posting the job. |
| `Location`             | City or area where the job is based. |
| `Min Salary`           | Minimum salary offered (numeric, in Indonesian Rupiah) if available. |
| `Max Salary`           | Maximum salary offered (numeric, in Indonesian Rupiah) if available. |
| `Salary Negotiation`   | Binary column (`1` if the salary is negotiable, `0` otherwise). |
| `Category`             | Main job category. |
| `Subcategory`          | More specific subcategory or specialization. |
| `Posted Date`          | Date time when the job was posted. |
| `Link`                 | Direct URL to the job’s detail page on Loker.id. |
| `Role`                 | Job role or function, usually found in the detailed job description. |
| `Type`                 | Type of employment, e.g., Full Time, Contract, Internship. |
| `Fresh Graduate`       | Binary indicator (`1` if fresh graduates are accepted, `0` otherwise). |
| `Min Experience`       | Minimum number of years of experience required. |
| `Max Experience`       | Maximum number of years of experience required. |
| `Level`                | Job seniority level. |
| `Description`          | Job description. |
| `Doctor/S3`            | Binary column: `1` if the job requires a doctoral degree (S3), `0` otherwise. |
| `Master/S2`            | Binary column: `1` if the job requires a master's degree (S2), `0` otherwise. |
| `Sarjana/S1`           | Binary column: `1` if the job requires a bachelor's degree (S1), `0` otherwise. |
| `Diploma/D1/D2/D3`     | Binary column: `1` if the job requires a diploma degree (D1/D2/D3), `0` otherwise. |
| `SMA/SMK/STM`          | Binary column: `1` if the job requires a high school degree, `0` otherwise. |
| `Skills`               | List of extracted skill keywords from the job description. |

## 🚧 Challenges
One of the main challenges encountered during the data scraping process was the risk of being temporarily blocked due to sending repeated requests in a short period. This often occurred when testing and debugging the scraping script multiple times, which unintentionally mimicked bot-like behavior. As a result, the website began restricting access, slowing down the development process. To resolve this, delays using time.sleep() were added between each request, and a more efficient scraping logic was implemented to reduce the number of unnecessary retries. Similarly, the translation step, which involved converting skill descriptions into English using a third-party library also presented several issues. Translating a large number of rows triggered rate limits from the translation service, especially when processing data in rapid succession. This not only disrupted the pipeline but also caused incomplete or failed translations. To mitigate this, a delay was introduced between translations, and the process was executed in batches to ensure compliance with the service's usage policies while maintaining performance and stability. Although both scraping and translating added significant time to the overall pipeline, these adjustments ensured smoother execution and more stable results.
