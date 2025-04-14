# telecomplus-datawarehouse-fabric
Repository for TelecomPlus data warehouse &amp; ETL project in Microsoft Fabric. Provides a BI solution for sales and customer service insights using a star schema.

# TelecomPlus Data Warehouse and ETL Project in Microsoft Fabric

## Overview

This project involved the development of a **data warehouse in Microsoft Fabric** for TelecomPlus, a telecommunications company providing mobile and internet services across the United States. The primary goal was to create a Business Intelligence (BI) solution to gain a detailed understanding of factors impacting sales and customer service, ultimately supporting better decision-making for future strategies and business growth. This was achieved by extracting data from various sources, transforming it through an **ETL (Extract, Transform, Load) flow**, and loading it into a data warehouse designed using the **Kimball dimensional modeling approach** with a **star schema**.

## Business Problem

TelecomPlus was facing several key challenges that hindered their ability to provide optimal service, meet localized customer needs, and maximize revenue growth:

*   **Inconsistent Service Quality Across Channels:** A lack of clear understanding regarding the performance of different communication channels (phone, email, chat, social media) within customer service.
*   **Products That Don’t Fit Local Needs:** Insufficient insight into regional customer preferences, potentially leading to offerings that don't align with local demands.
*   **Unclear Revenue Sources and Growth Opportunities:** Uncertainty about which locations and products were the most profitable, risking misallocation of resources.

## Business Needs & Goals

To address these problems, the project aimed to fulfill the following business needs:

*   **Improving Service Quality and Customer Experience:** Gain a clear understanding of the performance of each customer service channel and department by analyzing average waiting and resolution times, customer satisfaction scores, and interaction volumes.
*   **Customer Preferences and Product Trends:** Understand customer preferences in different areas to deliver the right products to the right audience by analyzing product subscriptions based on demographics and region.
*   **Revenue Analysis and Market Performance:** Analyze revenue patterns across locations, products, and time to identify growth opportunities and focus on profitable areas by tracking top-performing locations and revenue changes for product categories and types.

## Data Sources

The project utilized several data sources, primarily in the form of **CSV files hosted on a LakeHouse within Microsoft Fabric**:

*   `channel_info.csv`: Details the various communication channel platforms.
*   `reason_info.csv`: Captures the main reasons for customer interactions.
*   `customer_info.csv`: Provides demographic and location information about customers.
*   `agents_info.csv`: Contains details about TelecomPlus agents.
*   `plans_info.csv`: Details the subscription plans offered by TelecomPlus.
*   `customer_interactions_cases`: Logs each customer interaction with key metrics.
*   `signups_record`: Records information on customer subscriptions.
*   `uscoordinates_info.csv` (external source): Contains geographic coordinates for U.S. cities, used to enrich customer data.

## Data Warehouse Design

The data warehouse follows a **star schema design**, comprising **dimension tables** and **fact tables**.

### Dimension Tables

*   **Dim\_date:** Contains all dates between 2017 and 2022 with various time attributes (year, quarter, month, day).
*   **Dim\_customer:** Includes customer demographics (age, gender) and location information (city, state, region). Implements **Slowly Changing Dimension (SCD) Type 2** to track historical changes.
*   **Dim\_agent:** Stores agent details (name, role, team, department, experience).
*   **Dim\_channel:** Contains information about customer interaction channels (category, type, platform).
*   **Dim\_plan:** Details the subscription plans (product name, category, type, monthly price, data allowance, internet speed).
*   **Dim\_reason:** Describes the reasons for customer interactions (name, category, type, call priority).

### Fact Tables

*   **fact\_customer\_interactions\_cases:** Records customer interactions across channels, including wait times, resolution times, and satisfaction scores. Implements **incremental load**.
*   **fact\_signups\_record:** Captures data on new product subscriptions, including signup dates, termination dates, discounts, and final prices. Implements **incremental load**.

### Data Warehouse Schema

The fact tables are directly linked to the dimension tables using foreign keys in a star schema.

## ETL Process

The ETL process involved a **staging area** where data was extracted from the LakeHouse, transformed, and prepared for loading into the data warehouse. A **mixed approach of full and incremental load** was used.

### Loading Dimension Tables

Dimension tables were generally loaded using a **full load** approach. This involved **CopyData activities and DataFlows Gen2** to extract and transform data:

*   **Stg\_dim\_customer:** Involved merging customer data with geographic coordinates using a left outer join. Implemented as part of the **SCD Type 2** process.
*   **Stg\_dim\_agent:** Transformed using DataFlow Gen2 to handle varied date formats in the hire date column and calculate agent experience.
*   **Stg\_dim\_date:** Programmatically generated within Microsoft Fabric using a date range from 2017 to the current date.
*   **Stg\_dim\_reason:** Loaded using a Copy Data activity from `reason_info.csv` with no additional transformations.
*   **Stg\_dim\_channel:** Loaded using a Copy Data activity from `channel_info.csv` with no additional transformations.
*   **Stg\_dim\_plan:** Loaded using a Copy Data activity from `plans_info.csv` with no additional transformations.

### Loading Fact Tables

Fact tables were loaded using **DataFlows Gen2** and implemented an **incremental load** strategy.

*   **Stg\_fact\_customer\_interactions\_cases:** Involved correcting date formats and removing duplicate rows. Incremental load ensured only new interactions were processed.
*   **Stg\_fact\_signups\_record:** Involved merging with plan information to retrieve monthly prices and calculating subscription duration and final prices. Incremental load ensured only new sign-ups were processed.

### Extra ETL Work

*   **Incremental Load:** Implemented for the fact tables to process only new or modified records by comparing dates with the existing data in the data warehouse.
*   **Slowly Changing Dimension (SCD) Type 2:** Implemented for the `dim_customer` table to maintain historical records of customer information changes.

### Quality Check Validation

Standard quality check rules were followed to ensure data integrity, including checking business key integrity, uniqueness of dimension attributes, primary key integrity in fact tables, and foreign key relationships. A `log_quality_checks` table recorded any issues found, though no errors were detected.

### Data Loading Process into the Data Warehouse

A pipeline was used to load data from the staging area into the final data warehouse tables. Dimension tables (except `dim_customer`) were cleared before loading. Fact tables were loaded by appending new data, and the corresponding staging fact tables were cleared as part of the incremental load process. `dim_customer` had its own dedicated DataFlow Gen2 for SCD Type 2 implementation.

## Technology Used

*   **Microsoft Fabric**

## Lessons Learned

The project provided valuable experience with new tools like Fabric and reinforced concepts like dimensional modeling and ETL processes. Challenges included navigating the Fabric environment, implementing incremental load and slowly changing dimensions, and integrating diverse data sources. The project highlighted the importance of combining technical skills with a strong understanding of business needs.

## Conclusion

The data warehouse provides TelecomPlus with a well-structured system to transform raw data into actionable insights, enabling analysis of service performance, customer preferences, and revenue patterns, supporting better decision-making. Features like incremental loading and SCD Type 2 ensure efficiency and adaptability. The solution empowers TelecomPlus to improve customer experiences, optimize resource allocation, and drive sustained growth.
