# PyDataGenerator

A powerful Python library that generates templated data from XML specifications. Perfect for creating test data, database seeding, and realistic dataset generation without manual input.

**API Reference:** https://alexprodan99.github.io/pydatagenerator

## Features

- 🎯 **XML-Based Configuration**: Define your data generation rules in simple, readable XML
- 🎲 **Multiple Dataset Types**: Random numbers, sequences, categories, and time-series data
- 📊 **Flexible Output**: Generate data in any format using templates
- ⚡ **Scalable**: Generate millions of data points efficiently
- 🔧 **Easy Integration**: Simple Python API and command-line interface
- 📅 **Time-Series Support**: Generate time-series data with customizable date ranges

## Installation

### From pip

```bash
pip install pydatagenerator
```

### From source

```bash
git clone https://github.com/alexprodan99/pydatagenerator.git
cd pydatagenerator
pip install -e .
```

## Quick Start

### Basic Example: Generate Employee Data

Create an XML file `employees.xml`:

```xml
<pydatagenerator iterations="20">
    <dataset name="age" type="type.random-number-dataset" min="18" max="65"/>
    <dataset name="wage" type="type.random-number-dataset" min="1000" max="100000"/>
    <template>
        INSERT INTO Employees VALUES (#{age}, #{wage})
    </template>
</pydatagenerator>
```

Generate the data using the command line:

```bash
python -m pydatagenerator.data.impl.xml_parser -i employees.xml -o output.txt
```

Or use the scripts:

```bash
python scripts/data_generator.py -i employees.xml -o output.txt
```

**Output** (`output.txt`):

```
INSERT INTO Employees VALUES (42, 45670)
INSERT INTO Employees VALUES (31, 78234)
INSERT INTO Employees VALUES (55, 92100)
...
```

### Python API

```python
from pydatagenerator.xml.impl import XmlParser

# Parse XML from file
parser = XmlParser()
results = parser.parse_xml_from_file('employees.xml')

# Print results
for result in results:
    print(result)

# Or parse XML from string
xml_content = '''
<pydatagenerator iterations="5">
    <dataset name="id" type="type.sequence-number-dataset" start="1" increment="1"/>
    <dataset name="name" type="type.random-categorical-dataset">
        <categories value="Alice"/>
        <categories value="Bob"/>
        <categories value="Charlie"/>
    </dataset>
    <template>User ID: #{id}, Name: #{name}</template>
</pydatagenerator>
'''

results = parser.parse_xml_from_string(xml_content)
for result in results:
    print(result)
```

## Dataset Types

PyDataGenerator supports several dataset types to cover different data generation needs:

### 1. Random Number Dataset

Generates random numbers within a specified range.

**Type:** `type.random-number-dataset`

**Attributes:**
- `min` (required): Minimum value
- `max` (required): Maximum value
- `floating` (optional): Set to "true" for floating-point numbers (default: "false")

**Example:**

```xml
<dataset name="score" type="type.random-number-dataset" min="0" max="100" floating="true"/>
```

### 2. Sequence Number Dataset

Generates sequential numbers starting from a value and incrementing by a step.

**Type:** `type.sequence-number-dataset`

**Attributes:**
- `start` (required): Starting value
- `increment` (required): Increment step
- `floating` (optional): Set to "true" for floating-point numbers (default: "false")

**Example:**

```xml
<dataset name="id" type="type.sequence-number-dataset" start="1000" increment="1"/>
```

### 3. Random Categorical Dataset

Randomly selects from a predefined list of categories.

**Type:** `type.random-categorical-dataset`

**Attributes:**
- `categories` (required): List of category values

**Example:**

```xml
<dataset name="status" type="type.random-categorical-dataset">
    <categories value="active"/>
    <categories value="inactive"/>
    <categories value="pending"/>
</dataset>
```

### 4. Sequence Categorical Dataset

Cycles through categories in sequence.

**Type:** `type.sequence-categorical-dataset`

**Attributes:**
- `start` (required): Starting index
- `increment` (required): Increment step
- `categories` (required): List of category values
- `floating` (optional): Set to "true" for floating-point index arithmetic (default: "false")

**Example:**

```xml
<dataset name="department" type="type.sequence-categorical-dataset" start="0" increment="1">
    <categories value="Engineering"/>
    <categories value="Sales"/>
    <categories value="Marketing"/>
</dataset>
```

### 5. Random Number Time-Series Dataset

Generates random numbers with associated timestamps within specified ranges.

**Type:** `type.random-number-timeseries-dataset`

**Attributes:**
- `min_value` (required): Minimum numeric value
- `max_value` (required): Maximum numeric value
- `min_date` (required): Minimum date
- `max_date` (required): Maximum date
- `floating` (optional): Set to "true" for floating-point numbers (default: "false")
- `date_format` (optional): Date format string (default: "%Y-%m-%dT%H:%M:%SZ")

**Example:**

```xml
<dataset name="temperature" type="type.random-number-timeseries-dataset"
    min_value="10" max_value="35"
    min_date="2024-01-01T00:00:00Z" max_date="2024-12-31T23:59:59Z"
    floating="true" date_format="%Y-%m-%dT%H:%M:%SZ"/>
```

### 6. Sequence Number Time-Series Dataset

Generates sequential numbers with incrementing timestamps.

**Type:** `type.sequence-number-timeseries-dataset`

**Attributes:**
- `start_value` (required): Starting numeric value
- `increment_value` (required): Numeric increment step
- `start_date` (required): Starting date
- `increment_date` (required): Date increment (format: "DDd:HHh:MMm:SSs")
- `floating` (optional): Set to "true" for floating-point numbers (default: "false")
- `date_format` (optional): Date format string (default: "%Y-%m-%dT%H:%M:%SZ")

**Example:**

```xml
<dataset name="daily_revenue" type="type.sequence-number-timeseries-dataset"
    start_value="1000" increment_value="50"
    start_date="2024-01-01T00:00:00Z" increment_date="1d:0h:0m:0s"
    floating="false" date_format="%Y-%m-%dT%H:%M:%SZ"/>
```

## XML Structure

Every PyDataGenerator XML file must follow this structure:

```xml
<pydatagenerator iterations="N">
    <dataset name="column1" type="type.xxx-dataset" attribute1="value1" attribute2="value2"/>
    <dataset name="column2" type="type.yyy-dataset" attribute1="value1"/>
    <!-- More datasets -->
    
    <template>
        Your template content using #{column_name} placeholders
    </template>
</pydatagenerator>
```

**Key Elements:**
- `<pydatagenerator iterations="N">`: Root element specifying how many rows to generate (required)
- `<dataset>`: Defines a data column/field (required, at least one)
- `<template>`: Defines the output format using `#{dataset_name}` placeholders (required)

## Advanced Examples

### Example 1: Customer Database

Generate customer records with ID, name, email domain, and signup date:

```xml
<pydatagenerator iterations="1000">
    <dataset name="customer_id" type="type.sequence-number-dataset" start="1" increment="1"/>
    <dataset name="first_name" type="type.random-categorical-dataset">
        <categories value="John"/>
        <categories value="Jane"/>
        <categories value="Bob"/>
        <categories value="Alice"/>
    </dataset>
    <dataset name="last_name" type="type.random-categorical-dataset">
        <categories value="Smith"/>
        <categories value="Johnson"/>
        <categories value="Williams"/>
        <categories value="Brown"/>
    </dataset>
    <dataset name="signup_date" type="type.random-number-timeseries-dataset"
        min_value="0" max_value="0"
        min_date="2023-01-01T00:00:00Z" max_date="2024-12-31T23:59:59Z"
        date_format="%Y-%m-%d"/>
    
    <template>
        {"id": #{customer_id}, "first_name": "#{first_name}", "last_name": "#{last_name}", "signup_date": "#{signup_date}"}
    </template>
</pydatagenerator>
```

### Example 2: Sales Records with Time-Series

Generate daily sales data:

```xml
<pydatagenerator iterations="365">
    <dataset name="date" type="type.sequence-number-timeseries-dataset"
        start_value="0" increment_value="0"
        start_date="2024-01-01T00:00:00Z" increment_date="1d:0h:0m:0s"
        date_format="%Y-%m-%d"/>
    <dataset name="product_id" type="type.random-number-dataset" min="1" max="50"/>
    <dataset name="quantity" type="type.random-number-dataset" min="1" max="100"/>
    <dataset name="unit_price" type="type.random-number-dataset" min="10" max="500" floating="true"/>
    
    <template>
        #{date},#{product_id},#{quantity},#{unit_price}
    </template>
</pydatagenerator>
```

### Example 3: Product Catalog (Round-Robin Categories)

Generate products cycling through predefined categories:

```xml
<pydatagenerator iterations="100">
    <dataset name="product_id" type="type.sequence-number-dataset" start="1" increment="1"/>
    <dataset name="name" type="type.random-categorical-dataset">
        <categories value="Laptop"/>
        <categories value="Phone"/>
        <categories value="Tablet"/>
        <categories value="Monitor"/>
    </dataset>
    <dataset name="category" type="type.sequence-categorical-dataset" start="0" increment="1">
        <categories value="Electronics"/>
        <categories value="Computers"/>
        <categories value="Mobile"/>
    </dataset>
    <dataset name="price" type="type.random-number-dataset" min="100" max="5000" floating="true"/>
    <dataset name="stock" type="type.random-number-dataset" min="0" max="1000"/>
    
    <template>
        INSERT INTO Products VALUES (#{product_id}, '#{name}', '#{category}', #{price}, #{stock});
    </template>
</pydatagenerator>
```

## Command Line Usage

```bash
python scripts/data_generator.py -i <input_xml> -o <output_file>
```

**Arguments:**
- `-i, --input`: Path to input XML file (required)
- `-o, --output`: Path to output file (required)

**Example:**

```bash
python scripts/data_generator.py -i data_spec.xml -o generated_data.sql
```

## Development

### Install Development Dependencies

```bash
pip install "pydatagenerator[dev,test]"
```

### Run Tests

```bash
pytest
```

### Run Tests with Coverage

```bash
pytest --cov=src --cov-report=html
```

### Code Quality

```bash
# Format code
yapf -i -r src/

# Lint code
flake8 src/

# Check with pylint
pylint src/
```

## Project Structure

```
pydatagenerator/
├── src/pydatagenerator/
│   ├── data/
│   │   ├── abstract/          # Abstract base classes
│   │   └── impl/              # Concrete dataset implementations
│   │       ├── random_number_dataset.py
│   │       ├── sequence_number_dataset.py
│   │       ├── random_categorical_dataset.py
│   │       ├── sequence_categorical_dataset.py
│   │       ├── random_number_timeseries_dataset.py
│   │       ├── sequence_number_timeseries_dataset.py
│   │       └── dataset_handler_factory.py
│   └── xml/
│       ├── abstract/          # Abstract XML parser
│       └── impl/              # XML parser implementation
│           ├── xml_parser.py
│           └── xml_parser_util.py
├── scripts/
│   ├── data_generator.py      # CLI entry point
│   └── examples/
├── tests/                      # Unit tests
├── docs/                       # Documentation
└── pyproject.toml             # Project configuration
```

## Requirements

- Python >= 3.6
- lxml >= 5.2.1

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Author

**Alexandru Prodan**
- Email: prodanalexandru1999@gmail.com

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for a list of changes in each version.

## Support

For issues, questions, or suggestions, please open an issue on GitHub.