# Retail Demand Forecasting and Inventory Reorder Analysis

Iowa State University, MS Information Systems, Creative Component (2026)

A forecasting and reorder planning workflow that runs on two very different retail datasets, Walmart store sales and Instacart online grocery orders, through one shared weekly demand format.

Tableau dashboard: https://public.tableau.com/app/profile/anvitha.alla

## The question

Retailers have to decide how much stock to hold when demand is uncertain. Too little means lost sales. Too much means storage cost and tied up money. This project asks two things:

1. Can one weekly demand pipeline work for both in store retail and online grocery data?
2. Are simple baseline forecasts good enough to support reorder decisions?

## Data

Both datasets are public and come from Kaggle.

* Walmart Store Sales Forecasting. Weekly sales by store and department.
* Instacart Market Basket Analysis. Order level records that I aggregated into weekly demand by aisle.

The raw files are not included in this repository. Download them from Kaggle to rerun the work.

## Workflow

1. Cleaned both datasets in Python. Fixed null values and data types, and linked Instacart ids to aisle names.
2. Standardized both into one table with four fields: dataset, entity_id, week_start and y (weekly demand). The combined table has 423,579 rows, of which 421,570 are Walmart and 2,009 are Instacart.
3. Loaded the table into MySQL Workbench and ran validation queries for row counts, entity counts, null values and duplicates. These checks caught a cleaned file that contained only Instacart, and a Tableau export that was also missing Walmart. Both were fixed before the dashboard was built.
4. Built two baseline forecasts. Lag 1 predicts this week using last week. MA4 predicts this week using the average of the previous four weeks.
5. Evaluated both with MAE, RMSE and MAPE.
6. Extended the forecasts into reorder planning with safety stock, reorder point and recommended reorder quantity.
7. Built a Tableau dashboard with six views: Instacart demand trends, highest Walmart demand by month, actual against predicted demand, model comparison, and reorder quantity with safety stock for each dataset.

## Results

Instacart (lag 1 against MA4):
* MAE 6,623.77 against 8,242.21
* RMSE 27,502.72 against 27,834.34
* Lag 1 was better on all three measures.

Walmart (lag 1 against MA4):
* MAE 2,102.47 against 2,155.02, so lag 1 was slightly better.
* RMSE 7,206.83 against 6,518.65, so MA4 was better.
* Neither model won clearly. The answer depends on the metric.

MAPE values were very large, for example 609.89 for Walmart MA4, because many weekly demand values are small or close to zero. I treated MAE and RMSE as the reliable measures.

## Reorder planning

* Safety Stock = z x standard deviation of demand x square root of lead time in weeks
* Reorder Point = (predicted demand x lead time) + Safety Stock
* Recommended Reorder Quantity = max(0, Reorder Point minus Current Inventory)

## Challenges and how I handled them

1. **The two datasets did not match.** Walmart was already weekly sales by store and department. Instacart was order level data. I aggregated Instacart into weekly demand and put both into one four field structure so the same forecasting and evaluation steps could run on each.
2. **Entity ids did not line up.** The Instacart ids were hard to interpret on their own, so I linked them to aisle names. Some identifier fields were being read as numbers, so I converted them to text and handled the null values.
3. **A cleaned file silently dropped Walmart.** My combined weekly demand file contained only Instacart. I found it by loading the file into MySQL and checking which datasets and how many rows were present. I fixed the file before building anything on it.
4. **Tableau showed only Instacart.** One export had the same problem. I traced it back with SQL validation queries and rebuilt the data source.
5. **The scales were too different for one chart.** Walmart values were so much larger that they hid the Instacart bars. I split the reorder views into one worksheet per dataset.
6. **The metrics disagreed.** On Walmart, lag 1 had the lower MAE but MA4 had the lower RMSE. MAPE values were extremely large because many weekly values are small. I reported both datasets as they came out and relied on MAE and RMSE instead of claiming one model won everywhere.
7. **There was no inventory data.** I simulated current inventory from average weekly demand and labeled the reorder outputs as a demonstration of the method.

## Limitations

* The datasets contain no on hand inventory, so current inventory was simulated from average weekly demand. The reorder quantities show how the framework works. They are not real stocking recommendations.
* Aggregating to weekly demand removes some detail from the original data.
* Only two simple baseline models were tested.
* MAPE was not reliable on this data.

## Future work

* Test ARIMA, Prophet or LSTM models.
* Add holidays, promotions, pricing and seasonality.
* Replace simulated inventory with real inventory and lead time data.
* Apply the same framework to more retail datasets.

## Tools

Python, Jupyter Notebook, MySQL Workbench, Tableau Public

## Files

* The notebook forecast_model_comparison.ipynb builds the lag 1 and MA4 forecasts and the error metrics.
* The report PDF is the full written creative component.
* The presentation PPTX is the final oral presentation.
* The dashboard PDFs show the Tableau views.

To rerun the notebook, change the file paths at the top to match your own folders.
