# SQL Business Analytics on E-Commerce Database

# I. Introduction


The following report takes a mock, custom-built e-commerce database and uses SQL queries to extract data and insights valuable for a potential e-commerce business, such as website and traffic performance, product-level sales performance, and how customers access and interact with the website. Based on the results of the queries, **I conducted analysis to help the business understand various user-interaction trends with the website, as well as to make bidding recommendations for the growth and profitability of the mock business.**

This analysis was built on the course material in 'Advanced SQL: MySQL Data Analysis & Business Intelligence' by Maven Analytics. The database used in this project was created by John Pauler of Maven Analytics. 

Using `ipython-sql` package to connect MySQL server to Jupyter Notebook and make SQL queries.  

The full walk through of the analysis and code is available [here]().
 

# II. Database 

The database **mavenfuzzyfactory** contains customer interaction information with a mock e-commerce website within a three-year period from March 19, 2012 to March 19, 2015. 

### There are six related tables in the database

- **`website_sessions`**

  ![image](https://user-images.githubusercontent.com/52474489/142709634-9b929632-b9e6-49ac-8ef6-d4aceb841001.png)
    - Each instance describes a user's "session" in the website
    - primary key `website_session_id`

- **`website_pageviews`**
  
  ![image](https://user-images.githubusercontent.com/52474489/142709676-022faa71-cd87-4be1-8aa4-8c4f23573f2d.png)
    - Each instance describes each time a user has viewed a specific page in the website
    - primary key `website_pageview_id`

- **`products`**

  ![image](https://user-images.githubusercontent.com/52474489/142712066-0289bc45-4ffe-433e-8476-bca9e64e3737.png)
    - Each instance describes a product that is available for sale in the website
     - primary key `product_id`
  
- **`orders`**

  ![image](https://user-images.githubusercontent.com/52474489/142712091-61dc15d2-a6fa-47a5-a353-6b71530ac5c0.png)  
    - Each instance describes an order that a user has made
        - contains the `product_id` number of the first product ("primary product") a user added in their cart (`primary_product_id`). 
        - contains information about the total revenue (`price_usd`) and cost of goods (`order_id`) for each order
    - The primary key is `order_id`. 

- **`order_items`**

  ![image](https://user-images.githubusercontent.com/52474489/142712106-fb571cd5-5626-4b61-a5a5-2436208596dc.png)
    - Each instance describes an item that has been ordered by a specific user
        - contains associated `product_id`
        - contains an indicator if that item was the primary product (`is_product_id`)
    - Each instance is described by the primary key `order_item_id`. 

- **`order_item_refunds`**

  ![image](https://user-images.githubusercontent.com/52474489/142712118-91273554-ea78-417c-9954-4b27b288df18.png)
    - Each instance describes a refund event 
        - contains foreign keys `order_id` and `order_item_id`. 
    - The primary key is `order_item_refund_id`

<br>

**The database has the following schema:**
![image](https://user-images.githubusercontent.com/52474489/142564363-8aed1666-1489-47d6-9839-347647da2875.png)

# III. Analysis 

## A. Channel-Level Performance

First, we can examine the database by identifying and comparing the performance of different marketing channels. This particular website runs paid marketing campaigns to advertise its products. Paid traffic from users is generally tagged with **UTM parameters** where sessions are tagged with their respective **UTM Source** and **UTM Campaign**. A UTM source labels the source website that provided the traffic (search engines, social media sites), and UTM campaign is the type of marketing campaign used. A UTM campaign for a search engine can target specific keywords that a user types into the search engine, such as branded keywords (keywords associated with a specific brand, e.g. Gucci) or non-branded keywords (e.g. purse). Websites typically monitor these UTM parameters for each session (often times these parameters appear in the URL, as below). 

![image](https://user-images.githubusercontent.com/52474489/142711812-5c1894c4-b806-4ccc-b4d9-fd6d1e0b5ec7.png)

Specific UTM parameters describe a specific **marketing channel**. Performance and user-behaviors can be analyzed across different marketing channels. In this case, the primary UTM sources are `bsearch` and `gsearch` (search engines). The primary UTM campaigns are `brand` and `nonbrand`. 

  
**UTM sources**: A website session with a present `utm_source` parameter is an indicator that the traffic was paid for by marketing campaigns
  - `gsearch`: paid traffic through "gsearch" engine 
  - `bsearch`: paid traffic through "bsearch" engine 
  - `NULL`: If null, the session isn't a paid traffic


**UTM campaigns**: Refers to the type of marketing campaign that is paid for by the business. 
  - `nonbrand`: indicates a paid campaign that target keywords that is not associated with the business's brand
      - i.e. "teddy bear" 
  - `brand`: indicates a paid campaign that target keywords that is associated with the business's brand
      - i.e. "Mr. Fuzzy Bear" 
  - `NULL`: If null, there was no paid campaign associated with the website session 
    
    
**URL**: The URL of the original traffic source (indicated by variable `http_referer`)
  - `https://www.gsearch.com`
  - `https://www.bsearch.com`
  - `NULL`: If null, describes a session where a user has directly typed in the URL 
        

A website session in the database can be categorized into the following channels:
- `gsearch` source and `nonbrand` campaign: **gsearch Nonbrand Channel** 
- `bsearch` source and `nonbrand` campaign: **bsearch Nonbrand Channel**
- `gsearch` source and `brand` campaign: **gsearch Brand Channel**
- `bsearch` source and `brand` campaign: **bsearch Brand Channel**
- No UTM source and No UTM campaign: **Organic Search (unpaid)** 
    - User finds website through an organic search in the search engine, with no help of marketing channels. 
- No UTM source, No UTM campaign, No URL: **Direct Type In (unpaid)** 
    - User directly types in the URL of the website, without the help of search engines. 

A business would want to see growth in the paid channels so that they know they are getting their money's worth, but would also want the unpaid channels to grow at the same time so that they know the marketing campaigns are driving people to access the website on their own (without paid advertisement). 

We can ask the following business questions: 
- Which channels are driving the most website sessions and orders through the website? 
- How are user characteristics and conversion performance (order rate) different across different marketing channels? 
- How do you optimize bids and allocations of marketing budget?

### 1. Quarterly Traffic Volume 

![image](https://user-images.githubusercontent.com/52474489/142711853-680fe2bf-4034-4070-badd-43433e795cca.png)

For both brand and nonbrand campaigns, gsearch is the major search engine that users use to access the website. gsearch traffic also grows at a faster rate than bsearch, which is noticeable towards the end of the timeline. 

There are spikes of sessions that are noticeable for dates corresponding to Q4, which is where much of the holiday season lies (Thanksgiving, Black Friday, Christmas, New Years). Channels with nonbrand campaigns seem to have more of these "spikes" than channels with branded campaigns,  which makes sense given that holiday season shoppers are likely to search for nonbrand-related keywords such as "christmas gifts" or "black friday sale." **It would be then appropriate for the business to increase bidding for nonbrand campaigns during holiday season.**

### 2. Unpaid Traffic % (Organic Searches)

The following examines the proportion of unpaid channel traffic (organic searches) for all traffic in each search engine (bsearch and gsearch). This is to ensure that unpaid channel traffic is increasing, which is ideal as it would mean the website is driving in more and more traffic without the help of paid marketing campaigns. It would also allow us to ensure that despite the relatively low session volume, there is still growth in unpaid traffic. 

![image](https://user-images.githubusercontent.com/52474489/142711882-fcc375ab-3724-4849-a059-76c5ef3e209c.png)

It seems that especially after the Q2 of 2013, the unpaid traffic % seems to be consistently higher. To understand if paying for marketing campaigns for bsearch is worthwhile, the business can conduct the following "experiment:

Bid up bsearch marketing campaigns to see if the traffic from bsearch increases as a response, and to see what happens to bsearch's unpaid traffic %.

- If bsearch's traffic increases, and the unpaid traffic % for bsearch decreases, then the ad campaign is mainly driving paid traffic and the traffic is heavily reliant on the ad campaign. Spending more on the campaigns would not do much for the unpaid traffic. **Consider simply maintaining the bid for the bsearch channel**, depending on company financials or other metrics for bsearch such as the revenue it's driving (discussed later). 

- If bsearch traffic increases, and the unpaid traffic % for bsearch increases, then the ad campaign is driving unpaid traffic as well as paid traffic. In this case, it may be worth it to **increase the bid for the bsearch channel.** 

- If the bsearch traffic doesn't increase then **consider bidding down bsearch campaigns.** If the unpaid traffic % for bsearch increases even when the overall traffic for bsearch increased, it signifies that ad campaigns don't affect unpaid traffic (people are discovering the website through other means).

### 3. Conversion Rate

The **conversion rate** refers to a ratio between the number of sessions and the number of orders actually made on the website. How much % of the sessions for a certain marketing channel actually converted to orders can shed light on how much revenue a channel is bringing as well as how likely a potential customer is actually likely to buy something from the website using a given channel.

Examining the conversion rate for all channels

![image](https://user-images.githubusercontent.com/52474489/142711898-42503696-4958-48d0-aa28-2a3aa3d19e49.png)

It seems that most channels follow similar quarterly trends, even for unpaid channels. There was a gradual improvement in conversion rate across all channels, from 4-5% to 7-8%, which signifies a huge growth for the business. 

It is interesting to see that the conversion rate for brand keyword targetted channels for bsearch vastly improved after 2013 Q2. Given that no such dramatic spikes exist in terms of session traffic, and that only the brand channel for bsearch saw such improvement (and not nonbrand bsearch) is interesting. 

In fact, the period directly following 2013 Q2 is significant for several reasons: 

- Dramatic increase in conversion rate for 'bsearch brand'
- Noticeable increase in conversion rate for 'gsearch brand' 
- Increase in proportion of unpaid traffic for 'bsearch' (as seen previously)
- Dramatic increase in general traffic for 'gsearch', and some increase for 'bsearch' (as seen previously)

There is no information to what the business initiated after 2013 Q2, but it's safe to assume that above trends are all related as they all happen around the same time. I speculate that there must have been some sort of dramatic increase in bidding across all marketing channels, or other form of marketing through TV or banners. The above trends can potentially inform the **effects** of increasing marketing. 

When bidding up marketing:
- **Paid traffic for a commonly used search engine is increased at a more dramatic scale**
- **Proportion of unpaid traffic (organic search/direct type in) is increased, specifically for lesser used search engines such as bsearch**
- **Conversion rate improves relatively greater for customers coming through brand targetted channels** 

At the end of the day, conversion rate matters more than simply sessions alone because it directly translates into revenue. Given that the conversion rate responds well to increased traffic for sessions targetted by brand campaigns, it may make sense to try to increase bidding for these brand campaigns and see how the conversion rate reacts. However, it's important to note that customers entering brand related keywords in the search engine (e.g. "Mr. Fuzzy Bear") already have prior exposure to the brand. It's important to continue high investments in the nonbrand campaigns so that those who have not been exposed to the brand are exposed through these ad campaigns (e.g. people who just type in "teddy bear" in the search engine). 


