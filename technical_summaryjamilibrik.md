# A/B Test Report: Food and Drink Banner

## Purpose
A/B test was used as a type of randomized controlled experiment which involves randomly assigning users to either control or treatment groups to ensure that the only difference between the groups is the experiment. We want to conclude the treatment effect of a Globox business project or food and drink banner using inferential statistical analysis. A/B testing, also known as split testing, is a method used to compare two versions of a webpage, app feature, or other product offerings against each other to determine which one performs better. The purpose of A/B testing is multifaceted and includes improving user Experience. Also, this test was done to understand how changes affect user behavior. This can optimize interactions to enhance the user interface and user experience by testing different designs, layouts, and content related to a website banner. Moreover, Globox e-commerce company aims to increase Conversion Rates by Optimizing for Goals and testing variations to see which one leads to higher conversion rates, whether it's signing up, making a purchase, or any other desired action, and in this case the rate is measured through the spend and purchase activity of customers or users. In addition, reducing bounce Rates by identifying and eliminating elements that cause users to leave is crucial to increasing retention. 
The A/B testing is based on Data-Driven Decisions to remove Guesswork and make informed decisions based on data rather than intuition. The idea of the experimental design is validated by this test of the new features or changes in the website to see if they have the intended optimal effect before full rollout or launching. Thus, enhancing Content Engagement can be achieved by content Optimization and determining which content types, headlines, or calls to action are more effective in engaging users. Then, testing different personalization strategies is done to see which ones resonate more with the audience to develop the product design or banner after validating the new features in a controlled environment to gauge user response before a widespread launch. Risks should be minimized by Identifying potential issues with new features or designs before they affect all users. Also, this experiment is a marketing and advertising tool done by optimizing the ad and banner design, so testing different ad creatives and targeting strategies to improve ROI (return on investment) are important. 
The company needs to know the cost efficiency too by allocating resources to changes that have a proven positive impact and avoiding unnecessary costs to prevent investment in features or changes that do not improve or could potentially harm user experience or conversion rates. Therefore, the business can grow by boosting revenues and optimizing conversion rates and user experience including the average amount spent or purchased in USD($) per user. This can lead to higher retention rates, so A/B testing allows for continuous improvement in all aspects of a product or service by providing a scientific approach to making changes. By testing hypotheses about user behavior, businesses can incrementally leverage their offerings in ways that are directly aligned with user preferences and business objectives.

## Hypotheses
The first hypothesis is to test the difference in conversion rates between the two groups, the null hypothesis would assume that there is no difference in the conversion rates, so
 H0 would be: p1 = p2. 
p1 and p2 are the conversion rates for group A (control group) and group B (treatment group) respectively. 
However, the alternative hypothesis would assume that there is a difference between the conversion  proportions or rates, so 
H1: p1 != p2
For the second hypothesis test, we want to see whether there is a difference in the average amount spent per user between the two groups A and B, so the null hypothesis H0 would assume that there is no difference between the means and H0 would be:
             μ1 = μ2 
where  μ1 is the control and μ2 is the treatment group.
 On the other hand, the alternative hypothesis H1 would assume that there is a difference between the average amount spent per user, so H1:
            μ1 != μ2

## Methodology
Data was extracted from Beekeeper Studio using SQL query by selecting information related to user ID (id), country, group (A or B), gender, device, group, total amount spent in USD($), and whether the user converted or not (0 for no and 1 for yes). Data was organized using special codes from the SQL to select important metrics related to the user from three different tables (activity, group, and users). Then, data was cleaned and joined together in one table to be extracted to a Microsoft Excel, google spreadsheet, or CSV file. Thus, a user-level aggregate table was created to be further analyzed using google spreadsheet software and visualized using Tableau Public Desktop. Participants were randomly assigned users to each group to control and avoid selection bias, thus ensuring that the group metrics can be compared. 
### Test Design
- **Population:** 
The setup of the A/B test is as follows:

The experiment is only being run on the mobile website.
A user visits the GloBox main page and is randomly assigned to either the control (group A) or test group (experiment B). This is the join date for the user.
The page loads the banner if the user is assigned to the test group, and does not load the banner if the user is assigned to the control group.
The user subsequently may or may not purchase products from the website. It could be on the same day they join the experiment, or days later. If they do make one or more purchases, this is considered a “conversion”.
- **Duration:** The join date was January 25, 2023, as the start date, and the end date was February 6, 2023.
- **Success Metrics:**
The metrics measured were the conversion rate between the two groups which would tell the number of users who converted divided by the number of users in each group. Also, the second metric was the average amount spent per user in USD($) in each group which is calculated by dividing the sum of the amount spent by the number of users in each group.

## Results
### Data Analysis
- **Pre-Processing Steps:**
As I have mentioned above in the methodology I have extracted the data using SQL Query.
```sql

--1)CAN USER SHOW MORE THAN ONCE IN ACTIVITY TABLE?
SELECT uid, COUNT(*) AS NumberOfRows
FROM activity
GROUP BY uid
HAVING COUNT(*)>1;
--2) TYPE OF JOIN TO JOIN USERS TO ACTIVITY TABLE:
Select *
FROM users
LEFT JOIN activity ON activity.uid = users.id;
--3) Fill in null values:
SELECT 
    users.id,
    COALESCE(activity.spent, 0) AS spent
FROM users
LEFT JOIN activity ON users.id = activity.uid;
--4) start and end dates:
SELECT min(dt), max(dt)
FROM activity;
--5) Total users in experiment:
SELECT COUNT(uid) AS TotalUsers
FROM groups;
--6)users in control and treatment group:
SELECT "group", COUNT(uid) AS UsersInGroup
FROM groups
GROUP BY "group";
WITH users AS (
    SELECT u.id, a.spent
    FROM users u
    LEFT JOIN activity a ON u.id = a.uid
)
, GroupedUsers AS (
    SELECT id, max(spent) AS converted
    FROM users
    GROUP BY id
)
--7)conversion rate:
SELECT
     COUNT(CASE WHEN converted IS NOT NULL THEN 1 END) AS TotalConverted,
    COUNT(id) AS TotalUsers,
     count(converted)*1.0/count(id) AS ConversionRate
FROM GroupedUsers;
--8) user conversion rate for both groups:
WITH UserConversion AS (
    SELECT
        u.uid,
        u."group",
        MAX(a.spent) AS Converted
    FROM groups u
    LEFT JOIN activity a ON u.uid = a.uid
    GROUP BY u.uid, u."group"
)
--9)average amount spent:
SELECT
    "group" AS TestGroup,
    COUNT(CASE WHEN Converted IS NOT NULL OR Converted = 1 THEN 1 END) AS TotalConverted,
    COUNT(uid) AS TotalUsers,
    COUNT(CASE WHEN Converted IS NOT NULL OR Converted = 1 THEN 1 END) * 1.0 / COUNT(uid) AS ConversionRate
FROM UserConversion
GROUP BY "group";
WITH UserSpending AS (
    SELECT
        u.uid,
        u."group",
        COALESCE(sum(a.spent), 0) AS TotalSpent
    FROM groups u
    LEFT JOIN activity a ON u.uid = a.uid
    GROUP BY u.uid, u."group"
)

SELECT
    "group" AS TestGroup,
    AVG(TotalSpent) AS AverageAmountSpent
FROM UserSpending
GROUP BY "group";
--extraction:
WITH TotalSpent AS (
SELECT
        uid,
        COALESCE(SUM(spent), 0) AS total_spent
    FROM
        activity
    GROUP BY
        uid
)
  SELECT
    u.id,
    COALESCE(u.country,'Unknown') as country,
    COALESCE(u.gender,'Unknown') as gender,
    COALESCE(g.device,'Unknown') as device,
    COALESCE(g."group",'Unkown') as "group",
    CASE WHEN a.total_spent > 0 THEN 1 ELSE 0 END AS converted,
    COALESCE(a.total_spent,0) as total_spent,
    join_dt  --for novelty effect check
FROM
    users u
    LEFT JOIN groups g ON u.id = g.uid
    LEFT JOIN TotalSpent a ON u.id = a.uid;

```
Data were then analyzed through built-in and cheat formulas, and statistical calculations in Google spreadsheet to find the statistical values needed for the hypothesis testing and for visualization later in Tableau Public Desktop.
- **Statistical Tests Used:** 
We assumed a normal distribution of variables. The statistical tests used were two-sided z-tests with pooled proportions to test for the difference in conversion rate. After thorough calculations for the conversion rates, to the statistical values like standard deviation, error, and pooled variation, z-test score, the p-value for conversion rate was 0.0001. Also, the statistical variation of the confidence interval was constructed at a 95% confidence interval and significance value alpha of 0.05, so the critical value z for the rate was found at 1.96. Also, the margin error rate was calculated using formulas in Google spreadsheet along with the lower and higher confidence interval which were between 0.003 and 0.01, and the sample statistic rate for the difference in proportion or conversion rate between group B and A was 0.007. The population variance was assumed to be known since the sample was large(n>30). The formulas used were found on the cheat formula for statistics. We compute the margin of error = critical value * standard error. Then, we take the sample statistic subtract the margin of error to get the lower bound, and add the margin of error to get the upper bound. The confidence intervals for a difference in proportion were found with two sample-z-intervals while assuming that the proportions are unpooled.

 


​As for the difference in the average amount spent per user, a two-sided t-test was conducted with unpooled variance similar to the construction of a confidence interval. Also, the degrees of freedom (df), the sample statistic which is the difference in means was calculated along with the standard error, margin of error, T-statistic, and pooled standard deviation for mean. The p-value was found to be 0.94 at 95% confidence interval and a significance value alpha of 0.05. The difference in means was 0.0163 and the lower and upper confidence intervals were -0.439 and 0.471 respectively, so the confidence interval included 0.


- **Results Overview:** High-level summary. 
### Findings
   The p-value for the difference in conversion rates between groups B and A respectively was 0.0001 at a 95% confidence interval which is lower than the significance value alpha of 0.05. 
   However, for the difference in the means for the average amount spent per user between the treatment and control group, the p-value was 0.94 which is higher than the alpha significance level of 0.05 at 95 % confidence interval.

## Interpretation
- **Outcome of the Test(s):** Did it support the hypothesis?
Thus, for the difference in the conversion rates, the p-value of 0.007 signifies that the variation in the conversion rate or proportion is statistically significant at a 95% confidence interval, so we can reject the H0 null hypothesis that claims that the conversion rate between the two groups is equal. Moreover, the sample statistic for the difference in conversion rates was 0.007 bounded by 0.003 and 0.01 intervals. Therefore, the lower and upper interval levels are both above 0 which makes it stronger that the conversion rate is significant with the conversion rate of group B higher than that of group B. Also, I assumed 80% statistical power and 10% minimum detectable effect. We can say that the variation in conversion rate is statistically significant but not statistically powerful since the confidence interval is still within 1% minimum detectable effect although the upper bound interval is slightly above. Also, the number of users in the experiment for 12 days was 489000 which is less than the value needed for statistical power which was 769000 as calculated by STATSIG.
As for the difference in the average amount spent per user between groups A and B, the p-value was 0.94 at a 95% confidence interval which is greater than the significance level alpha of 0.05. Thus, the variation in means in the two groups was not statistically significant. Moreover, the sample statistic with the difference in means was 0.0163 ranging between the lower and upper confidence interval levels of -0.439 and 0.471. This reveals that the confidence interval contains 0, and the difference is not significant. Also, the confidence interval is wider when comparing the difference in means between the two groups which suggests that a higher sample size can be used to minimize the error and fluctuation in the variation in means between group B and A.
The novelty effect was also tested to determine the correlation between the joining time and the metrics used to compare groups A and B. I have seen that the fluctuations in the average amount spent per user and in conversion rate between groups A and B do not contain any novelty or spike effect that seems irrelevant during the 12 days of study. If there were a spike or novelty effect, the values would rise at the beginning of the experiment and drop sharply after that, which is not the case here, since the variations in the amount spent and conversion rate between the treatment and control groups were stable and consistent with time, thus eliminating the possibility of any novelty effect on the retention, conversion, and engagement of the users between the two groups. 
- **Confidence Level:** 
The confidence level was estimated at a 95% confidence interval, and then the margin of error and standard error were calculated.

## Conclusions
- **Key Takeaways:** What did we learn?
 Although the difference between conversion rates in groups B and A was significant, the variation wasn't statistically powerful, since the confidence interval is still close to the 1% minimum detectable effect. Both lower and upper intervals of the conversion rate confidence interval were above 0 at 95% confidence, where we can reject H0, and say that the conversion rate difference between group B and A was statistically significant, where the conversion proportion of B is higher than A, since the sample statistic and boundaries of the interval were above 0. The sample sizes were not high enough to say that the variation in conversion rate was statically powerful at 80% statistical power and 10% minimum detectable effect.
On the other hand, the difference between the average amount spent was not statistically significant between the two groups due to the p-value being higher than the alpha value. Also, the lower and upper confidence intervals for the variation of means contain 0 at 95% confidence interval and 5% significance level, so we fail to reject the null hypothesis H0 in this case. The confidence interval is wide for the variation of means between groups B and A, so although the upper confidence interval is above 1% minimum detectable effect, the lower confidence interval is still negative and below 0. Moreover, at 80% statistical power and 10% MDE, the sample size is still way below the desired optimum sample size of 38 million as found on the statulator. Thus, the difference in means between the 2 groups is neither statistically powerful nor statistically significant. 
- **Limitations/Considerations:** 
 The sample size was not high enough to measure statistical power at 80% power and 10% minimum detectable effect, and only one of the metrics which is the conversion proportion difference was statistically significant, while the mean spent between the groups was not. 
## Recommendations
- **Next Steps:** 
It is recommended to reiterate the experiment since only one of the metrics was statistically significant in difference which is the conversion rate but not statiscally powerful due to the lower sample size, while the difference in the average amount spent between the groups wasn't significant or powerful significantly. It is advised to repeat the experiment.
- **Further Analysis:** 
It is better to reiterate the experiment by increasing the time for the users to join the two groups so that the sample size can increase to reach the desired size for statistical power. The globox A/B test needs more than 12 days to be tested, with at least 16-17 days instead to be more confident in validating the results and finding statistical significance and power in the variation in the conversion rate and average amount spent per user between the treatment and control group. Also, the sample size should increase from 49K to 76K at least to have better trusted results in order to launch the experiment after finding statistical significance and power in the difference in metrics between the 2 groups to avoid risks, loss of money on projects that might not be statistically proven, validated and trusted yet.
