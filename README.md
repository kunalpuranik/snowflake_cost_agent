# Snowflake Cost Analysis Agent

## Overview

This project provides a complete solution for analyzing Snowflake costs using Cortex Analyst. It includes optimized SQL views and a semantic model that enables natural language queries about your Snowflake spending.

## What's Included

- **cost_agent.ipynb**: A Jupyter notebook that sets up everything needed for the cost analysis agent
- **query.sql**: Original SQL queries (reference only)
- **README.md**: This file with setup instructions

## Features

The cost analysis agent can answer questions about:

- **Daily costs** broken down by service type
- **Warehouse costs** with compute and cloud services breakdown
- **Product category costs** (AI & ML, Data Transformation, Storage, etc.)
- **Query-level costs** with user and database attribution
- **Cost trends** with rolling averages

## Prerequisites

1. **Snowflake Account** with Cortex features enabled
2. **ACCOUNTADMIN Role** (required to access ACCOUNT_USAGE views)
3. **Snowsight Access** (Snowflake's web UI)

## Setup Instructions

### Step 1: Import the Notebook

1. Log into your Snowflake account via Snowsight
2. Navigate to **Projects** → **Notebooks**
3. Click **"Import .ipynb file"** or **"+ Notebook"** → **"Import from .ipynb"**
4. Upload the `cost_agent.ipynb` file
5. Select any warehouse to run the notebook (the notebook will create its own dedicated warehouse)

### Step 2: Configure the Notebook

1. Open the imported notebook
2. In the first code cell (Step 1), update these variables:
   ```sql
   SET DATABASE_NAME = 'COST_ANALYSIS_DB';  -- Your database name
   SET SCHEMA_NAME = 'COST_VIEWS';          -- Your schema name
   SET COST_PER_CREDIT = 2.5;               -- Your cost per credit in USD
   ```

### Step 3: Run All Cells - Everything is Automated! 🚀

1. Click **"Run All"** in the notebook
2. The notebook will automatically:
   - **Create a dedicated warehouse** (`COST_AGENT_WH`) for the agent
   - Create a database and schema (if they don't exist)
   - Create 5 optimized views for cost analysis
   - Generate a semantic model (YAML)
   - Upload the semantic model to a Snowflake stage
   - **Automatically create the Cortex Analyst agent**

### Step 4: Start Using the Agent!

After running all cells:

1. Navigate to **Snowflake Intelligence** in Snowsight (left sidebar)
2. Find your **Snowflake Cost Analyst** agent
3. Start asking questions immediately!

You can now ask natural language questions like:

- "What is my daily AI cost?"
- "Show me costs for warehouse COMPUTE_WH"
- "Show me the snowflake cost for the last 30 days. Break it down by product categories."
- "What are my top 5 most expensive warehouses this month?"
- "How much did I spend on compute vs storage last week?"
- "What is the trend of my daily costs over the past 90 days?"
- "Which users are generating the most costs?"
- "What databases are most expensive to query?"
- **"How much does the cost agent itself cost?"** ← Track the agent's own costs!

## What Gets Created

### Dedicated Warehouse
**COST_AGENT_WH** - A dedicated XSMALL warehouse that:
- Runs all cost agent queries
- Auto-suspends after 1 minute of inactivity
- Allows you to track exactly how much the cost agent itself costs
- Check its costs with: `SELECT * FROM WAREHOUSE_COSTS_SUMMARY WHERE WAREHOUSE_NAME = 'COST_AGENT_WH'`

## Views Created

The notebook creates 5 optimized views:

### 1. DAILY_COSTS_BY_SERVICE
Daily costs broken down by Snowflake service type (Compute, Storage, Snowpark, etc.)

### 2. WAREHOUSE_COSTS_SUMMARY
Warehouse-level costs with breakdown of compute vs cloud services

### 3. QUERY_COSTS_DETAIL
Query-level cost estimates with user, database, and execution details

### 4. COST_BY_PRODUCT_CATEGORY
Costs grouped by major product categories:
- AI & Machine Learning
- Data Transformation & Compute
- Data Storage
- Data Sharing & Collaboration
- Cloud Services
- Serverless Features
- Search & Optimization

### 5. COST_SUMMARY_METRICS
Daily cost summaries with 7-day and 30-day rolling averages

## Cost Calculation Methodology

### Warehouse and Service Costs
- Uses `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY` for actual credit consumption
- Uses `SNOWFLAKE.ACCOUNT_USAGE.METERING_DAILY_HISTORY` for service-level costs
- Multiplies credits by your configured cost per credit

### Query-Level Costs
- Proportionally allocates warehouse costs to queries based on execution time
- Formula: `(Query Hours / Total Warehouse Hours) × Warehouse Credits × Cost Per Credit`
- Provides estimates for understanding which queries/users drive costs

## Important Notes

### Data Latency
- Views use `SNOWFLAKE.ACCOUNT_USAGE` which has a latency of **45 minutes to 3 hours**
- Historical data is available for up to **1 year** (90 days for query details)

### Cost Per Credit
- Default is set to $2.5 per credit
- Update the `$COST_PER_CREDIT` variable to match your actual rate
- You can find your rate in your Snowflake contract or billing dashboard

### Permissions
- Requires **ACCOUNTADMIN** role to access ACCOUNT_USAGE views
- To share with other users, grant them access to the database, schema, and views (see Step 8 in the notebook)

## Customization

### Adding More Verified Queries
Edit the `verified_queries` section in the semantic model (Cell 14) to add more example questions and their corresponding SQL.

### Modifying Product Categories
Edit the `COST_BY_PRODUCT_CATEGORY` view (Cell 8) to adjust how services are categorized.

### Adjusting Time Ranges
- Views store 365 days of data (90 days for query details)
- Modify the `WHERE` clauses in view definitions to change retention periods

## Troubleshooting

### Views are empty
- Wait a few hours for ACCOUNT_USAGE data to populate
- Ensure you have activity in your Snowflake account

### Agent doesn't understand questions
- Try rephrasing your question
- Add more verified queries to the semantic model
- Check that the semantic model was uploaded correctly to the stage

### Permission errors
- Ensure you're using ACCOUNTADMIN role
- Grant appropriate permissions to other users (see Step 8 in notebook)

### Cost calculations seem incorrect
- Verify your `$COST_PER_CREDIT` setting
- Remember that query costs are estimates based on proportional allocation
- Check your Snowflake billing dashboard for actual costs

## Architecture

```
┌─────────────────────────────────────────┐
│   SNOWFLAKE.ACCOUNT_USAGE Views         │
│   (Raw billing and usage data)          │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│   Optimized Cost Views                  │
│   - DAILY_COSTS_BY_SERVICE              │
│   - WAREHOUSE_COSTS_SUMMARY             │
│   - QUERY_COSTS_DETAIL                  │
│   - COST_BY_PRODUCT_CATEGORY            │
│   - COST_SUMMARY_METRICS                │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│   Semantic Model (YAML)                 │
│   - Defines tables, dimensions, measures│
│   - Provides synonyms and descriptions  │
│   - Includes verified queries           │
└──────────────┬──────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────┐
│   Cortex Analyst Agent                  │
│   - Natural language interface          │
│   - Generates SQL from questions        │
│   - Returns results and visualizations  │
└─────────────────────────────────────────┘
```

## Support and Feedback

For issues or questions:
1. Check the troubleshooting section above
2. Review Snowflake's Cortex Analyst documentation
3. Verify your Snowflake account has Cortex features enabled

## License

This project is provided as-is for use with Snowflake accounts.

## Version History

- **v1.0** (2025-11-11): Initial release with 5 views and semantic model

