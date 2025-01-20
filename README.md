# Introduction

A dive into the data job market. Focusing on data analyst roles this project explores top-paying jobs, in demand skills and where high demand meets high salary.

SQL queries in [project_sql folder](/project_sql/)

# Background

Driven by a quest to navigate the data analyst job market more efficiently, this project was born from the desire to pinpoint top-paid and in-demand skills, streamlining others to work to find optimal jobs.

Data hails from [Luke Brousse SQL course](https://lukebarousse.com/sql).

### Questions to answer were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn?

# Tools used

- **SQL**: The backbone of the analysis, allowing to query the database and unearth critical insigths.
- **PostgreSQL**: The chosen database management system, ideal for handling the job posting data.
- **Git & Github**: Essential for version control and sharing SQL scripts and analysis, ensuring collaboration and project tracking.
- **Excel**: Used to make tables, pivot tables and graphs to sumarize the data better and show it in a friendly way.

# The Analysis

Each query for this project aimed at investigating specific aspects of the data analyst job market.

#### 1. Top Paying Data Analyst Jobs

To identify the highest-paying roles. Filtered data analyst positions by average yearly salary and location, focusing on remote jobs. This query highlights the high paying opportunities in the field.

```SQL
SELECT
    job_id,
    name AS company_name,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE 
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND 
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10 
```
Here are some bullet points found with this analysis: 

1. The highest-paying data analyst position offers an exceptional salary of $650,000 per year at Mantys, which is nearly twice the salary of the second-highest position (Meta's Director of Analytics at $336,500). This significant outlier suggests either a highly specialized role or potentially includes additional compensation components like equity or bonuses.
2. All positions listed are fully remote ("Anywhere" location) and full-time, indicating a strong market for flexible work arrangements in high-paying data analytics roles. This suggests companies are willing to offer competitive salaries regardless of employee location, prioritizing talent over geographical constraints.
3. The salary range for senior/principal data analyst positions (excluding the Mantys outlier) consistently falls between $184,000 - $217,000, with companies like SmartAsset, Motional, and Get It Recruit offering similar compensation packages. This suggests a relatively stable market rate for senior-level data analytics talent.

#### 2. Skills for Top Paying Jobs

To understand what skills are required for the top-paying jobs, we joined the job postings with the skills data, providing insights into what employers value for high-compensation roles.

```SQL
WITH top_paying_jobs AS (
    SELECT
        job_id,
        name AS company_name,
        job_title,
        job_location,
        job_schedule_type,
        salary_year_avg,
        job_posted_date
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE 
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND 
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10 
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY 
    salary_year_avg DESC
```

Here is a breakdown of the most demanded skills for the top 10 highesr paying data analyst jobs in 2023:

- **SQL** is leading with a count of 8.
- **Python** follows closely with 7.
- **Tableau** is also highly sought after with a count of 6. There are other skills like **R**, **Snowflake**, **Pandas** and **Excel** varying degrees of demand.


#### 3. In-Demand Skills for Data Analysts

This query helped identify the skills most frequently requested in job postings, directing focus to areas with high demands.

```SQL
SELECT 
        skills,
        COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
GROUP BY
        skills
ORDER BY
    demand_count DESC
LIMIT 5
```

Here is a breakdown of the most demanded skills for data analysts in 2023.

- **SQL** and **Excel** remain fundamental, emphasizing the need for strong foundational skills in data processing and spreadsheet manipulation.
- **Programming** and **Visualization Tools** like **Python**, **Tableau** and **Power Bi** are essential, pointing towards the increasing importance of technical skills in data storytelling and decision support.

#### 4. Skills Based on Salary

Exploring the average salaries associated with different skills revealed which skills are the highest paying.

```SQL
SELECT 
        skills,
        ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
GROUP BY
        skills
ORDER BY
    avg_salary DESC
LIMIT 25
```

Here's a breakdown of the results for top paying skills for Data Analysts: 

- **High Demand for Big Data & ML Skills**: Top salaries are commanded by analysts skilled in big data technologies, machine learning tools and Python libraries, reflecting the industry's high valuation of data processing and predictive modeling capabilities.
- **Software Development & Deployment Profficiency**: Knowledge in development and development tools indicates a lucrative crossover between data analysis and enginerring with a premium on skills that facilitate automation and efficient data pipeline management.
- **Cloud Computing Expertise**: Familiarity with cloud and data engineering tools underscores the growing importance of cloud-based analytics environments, suggesting that cloud proficiency significantly boosts earning potential in data analytics.

#### 5. Most Optimal Skills to Learn

Combining insights from demand and salary data, this query aimed to pinpoint skills that are both in high demand and have high salaries, offering a strategic focus for skill development.

```SQL
SELECT 
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = True
GROUP BY 
    skills_dim.skill_id
HAVING
    COUNT(skills_job_dim.job_id) > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

Breakdown of the most optimal skills for Data Analytics 2023:

- **High-Demand Programming Languages**: Python and R stand out for their high demand, with demand counts of 236 and 148 respectively. Despite the demand their average salaries indicate that proficiency in these languages is highly valued but also widely available.
- **Cloud Tools and Technologies**: SKills in specialized technologies such as Snowflake, Azure, AWS and BigQuery show significant demand with relatively high average salaries, pointing towards the growing importance of cloud platforms and big data technologies in data analysis.
- **Business Intelligence and Data Visualization Tools**: Tableau and Looker, with demand counts of 230 and 49 respectively highlight the critical role of data visualization and business intelligence in deriving actionable insights from data.
- **Database Technologies**: The demand for skills in traditional and NOSQL databases reflects the enduring need for data storage, retrieval and management expertise.

# Conclusions

### Insights
From the analysis, several general insights emerged:

1. **Top Paying Data Analysis Jobs**: The highest paying jobs tend to be also some of the more niche ones, with several tools that were asked for very few companies.
2. **Skills for Top-Paying Jobs**: High-paying data analyst jobs required advanced proficiency in SQL, suggesting its a critical skill for earning a top salary.
3. **Most In-Demand Skills**: SQL is also the most demanded skill in the data analyst job market, this making it essential for job seekers.
4. **Skills with Higher Salaries**: Specialized skills are associated with highest average salaries, indicating a premium on niche expertise. 
5. **Optimal Skills for Job Market Value**: SQL leads in demand and offers for a high average salary, positioning it as one of the most optimal skills to learn to maximize their market value. 