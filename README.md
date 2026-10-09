# Retail Sales Analytics & Customer Segmentation

Python, SQL, and Machine Learning educational portfolio for an MSc Data Science application.

**AI disclosure:** Substantial project implementation and writing were AI-generated. This is a learning portfolio starting point. The applicant must independently understand and validate the work and accurately describe personal contributions.

## Quick start on Windows with VS Code

Extract this folder, open it in VS Code, then run in PowerShell:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-lock.txt
# Workbook is bundled. Download again only if it is missing:
# .\.venv\Scripts\python.exe download_data.py
.\.venv\Scripts\python.exe run_pipeline.py
.\.venv\Scripts\python.exe -m pytest -q
.\.venv\Scripts\python.exe -m streamlit run dashboard/app.py
```

Select `.venv` as the Python interpreter and notebook kernel in VS Code. Install Microsoft's Python and Jupyter extensions. Run the four notebooks in numbered order after the pipeline. Python 3.12 is recommended to match the tested environment. Requirements-lock pins the versions actually used here, but Windows execution itself was not tested. Set `$env:OMP_NUM_THREADS="1"` if numerical libraries oversubscribe CPU threads.

## Project structure and reproducibility

`data/raw/Online Retail.xlsx` is the real, unchanged source. `run_pipeline.py` rebuilds processed CSVs, SQLite, models, metrics, and 16 charts. `run_supplemental.py` adds the dendrogram, bootstrap interval, and outlier sensitivity. `build_documents.py` regenerates report and interview PDF files after these analyses. `presentation/build.mjs` generates the presentation using the OpenAI artifact-tool runtime and is environment-specific, unlike the portable core Python project. All notebook code cells were executed in-process with IPython and saved outputs. Standard Jupyter kernel startup was blocked by sandbox network-interface restrictions. Run `python run_supplemental.py` after the main pipeline and `python build_documents.py` after both to refresh report statistics. The interview text is explanatory and should be checked after substantive methodological changes.

The full dataset and generated outputs are included in the ZIP. Large reproducible raw and database files are gitignored by default. Commit source, notebooks, small charts, reports, and documentation, then provide the UCI download procedure. No GitHub repository was created or published. Models are joblib artifacts: only load trusted files and use compatible dependencies. Ward models store training labels and cannot predict new customer clusters directly.

## Business problem and research objectives

Understand sales performance, purchasing behavior, customer value, and observed retention. Compare whether K-Means and Ward produce useful geometric groupings of standardized RFM features. This project performs descriptive unsupervised analysis, not supervised churn prediction or causal revenue optimization.

## Dataset source, license, and limitations

UCI Online Retail, dataset 352, Chen (2015), DOI https://doi.org/10.24432/C5BW33 . Source: https://archive.ics.uci.edu/dataset/352/online+retail . CC BY 4.0 https://creativecommons.org/licenses/by/4.0/ . Original dates: December 2010 to December 2011. Unit prices are GBP. Many customers are wholesalers. Dataset and transformed copies retain this attribution. Code is offered under MIT (LICENSE); dataset rights remain CC BY 4.0. Read `reports/results.json` for the workbook SHA-256 and observed missingness.

## Technologies

Python, Pandas, NumPy, SciPy, scikit-learn, SQLite, Matplotlib, Seaborn, Plotly, Streamlit, Jupyter, pytest, joblib, and ReportLab.

## Problem statement and research objectives

The business problem is to understand when sales occur, which products generate transaction value, and how purchasing behavior differs across observed customers. Research questions ask whether a small RFM feature space supports interpretable groups, how centroid-based and hierarchical clustering compare, and how missing customer identity limits conclusions. Deliverables include an auditable pipeline, executed notebooks, SQL outputs, saved model artifacts, and a dashboard.

## Literature and methodological background

The dataset-associated study by Chen, Sain, and Guo motivates RFM segmentation in online retail. This project implements its own transparent workflow rather than reproducing that paper’s experiments. K-Means minimizes within-cluster squared Euclidean distances. Ward hierarchical clustering merges the pair of clusters that produces the smallest increase in within-cluster variance. Silhouette compares within-cluster proximity with the nearest alternative cluster. PCA rotates a centered feature space into orthogonal directions of maximum variance. Official scikit-learn documentation provides implementation and metric definitions.

## Dataset description and provenance

UCI Online Retail (dataset 352) contains invoices, stock codes, descriptions, quantities, timestamps, unit prices in sterling, customer identifiers, and countries. Observed dates span 1 December 2010 through 9 December 2011. The retailer sells gifts and includes wholesalers. The original workbook is bundled without alteration. The UCI page licenses the dataset under CC BY 4.0. Attribution is required when sharing adaptations. This project is unrelated to a current retailer and uses no synthetic transactions. Source SHA-256: 43465a06f2ccf7c8b5bd2892bc7defb52f97487934fe93b16ae4c3936424676d

## Data preprocessing

Observed missing values include 135,080 customer IDs and 1,454 descriptions. This direct workbook audit takes precedence over generic metadata about missingness. Remove 5,268 exact duplicates as an explicit assumption: repeated identical lines may be genuine, so this choice can underestimate sales. Then exclude 11,763 rows through a combined invalid-sales mask. Cancellation, nonpositive quantity, and nonpositive price flags overlap and must not be added together. Convert timestamps and numeric fields, trim identifiers and text, and assign Unknown to missing descriptions. Preserve anonymous positive sales for EDA but exclude them from customer models. Revenue equals quantity times unit price.

## Outliers and consistency

IQR upper-fence counts are {'Quantity': 27111, 'UnitPrice': 37827, 'Revenue': 42624}. These flags identify unusual observations, not proven errors. Wholesale purchases can be large, so the main analysis retains extremes and applies log1p transformation for clustering. Products have 217 stock codes with multiple descriptions, so the SQL product dimension uses a modal description. No multi-customer invoice conflict was observed. Signed transaction value after deduplication is £9,726,006.95, but it includes administrative and return entries and is not an audited accounting figure. The main sales KPI is positive transaction value, including non-merchandise stock codes such as postage.

## EDA results

The dataset contains 19,960 positive-sale invoices with average order value £533.17. Identified customers contribute £8,887,208.89; anonymous sales contribute the difference. The observed repeat-customer share is 65.58%. Monthly totals, product revenue and units, order-value skew, country contributions, weekday patterns, and cohort retention are exported as charts and underlying CSV results. November 2011 is the highest full-month positive-sales period in this workflow. The terminal December period covers only nine days. One annual cycle cannot establish stable recurring seasonality.

## SQL relational analysis

The database contains customers, products, invoices, and sales_lines with primary keys, foreign keys, positive-value constraints, and join indexes. Fourteen queries cover revenue, orders, country performance, product sales, customer rankings, frequency, recency, cumulative revenue, and prior-month comparisons. CTEs build order and customer aggregates. DENSE_RANK, LAG, and window SUM demonstrate ranking and time-oriented SQL. SQLite foreign-key checks and SQL-to-Pandas revenue reconciliation validate the imported database.

## RFM methodology

At reference date 10 December 2011, recency is calendar days since the last observed purchase, frequency counts distinct invoices, and monetary value sums positive transaction value. Recency scores reverse percentile ranks. Frequency and monetary scores increase with percentile ranks. Scores range from 1 to 5 and preserve ties, so group sizes need not be equal. Ordered rules assign High-value first (R>=4 and M>=4), then Loyal (F>=4 and R>=3), then New (observed tenure<=30 days and frequency<=2), then At-risk (R<=2 and F>=3 or M>=3), then Inactive (R<=2), with Regular as the default. Rule precedence means a newly observed high-value customer receives High-value. These thresholds are business assumptions rather than learned churn labels.

## Machine Learning methodology

Select Recency, Frequency, and Monetary. Apply log(1+x) to reduce right skew and StandardScaler to give features comparable variance. Use identical features for both algorithms, evaluate k=2 through 8, and select maximum sampled silhouette separately for each. K-Means uses k-means++ initialization, 20 restarts, and seed 42. Ward uses Euclidean geometry and variance-minimizing merges. No train/test accuracy is appropriate because there are no labels and the goal is descriptive segmentation of the observed population. This internal selection is exploratory, not evidence of future generalization. The elbow curve provides a diagnostic rather than an automatic independent selection rule.

## Experimental results

K-Means selects k=2 with sampled silhouette 0.4252, Davies–Bouldin 0.8913, and Calinski–Harabasz 4376.83. Ward selects k=2 with sampled silhouette 0.4096, Davies–Bouldin 0.8353, and Calinski–Harabasz 3091.29. Silhouette uses a fixed random sample of at most 2,000 customers. Other metrics use all customers. Higher silhouette and Calinski–Harabasz are preferred, while lower Davies–Bouldin is preferred. The metrics disagree on the relative ordering, so neither algorithm is universally superior. Agreement ARI is 0.3926. Five alternative K-Means seeds give ARI 1.0 relative to the selected fit, which establishes seed stability only.

## Advanced statistical analytics

The first two PCA components explain 93.86% of transformed feature variance. Clusters are fit in three dimensions, and the PCA display is a lossy projection. Spearman correlation between frequency and monetary value is 0.8072. Its computed p-value underflows to zero and should be reported as p<0.001, not literally zero. The relationship partly follows from monetary value accumulating with orders and cannot establish causation. A customer bootstrap with 1,000 resamples gives a conditional 95% interval of 64.22%–66.94% for repeat share. Removing the top 1% by monetary value and refitting K-Means with fixed k and the same scaler yields ARI 0.9339 on retained customers. A dendrogram uses a fixed random sample of 500 customers and must not be interpreted as the full fitted hierarchy.

## Customer retention and business recommendations

Cohorts use first observed purchase month. Each cell divides distinct active cohort customers by original cohort size. Observable inactivity is zero and unobservable future months remain missing. December 2011 activity is censored by the early endpoint. High-value and loyal groups are candidates for service improvements and relevant repeat-purchase offers. At-risk groups are candidates for consent-based reactivation trials. New customers may benefit from onboarding. Inactive groups require a cost-sensitive outreach strategy. Estimate treatment effects with randomized experiments, holdout groups, and incremental margin before deploying campaigns.

## Discussion and cluster interpretation

The two-cluster solution captures a broad purchasing gradient rather than six discrete marketing personas. Median RFM values and customer/revenue shares are the basis for describing each numbered cluster. Numerical labels have no ordinal meaning and may switch across runs. The rule-based RFM segment and learned cluster answer different questions. Return-linked monetary value, longer history, product categories, and temporal evaluation could change these groups. Ward does not expose a predict method for new customers. Saved Ward assignments refer only to the training population.

## Limitations

This is one historical retailer with wholesale customers, no profit margins, no demographics, and a short observation period. Customer-ID missingness may bias segmentation. Exact duplicates may represent genuine repeat lines. Positive-sales analysis ignores returns in customer monetary values and retains operational stock codes. First observed dates are left-censored and later retention is right-censored. Internal clustering metrics reward geometric separation and do not establish marketing usefulness. Bootstrap intervals assume independent customers within this observed population. Joblib artifacts require trusted sources and compatible dependency versions.

## Conclusion and future work

The project provides an executable example of audit-driven retail analytics and interpretable unsupervised segmentation. Its strongest contribution is a traceable link between raw data, cleaning assumptions, reported metrics, model artifacts, and user-facing outputs. Future work should link returns to purchases, examine net value and margins, collect multiple annual cycles, test temporal stability, explore alternative distance models, and evaluate retention interventions through experiments.

## Results and charts

![Monthly sales](visualizations/01_monthly_sales.png)
![Model selection](visualizations/13_silhouette.png)
![Cohort retention](visualizations/09_cohort_retention.png)

All 18 high-resolution PNGs are under `visualizations/`. Customer and cluster profiles, query result CSVs, evaluation tables, models, and retention matrices are included. For exact model metrics, consult `reports/results.json` and `data/processed/cluster_metrics.csv`. Dashboard filters affect sales and the displayed customer subset; they do not redefine the fixed full-history segmentation.

## Validation and use

See `reports/validation.json` and `reports/VALIDATION.md` for executed checks and explicit limitations. These files describe assistant-side testing, not work independently completed by the applicant. No accuracy or business lift is claimed. See the academic PDF, slides, resume guidance, and 25 interview answers under `reports/`.

## References

Chen, D. (2015). Online Retail [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33. CC BY 4.0: https://creativecommons.org/licenses/by/4.0/ .
Chen, D., Sain, S. L., and Guo, K. (2012). Data mining for the online retail industry: A case study of RFM model-based customer segmentation using data mining. Journal of Database Marketing & Customer Strategy Management, 19, 197–208. https://doi.org/10.1057/dbm.2012.17 .
Scikit-learn documentation: Clustering. https://scikit-learn.org/stable/modules/clustering.html .
Scikit-learn documentation: Silhouette score. https://scikit-learn.org/stable/modules/generated/sklearn.metrics.silhouette_score.html .
Scikit-learn documentation: PCA. https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html .
Sources accessed 8 October 2026.