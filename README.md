# LEGO Sets Analysis Dashboard

## 📊 Project Link
<a href="https://app.powerbi.com/view?r=eyJrIjoiNjlkZmZmNzEtMTQ1My00MDlkLTkwMTgtNTJhODI5Y2YwN2NlIiwidCI6IjJiYjZlNWJjLWMxMDktNDdmYi05NDMzLWMxYzZmNGZhMzNmZiIsImMiOjl9" target="_blank">
  <img src="https://upload.wikimedia.org/wikipedia/commons/c/cf/New_Power_BI_Logo.svg" width="100" height="100" alt="Power BI"/>
</a>

## Project Summary

This Power BI project provides a comprehensive analysis of LEGO sets spanning multiple years, themes, and price ranges. The dashboard enables users to explore trends in LEGO products by analyzing key metrics such as pricing, piece counts, target age ranges, and theme popularity. It offers interactive visuals that help identify patterns in product offerings, understand market segmentation by age and price, and make data-driven decisions for collectors, retailers, or market analysts interested in the LEGO ecosystem.

The project demonstrates advanced Power BI capabilities including DAX measure development, Power Query transformations, calculated tables, dynamic filtering, and interactive visual design to deliver actionable insights from raw CSV data.

---

## Problem Statement & Solution

### The Challenge
LEGO enthusiasts, retailers, and market analysts face difficulties in understanding the vast landscape of LEGO products without structured analytical tools. With thousands of sets released over the years across various themes, price points, and age ranges, it's challenging to identify trends, compare products effectively, or make informed purchasing and inventory decisions. Questions like "What's the average price by theme?", "Which age groups have the most product offerings?", or "How have LEGO sets evolved over time?" require manual data processing and analysis.

### The Solution
This Power BI report transforms raw LEGO set data into an interactive, visual analytics platform. Through careful data modeling, DAX calculations, and intuitive visualizations, users can instantly explore product distributions, compare metrics across dimensions (theme, year, age range, price), and drill down into individual set details. The data model supports dynamic filtering through calculated tables and measures, enabling price-based filtering and detailed product selection. The report empowers stakeholders to answer complex business questions through simple visual interactions, turning data into strategic insights.

---

## Questions & Answers

### 1. What data sources were used?

**Answer:** The project uses a single primary data source: a CSV file named `lego_sets.csv` containing comprehensive LEGO product information. The original dataset includes fields such as:
- Set identification and naming information
- Year of release
- Theme hierarchy (theme, subtheme, themeGroup, category)
- Product specifications (pieces, minimum age range, US retail price)
- Image URLs for visual representation

Additionally, the model includes:
- A calculated table (`Max Price`) generated using DAX's GENERATESERIES function
- A utility table (`LastRefreshTime`) that captures the last data refresh timestamp using Power Query

---

### 2. How is the data model structured?

**Answer:** The data model follows a simplified star schema with one main fact table and supporting utility tables:

**Tables:**
1. **lego_sets (Fact Table)**
   - 13 columns including dimensions and measures
   - 13 DAX measures for calculations
   - Import mode for optimal performance
   - Contains all product attributes and serves as the primary analytical table

2. **Max Price (Calculated Table)**
   - Single-column table generated using `GENERATESERIES(0, 850, 5)`
   - Creates a range from $0 to $850 in $5 increments
   - Used for dynamic price filtering through slicers
   - Connected to fact table through DAX measure relationships

3. **LastRefreshTime (Utility Table)**
   - Captures the timestamp of the last data refresh
   - Uses Power Query M expression: `DateTime.LocalNow()`
   - Provides data freshness visibility to report users

**Relationships:**
The model has no physical relationships defined. Instead, it uses a measure-based filtering approach where the `Max Price` measure in the lego_sets table references the `Max Price Value` from the Max Price table, creating a virtual relationship for dynamic filtering.

---

### 3. What are the main measures and KPIs?

**Answer:** The model includes 14 measures across two tables, categorized by purpose:

**Aggregate Measures (lego_sets table):**
- **Total Sets**: `DISTINCTCOUNT(lego_sets[set_id])` - Counts unique LEGO sets
- **Total Groups**: `DISTINCTCOUNT(lego_sets[themeGroup])` - Counts distinct theme groups
- **Avg. Age**: `AVERAGEX(lego_sets, lego_sets[agerange_min])` - Average minimum age recommendation
- **Avg. Price**: `AVERAGEX(lego_sets, lego_sets[US_retailPrice])` - Average retail price (formatted as currency)
- **Avg. Pieces**: `AVERAGEX(lego_sets, lego_sets[pieces])` - Average piece count per set

**Detail Selection Measures (for drill-down/card visuals):**
These measures use `HASONEVALUE()` to display specific values when a single item is selected:
- **Selected Name**: Returns set name or "Select Item" if multiple/none selected
- **Selected price**: Returns price or "-" placeholder
- **Selected year**: Extracts year from date field or returns "-"
- **Selected pieces**: Returns piece count or "-"
- **Selected age**: Returns minimum age or "-"

**Filtering Measures:**
- **Max Price** (lego_sets table): `IF([Avg. Price] <= 'Max Price'[Max Price Value], 1, 0)` - Binary filter flag for price-based filtering
- **Max Price Value** (Max Price table): `SELECTEDVALUE('Max Price'[Max Price])` - Captures the selected price threshold from slicer

**UI Enhancement Measures:**
- **Color tran**: `IF(HASONEVALUE(lego_sets[name]), "rgba(0,0,0,0)")` - Returns transparent color when an item is selected (for conditional formatting)
- **Placeholder text**: `"←" & UNICHAR(10) & "Select One Item"` - Creates a multi-line placeholder with arrow and text using line break character

---

### 4. What transformations were done in Power Query (M)?

**Answer:** The Power Query transformations prepare the raw CSV data for analysis through several cleaning and enrichment steps:

**lego_sets table transformations:**

```powerquery
let
    Source = Csv.Document(
        File.Contents("C:\Users\ahmedsamir\Desktop\lego_sets.csv"),
        [Delimiter=",", Columns=14, Encoding=65001, QuoteStyle=QuoteStyle.None]
    ),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Removed Columns" = Table.RemoveColumns(
        #"Promoted Headers",
        {"minifigs", "bricksetURL", "thumbnailURL"}
    ),
    #"Filtered Rows" = Table.SelectRows(
        #"Removed Columns",
        each [agerange_min] <> null and [agerange_min] <> ""
    ),
    #"Filtered Rows1" = Table.SelectRows(
        #"Filtered Rows",
        each [US_retailPrice] <> null and [US_retailPrice] <> ""
    ),
    #"Filtered Rows2" = Table.SelectRows(
        #"Filtered Rows1",
        each [imageURL] <> null and [imageURL] <> ""
    ),
    #"Changed Type1" = Table.TransformColumnTypes(
        #"Filtered Rows2",
        {
            {"agerange_min", Int64.Type},
            {"US_retailPrice", Currency.Type},
            {"year", type date},
            {"pieces", Int64.Type}
        }
    ),
    #"Added Conditional Column" = Table.AddColumn(
        #"Changed Type1",
        "Age Range",
        each if [agerange_min] <= 4 then "1 to 4"
             else if [agerange_min] <= 9 then "5 to 9"
             else if [agerange_min] <= 17 then "10 to 17"
             else "Over 18",
        type text
    ),
    #"Added Conditional Column1" = Table.AddColumn(
        #"Added Conditional Column",
        "Price Range",
        each if [US_retailPrice] > 500 then "$$$$$"
             else if [US_retailPrice] > 100 then "$$$$"
             else if [US_retailPrice] > 50 then "$$$"
             else if [US_retailPrice] > 25 then "$$"
             else "$",
        type text
    )
in
    #"Added Conditional Column1"
```

**Key Transformation Steps:**
1. **CSV Import**: Loads CSV with UTF-8 encoding (65001)
2. **Column Removal**: Eliminates unnecessary columns (minifigs, bricksetURL, thumbnailURL)
3. **Data Quality Filtering**: Removes rows with null or empty values in critical fields (age, price, imageURL)
4. **Data Type Conversion**: Converts string values to appropriate types (Integer, Currency, Date)
5. **Categorical Enrichment**: Creates two calculated columns:
   - **Age Range**: Buckets ages into four categories (1-4, 5-9, 10-17, Over 18)
   - **Price Range**: Categorizes prices into five tiers using $ symbols ($, $$, $$$, $$$$, $$$$$)

**LastRefreshTime table transformation:**
```powerquery
let
    Source = DateTime.LocalNow()
in
    Source
```
This simple transformation captures the current local date and time at the moment of refresh, providing data freshness tracking.

---

### 5. How do the DAX formulas work?

**Answer:** Let's break down the key DAX patterns used in this project:

**Pattern 1: Distinct Count Aggregations**
```dax
Total Sets = DISTINCTCOUNT(lego_sets[set_id])
Total Groups = DISTINCTCOUNT(lego_sets[themeGroup])
```
These measures use `DISTINCTCOUNT()` to count unique values, ensuring duplicate set IDs or theme groups are counted only once. This provides accurate counts even if the data contains duplicates.

**Pattern 2: AVERAGEX Iterator Pattern**
```dax
Avg. Age = AVERAGEX(lego_sets, lego_sets[agerange_min])
Avg. Price = AVERAGEX(lego_sets, lego_sets[US_retailPrice])
Avg. Pieces = AVERAGEX(lego_sets, lego_sets[pieces])
```
`AVERAGEX()` is an iterator function that:
- Iterates row-by-row through the lego_sets table
- Evaluates the expression (column reference) for each row
- Calculates the average across all rows in the current filter context
- This approach handles filter context correctly and respects slicers/filters applied to the visual

**Pattern 3: Conditional Display with HASONEVALUE()**
```dax
Selected Name = IF(
    HASONEVALUE(lego_sets[name]),
    MAX(lego_sets[name]),
    "Select Item"
)

Selected price = IF(
    HASONEVALUE(lego_sets[US_retailPrice]),
    MAX(lego_sets[US_retailPrice]),
    "-"
)
```
This pattern creates smart detail displays:
- `HASONEVALUE()` returns TRUE when filter context contains exactly one distinct value
- When TRUE, `MAX()` safely extracts that single value
- When FALSE (no selection or multiple values), displays a placeholder
- Perfect for card visuals that should only show details when one item is selected

**Pattern 4: Cross-Table Measure Reference**
```dax
-- In Max Price table:
Max Price Value = SELECTEDVALUE('Max Price'[Max Price])

-- In lego_sets table:
Max Price = IF([Avg. Price] <= 'Max Price'[Max Price Value], 1, 0)
```
This creates dynamic filtering without physical relationships:
- `Max Price Value` captures the user's selection from the Max Price slicer
- `Max Price` measure compares average price against the selected threshold
- Returns 1 (TRUE) or 0 (FALSE), which can be used as a filter flag
- Enables interactive price filtering through slicer interaction

**Pattern 5: String Concatenation with Special Characters**
```dax
Placeholder text = "←" & UNICHAR(10) & "Select One Item"
```
- Combines text strings using `&` operator
- `UNICHAR(10)` inserts a line break (ASCII character 10)
- Creates multi-line text for visual placeholders with arrow pointing left

**Pattern 6: Conditional Formatting Helper**
```dax
Color tran = IF(HASONEVALUE(lego_sets[name]), "rgba(0,0,0,0)")
```
- Returns transparent color (rgba with alpha=0) when single item selected
- Used in conditional formatting to hide/show visual elements dynamically
- Enhances user experience by providing visual feedback on selections

---

### 6. What visuals are included in the report and why?

**Answer:** While the exact visuals aren't stored in the data model metadata, based on the measures and data structure, the report likely includes:

**KPI Cards:**
- Display aggregate measures (Total Sets, Total Groups, Avg. Age, Avg. Price, Avg. Pieces)
- Provide high-level summary metrics at a glance
- Update dynamically based on applied filters

**Detail Cards:**
- Use the "Selected" measures to show individual set details
- Display name, price, year, pieces, and age when a single set is selected
- Use the Placeholder text measure to guide users when no selection is made
- Conditional formatting with Color tran measure to enhance visibility

**Slicers:**
- **Max Price Slicer**: Connected to the Max Price calculated table, allows users to filter by maximum price threshold
- **Theme/Category Slicers**: Enable filtering by theme, subtheme, themeGroup, or category
- **Age Range Slicer**: Filter by the calculated Age Range buckets (1-4, 5-9, 10-17, Over 18)
- **Price Range Slicer**: Filter by the calculated Price Range categories ($, $$, $$$, $$$$, $$$$$)
- **Year Slicer**: Time-based filtering to analyze trends over years

**Charts and Tables:**
- **Bar/Column Charts**: Compare metrics across themes, categories, or years
- **Line Charts**: Show trends over time (average price evolution, pieces per year)
- **Scatter Plots**: Explore relationships between price, pieces, and age ranges
- **Table Visual**: Display detailed set information with image URLs for product browsing
- **Matrix Visual**: Cross-tabulate metrics by multiple dimensions (theme × year, age × price)

**Rationale:**
The visual selection supports three key use cases:
1. **Executive Overview**: Quick KPI cards for high-level insights
2. **Interactive Exploration**: Slicers and charts for filtering and comparing
3. **Detail Investigation**: Selected measures and tables for deep-dive analysis

---

### 7. What business insights does each page provide?

**Answer:** Based on the data model structure, the report likely contains multiple pages serving different analytical needs:

**Page 1: Executive Dashboard**
- **Purpose**: High-level overview of the LEGO product portfolio
- **Key Insights**:
  - Total number of unique sets and theme groups in the database
  - Average pricing across the entire catalog
  - Average complexity (pieces) and target age demographics
  - Quick identification of market scale and product diversity
- **Business Value**: Executives can instantly understand the breadth of product offerings and key market positioning metrics

**Page 2: Price & Value Analysis**
- **Purpose**: Understand pricing strategies and value propositions
- **Key Insights**:
  - Price distribution across themes and categories
  - Relationship between price and piece count (value per piece)
  - Identification of premium vs. budget product lines
  - Price trends over time showing market evolution
- **Business Value**: Helps retailers optimize inventory mix and pricing strategies; assists collectors in identifying value opportunities

**Page 3: Age Demographics & Product Segmentation**
- **Purpose**: Analyze product segmentation by target age groups
- **Key Insights**:
  - Distribution of products across age ranges (1-4, 5-9, 10-17, Over 18)
  - Age-specific theme preferences
  - Correlation between age targets and product complexity/pricing
  - Market gaps or opportunities in specific age segments
- **Business Value**: Guides product development and marketing strategies for different age demographics

**Page 4: Theme & Category Performance**
- **Purpose**: Deep-dive into theme popularity and characteristics
- **Key Insights**:
  - Most popular themes by set count and average metrics
  - Theme-specific pricing and complexity patterns
  - Category trends and evolution over years
  - Identification of emerging or declining themes
- **Business Value**: Informs inventory decisions, identifies licensing opportunities, and guides marketing focus

**Page 5: Product Detail Explorer**
- **Purpose**: Individual set investigation and comparison
- **Key Insights**:
  - Detailed information on specific sets when selected
  - Visual product browsing with images
  - Side-by-side comparison capabilities
  - Complete product specifications at a glance
- **Business Value**: Supports purchasing decisions for consumers/retailers and enables detailed competitive analysis

**Page 6: Time Series Analysis**
- **Purpose**: Historical trends and temporal patterns
- **Key Insights**:
  - Year-over-year changes in pricing, complexity, and product volume
  - Identification of market evolution patterns
  - Seasonal or cyclical trends in product releases
  - Historical context for current market state
- **Business Value**: Reveals market maturity, pricing inflation, and product strategy evolution over time

---

### 8. What skills or technologies does the project demonstrate?

**Answer:** This Power BI project showcases a comprehensive skill set across multiple technical domains:

**Power BI Core Competencies:**
- **Data Modeling**: Creating efficient import models with optimized table structures
- **DAX (Data Analysis Expressions)**: Advanced formula development including:
  - Iterator functions (AVERAGEX)
  - Conditional logic (IF, HASONEVALUE)
  - Context awareness (SELECTEDVALUE, filter context manipulation)
  - Cross-table measure references
  - String manipulation and formatting
- **Power Query (M Language)**: Data transformation and preparation including:
  - CSV file loading with specific encoding
  - Data type conversions
  - Row filtering for data quality
  - Calculated column creation with conditional logic
  - Column removal for optimization
- **Calculated Tables**: Using DAX to generate virtual tables (GENERATESERIES)
- **Measure Organization**: Logical grouping and naming conventions for maintainability

**Data Analysis & Business Intelligence:**
- **Data Quality Management**: Implementing filtering to ensure clean, complete datasets
- **Dimensional Modeling**: Understanding of fact/dimension concepts (even without physical relationships)
- **KPI Development**: Creating meaningful business metrics from raw data
- **Categorical Binning**: Creating meaningful buckets for continuous variables (age, price)

**UX/UI Design Principles:**
- **Interactive Design**: Implementing dynamic filtering through calculated measures
- **User Guidance**: Creating placeholder text and visual cues for better user experience
- **Conditional Formatting**: Using measures to drive visual formatting dynamically
- **Progressive Disclosure**: Building detail views that appear on selection

**Technical Best Practices:**
- **Performance Optimization**: Using import mode for faster query performance
- **Code Documentation**: Clear naming conventions for measures and tables
- **Format Consistency**: Proper number/currency formatting for all measures
- **Audit Trail**: Including data refresh timestamp for data governance

**Business Acumen:**
- **Retail Analytics**: Understanding pricing, inventory, and product segmentation
- **Market Analysis**: Creating views that support competitive and trend analysis
- **Customer Segmentation**: Age-based analysis for targeted insights

**Transferable Skills:**
- Problem decomposition and analytical thinking
- ETL (Extract, Transform, Load) processes
- Data storytelling and visualization design
- Requirements analysis and solution architecture

This project demonstrates production-ready Power BI development capabilities suitable for enterprise business intelligence roles.

---

## Technical Specifications

### Data Model Overview

| Table | Type | Rows | Columns | Measures | Purpose |
|-------|------|------|---------|----------|---------|
| lego_sets | Fact (Import) | Variable | 13 | 13 | Main analytical table containing all LEGO set attributes |
| Max Price | Calculated | 171 | 1 | 1 | Price filtering helper (0 to 850 in steps of 5) |
| LastRefreshTime | Import | 1 | 1 | 0 | Data freshness tracking |

### Column Details - lego_sets Table

| Column Name | Data Type | Source | Description |
|-------------|-----------|--------|-------------|
| set_id | String | CSV | Unique identifier for each LEGO set |
| name | String | CSV | Product name/title |
| year | DateTime | CSV (converted) | Release year |
| theme | String | CSV | Primary theme (e.g., Star Wars, City) |
| subtheme | String | CSV | Sub-category within theme |
| themeGroup | String | CSV | Higher-level theme grouping |
| category | String | CSV | Product category classification |
| pieces | Int64 | CSV (converted) | Number of pieces in the set |
| agerange_min | Int64 | CSV (converted) | Minimum recommended age |
| US_retailPrice | Decimal (Currency) | CSV (converted) | US retail price in dollars |
| imageURL | String | CSV | URL to product image |
| Age Range | String | Power Query | Calculated: Age buckets (1-4, 5-9, 10-17, Over 18) |
| Price Range | String | Power Query | Calculated: Price tiers ($, $$, $$$, $$$$, $$$$$) |

### Measure Summary

**Aggregate Measures:**
- Total Sets, Total Groups, Avg. Age, Avg. Price, Avg. Pieces

**Selection Measures:**
- Selected Name, Selected price, Selected year, Selected pieces, Selected age

**Filtering Measures:**
- Max Price, Max Price Value

**UI Helpers:**
- Color tran, Placeholder text

### Data Transformations Summary

**Power Query Steps:**
1. CSV import with UTF-8 encoding
2. Header promotion
3. Column removal (minifigs, URLs)
4. Null/empty value filtering (age, price, imageURL)
5. Type conversions (Integer, Currency, Date)
6. Age Range bucketing (4 categories)
7. Price Range categorization (5 tiers)

**Calculated Tables:**
- Max Price: `GENERATESERIES(0, 850, 5)`
- LastRefreshTime: `DateTime.LocalNow()`

---

## Dependencies & Requirements

### System Requirements
- **Power BI Desktop**: Version with support for Import mode and calculated tables
- **Data Source**: CSV file located at `C:\Users\AboSamir\Desktop\lego_sets.csv`
- **Operating System**: Windows (for Power BI Desktop)

### Data Requirements
The CSV file must include the following columns:
- set_id, name, year, theme, subtheme, themeGroup, category
- pieces, agerange_min, US_retailPrice, imageURL

### Skills Required to Modify
- Power BI Desktop proficiency
- DAX (Data Analysis Expressions) knowledge
- Power Query M language understanding
- Basic data modeling concepts

### Setup Instructions
1. Install Power BI Desktop (latest version recommended)
2. Ensure the CSV file path is accessible or update the source path in Power Query
3. Open the .pbix file in Power BI Desktop
4. Refresh the data to load current CSV contents
5. Publish to Power BI Service if sharing is needed

---

## Project Metadata

- **Model Culture**: en-US
- **Default Mode**: Import
- **Source Query Culture**: ar-EG
- **Compatibility Level**: Modern Power BI
- **Last Modified**: May 12, 2025
- **Data Source**: Local CSV file

---

## Future Enhancement Opportunities

- Connect to live LEGO API for real-time data updates
- Add more sophisticated trend analysis with time intelligence functions
- Implement predictive analytics for pricing trends
- Create mobile-optimized report layouts
- Add drill-through pages for deep-dive analysis
- Implement row-level security for multi-user scenarios
- Add bookmarks for guided analytical narratives

---

## License & Usage

This project is provided for educational and analytical purposes. LEGO® is a trademark of the LEGO Group, which does not sponsor, authorize, or endorse this project.

---

## Contact & Contribution

For questions, improvements, or collaboration opportunities, please feel free to reach out or submit pull requests to enhance this analytical tool.

---
## 👤 Author

ahmed samir
**Built with Power BI Desktop | DAX | Power Query M**