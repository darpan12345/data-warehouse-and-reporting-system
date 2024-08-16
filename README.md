# data-warehouse-and-reporting-system
data warehouse and reporting system using sql
This project will give you hands-on experience with data warehousing concepts, ETL processes, and reporting.
DATABASE schema creation ____
-- Dimension Tables
CREATE TABLE DimProduct (
    ProductID INT PRIMARY KEY,
    ProductName VARCHAR(255),
    Category VARCHAR(255),
    Price DECIMAL(10, 2)
);

CREATE TABLE DimCustomer (
    CustomerID INT PRIMARY KEY,
    CustomerName VARCHAR(255),
    Email VARCHAR(255)
);

CREATE TABLE DimDate (
    DateID INT PRIMARY KEY,
    Date DATE,
    Month INT,
    Year INT
);

-- Fact Table
CREATE TABLE FactSales (
    SaleID INT PRIMARY KEY,
    ProductID INT,
    CustomerID INT,
    DateID INT,
    Quantity INT,
    TotalAmount DECIMAL(10, 2),
    FOREIGN KEY (ProductID) REFERENCES DimProduct(ProductID),
    FOREIGN KEY (CustomerID) REFERENCES DimCustomer(CustomerID),
    FOREIGN KEY (DateID) REFERENCES DimDate(DateID)
);


ETL PROCESS 

import pandas as pd
import sqlalchemy

# Database connection
engine = sqlalchemy.create_engine('mysql+pymysql://username:password@localhost/mydatabase')

# Extract
def extract_data():
    product_df = pd.read_csv('products.csv')
    customer_df = pd.read_csv('customers.csv')
    sales_df = pd.read_csv('sales.csv')
    return product_df, customer_df, sales_df

# Transform
def transform_data(product_df, customer_df, sales_df):
    # Example transformation: Clean data
    product_df = product_df.dropna()
    customer_df = customer_df.dropna()
    sales_df = sales_df.dropna()
    return product_df, customer_df, sales_df

# Load
def load_data(product_df, customer_df, sales_df):
    product_df.to_sql('DimProduct', con=engine, if_exists='replace', index=False)
    customer_df.to_sql('DimCustomer', con=engine, if_exists='replace', index=False)
    sales_df.to_sql('FactSales', con=engine, if_exists='replace', index=False)

# ETL Process
product_df, customer_df, sales_df = extract_data()
product_df, customer_df, sales_df = transform_data(product_df, customer_df, sales_df)
load_data(product_df, customer_df, sales_df)


BASIC REPORTING QUERY ___

SELECT
    p.ProductName,
    SUM(f.Quantity) AS TotalQuantity,
    SUM(f.TotalAmount) AS TotalRevenue
FROM
    FactSales f
JOIN
    DimProduct p ON f.ProductID = p.ProductID
GROUP BY
    p.ProductName;


REPORTING QUEUE USING PANDAS

import pandas as pd
import sqlalchemy

# Database connection
engine = sqlalchemy.create_engine('mysql+pymysql://username:password@localhost/mydatabase')

# Load data from database
query = """
SELECT
    p.ProductName,
    SUM(f.Quantity) AS TotalQuantity,
    SUM(f.TotalAmount) AS TotalRevenue
FROM
    FactSales f
JOIN
    DimProduct p ON f.ProductID = p.ProductID
GROUP BY
    p.ProductName;
"""

report_df = pd.read_sql(query, con=engine)

# Display report
print(report_df)


