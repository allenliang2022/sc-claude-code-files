# E-commerce Analytics Platform - AI Coding Instructions

## Architecture Overview

This is a **modular e-commerce analytics platform** with two interfaces: a Jupyter notebook for deep analysis and a Streamlit dashboard for business users. The architecture follows a three-layer pattern:

1. **Data Layer** (`data_loader.py`) - ETL pipeline for 6 related e-commerce CSV files
2. **Business Logic** (`business_metrics.py`) - Metrics calculations and visualization generation  
3. **Presentation Layer** (`dashboard.py` + `EDA_Refactored.ipynb`) - Two distinct UIs for different user personas

## Critical Configuration Pattern

**All analysis is driven by year-based configuration variables** set at the top of notebooks:

```python
ANALYSIS_YEAR = 2023        # Primary analysis year
COMPARISON_YEAR = 2022      # For YoY comparisons (optional)
ANALYSIS_MONTH = None       # Filter to specific month or None for full year
DATA_PATH = 'ecommerce_data/'
```

These variables cascade through the entire analysis pipeline. **Never hardcode years** - always reference these constants.

## Data Flow Architecture

### Entry Point Pattern
```python
# Standard initialization across all files
from data_loader import load_and_process_data
loader, processed_data = load_and_process_data('ecommerce_data/')
```

### Sales Dataset Creation
The core analytical dataset is created via `loader.create_sales_dataset()` which:
- Joins 6 CSV files (orders, order_items, products, customers, reviews, payments)
- Applies year/month/status filtering
- Creates derived columns: `purchase_year`, `purchase_month`, `total_item_value`, `delivery_speed_category`

### Business Metrics Pattern
```python
from business_metrics import BusinessMetricsCalculator
metrics_calc = BusinessMetricsCalculator(sales_data)
report = metrics_calc.generate_comprehensive_report(
    current_year=ANALYSIS_YEAR,
    previous_year=COMPARISON_YEAR
)
```

## CSV File Dependencies

The platform expects exactly 6 CSV files in `ecommerce_data/`:
- `orders_dataset.csv` - Order details with dates and status
- `order_items_dataset.csv` - Individual items with pricing
- `products_dataset.csv` - Product catalog with categories
- `customers_dataset.csv` - Customer geographic data
- `order_reviews_dataset.csv` - Review scores and feedback
- `order_payments_dataset.csv` - Payment method data

**Key Join Relationships**: `order_id` connects orders→items→reviews, `product_id` connects items→products, `customer_id` connects orders→customers.

## Visualization Conventions

### Plotly for Dashboard (`dashboard.py`)
- Interactive charts with consistent color scheme: `#1f77b4` (primary), `#ff7f0e` (secondary)
- Currency formatting: `format_currency()` converts to K/M notation ($300K, $2M)
- Trend indicators: Green arrows (↗) for positive, red (↘) for negative changes

### Matplotlib + Seaborn for Notebooks (`business_metrics.py`)
- Business-oriented color palettes via `sns.color_palette('viridis')`
- Consistent figure sizing: `figsize=(12, 6)` for charts, `(15, 8)` for complex plots

## Streamlit Dashboard Architecture

### Caching Strategy
```python
@st.cache_data
def load_dashboard_data():
    # Expensive data loading cached across sessions
```

### Layout Pattern
- **Header**: Year filter (applies globally)
- **KPI Row**: 4 metric cards with trend indicators  
- **Charts Grid**: 2x2 layout with responsive sizing
- **Bottom Row**: Customer experience metrics

Year changes trigger complete dashboard refresh through `st.rerun()`.

## Development Workflows

### Adding New Metrics
1. Extend `BusinessMetricsCalculator` class with new calculation method
2. Add corresponding visualization to `MetricsVisualizer` 
3. Update dashboard layout to display new metric
4. Add to notebook analysis sections

### Testing Changes
```bash
# Dashboard development
streamlit run dashboard.py

# Notebook analysis  
jupyter notebook EDA_Refactored.ipynb
```

### Data Validation
The `EcommerceDataLoader._validate_data()` method checks for required columns. When adding new metrics, ensure your calculations handle missing data gracefully with `pd.isna()` checks.

## Project-Specific Patterns

- **Error Handling**: Uses `warnings.filterwarnings('ignore')` and try-catch with user-friendly messages
- **Optional Dependencies**: Graceful degradation when seaborn unavailable (`HAS_SEABORN` flag)
- **Windows Shell**: All terminal commands documented for PowerShell (see README)
- **Modular Imports**: Each module can be imported independently for reuse in other projects

## Key Files for Understanding Context

- `data_loader.py` lines 254-268: Main entry point function
- `business_metrics.py` lines 420-471: Comprehensive report generation
- `dashboard.py` lines 100-150: Caching and formatting utilities
- `EDA_Refactored.ipynb` cell 3: Configuration constants that drive entire analysis
