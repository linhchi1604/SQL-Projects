# A. Customer Journey
### Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.
```
SELECT s.customer_id, p.plan_name, s.start_date
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans AS p
ON s.plan_id = p.plan_id
WHERE s.customer_id IN (1, 2, 11, 13, 15, 16, 18, 19)
ORDER BY s.customer_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/ce7e46e4-7f3e-4fcb-96be-037890d7be66)
<br/>
**Common Customer Journey:** After 7-day free trial, customers often change to basic or pro monthly plan. After a while, they would either upgrade to pro annual plan or churn.
- Customer 1: started with trial and changed to basic monthly after 7-day trial
- Customer 2: started with trial and changed to pro annual after 7-day trial
- Customer 11: started with trial and churned after 7-day trial
- Customer 13: started with trial, changed to basic monthly after 7-day trial, and upgraded to pro monthly 3 months later
- Customer 15: started with trial, changed to pro monthly after 7-day trial, and churned after 1 month
- Customer 16: started with trial, changed to basic monthly after 7-day trial, and upgraded to pro annual 4 months later
- Customer 18: started with trial and changed to pro monthly after 7-day trial
- Customer 19: started with trial, changed to pro monthly after 7-day trial, and upgraded to pro annual 2 months later
