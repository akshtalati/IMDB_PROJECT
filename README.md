# 🎬 IMDb Data Management & Business Intelligence (IMDM)

## 📌 Project Overview
This project implements an end-to-end Business Intelligence (BI) system using large-scale IMDb datasets. The objective is to transform raw, highly normalized movie metadata into an analytics-ready dimensional model that enables accurate querying, scalable analysis, and meaningful business insights.

The project strictly follows data warehousing and BI best practices, with a strong emphasis on fact table grain definition, aggregation correctness, disciplined ETL design, and trustworthy dashboarding.

---

## 🎯 Business Objectives
- Transform raw IMDb data into a reliable analytical data model
- Enable genre, rating, profession, and time-based analysis
- Prevent common BI issues such as double counting and metric inflation
- Deliver dashboards that support decision-making rather than raw reporting

---

## 🧩 Key Business Questions
- Which genres consistently produce the highest-rated movies?
- How have movie releases and ratings evolved over time?
- Which professions (actors, directors, writers) are associated with highly rated titles?
- How do ratings vary across genres and release years?

---

## 📂 Data Sources
Public IMDb datasets, including:
- Title basics (movies, genres, release years)
- Title ratings
- Name basics (people and professions)
- Title principals (movie–person relationships)

**Data Characteristics**
- ~90 million rows
- ~14 GB of raw data
- Highly normalized structure with multiple many-to-many relationships

---

## 🏗️ Architecture Overview
```
IMDb Raw Files  
→ ETL (Azure Data Factory / Alteryx)  
→ Snowflake Staging Tables  
→ Dimensional Model (Star Schema)  
→ SQL Analytics  
→ Tableau / Power BI Dashboards
```

---

## 🧠 Data Modeling Approach

### Dimensional Model
A star schema was designed to prioritize analytical clarity, performance, and correctness.

### Fact Table
**Fact_Title_Ratings**
- Measures: average_rating, num_votes
- Grain: one row per movie per rating snapshot

### Dimension Tables
- **Dim_Title** – movie metadata
- **Dim_Genre** – normalized genres
- **Dim_Person** – actors, directors, writers
- **Dim_Profession** – profession categories
- **Dim_Date** – calendar attributes

### Bridge Tables
- **Bridge_Title_Genre** (Title ↔ Genre)
- **Bridge_Title_Person** (Title ↔ Person)

Bridge tables resolve many-to-many relationships without duplicating fact measures.

---

## 🔄 ETL & Data Integration

### Tools Used
- Azure Data Factory – orchestration and scheduling
- Alteryx – data cleansing and transformation
- Snowflake – cloud data warehouse

### ETL Design Principles
- Controlled flattening of only analytics-relevant attributes
- Deduplication and referential integrity enforcement
- Validation of row counts and aggregates at every stage

---

## 🧠 Design Decisions & Trade-offs

### Dimensional Modeling vs Fully Normalized Schema
**Decision:** Use a star schema instead of a fully normalized design  
**Rationale:** BI systems prioritize query simplicity, performance, and readability  
**Trade-off:** Acceptable redundancy in exchange for analytical clarity

---

### Handling Many-to-Many Relationships
**Decision:** Introduce bridge tables rather than flattening relationships  
**Rationale:** Prevents row explosion and duplicated ratings  
**Trade-off:** Slightly more complex SQL joins

---

### Fact Table Grain Definition
**Decision:** One row per movie per rating snapshot  
**Rationale:** Ensures consistent aggregation and prevents inflated metrics  
**Trade-off:** Requires strict join discipline (intentionally enforced)

---

### Controlled Flattening During ETL
**Decision:** Flatten only attributes required for analytics  
**Rationale:** Full flattening increases duplication risk and table size  
**Trade-off:** More joins, but significantly higher correctness and scalability

---

### Tool Selection
| Tool | Reason | Trade-off |
|------|--------|-----------|
| Snowflake | Scalable, BI-friendly | Cloud cost |
| Azure Data Factory | Robust orchestration | Verbose configuration |
| Alteryx | Rapid transformations | License-based |
| Tableau / Power BI | Industry-standard BI tools | Learning curve |

---

## 🧮 SQL Analytics Layer
All SQL queries strictly respect the defined fact table grain and avoid duplication through correct joins, aggregation order, and validation checks.

---

## 🧪 Appendix: SQL Examples

### Top Rated Genres
```sql
SELECT
    g.genre_name,
    ROUND(AVG(f.average_rating), 2) AS avg_rating
FROM fact_title_ratings f
JOIN bridge_title_genre tg
    ON f.title_id = tg.title_id
JOIN dim_genre g
    ON tg.genre_id = g.genre_id
GROUP BY g.genre_name
ORDER BY avg_rating DESC;
```

### Movie Releases Over Time
```sql
SELECT
    d.release_year,
    COUNT(DISTINCT f.title_id) AS movie_count
FROM fact_title_ratings f
JOIN dim_date d
    ON f.release_date_id = d.date_id
GROUP BY d.release_year
ORDER BY d.release_year;
```

### Professions Associated with High Ratings
```sql
SELECT
    p.profession_name,
    ROUND(AVG(f.average_rating), 2) AS avg_rating
FROM fact_title_ratings f
JOIN bridge_title_person tp
    ON f.title_id = tp.title_id
JOIN dim_profession p
    ON tp.profession_id = p.profession_id
GROUP BY p.profession_name
ORDER BY avg_rating DESC;
```

### Validation Query
```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(DISTINCT title_id) AS unique_titles
FROM fact_title_ratings;
```

---

## 📊 Visualization Layer
Dashboards built using Tableau / Power BI include:
- Top-rated movies by genre
- Ratings trends over time
- Genre and profession comparisons
- Interactive filters and KPI-driven storytelling

The visualization layer focuses on interpretability, analytical correctness, and business insight delivery.

---

## ✅ Key Outcomes
- Successfully handled large-scale IMDb data
- Designed a robust dimensional model
- Eliminated common BI aggregation errors
- Delivered insight-driven dashboards aligned with industry best practices

- 
