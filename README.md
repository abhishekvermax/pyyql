# PYYql - Declarative PySpark SQL Engine

[![Python Version](https://img.shields.io/badge/python-3.7%2B-blue.svg)](https://www.python.org/downloads/)
[![PySpark Version](https://img.shields.io/badge/pyspark-3.0%2B-orange.svg)](https://spark.apache.org/downloads.html)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)]()

> **Transform complex PySpark SQL operations into simple, debuggable YAML configurations**

## 🎯 Vision

PYYql bridges the gap between SQL's declarative nature and PySpark's programmatic complexity. By defining data transformations in YAML, developers can create maintainable, version-controlled, and easily debuggable data pipelines that preserve column-level lineage and business logic as living documentation. Every transformation becomes traceable, auditable, and collaborative across technical and business teams.

## ✨ Why PYYql?

### **The Problem**
- **Complex PySpark Code**: Multi-table joins and aggregations become verbose and hard to maintain
- **SQL Debugging Nightmare**: Traditional SQL offers limited debugging and traceability options  
- **Non-Version-Controllable**: SQL queries are often stored as strings, making them difficult to version and review
- **Lost Data Lineage**: Difficulty tracking column transformations and business logic through complex pipelines
- **Metadata Chaos**: Business rules and column meanings scattered across code comments and documentation
- **Steep Learning Curve**: PySpark DataFrame API requires deep knowledge for complex operations

### **The Solution**
PYYql transforms this:
```python
# Complex PySpark code
df1.alias("emp") \
   .join(df2.alias("man"), F.col("emp.manager_id") == F.col("man.manager_id"), "left") \
   .join(df3.alias("dep"), F.col("man.department_id") == F.col("dep.department_id"), "left") \
   .select(
       F.col("emp.emp_name").alias("employee_name"),
       F.col("dep.department_name").alias("dept_name")
   ) \
   .where(F.col("emp.status") == "active") \
   .orderBy("employee_name")
```

Into this:
```yaml
# Clean, declarative YAML
constructed_table_name: HR_view

dependencies:
  emp: { table_name: employee, type: source }
  man: { table_name: manager, type: source }
  dep: { table_name: department, type: source }

join_conditions:
  - ("emp", "man", "emp.manager_id", "man.manager_id")
  - ("man", "dep", "man.department_id", "dep.department_id")

select:
  emp.emp_name: employee_name
  dep.department_name: dept_name

filter_condition:
  - emp.status == "active"

sort_condition:
  - (emp.emp_name, asc)
```

## 🚀 Key Benefits

### **🔍 Enhanced Debugging**
- **Step-by-step Execution**: Trace exactly what happens at each transformation step
- **Intermediate Results**: Inspect data at any point in the pipeline
- **Clear Error Messages**: Know exactly which YAML section caused issues

### **📊 Data Lineage & Metadata Preservation**
- **Column-Level Traceability**: Track every column transformation from source to target
- **Business Logic Documentation**: YAML configurations serve as living documentation of business rules
- **Metadata Preservation**: Maintain data definitions, transformations, and business context in version-controlled configs
- **Audit Trail**: Complete history of data transformation changes for compliance and debugging
- **Impact Analysis**: Easily identify downstream effects of schema or business logic changes

### **⚙️ Configuration-Driven Architecture**
- **Business Logic as Code**: Transform domain knowledge into maintainable, testable configurations
- **Environment Consistency**: Same YAML configs work across dev, staging, and production
- **Dynamic Pipeline Generation**: Generate complex data pipelines from simple configuration files
- **Separation of Concerns**: Business logic separate from implementation details

### **📚 Improved Maintainability** 
- **Declarative Syntax**: Focus on *what* you want, not *how* to implement it
- **Version Control Friendly**: YAML configs are easily diffed and reviewed
- **Self-Documenting**: Configuration serves as both code and documentation
- **Schema Evolution**: Track and manage data schema changes over time

### **🧪 Better Testing**
- **Isolated Testing**: Test individual transformations independently
- **Mock-Friendly**: Easy to create test scenarios with sample data
- **Regression Prevention**: Configuration changes are explicit and reviewable
- **Business Rule Validation**: Validate business logic without complex PySpark setup

### **👥 Team Collaboration**
- **Lower Barrier to Entry**: SQL analysts can contribute without deep PySpark knowledge
- **Consistent Patterns**: Standardized way to express data transformations
- **Knowledge Sharing**: Configurations can be shared across teams and projects
- **Cross-functional Communication**: Business stakeholders can review and understand transformations

## 📦 Installation

```bash
# Install from PyPI (coming soon)
pip install pyyql

# Or install from source
git clone https://github.com/your-org/pyyql.git
cd pyyql
pip install -e .
```

### Prerequisites
- Python 3.7+
- PySpark 3.0+
- PyYAML

## 🏃 Quick Start

### 1. Define Your Data Pipeline in YAML

Create a `my_transformation.yaml` file:

```yaml
constructed_table_name: customer_orders

dependencies:
  customers: { table_name: customer_data, type: source }
  orders: { table_name: order_data, type: source }

join_conditions:
  - ("customers", "orders", "customers.customer_id", "orders.customer_id")

select:
  customers.customer_name: customer_name
  customers.email: customer_email
  orders.order_total: order_amount
  orders.order_date: order_date

filter_condition:
  - orders.order_total > 100

sort_condition:
  - (orders.order_date, desc)
```

### 2. Execute with Python

```python
from pyspark.sql import SparkSession
from pyyql import YQL
from your_data_loader import load_dataframes

# Initialize Spark
spark = SparkSession.builder.appName("PYYql Demo").getOrCreate()

# Load your DataFrames
df_dict = {
    "customer_data": spark.read.csv("customers.csv", header=True),
    "order_data": spark.read.csv("orders.csv", header=True)
}

# Execute transformation
yql = YQL(yaml_path="my_transformation.yaml", df_named_dict=df_dict)
result_df = yql.run()
result_df.show()
```

## 🔗 Data Lineage & Business Logic Traceability

### **Column-Level Lineage Tracking**

PYYql automatically maintains complete column lineage through its configuration-based approach:

```yaml
# Every transformation is explicitly documented
select:
  customers.first_name: customer_first_name        # Direct mapping
  customers.last_name: customer_last_name          # Direct mapping  
  orders.order_date: transaction_date              # Business terminology
  orders.total_amount: revenue                     # Business context
```

**Benefits:**
- 📍 **Source Identification**: Know exactly which source column feeds each output
- 🔄 **Transformation History**: Track how business rules evolve over time
- 📋 **Impact Analysis**: Identify all downstream dependencies when source schemas change
- 🏷️ **Business Context**: Maintain semantic meaning alongside technical transformations

### **Metadata as Business Logic**

YAML configurations serve as living documentation of your data transformation business rules:

```yaml
constructed_table_name: monthly_customer_revenue

# Business rule: Include only active customers with orders
dependencies:
  active_customers: { table_name: customers, type: source }
  valid_orders: { table_name: orders, type: source }

# Business rule: Customer-order relationship via customer_id
join_conditions:
  - ("active_customers", "valid_orders", "customers.id", "orders.customer_id")

# Business rule: Only include high-value transactions
filter_condition:
  - orders.amount >= 100
  - customers.status == 'ACTIVE'
  - orders.order_date >= '2024-01-01'

# Business rule: Revenue calculation and customer segmentation
select:
  customers.customer_id: customer_id
  customers.customer_name: customer_name
  customers.segment: customer_tier
  orders.amount: transaction_amount
  orders.order_date: revenue_date
```

### **Audit Trail & Compliance**

Every change to business logic is tracked through version control:

```bash
# View the evolution of business rules
git log --oneline customer_revenue_model.yaml

# See exactly what business logic changed
git diff HEAD~1 customer_revenue_model.yaml

# Understand the business impact of changes
git blame customer_revenue_model.yaml
```

### **Cross-Team Communication**

YAML configurations bridge technical and business teams:

- **Data Engineers**: Implement the technical transformation
- **Business Analysts**: Define and validate business rules
- **Data Governance**: Ensure compliance and data quality
- **Data Scientists**: Understand feature engineering logic
- **Stakeholders**: Review and approve business logic changes

## 📖 YAML Schema Reference

### Complete Schema Structure

```yaml
# Table name for the final result
constructed_table_name: "result_table_name"

# Define source tables and their aliases  
dependencies:
  alias1:
    table_name: "actual_table_name"
    type: "source"  # or "model" for dependent transformations
    source: "data_source_name"

# Define how tables should be joined
join_conditions:
  - ("table1_alias", "table2_alias", "table1.key_column", "table2.key_column")
  - ("table2_alias", "table3_alias", "table2.key_column", "table3.key_column")

# Select columns with optional aliasing
select:
  original_column_name: aliased_column_name
  table_alias.column_name: new_column_name

# Filter conditions (WHERE clause)
filter_condition:
  - "column_name == 'value'"
  - "numeric_column > 100"
  - "date_column >= '2023-01-01'"

# Group by columns and aggregations
group_condition:
  - "column_name"
  
# Having conditions (post-aggregation filters)  
having_condition:
  - "COUNT(*) > 5"

# Sort specification
sort_condition:
  - "(column_name, asc)"
  - "(other_column, desc)"
```

### Schema Sections Explained

#### **Dependencies**
Maps table aliases to actual DataFrame names in your Python dictionary.

```yaml
dependencies:
  emp:                    # Alias used in joins and selections
    table_name: employee  # Key in your df_named_dict
    type: source         # source | model
    source: hr_database  # Data source identifier
```

#### **Join Conditions** 
Defines relationships between tables using tuple format.

```yaml
join_conditions:
  # Format: (left_table, right_table, left_key, right_key)
  - ("customers", "orders", "customers.id", "orders.customer_id")
  - ("orders", "products", "orders.product_id", "products.id")
```

#### **Select Clause**
Maps source columns to output column names.

```yaml
select:
  # Format: source_column: output_column
  customers.name: customer_name
  orders.total: order_amount
  products.price: product_price
```

## 🛠 API Reference

### YQL Class

Main interface for executing YAML-defined transformations.

```python
from pyyql import YQL

yql = YQL(yaml_path="path/to/config.yaml", df_named_dict=dataframes)
result = yql.run()
```

**Parameters:**
- `yaml_path` (str): Path to YAML configuration file
- `df_named_dict` (Dict[str, DataFrame]): Dictionary mapping table names to PySpark DataFrames

**Returns:**
- `DataFrame`: Transformed PySpark DataFrame

### PYYql Class  

Lower-level interface for advanced usage.

```python
from pyyql import PYYql

engine = PYYql(yaml_path="config.yaml")
conditions = engine._get_join_condition()
aliases = engine._select_alias()
```

## 📁 Project Structure

```
pyyql/
├── main/
│   ├── __init__.py
│   ├── pyyql.py          # Core transformation engine
│   ├── yaml_engine.py    # YAML parsing utilities  
│   └── yql.py           # Main user interface
├── test/
│   ├── resources/        # Test data and configurations
│   ├── test_*.py        # Test suites
│   └── samplesparksession.py
├── README.md
├── setup.py
└── requirements.txt
```

## 🧪 Testing

Run the test suite:

```bash
# Run all tests
python -m pytest test/

# Run specific test file
python -m unittest test.test_yaml_engine

# Run with coverage
pytest --cov=main test/
```

### Example Test Configuration

See `test/resources/sample_yaml.yaml` for a complete example configuration used in tests.

## 🔄 Current Status & Roadmap

### ✅ Implemented Features
- **Multi-table JOINs** with automatic duplicate column handling
- **SELECT projections** with column aliasing  
- **YAML configuration parsing** and validation
- **Basic test framework** with sample data

### 🚧 In Development (Phase 1)
- **WHERE/FILTER conditions** 
- **GROUP BY with aggregations**
- **ORDER BY/SORT operations**
- **Complete run() method implementation**

### 📋 Planned Features (Phase 2+)
- **HAVING clauses**
- **Window functions** (ROW_NUMBER, RANK, LAG, LEAD)
- **UNION operations**
- **Subqueries and CTEs** 
- **CASE WHEN statements**
- **Advanced aggregation functions**
- **User-defined functions (UDF) support**

## 🤝 Contributing

We welcome contributions! Here's how to get started:

### Development Setup

```bash
# Clone the repository
git clone https://github.com/your-org/pyyql.git
cd pyyql

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements-dev.txt

# Run tests
python -m pytest test/
```

### Contribution Guidelines

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Write** tests for your changes
4. **Ensure** all tests pass
5. **Commit** your changes (`git commit -m 'Add amazing feature'`)
6. **Push** to the branch (`git push origin feature/amazing-feature`)
7. **Open** a Pull Request

### Areas for Contribution

- 🐛 **Bug fixes** and error handling improvements
- ✨ **New SQL operations** (GROUP BY, HAVING, Window functions)
- 📚 **Documentation** improvements and examples
- 🧪 **Test coverage** expansion
- 🎨 **YAML schema** enhancements
- ⚡ **Performance** optimizations

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Apache Spark** team for the powerful DataFrame API
- **PyYAML** project for excellent YAML parsing capabilities
- **Open source community** for inspiration and best practices

## 📞 Support & Community

- 🐛 **Issues**: [GitHub Issues](https://github.com/your-org/pyyql/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/your-org/pyyql/discussions)  
- 📧 **Email**: support@pyyql.org
- 📖 **Documentation**: [Full Documentation](https://pyyql.readthedocs.io)

---

**Made with ❤️ for the PySpark community**