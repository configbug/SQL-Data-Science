## SOLUTION

![image](https://user-images.githubusercontent.com/1278217/199652729-c3b50673-894c-405e-8e58-91304c14f86c.png)
![image](https://user-images.githubusercontent.com/1278217/199652767-cd64e71c-4880-40af-860e-b94333910d94.png)
![image](https://user-images.githubusercontent.com/1278217/199652859-27236fc3-93b0-476f-9e3b-c9569c73a365.png)

## CODE USED

> **QUESTION 1 :** Compare the final_assignments_qa table to the assignment events we captured for user_level_testing. Write an answer to the following question: Does this table have everything you need to compute metrics like 30-day view-binary?

>> ANSWER : No. The created_at date is needed.

```
SELECT 
  * 
FROM 
  dsv1069.final_assignments_qa
```


> **QUESTION 2 :** Write a query and table creation statement to make final_assignments_qa look like the final_assignments table. If you discovered something missing in part 1, you may fill in the value with a place holder of the appropriate data type. 

>> ANSWER : 

```
SELECT item_id,
       test_a AS test_assignment,
       (CASE
            WHEN test_a IS NOT NULL then 'test_a'
            ELSE NULL
        END) AS test_number,
       (CASE
            WHEN test_a IS NOT NULL then '2020-01-01 00:00:00'
            ELSE NULL
        END) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION
SELECT item_id,
       test_b AS test_assignment,
       (CASE
            WHEN test_b IS NOT NULL then 'test_b'
            ELSE NULL
        END) AS test_number,
       (CASE
            WHEN test_b IS NOT NULL then '2020-01-01 00:00:00'
            ELSE NULL
        END) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION
SELECT item_id,
       test_c AS test_assignment,
       (CASE
            WHEN test_c IS NOT NULL then 'test_c'
            ELSE NULL
        END) AS test_number,
       (CASE
            WHEN test_c IS NOT NULL then '2020-01-01 00:00:00'
            ELSE NULL
        END) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION
SELECT item_id,
       test_d AS test_assignment,
       (CASE
            WHEN test_d IS NOT NULL then 'test_d'
            ELSE NULL
        END) AS test_number,
       (CASE
            WHEN test_d IS NOT NULL then '2020-01-01 00:00:00'
            ELSE NULL
        END) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION
SELECT item_id,
       test_e AS test_assignment,
       (CASE
            WHEN test_e IS NOT NULL then 'test_e'
            ELSE NULL
        END) AS test_number,
       (CASE
            WHEN test_e IS NOT NULL then '2020-01-01 00:00:00'
            ELSE NULL
        END) AS test_start_date
FROM dsv1069.final_assignments_qa
UNION
SELECT item_id,
       test_f AS test_assignment,
       (CASE
            WHEN test_f IS NOT NULL then 'test_f'
            ELSE NULL
        END) AS test_number,
       (CASE
            WHEN test_f IS NOT NULL then '2020-01-01 00:00:00'
            ELSE NULL
        END) AS test_start_date
FROM dsv1069.final_assignments_qa;
```

> **QUESTION 3 :** Use the final_assignments table to calculate the order binary for the 30 day window after the test assignment for item_test_2 (You may include the day the test started)

>> ANSWER : 

```
SELECT test_assignment,
       COUNT(DISTINCT item_id) AS number_of_items,
       SUM(order_binary) AS items_ordered_30_days
FROM
  (SELECT item_test_2.item_id,
          item_test_2.test_assignment,
          item_test_2.test_number,
          item_test_2.test_start_date,
          item_test_2.created_at,
          MAX(CASE
                  WHEN (created_at > test_start_date
                        AND DATE_PART('day', created_at - test_start_date) <= 30) THEN 1
                  ELSE 0
              END) AS order_binary
   FROM
     (SELECT final_assignments.*,
             DATE(orders.created_at) AS created_at
      FROM dsv1069.final_assignments AS final_assignments
      LEFT JOIN dsv1069.orders AS orders
        ON final_assignments.item_id = orders.item_id
        WHERE test_number = 'item_test_2') AS item_test_2
   GROUP BY item_test_2.item_id,
            item_test_2.test_assignment,
            item_test_2.test_number,
            item_test_2.test_start_date,
            item_test_2.created_at) AS order_binary
GROUP BY test_assignment;
```

> **QUESTION 4 :** Use the final_assignments table to calculate the view binary, and average views for the 30 day window after the test assignment for item_test_2. (You may include the day the test started)

>> ANSWER : 

```
SELECT
  test_assignment,
  COUNT(item_id) AS items,
  SUM(view_binary_30d) AS viewed_items,
  CAST(100 * SUM(view_binary_30d) / COUNT(item_id) AS FLOAT) AS viewed_percent,
  SUM(views) AS views,
  SUM(views) / COUNT(item_id) AS average_views_per_item
FROM
  (
    SELECT
      f.test_assignment,
      f.item_id,
      MAX(
        CASE
          WHEN item_views.event_time > f.test_start_date THEN 1
          ELSE 0
        END
      ) AS view_binary_30d,
      COUNT(item_views.event_id) AS views
    FROM
      dsv1069.final_assignments f
      LEFT OUTER JOIN (
        SELECT
          event_time,
          event_id,
          CAST(parameter_value AS INT) AS item_id
        FROM
          dsv1069.events
        WHERE
          event_name = 'view_item'
          AND parameter_name = 'item_id'
      ) item_views ON f.item_id = item_views.item_id
      AND item_views.event_time >= f.test_start_date
      AND DATE_PART(
        'day',
        item_views.event_time - f.test_start_date
      ) <= 30
    WHERE
      f.test_number = 'item_test_2'
    GROUP BY
      f.test_assignment,
      f.item_id
  ) item_orders
GROUP BY
  test_assignment;
```

> **QUESTION 5, 6 :** Use the https://thumbtack.github.io/abba/demo/abba.html to compute the lifts in metrics and the p-values for the binary metrics ( 30 day order binary and 30 day view binary) using a interval 95% confidence. 

>> ANSWER : (See charts above) Test Group 1 had 2.5% more items viewed compared to Test Group 0, the p-value for this increase was 0.2, which does not meet our threshold of significance. There are no statistically significant changes in the percentage seen of the metrics as a result of treatment.
