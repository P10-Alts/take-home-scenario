# Data Engineering Take-Home Assignment

### Overview

Build a data enrichment pipeline that integrates Preqin alternative investment data with our Salesforce Account records to enhance our investment committee's decision-making process and relationship management capabilities.

This repo is to be used as the guideline for the exercise. Please to not attempt to push any changes to the repo.

Be prepared to run your solution locally during the presentation once you complete the exercise.
We will want to see the following functionality, and an explanation behind your implementation.

- Fuzzy matching between Salesforce accounts and Preqin accounts
- Data quality scoring between Salesforce accounts and Preqin accounts
- Enriched Salesforce opportunities using Preqin investor data

**Required Tooling**
- python
- SQL

**Recommended Tooling**
- DuckDB
- Docker

**Estimated Time**: 3-4 hours

**Due Date**: 5 days from receipt

---

## Business Context

You're joining our Data Solutions team as a Data Engineer. Our firm manages relationships with institutional investors, fund managers, and service providers in the alternative investment ecosystem. Currently, our sales and relationship management teams rely on Salesforce to track accounts, but they lack comprehensive market intelligence about these organizations' investment activities, fund performance, and strategic positioning.

**The Challenge:**
Our Salesforce Account records contain basic company information, but our relationship managers need deeper insights to:

- Understand prospects' investment strategies and fund performance
- Identify cross-selling opportunities across different asset classes
- Prioritize outreach based on fund raising activity and AUM growth
- Tailor proposals based on geographic and sector focus areas

**The Solution:**
Integrate Preqin's comprehensive alternative investment database with our Salesforce Account data to create enriched account profiles that provide 360-degree visibility into our prospects' and clients' investment activities.

---

## Data Sources Overview

### 1. Salesforce Account Data (Primary Source)

**Access Method:** CSV
- [salesforce_accounts.csv](https://github.com/P10-Alts/take-home-scenario/edit/main/mock_data/salesforce_accounts.csv)
- [salesforce_contacts.csv](https://github.com/P10-Alts/take-home-scenario/edit/main/mock_data/salesforce_contacts.csv)
- [salesforce_opportunities.csv](https://github.com/P10-Alts/take-home-scenario/edit/main/mock_data/salesforce_opportunities.csv)


**Key Salesforce Objects:**

```
Account (Standard Object):
- Id, Name, Type, Industry, BillingCountry
- Website, Phone, NumberOfEmployees
- AnnualRevenue, Ownership, AccountSource
- Custom Fields: Account_Type__c, Investment_Focus__c, AUM_Range__c, Preqin_External_Id

Contact (Standard Object):
- AccountId, Name, Title, Email, Phone
- Custom Fields: Seniority_Level__c, Decision_Maker__c

Opportunity (Standard Object):
- AccountId, Name, StageName, Amount, CloseDate
- Type, LeadSource, Custom Fields: Service_Line__c
```

### 2. Preqin Alternative Investment Data (Enhancement Source)

**Access Method:** JSON
- [preqin_fund_managers.json](https://github.com/P10-Alts/take-home-scenario/edit/main/mock_data/preqin_fund_managers.json)
- [preqin_investors.json](https://github.com/P10-Alts/take-home-scenario/edit/main/mock_data/preqin_investors.json)

**Available Preqin Endpoints:**

```
Fund Managers:
GET /fund-managers/search
- firm_name, headquarters, total_aum, funds_managed
- asset_classes, geographic_focus, investment_strategy
- founded_year, employee_count, recent_fundraising

Investors:
GET /investors/search
- investor_name, investor_type, headquarters, total_commitments
- asset_class_allocations, geographic_preferences
- recent_commitments, portfolio_companies
```

### 3. Company Matching Challenges

**Entity Resolution Required:**

- Salesforce Account names may not exactly match Preqin firm names
- Multiple legal entities under same parent company
- Subsidiary vs. parent company relationships
- Name variations (abbreviations, legal suffixes, etc.)

**Example Matching Scenarios:**

```
Salesforce Account: "KKR & Co. Inc."
Preqin Firm: "Kohlberg Kravis Roberts & Co. L.P."

Salesforce Account: "Blackstone Group"
Preqin Firm: "The Blackstone Group L.P."

Salesforce Account: "TPG Capital"
Preqin Firm: "Texas Pacific Group"
```

---

## Technical Requirements

Implement an ETL pipeline leveraging python/SQL to read data from the Salesforce data sources, and enrich with the Preqin feed api responses.
The enriched account records should be persisted to a SQL table with the schema defined below.

We recommend hosting a db locally via DuckDB for this, but feel free to implement this however you prefer (Docker etc.)

The final product should include the following features:

### 1. Entity Matching Algorithm

Implement sophisticated matching logic to connect Salesforce Accounts with Preqin entities:

```python
def match_account_to_preqin(sf_account, preqin_firms):
    """
    Multi-stage matching process:
    1. Exact name matching (case-insensitive)
    2. Fuzzy string matching (>85% similarity)
    3. Website domain matching
    4. Alternative name/DBA matching
    5. Manual review queue for edge cases
    """
    pass
```

### 2. Data Quality Scoring

```python
def calculate_enrichment_quality(match_confidence, data_completeness, data_freshness):
    """
    Composite score considering:
    - Matching confidence (0.0-1.0)
    - Data completeness (% of fields populated)
    - Data freshness (days since last Preqin update)
    """
    pass
```

### 3. Competitor Analysis

```python
def analyze_competitor_opportunities(account_preqin_data, firm_portfolio):
    """
    Identify business development opportunities:
    - Competitive intelligence
    - Cross-selling opportunities
    """
    pass
```

### Target Enrichment Schema

Enhance Salesforce Account records with the following Preqin-derived fields:

```sql
-- New Custom Fields to Add to Salesforce Account Object
CREATE TABLE account_enrichment (
    salesforce_account_id VARCHAR(18) PRIMARY KEY,

-- Preqin Matching Info
    preqin_firm_id VARCHAR(50),
    preqin_firm_name VARCHAR(255),
    matching_confidence_score DECIMAL(3,2),
    matching_method VARCHAR(50),-- 'exact', 'fuzzy', 'manual', 'domain'

-- Fund Manager Profile
    preqin_firm_type VARCHAR(50),-- 'Fund Manager', 'Investor', 'Service Provider'
    total_aum_usd DECIMAL(15,2),
    funds_managed_count INTEGER,
    founded_year INTEGER,
    headquarters_city VARCHAR(100),
    headquarters_country VARCHAR(50),

-- Investment Strategy
    primary_asset_class VARCHAR(100),
    secondary_asset_classes TEXT,-- JSON array
    geographic_focus TEXT,-- JSON array
    investment_strategies TEXT,-- JSON array

-- Recent Activity (Last 24 Months)
    active_fundraising_count INTEGER,
    recent_fund_closes_count INTEGER,
    recent_deals_count INTEGER,
    largest_recent_fund_size DECIMAL(15,2),

-- Competitor Analysis
    competitive_overlap_score DECIMAL(3,2),

-- Metadata
    last_enriched_date DATE,
    data_freshness_score DECIMAL(3,2),
    enrichment_status VARCHAR(20)-- 'enriched', 'partial', 'no_match', 'error'
);
```
