# Norgesgruppen Synthetic Customer Reviews Dataset

This repository contains synthetic customer review data for Norgesgruppen stores, generated according to the plan in `Norgesgruppen-SyntheticData-Plan.md`.

## 📁 Generated Files

### 1. `customers.csv` (500 customers)
Synthetic customer personas with the following columns:
- `customer_id`: Unique identifier for each customer
- `name`: Norwegian name (generated with Faker)
- `city`: Customer's city
- `persona`: One of four types:
  - **Bargain Hunter** (24%): Focuses on price and sales
  - **Quality Seeker** (27%): Values fresh produce and premium brands
  - **Busy Parent** (30%): Prioritizes convenience and availability
  - **Student** (20%): Budget-conscious, simple needs

### 2. `customer_reviews.csv` (5,000 reviews)
Synthetic customer reviews with the following schema:

| Column | Type | Description |
|--------|------|-------------|
| `review_id` | Integer | Unique identifier for each review |
| `customer_id` | Integer | Links to customers.csv |
| `store_name` | String | Store name from store_list.csv |
| `store_brand` | String | Store brand (KIWI, MENY, SPAR, etc.) |
| `review_date` | Datetime | Review date (June-August 2025, summer period) |
| `rating` | Integer | Star rating (1-5) |
| `review_title` | String | Short review title |
| `review_text` | String | Full Norwegian review text |
| `product_name` | String | Product name (if mentioned, otherwise null) |
| `product_category` | String | Product category (if mentioned, otherwise null) |
| `sentiment_label` | String | Positive/Negative/Neutral |
| `sentiment_score` | Float | Sentiment score (-1 to 1) |

### 3. Input Data Files
- `store_list.csv`: 848 Norgesgruppen stores (KIWI, MENY, SPAR, EUROSPAR, Joker, Nærbutikken, Jacobs)
- `product_list.csv`: 36 products across categories (Grillmat, Sjømat, Frukt og Grønt, etc.)

## 📊 Dataset Statistics

**Review Distribution:**
- Rating 1: 419 reviews (8.4%)
- Rating 2: 901 reviews (18.0%)
- Rating 3: 1,507 reviews (30.1%)
- Rating 4: 1,138 reviews (22.8%)
- Rating 5: 1,035 reviews (20.7%)

**Sentiment Distribution:**
- Positive: 2,173 reviews (43.5%)
- Neutral: 1,507 reviews (30.1%)
- Negative: 1,320 reviews (26.4%)

**Average Rating:** 3.29/5.0

## 🎯 Review Scenarios

### Positive Scenarios
- Excellent service
- Great product quality
- Fully stocked shelves
- Good sales/offers
- General positive experience

### Negative Scenarios
- Out of stock items
- Long queues
- Poor product quality
- Messy store
- High prices

### Neutral Scenarios
- General observations
- Price comparisons
- Quick visits

## 🚀 Usage for Hackathon

### Analysis Ideas

#### 1. Store Performance Analysis
```python
# Average rating by store brand (already in dataset)
brand_ratings = reviews_df.groupby('store_brand')['rating'].mean()

# Top-rated individual stores
store_ratings = reviews_df.groupby('store_name')['rating'].mean().sort_values(ascending=False)
```
Or 
```sql
select store_brand, 
       count(*) as review_count,
       avg(rating) as avg_rating
from hackathon.synthetic_data_raw.customer_reviews
group by store_brand;
```

#### 2. Product Sentiment Analysis
```python
# Filter reviews with product mentions
product_reviews = reviews_df[reviews_df['product_name'].notna()]

# Most mentioned products
product_counts = product_reviews['product_name'].value_counts()

# Negative sentiment by product
negative_products = product_reviews[product_reviews['sentiment_label'] == 'Negative']
negative_product_counts = negative_products['product_name'].value_counts()

# Average rating by product category
category_ratings = product_reviews.groupby('product_category')['rating'].mean()
```

Or
```sql
SELECT 
    product_name,
    COUNT(*) as review_count,
    AVG(rating) as avg_rating,
    AVG(sentiment_score) as avg_sentiment,
    COUNT(CASE WHEN sentiment_label = 'POSITIVE' THEN 1 END) as positive_reviews,
    COUNT(CASE WHEN sentiment_label = 'NEGATIVE' THEN 1 END) as negative_reviews
FROM norgesgruppen_hackathon.synthetic_data_raw.customer_reviews
WHERE product_name IS NOT NULL
GROUP BY product_name
ORDER BY review_count DESC;

SELECT 
product_name,
COUNT(*) as product_count
FROM norgesgruppen_hackathon.synthetic_data_raw.customer_reviews
GROUP BY product_name
ORDER BY product_count DESC;

SELECT 
    product_name,
    COUNT(*) as negative_review_count,
    AVG(sentiment_score) as avg_sentiment_score
FROM norgesgruppen_hackathon.synthetic_data_raw.customer_reviews
WHERE UPPER(sentiment_label) = 'NEGATIVE'
GROUP BY product_name
ORDER BY negative_review_count DESC;

```

#### 3. Time-Based Patterns (Summer 2025)
```python
# Convert review_date to datetime
reviews_df['review_date'] = pd.to_datetime(reviews_df['review_date'])

# Reviews by month (June, July, August 2025)
reviews_df.groupby(reviews_df['review_date'].dt.to_period('M')).size()

# Reviews by day of week
reviews_df['day_of_week'] = reviews_df['review_date'].dt.day_name()
reviews_df.groupby('day_of_week')['rating'].mean()
```

#### 4. Customer Persona Insights
```python
# Join reviews with customers
reviews_customers = reviews_df.merge(customers_df, on='customer_id')

# Rating patterns by persona
persona_ratings = reviews_customers.groupby('persona')['rating'].mean()
```

## 🛠️ Regenerating the Data

### Prerequisites
```bash
pip install -r requirements.txt
```

### Generate Customers
```bash
python generate_customers.py
```

### Generate Reviews
```bash
python generate_reviews.py
```

## 📝 Data Quality Features

✅ **Realistic Norwegian Text**: All reviews written in Norwegian  
✅ **Persona-Driven**: Customer personas influence review sentiment and content  
✅ **Store Brand Awareness**: Reviews reflect expectations for different store brands  
✅ **Product Integration**: Separate columns for product name and category for easy filtering  
✅ **Summer Period Focus**: Reviews span June-August 2025 (peak summer season)  
✅ **Sentiment Alignment**: Sentiment scores match rating levels  
✅ **Analysis-Ready Structure**: Flat table structure, no nested data or JSON parsing needed  

## 🔗 Join Capabilities

This dataset can be joined with:
1. **customers.csv** via `customer_id` - for persona-based analysis
2. **store_list.csv** via `store_name` - for additional store metadata
3. **product_list.csv** via `product_name` - for additional product details
4. **Hackathon sales data** via `store_name` + `review_date` - to correlate reviews with sales performance

## 📌 Notes

- All customer names and data are synthetic (generated with Faker)
- Reviews are template-based with randomized elements
- Sentiment scores are derived from ratings (not ML-based for simplicity)
- Store names and brands are directly included for easy analysis (no joins required)
- Product mentions included as separate `product_name` and `product_category` columns
- Not all reviews mention products (approximately 60-70% do)
- Date range covers summer 2025 (June 1 - August 31) for seasonal analysis

## 🎓 Educational Purpose

This dataset is designed for a data analytics hackathon to demonstrate:
- Data integration across multiple tables
- Sentiment analysis
- Customer segmentation
- Store performance metrics
- Product popularity and feedback analysis

---

**Generated:** October 2025  
**Total Records:** 5,500 (500 customers + 5,000 reviews)  
**Language:** Norwegian (Bokmål)

