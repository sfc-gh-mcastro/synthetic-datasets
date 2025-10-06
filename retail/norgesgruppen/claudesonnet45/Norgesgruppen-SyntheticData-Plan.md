### **Objective**

To create a synthetic, yet realistic, customer review dataset for Norgesgruppen stores. This dataset should be structured to allow for joins with existing store data (@store\_list.csv), product data (@product\_list.csv), and the hackathon's structured sales data. The reviews should reflect a range of customer experiences, sentiments, and topics relevant to grocery shopping in Norway.

---

### **Phase 1: Defining the Dataset Schema**

First, let's define the structure of your final output file, which we can call customer\_reviews.csv. A well-defined schema is the foundation of your project.

**Proposed customer\_reviews.csv Schema:**

| Column Name | Data Type | Description | Example |
| :---- | :---- | :---- | :---- |
| review\_id | Integer / UUID | A unique identifier for each review. | 10543 or a3b8-f4c1-4d3e-8b1a |
| customer\_id | Integer / UUID | A unique identifier for each synthetic customer. Allows tracking reviews from the same person. | 582 or c2d9-a5b3-4f1e-9c2b |
| store\_id | Integer | The ID of the store being reviewed. This is the **key to join** with @store\_list.csv. | 2145 |
| review\_date | Date / Datetime | The date and time the review was posted. Should fall within a realistic timeframe (e.g., last 2 years). | 2024-09-15T18:30:00 |
| rating | Integer (1-5) | The star rating given by the customer. | 4 |
| review\_title | String | A short, catchy title for the review. | "Bra utvalg, men litt rotete" |
| review\_text | String | The full text of the customer's review. This is the core of the dataset. | "Veldig fornøyd med ferskvareavdelingen hos Meny..." |
| mentioned\_product\_ids | List\[Integer\] | A list of product IDs from @product\_list.csv mentioned in the review. **Key for joining** to sales data. | \[101, 305\] (e.g., for "Grovbrød" and "Lettmelk") |
| sentiment\_label | String | A categorical sentiment label derived from the review. | 'Positive', 'Negative', 'Neutral' |
| sentiment\_score | Float (-1 to 1\) | A numerical sentiment score. Useful for more nuanced analysis. | 0.75 |

---

### **Phase 2: The Generation Plan (Step-by-Step)**

This phase outlines the process of creating the data to fill the schema defined above.

#### **Step 1: Create Synthetic Customer Personas**

You can't have reviews without customers. Creating simple personas will help generate more varied and realistic reviews.

* **Task:** Generate a customers.csv file.  
* **Columns:** customer\_id, name, city (from @store\_list.csv), persona.  
* **Personas could include:**  
  * **The Bargain Hunter:** Focuses on price, sales (tilbud), and the "First Price" brand. Their reviews often mention value for money.  
  * **The Quality Seeker:** Focuses on fresh produce (ferskvarer), organic options (økologisk), and premium brands. Shops mostly at Meny or Jacob's.  
  * **The Busy Parent:** Values speed, store layout, and stock availability of everyday items. Often complains about queues (kø) and empty shelves.  
  * **The Student:** Buys simple, cheap items. Shops at Kiwi or Joker. Reviews might be short and to the point.

#### **Step 2: Define Review Scenarios & Templates**

This is the creative core of the project. A scenario-based approach will ensure your reviews are not just random words but tell a story. All reviews should be generated in **Norwegian**.

* **Positive Scenarios:**  
  * **Excellent Service:** "Fantastisk hjelp fra en ansatt i dag\!" (Links to a high rating).  
  * **Great Product Quality:** "Alltid ferske og fine grønnsaker hos dere." (Can link to a mentioned\_product\_id from the vegetable category).  
  * **Fully Stocked:** "Tommel opp for at dere alltid har 'Grovbrød fra Bakeriet' på lager\!" (Links to a specific mentioned\_product\_id).  
  * **Good Sale/Offer:** "Elsker når 'Kaffe fra Evergood' er på tilbud\!" (Connects to sales data themes).  
* **Negative Scenarios:**  
  * **Out of Stock:** "Irriterende at dere var tomme for lettmelk igjen." (Links to mentioned\_product\_id).  
  * **Long Queues:** "Altfor lang kø i kassen rundt klokken 16." (Could be correlated with high-volume sales hours in the structured data).  
  * **Poor Quality:** "Kjøpte en avokado som var helt brun inni." (Links to mentioned\_product\_id).  
  * **Messy Store:** "Veldig rotete i butikken i dag, vanskelig å finne frem."  
* **Neutral Scenarios:**  
  * **General Observation:** "Helt grei butikk for en rask handel."  
  * **Price Observation:** "Prisene her er på linje med andre butikker i området."

#### **Step 3: The Generation Script**

This is where you'll write the code (e.g., in Python) to execute the plan.

1. **Load Inputs:** Read @store\_list.csv, @product\_list.csv, and your new customers.csv into memory (e.g., using pandas).  
2. **Initialize Output:** Create an empty list or DataFrame to hold your generated reviews.  
3. Main Generation Loop: Loop for the desired number of reviews (e.g., 5,000). In each iteration:  
   a. Pick a Customer: Randomly select a customer\_id from your customer list. Their persona will influence the scenario choice.  
   b. Pick a Store: Randomly select a store\_id from the store list. You could add logic to make customers more likely to review stores in their own city.  
   c. Pick a Scenario: Based on the customer's persona, probabilistically choose a scenario from your list (e.g., the Bargain Hunter is 50% likely to talk about price).  
   d. Determine Rating: The chosen scenario dictates the rating (e.g., "Out of Stock" scenario maps to a rating of 1 or 2).  
   e. Generate review\_text: Use templates combined with data from your files.  
   \* Example Template: "{product\_name} var dessverre utsolgt i dag hos {store\_name}. Veldig skuffende\!"  
   \* Fill the template:  
   \* Select a random product from @product\_list.csv \-\> product\_name and product\_id.  
   \* Get the store\_name from @store\_list.csv using the selected store\_id.  
   \* Populate mentioned\_product\_ids with the chosen product\_id.  
   f. Generate Metadata: Create a random review\_date, a unique review\_id, etc.  
   g. Append to Output: Add the newly generated review record to your output list.

#### **Step 4: Post-Processing**

Once the main loop is done:

1. **Sentiment Analysis:** Iterate through your generated reviews. For each review\_text, use a pre-trained sentiment analysis model (ideally one that supports Norwegian) to calculate the sentiment\_score and assign a sentiment\_label.  
2. **Save to CSV:** Convert your list of review records into a pandas DataFrame and save it as customer\_reviews.csv.

---

### **Phase 3: Planning for the Hackathon \- How to Use the Data**

This is the "why" of the project. Explain how the hackathon participants can leverage this new dataset.

1. **Store Performance Analysis:**  
   * **Join:** customer\_reviews.csv on store\_id with @store\_list.csv.  
   * **Question:** Which store *chain* (Meny, Kiwi, Joker) has the highest average rating? Are there regional differences in customer satisfaction (e.g., Oslo vs. Bergen)?  
2. **Connecting Reviews to Sales Performance:**  
   * **Join:** customer\_reviews.csv on store\_id and review\_date with the structured\_sales\_data.csv.  
   * **Question:** Do periods with a high number of negative reviews (e.g., about queues or stock issues) correlate with a dip in revenue for a specific store?  
3. **Product-Specific Sentiment:**  
   * **Join:** customer\_reviews.csv (on mentioned\_product\_ids) with product\_list.csv and the sales data (on product\_id).  
   * **Question:** What is the customer sentiment surrounding the top-selling products? Do negative reviews mentioning a product being "out of stock" align with periods where that product had zero sales in the structured data? This can help quantify the impact of stockouts.

### **Suggested Tools & Technologies**

* **Language:** Python  
* **Libraries:**  
  * pandas: For data manipulation and handling CSVs.  
  * numpy: For random sampling and numerical operations.  
  * Faker: To generate realistic synthetic names for customers.  
  * transformers (Hugging Face): To use a pre-trained model for Norwegian sentiment analysis (e.g., NbAiLab/nb-bert-base-sentiment). This is more advanced but gives much better results.  
  * For a simpler approach, you could pre-assign sentiment based on the chosen scenario.

By following this plan, you will produce a high-quality, synthetic dataset that is perfectly tailored for a data-driven hackathon. Good luck\!