# SQL Business Analytics on E-Commerce Database

The following README.md file includes the analysis without code. The full walk-through of the analysis, SQL queries, and codes for Python visualizations are available [here](https://nbviewer.org/github/yunhwanchoi/SQL-Analysis-E-Commerce-Database/blob/main/SQL%20Business%20Analytics%20E-Commerce%20Database%20%28Code%29.ipynb).

# Table of Contents 

[I. Introduction](#1)

[II. Database](#2)

[III. Analysis](#3)
- [A. Channel-Level Performance](#3a)
  - [1. Quarterly Traffic Volume](#3a1)
  - [2. Unpaid Traffic % (Organic Searches)](#3a2)
  - [3. Conversion Rate](#3a3)
- [B. Page-Level Performance](#3b)
  - [1. Calculating Clickthrough Rates for Each Page Type](#3b1)
  - [2. Bounce Rates for Landing Pages ](#3b2)
  - [3. Order Rates for Billing Pages](#3b3)
- [C. Product-Level Performance](#3c)
  - [1. Monthly Trending for Number of Orders per Product](#3c1)
  - [2. Cross-Sold Products](#3c2)
  - [3. Monthly Trending for Refunds](#3c3)
 
[IV. Conclusion](#4)

# I. Introduction<a class="anchor" id="1"></a>


The following report takes a mock, custom-built e-commerce database and uses SQL queries to extract data and insights valuable for a potential e-commerce business, such as website and traffic performance, product-level sales performance, and how customers access and interact with the website. Based on the results of the queries, **I conducted analysis to help the business understand various user-interaction trends with the website, as well as to make bidding recommendations for the growth and profitability of the mock business.**

The summary of the insights and  of this analysis can be found in the [Conclusion](#4).


This analysis was built on the course material in 'Advanced SQL: MySQL Data Analysis & Business Intelligence' by Maven Analytics. The database used in this project was created by John Pauler of Maven Analytics. 

I used `ipython-sql` package to connect MySQL server to Jupyter Notebook and make SQL queries, and pandas/matplotlib/seaborn for further visualizations.  
 

# II. Database<a class="anchor" id="2"></a>

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

# III. Analysis <a class="anchor" id="3"></a>

## A. Channel-Level Performance<a class="anchor" id="3a"></a>

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

### 1. Quarterly Traffic Volume<a class="anchor" id="3a1"></a>

![image](https://user-images.githubusercontent.com/52474489/142711853-680fe2bf-4034-4070-badd-43433e795cca.png)

For both brand and nonbrand campaigns, gsearch is the major search engine that users use to access the website. gsearch traffic also grows at a faster rate than bsearch, which is noticeable towards the end of the timeline. 

There are spikes of sessions that are noticeable for dates corresponding to Q4, which is where much of the holiday season lies (Thanksgiving, Black Friday, Christmas, New Years). Channels with nonbrand campaigns seem to have more of these "spikes" than channels with branded campaigns,  which makes sense given that holiday season shoppers are likely to search for nonbrand-related keywords such as "christmas gifts" or "black friday sale." **It would be then appropriate for the business to increase bidding for nonbrand campaigns during holiday season.**

### 2. Unpaid Traffic % (Organic Searches)<a class="anchor" id="3a2"></a>

The following examines the proportion of unpaid channel traffic (organic searches) for all traffic in each search engine (bsearch and gsearch). This is to ensure that unpaid channel traffic is increasing, which is ideal as it would mean the website is driving in more and more traffic without the help of paid marketing campaigns. It would also allow us to ensure that despite the relatively low session volume, there is still growth in unpaid traffic. 

![image](https://user-images.githubusercontent.com/52474489/142711882-fcc375ab-3724-4849-a059-76c5ef3e209c.png)

It seems that especially after the Q2 of 2013, the unpaid traffic % seems to be consistently higher. To understand if paying for marketing campaigns for bsearch is worthwhile, the business can conduct the following "experiment:

Bid up bsearch marketing campaigns to see if the traffic from bsearch increases as a response, and to see what happens to bsearch's unpaid traffic %.

- If bsearch's traffic increases, and the unpaid traffic % for bsearch decreases, then the ad campaign is mainly driving paid traffic and the traffic is heavily reliant on the ad campaign. Spending more on the campaigns would not do much for the unpaid traffic. **Consider simply maintaining the bid for the bsearch channel**, depending on company financials or other metrics for bsearch such as the revenue it's driving (discussed later). 

- If bsearch traffic increases, and the unpaid traffic % for bsearch increases, then the ad campaign is driving unpaid traffic as well as paid traffic. In this case, it may be worth it to **increase the bid for the bsearch channel.** 

- If the bsearch traffic doesn't increase then **consider bidding down bsearch campaigns.** If the unpaid traffic % for bsearch increases even when the overall traffic for bsearch increased, it signifies that ad campaigns don't affect unpaid traffic (people are discovering the website through other means).

### 3. Conversion Rate<a class="anchor" id="3a3"></a>

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

## B. Page-Level Performance <a class="anchor" id="3b"></a>

The following analysis examines the clickthrough rate for each type of page in the website, and quantifies the performance of the website across different landing pages and different billing pages.

### 1. Calculating Clickthrough Rates for Each Page Type<a class="anchor" id="3b1"></a>

In order to make an order, a user goes through the following sequence of the following types of pages in the website
- Landing page 
- `/products`
- Selected Product page (one of four possible products)
- `/cart`
- `/shipping`
- Billing page 
- `/thankyou`
    
    
A **clickthrough rate** is defined as how often a user clicks to the next page for a certain page. From the database using the `website_sessions` and `website_pageviews` table, I was able to compare the cilckthrough rates for different types of pages in the website to identify if there are any interesting types of pages or detect outliers in terms of the clicking patterns for these pages.

![image](https://user-images.githubusercontent.com/52474489/142719868-5fdda8a2-bcb3-4994-a729-a80ef32fc860.png)

The above resulting table lists the clickthrough rates of different page types in order of sequence. 
- The clickthrough rate for the lander page is little over half. It makes sense that a good amount of eyeshoppers stumble upon the page with no intention of buying the product and exit the page. 
    - **There may be opportunities to improve the landing page performance** by making the design or structure a more accessible or appealing for the user. 
- The products page seems to be an intermediary page that lists the products, so it makes sense that users are clicking through it in a high rate. It would be concerning if the clickthrough rate was low at this page (problems with interface, product images, etc) but this isn't the case here.
- It seems that the selected product page has the lowest clickthrough rate, which makes sense as many of the purchase decisions are likely to be made here and the pages beyond this page consist of steps for actually purchasing the product. 
- The pages beyond selected product page have relatively high clickthrough rates
- The billing page is the step right before payment/order is made, and there is a slight dip in the clickthrough rate. 
    - **It is worth examining if there are improvements that could be made on the billing page** to encourage more orders. 

The **landing page** describes the page of the website that a customer first sees when visiting the website. The business has tried out multiple different landing pages. There are 6 variations of **landing pages** that the business has tested out in the duration specified in the database.
- `/home`
- `/lander-1`
- `/lander-2`
- `/lander-3`
- `/lander-4` 
- `/lander-5` 

And 2 variations of **billing pages** 
- `/billing`
- `/billing-2`

I measured the performance across different variations of both types of pages. 

### 2. Bounce Rates for Landing Pages <a class="anchor" id="3b2"></a>

**Bounce rate** refers to the number of sessions where a user did not make it past the landing page over the total number of sessions. The bounce rates were calculated for each of the six landing pages. Different landing pages were used for different points in the duration of the business. 

![image](https://user-images.githubusercontent.com/52474489/142719884-d3b9baa6-afda-4310-af31-f5972c7307e8.png)

`/lander-5`, the latest created landing page had the lowest bounce rate. The business is of date using the most effective landing page by a good margin and no change seems to be necessary. 

### 3. Order Rates for Billing Pages<a class="anchor" id="3b3"></a>

The billing page in this website is the step right before an order is made and the construction of this page can be important to encouraging users to make an order. There are two billing pages used throughout the course of the business, `/billing` and `/billing-2`. `/billing-2` is newly implemented and I wanted to verify that the new billing page is converting more orders. 

![image](https://user-images.githubusercontent.com/52474489/142719895-89fdb1ab-dfeb-41b1-9fe6-adb2be8ecb87.png)

`/billing-2` was created around only a year after the beginning of the website and received a large majority of all billing sessions in the database. The proportion of users who made orders on the billing page (order rate) is significantly higher for `/billing-2`, verifying its value over `/billing`. 

## C. Product Level Performance<a class="anchor" id="3c"></a>

The performance of the eCommerce business can be analyzed at the product-level as well. 

The business has originally sold 1 product, then has since added one new product at a time. There are four stuffed animal products in total that have been sold
- Product 1: The Original Mr. Fuzzy Bear
- Product 2: The Forever Love Bear
- Product 3: The Birthday Sugar Panda
- Product 4: The Hudson River Mini Bear 

The company revenue information is listed for every order made in the `orders` table, where `price_usd` refers to the revenue for the order and `cogs_usd` refers to the cost of good. The **profit margin** for each order would be represented by `price_usd` - `cogs_usd` and is a metric of interest for product-level analysis.

For each of the four products, the price, cost of goods, and profit margin are as below:

![image](https://user-images.githubusercontent.com/52474489/142719908-4deef794-379f-46da-8f30-4c6c74003387.png)

### 1. Monthly Trending for Number of Orders per Product <a class="anchor" id="3c1"></a>

Extracting the profit margin of orders for each product at every month

![image](https://user-images.githubusercontent.com/52474489/142719921-cce3f00a-8a9a-495b-9a4f-50aee83ee523.png)

- From the graph above, product 1 is the earliest launched and the greatest revenue driving product of the website. Product 2 was listed some time after, and product 3 and 4 launched around a similar time. 
- For every holiday season of this graph (november~december) a spike in sales is clearly visible, followed by a dramatic drop. After the latest holiday season, all products seems to have dropped significantly. It is worth investigating if this is part of the seasonal trend or something else is contributing to the overall decrease in sales. 
- It is worth noting that sales trend for product 3 and product 4 follow each other tightly, while the profit from product 4 consistently remains lower than product 3. Given that product 4 is cheaper than 3, it is possible that these items are frequently bought together.

### 2. Cross-Sold Products <a class="anchor" id="3c2"></a>

It is important to understand which products are likely to be sold together and at which rate they are being sold together. This information can be valuable when designing pages of any eCommerce website to feed into a recommendation system or something similar that suggests additional items based on the item that user has added to the cart. 

In the mavenfuzzyfactory database, each order can have 1 or more products, with the first one added to the cart by the user marked as the **primary product** in the database. Other products in the order, if existing, are **cross-sold products**. It may be interesting to examine user purchase patterns depending on which product the primary item is.

The following code extracts the number of orders for all the combinations of primary/cross-sold products that exist in the database, and calculates the **cross sell rate**, which is the number of orders for a given combination of primary/cross-sold product over the total number of orders for that primary product. For example, for orders that have primary product as product 1, what % of them had an add-on cross-sold product as product 2? I only pull out orders from the point where all four products are listed in the website, which is December 5, 2014. 

![image](https://user-images.githubusercontent.com/52474489/142719958-f1b4ce29-f126-4d7c-ac83-ee4dbdc3fbdb.png)

Visualizing above data with a seaborn heatmap:

![image](https://user-images.githubusercontent.com/52474489/142719983-671681df-729f-4746-9abb-aafcf3cf9e53.png)

![image](https://user-images.githubusercontent.com/52474489/142719988-e6aa4e4a-4b54-4446-81bb-e2b47c943fa7.png)

It seems for primary products 1-3, product 4 was the most commonly sold as the add-on item. Out of all combinations of primary/cross-sold products, orders with primary product as product 1 and the cross-sold product as product 4 was by far the most common. Product 3 was also commonly cross-sold with primary product 1, but not much for other primary products. Also notable is that users most commonly chose product 1 as a primary product. This may have to do with the fact that product 1 (Mr. Fuzzy Bear) was on the market for the longest and possibly became the driving source of the marketing of the business. 

Product 4 is the cheapest item on the website and is barely chosen as the primary product, being sold mostly as a cross-sold product. Understanding this customer behavior, it is recommended to **continue marketing Product 4 as an add-on items to other products, especially for cases where users chose product 1 as the primary product.** This can be done directly by asking users in the website if they would like to add product 4 to their cart as well, or suggesting that *many customers also purchase product 4*. Offering seasonal discounts for adding on product 4 as the cross-sold product may increase overall sales for product 4 and contribute to the overall profit margin.

### 3. Monthly Trending for Refunds <a class="anchor" id="3c3"></a>

Monitoring trends with refund helps the business troubleshoot if there are any issues with a certain product. Following shows monthly refund rate trends for each product.

![image](https://user-images.githubusercontent.com/52474489/142720001-023fc7ce-1e95-491e-bd75-49ecb4cefb96.png)

- It seems that often times refund rates for each product moves closely together for the most part, which is interesting. When refund rates collectively increase/decrease for all products may reflect an increase/decrease in the manufacturing/packaging quality for all products for that time period. 
- Overall, product 3 seems to be refunded the most, then product 1, product 2, and product 4. There is a very high spike of refunds only in product 1 around the summer of 2014 but this seems to be taken care of as the refund rates dramatically lowered after a certain point. 
- **The refund rates for product 3 hovers around 4% to 8%, which is concerning and definitely worth looking into, by both inspecting the quality of the product and incorporating feedback from customer reviews.**


# IV. Conclusion<a class="anchor" id="4"></a>


To summarize some of the insights gained about the mock business after querying the database, as well as recommendations for the business:

**Channel-Level Performance**
- Majority of traffic comes from paid marketing channels for gsearch, specifically for nonbrand gsearch campaigns. Nonbrand channels seem to be more susceptible to seasonality, especially during the holiday season at the end of the year. **Increase bidding for nonbrand campaigns during the holiday season.**


- Proportion of unpaid traffic for bsearch channels is slightly (but consistently) greater than gsearch channels. **Consider bidding up bsearch marketing campaigns, or conduct some sort of A/B Testing to see how the bsearch traffic responds to marketing changes and if the proportion of unpaid bsearch traffic changes at all. This will help understand how the ad campaign is influencing bsearch's traffic.**


- The period after 2013 Q2 saw dramatic changes in traffic and conversion rates for the website sessions. I hypothesized that marketing campaigns were significantly bidded up at this point. When bidding up marketing 
    - Paid traffic for a commonly used search engine is increased at a more dramatic scale
    - Proportion of unpaid traffic (organic search/direct type in) is increased, specifically for lesser used search engines such as bsearch
    - Conversion rate improves relatively greater for customers coming through brand targetted marketing campaigns


**Page-Level Performance**

- The clickthrough rate for the lander page is little over half. **There may be opportunities to improve the landing page performance by making the design or structure a more accessible or appealing for the user.**

-  The clickthrough rate for the billing page is relatively lower and is the most important page to optimize clickthrough rate from. **It is worth examining if there are improvements that could be made on the billing page to encourage more orders.**

- The bounce rate for the most recently implemented `/lander-5` has the lowest bounce rate compared to previously tested lander pages. **If possible, examine the changes made for `/lander-5` to inform further improvements on the lander page.**

- The order rate for the most recently implemented `/billing-2` has a higher order rate compared to the previously tested billing page. **If possible, examine the changes made for `/billing-2` to inform further improvements on the billing page.**

**Product-Level Performance**

- **Investigate the collective drop in the volume of sales from 2015 Q1.**

- Product 4 is the most cross-sold item and is rarely bought as a primary product. **Continue marketing Product 4 as an add-on items to other products, especially for cases where users chose product 1 as the primary product.**

- The refund rates for product 3 is consistently the highest compared to all other products, hovering around 4% to 8%. **Look into product 3 by inspecting the quality of the product and incorporating feedback from customer reviews.**










