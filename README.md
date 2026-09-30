            ##### Investment Banking Operations Analytics & Financial Assurance Suite ########
                             
An end-to-end analytical project designed for Investment Banking Operations, Settlement Desks, and Revenue Assurance teams. The suite combines Python data engineering, predictive modeling (OLS/Logistic Risk Scoring), SQLite persistence, and executive Power BI reporting across two core dashboards:

1. **Client Experience & Operations Health**
2. **Financial Ops & Revenue Assurance**
   
-------------------------

## Dataset Source & Architecture

1. **Source**
	Source: Synthetically generated high-frequency investment banking operational dataset modelled after institutional clearing, trade settlement, and custodian fee structures.
	Storage / Database Engine: Embedded SQLite Database (investment_bank_ops.db).
	Ingestion Method: Python (sqlite3 / pandas) pipeline transforming raw transactional records into normalized relational tables for Power BI reporting.

2. **Core Tables, Row Counts & Column Specifications**
  The data model consists of 5 interconnected tables totaling 10,000 primary trade transactions

3. **Detailed Column Schema (fact_trades)## System Architecture & Data Pipeline**

  The primary transactional table (fact_trades) contains the following 16 columns:
	
  trade_id — Unique identifier for each trade execution.
	trade_date — Date of trade execution and settlement logging.
	client_id — Foreign key mapping to institutional client profiles (dim_clients).
	asset_class — Category of instrument (Equities, Fixed Income, OTC Derivatives, FX Options).
	notional_amount — Total gross trade value in USD.
	internal_clearing_fee — Standard contractual fee calculated by internal systems.
	custodian_charged_fee — Actual fee charged by external clearing custodians.
	settlement_delay_days — Number of business days settlement was delayed beyond T+1/T+2.
	trade_failed — Binary flag (1="Failed Settlement" , 0="Settled On Time" ).
	fail_reason — Root cause breakdown (System Discrepancy, Liquidity Issue, Documentation Delay, Counterparty Fail).
	fee_discrepancy — Variance measure ("custodian_charged_fee"-"internal_clearing_fee" ).
	is_fee_mismatch — Binary indicator flag (1="Discrepancy">$5.00, 0="Matched" ).
	sla_breach_flag — Binary indicator (1="Resolution Time">24" hours" ).
	abs_discrepancy — Absolute monetary value of fee variance (|"fee_discrepancy"|).
	discrepancy_category — Categorical label (Custodian Overcharge, Matched, Internal Undercharge).
	penalty_rate_bps — Regulatory CSDR / T+1 penalty rate applied based on asset class notionals.
  
---------------------------

## System Architecture & Data Pipeline

[ Raw IB Operations Data ] 

[ Python Data Pipeline ] 
├─ Scenario 1: Resolution Time & MTTR Analysis 
├── Scenario 2: CSAT Driver Modeling & Risk Scoring 
├── Scenario 3: Early Warning System (Logistic Regression) 
├── Scenario 4: Fee Reconciliation & Revenue Leakage ($32.5k) 
├── Scenario 5: Settlement Volume Forecasting (OLS Model) 
└── Scenario 6: CSDR / T+1 Regulatory Penalty Simulation ($153.4k)

[ SQLite Database: investment_bank_ops.db ] 
├── fact_trades 
├── dim_clients
├── dim_client_scorecard 
├── fact_cx 
└── fact_volume_forecast

[ Power BI Desktop Analytics ] 
├── Page 1: CX & Operations Health 
└── Page 2: Financial Ops & Revenue Assurance

-------------------------------------

## Prerequisites & Python Environment Setup ### 
Required Python Packages
 ```bash pip install pandas numpy scikit-learn sqlite3

-------------------------------------

Python SQLite Connector Script
--------------------

## Comprehensive Analysis, Modeling Tests & Outcomes

### Scenario 1: Resolution Time (MTTR) & Operational SLA Performance
* **Objective:** Evaluate operational issue resolution speed across trade exception categories.
* **Outcome:** Mean Time to Resolve (MTTR) across all operational tickets stands at 24.3 hours. Trade Exceptions and Settlement Delays drive the highest friction, averaging over 38 hours to resolve.

### Scenario 2 & 3: CSAT Drivers & Predictive Early Warning Model
* **Statistical Test:** Logistic Regression & Ordinary Least Squares (OLS) Regression.
* **Findings:** Resolution time exceeding **40 hours** causes a steep cliff in Client Satisfaction (CSAT drops below 2.0). 
* **Early Warning Machine Learning Model:** Flagged **528 high-risk trades** with failure probabilities > 35%, enabling pre-settlement intervention before operational breach.

### Scenario 4: Fee Reconciliation & Revenue Leakage Audit
* **Methodology:** Evaluated internal contractual clearing rates against custodian charged fees:
  Discrepancy = Custodian Fee - Internal Fee
* **Threshold Filter:** Variance > 5.00
* **Key Outcome:** Identified **$32,545.77 in total overcharges** across **698 trades** (6.98% mismatch rate). Net revenue leakage (undercharging clients internally) was **$0.00**, indicating internal billing accuracy. Equities drove **$14.3K (44%)** of all overcharges due to systematic fee tier misapplication by clearing custodians.

### Scenario 5: Settlement Volume Forecasting
* **Statistical Model:** Linear Regression (OLS) fitted on 365 days of historical settlement data:
  	Volume = beta_0 + beta_1 * DayIndex
* **Key Outcome: Baseline mean volume of 27.4 trades per day. Model slope Beta_1 = -0.0010 projects stable forward settlement volume averaging 27.2 trades per day over the next 30 days. No capacity bottlenecks predicted..

### Scenario 6: CSDR / T+1 Regulatory Penalty Simulation
* **Regulatory Calculation:** 
Penalty Cost = Notional Amount * (Penalty Rate in bps / 10000) * Delay Days
Penalty Rates: Equities (1.0 bps), Fixed Income (0.5 bps), OTC Derivatives (1.5 bps), FX Options (1.5 bps)
* **Key Outcome:** Total portfolio penalty exposure equals $153,372.06 across 1,626 failed trades. OTC Derivatives carry the highest total capital drag ($59,815.55), while FX Options represent the highest unit cost per failure ($136.82 per trade).

----------------

💡 Key Business Insights & Strategic Recommendations

1. Revenue Leakage: $32.5k Lost in Custodian Overcharges
	What the Numbers Say: Out of 10,000 trades, 698 trades (6.98% of all transactions) were overcharged by clearing custodians, resulting in $32,545.77 in direct financial overpayments. Equities ($14.3K) and Fixed Income ($7.9K) account for over 68% of all overcharges.
	Why it Matters: Custodians are applying incorrect fee tiers on routine trades. Because internal system fees matched contractual agreements perfectly, 100% of this leakage is recoverable money owed back to the bank.
	Actionable Recommendation: Immediately execute the Dispute Audit Queue (Zone 4) to issue formal clawback claims against external custodians. Going forward, implement automated fee validation before settlement instruction release to block overcharged trades at gateway level.

2. Regulatory Capital Drag: $153.4k CSDR Penalty Exposure
	What the Numbers Say: Out of 1,626 trade failures, total regulatory fine exposure under European CSDR / T+1 settlement rules equals $153,372.06. OTC Derivatives ($59.82K) and Equities ($56.12K) represent over 75% of total penalty cost. FX Options carry the highest failure penalty rate ($136.82 per trade).
	Why it Matters: A trade fail in OTC Derivatives or FX Options costs up to 3x more than a fail in Fixed Income due to higher penalty rates and larger notionals.
	Actionable Recommendation: Prioritize settlement ops resources strictly by penalty risk per trade, not simple queue order. Establish an automated escalation rule: any OTC Derivative or FX Option trade delayed by >1 day must trigger an automatic pre-fail alert to the settlement desk manager.

3. Client Churn Risk: The "40-Hour" CSAT Drop-off
	What the Numbers Say: Client satisfaction (CSAT) drops precipitously from 4.8/5.0 down to 1.0 as trade resolution time (MTTR) crosses 40 hours. Furthermore, 528 trades were identified as having a >35% probability of failure before settlement date.
	Why it Matters: Institutional clients tolerate occasional errors, but they do not tolerate prolonged silence. Trade exception resolution times exceeding two days directly cause institutional client churn.
	Actionable Recommendation: Introduce a strict 36-Hour SLA Ceiling for trade exception resolution. Deploy the Early Warning Queue to flag high-risk trades before they fail, allowing operations teams to proactively reach out to clients prior to SLA breach.

4. Daily Operational Capacity & Settlement Volume Stability
	What the Numbers Say: Daily settlement volume averages 27 trades/day with a maximum historical peak of 46 trades/day. 30-day forward predictive modeling shows a flat, stable volume line.
	Why it Matters: Existing operational team bandwidth is sufficient to handle incoming trade volume without needing extra headcount. Operational failures are caused by workflow delays, not volume overcapacity.
	Actionable Recommendation: Reallocate existing operational headcount from manual trade entry to automated exception management and fee audit reconciliation.
