# My Sql codes


## For 1st Business Question 

         SELECT 
         DISTINCT(product_name),
         base_price
         FROM 
         fact_events
         JOIN
         dim_products USING (product_code)

## For 2nd Business Question 
         SELECT 
         city, 
         COUNT(store_id) AS store_count
         FROM
         dim_stores
         GROUP BY city
         ORDER BY store_count DESC;

## For 3rd Business Question 
         SELECT 
         campaign_name,
         ROUND(SUM(base_price * `quantity_sold(before_promo)`) / 1000000,2) AS total_revenue_before_promotion,
         ROUND(SUM(CASE
                WHEN promo_type = 'BOGOF' THEN base_price * 0.5 * (`quantity_sold(after_promo)` * 2)
                WHEN promo_type = '500 Cashback' THEN (base_price - 500) * `quantity_sold(after_promo)`
                WHEN promo_type = '50% OFF' THEN base_price * 0.5 * `quantity_sold(after_promo)`
                WHEN promo_type = '33% OFF' THEN base_price * 0.67 * `quantity_sold(after_promo)`
                WHEN promo_type = '25% OFF' THEN base_price * 0.75 * `quantity_sold(after_promo)` END) / 1000000,2) AS total_revenue_after_promotion
         FROM
         fact_events
         JOIN
         dim_campaigns USING (campaign_id)
         GROUP BY campaign_name;


## For 4th Business Question 
         With  Diwali_campaign_sale as ( Select category , 
         Round(Sum((
              Case 
              When promo_type = "BOGOF" Then `quantity_sold(after_promo)`*2
              Else `quantity_sold(after_promo)`
              End
          - `quantity_sold(before_promo)`) * 100)
          / Sum(`quantity_sold(before_promo)`),2) as `ISU%` 
          From fact_events 
          Join 
          dim_products using(product_code)
          Join 
          dim_campaigns using(campaign_id)
          Where campaign_name = "Diwali"
          Group by category)

          Select 
          Category , 
          `ISU%`, 
          row_number() Over(order by `ISU%` desc) as rank_order 
          From Diwali_campaign_sale ;
     
## For 5th Business Question 

        SELECT 
        product_name,
        category,
        ROUND((SUM(CASE
                WHEN promo_type = 'BOGOF' THEN base_price * 0.5 * (`quantity_sold(after_promo)` * 2)
                WHEN promo_type = '500 Cashback' THEN (base_price - 500) * `quantity_sold(after_promo)`
                WHEN promo_type = '50% OFF' THEN base_price * 0.5 * `quantity_sold(after_promo)`
                WHEN promo_type = '33% OFF' THEN base_price * 0.67 * `quantity_sold(after_promo)`
                WHEN promo_type = '25% OFF' THEN base_price * 0.75 * `quantity_sold(after_promo)`
                ELSE 0
               END) 
               - SUM(base_price * `quantity_sold(before_promo)`)) / SUM(base_price * `quantity_sold(before_promo)`) * 100,2) AS `IR%`
          FROM
          fact_events
          JOIN
          dim_products USING (product_code)
          GROUP BY product_name , category
          ORDER BY `IR%` DESC
          LIMIT 5;
