# CodeAlpha Task 1: Web Scraping | Kenyan E-commerce Dataset

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/evanskips2727-del/CodeAlpha_WebScraping/blob/main/Task1_Web_Scraping.ipynb)
![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![License](https://img.shields.io/badge/Code%20license-MIT-lightgrey)

An ethical, reproducible scraping pipeline that turns a Kenyan electronics retailer's public catalogue into a clean, analysis-ready dataset of **1,293 products** across **5 categories** and **65 brands**.

Built for the **CodeAlpha Data Analytics Internship** (Task 1). The same dataset powers Task 2 (EDA) and Task 3 (Visualization).

---

## Why this project is different

| Practice | What I did |
|---|---|
| **Source vetting** | Tested 9 Kenyan sites before writing a scraper, and rejected the ones that blocked bots or disallowed scraping |
| **Compliance** | Checked `robots.txt` first, and the scraper re-checks every URL against it at run time |
| **Politeness** | 1.5-2.5 s random delay, retries with back-off, one reused session, public pages only |
| **Robust extraction** | Reads the structured JSON the site embeds in each page, instead of fragile HTML selectors |
| **Data quality** | Documented every cleaning decision, validated the output, and listed the limitations openly |

## Source selection

| Site | Result | Decision |
|---|---|---|
| Jumia Kenya | Cloudflare challenge (403) | Excluded: access is deliberately restricted |
| Kilimall | `robots.txt` disallows `/product*`, `/item*`, `/api/*` | Excluded: the data pages are off-limits |
| Jiji, Carrefour, Naivas, PigiaMe, Phone Place | HTTP 403 | Excluded |
| Gadgets Kenya | Manufacturer's company site, not a shop | Excluded |
| **Masoko** | `robots.txt` allows all crawling (checked 11 Oct 2026) | **Selected** |

## Method

1. **Discover:** each Masoko category page embeds its product list as JSON in a Next.js `__NEXT_DATA__` script tag.
2. **Extract:** `requests` and `BeautifulSoup` locate the tag, and `json` parses it.
3. **Paginate:** follow the site's own `nextPage` value until the last page of each category.
4. **Deduplicate:** the same product can appear in several categories, so rows are deduplicated by product `id` (1,563 raw rows became 1,293 unique products).
5. **Clean and engineer features:** see below.
6. **Validate:** automated checks on IDs, prices and required fields before saving.

### Cleaning and feature engineering

- Prices converted from text to numeric
- New fields: `final_price`, `discount_kes`, `discount_pct`, `has_discount`
- Brand extracted from product names, with corrections for multi-word brands and Apple product lines
- RAM, storage and screen size extracted from product names with regex
- Gaming items labelled as **Hardware/Accessory**, **Video Game** or **Gift Card**, because game titles were being read as brands
- SKU decoded into a product-type code and a source/channel code
- Fields that were empty or constant for every row were dropped

## Dataset snapshot

Scraped on 11 Oct 2026.

| Metric | Value |
|---|---|
| Unique products | 1,293 |
| Brands | 65 |
| Categories | Accessories 735, Mobile phones 299, Gaming 132, Laptops 74, Apple devices 53 |
| Discounted listings | 24.7% |
| Discount depth | Median ~10%, mean ~15.8%, max 72.4% |
| Median final price | KES 17,600 (mean KES 65,904, so prices are strongly right-skewed) |
| Price range | KES 94 to KES 646,758 |
| Largest brands by listings | Apple (293), JBL (112), Samsung (67), Hama (55), Havit (51) |

Early observations to be tested properly in the EDA:

- The deepest discounts seen (63-72%) are on budget Vision Plus accessories.
- 37 product names are listed twice with different SKU channel codes and a median price gap of about 3%. The site does not explain the codes, so the cause is not claimed here.

## Data dictionary

| Column | Description |
|---|---|
| `id`, `sku`, `name`, `urlPath`, `thumbnail` | Identifiers and product text from the site |
| `price`, `specialPrice` | List price and discounted price in KES |
| `final_price` | Price a customer pays (special price when present) |
| `discount_kes`, `discount_pct`, `has_discount` | Engineered discount features |
| `brand` | Brand parsed from the product name |
| `product_type` | Hardware/Accessory, Video Game or Gift Card |
| `ram_gb`, `storage_gb`, `screen_in` | Specs parsed from the product name (blank when not applicable) |
| `category`, `category_group`, `page` | Where the product was found |
| `sku_type`, `sku_channel` | Codes decoded from the SKU prefix |
| `stock_qty` | Displayed stock quantity (indicative only, see limitations) |
| `has_campaign`, `campaign` | Promotion flag and raw campaign data |
| `price_anomaly` | Data-quality flag (no anomalies found) |
| `typeId`, `scraped_at` | Product type from the site and scrape timestamp |

## Repository structure

```
CodeAlpha_WebScraping/
├── Task1_Web_Scraping.ipynb      # full pipeline: robots check, scraper, cleaning, validation
├── data/
│   ├── masoko_products_raw.csv   # deduplicated scrape output
│   └── masoko_products_clean.csv # analysis-ready dataset
├── requirements.txt
├── LICENSE
└── README.md
```

## Run it yourself

**In Colab (easiest):** click the *Open in Colab* badge at the top and run the notebook from top to bottom. A full scrape takes about 8 minutes.

**Locally:**

```bash
git clone git clone https://github.com/evanskips2727-del/CodeAlpha_WebScraping.git
cd CodeAlpha_WebScraping
pip install -r requirements.txt
jupyter notebook Task1_Web_Scraping.ipynb
```

Re-running will produce a newer snapshot, so counts and prices will differ slightly from the figures above.

## Limitations

- Rating, review and vendor fields exist in the site's data structure but were **empty for every product**, so no review-based analysis is possible from this source.
- `stock_qty` appears capped (about 41% of rows show exactly 10). Treat it as indicative only.
- A product listed in several categories is kept under the first one scraped, so smaller categories such as Apple devices are under-counted.
- The `computing/routers` category returned no data and was skipped.
- Brand and spec extraction is rule-based on product names and may contain errors.
- The data is a single-day snapshot, so it cannot show price changes over time.

## Ethics and responsible use

- Only publicly accessible catalogue pages were requested. There was no login, no personal data and no circumvention of blocks.
- Request volume was kept low, with delays between requests.
- The data belongs to its original owner. It is shared here for **educational and portfolio purposes only**, and the MIT license covers the code, not the data.
- If the site owner would like the dataset removed, please open an issue and it will be taken down.

## Roadmap: CodeAlpha Data Analytics internship

- [x] **Task 1:** Web scraping (this repository)
- [ ] **Task 2:** Exploratory data analysis
- [ ] **Task 3:** Data visualization
- [ ] **Task 4:** Sentiment analysis

## Author

**Evans Kiplangat** | Data Analyst and Applied Statistician, Nairobi, Kenya

[GitHub](https://github.com/evans25575) | [LinkedIn](https://linkedin.com/in/evans-kiplangat-375646179) | [Portfolio](https://evans25575.github.io/Evans---portfolio-/)

*Completed as part of the [CodeAlpha](https://www.codealpha.tech) Data Analytics Internship.*
