# **Kaggle SQL**

**Structured Query Language**

## Course #1 -> **Getting Started With SQL and BigQuery**

We'll build our SQL skills using BigQuery, a web service that lets you apply SQL to huge datasets.

<pre><code>from google.cloud import bigquery</code></pre>

- The first step in the workflow is to create a <code>Client</code> object. As you'll soon see, this <code>Client</code> object will play a central role in retrieving information from BigQuery datasets.

<pre><code>client = bigquery.Client()</code></pre>

- In BigQuery, each dataset is contained in a corresponding project. In this case, our <code>hacker_news</code> dataset is contained in the <code>bigquery-public-data</code> project. To access the dataset,
  - We begin by constructing a reference to the dataset with the <code>dataset()</code> method.
  - Next, we use the <code>get_dataset()</code> method, along with the reference we just constructed, to fetch the dataset.

<pre>
<code>dataset_ref = client.dataset("hacker_news", project = "bigquery-public-data")</code>
<code>dataset = client.get_table(dataset_ref)</code>
</pre>

Every dataset is just a collection of tables. You can think of a dataset as a spreadsheet file containing multiple tables, all composed of rows and columns.

We use the <code>list_tables()</code> method to list the tables in the dataset.

<pre>
# List all the tables in the "hacker_news" dataset
<code>tables = list(client.list_tables(dataset))</code>

# Print names of all tables in the dataset (there are four!)
<code>for table in tables:</code>
<code>  print(table.table_id)</code>
</pre>

Similar to how we fetched a dataset, we can fetch a table. In the code cell below, we fetch the <code>full</code> table in the <code>hacker_news</code> dataset.

<pre>
<code>table_ref = dataset_ref.table("full")</code>
<code>table = client.get_dataset(table_ref)</code>
</pre>

<img src='./images/google_cloud_bigquery_workflow.png' 
style="float: center; margin-right: 20px;"/>

#### **Table schema**

The structure of a table is called its **schema**. We need to understand a table's schema to effectively pull out the data we want.

In this example, we'll investigate the `full` table that we fetched above.

<pre>
# Print information on all the columns in the "full" table in the "hacker_news" dataset
<code>table.schema</code>
</pre>

Each `SchemaField` tells us about a specific column (which we also refer to as a **field**). In order, the information is:

- The **name** of the column
- The **field** type (or datatype) in the column
- The **mode** of the column (`'NULLABLE'` means that a column allows NULL values, and is the default)
- A **description** of the data in that column

Viewing `table`:

```
# Preview the first five lines of the "full" table
client.list_rows(table, max_results=5).to_dataframe()
```

Viewing especific `columns`:

```
# Preview the first five entries in the "by" column of the "full" table
client.list_rows(table, selected_fields=table.schema[:1], max_results=5).to_dataframe()
```

---

## Course #2 -> Select, From & Where

### **Introduction**

Now that you know how to access and examine a dataset, you're ready to write your first SQL query! As you'll soon see, SQL queries will help you sort through a massive dataset, to retrieve only the information that you need.

We'll begin by using the keywords **SELECT**, **FROM**, and **WHERE** to get data from specific columns based on conditions you specify.

For clarity, we'll work with a small imaginary dataset `pet_records` which contains just one table, called `pets`.

<img src='./images/imaginary_dataset.png' 
style="float: center; margin-right: 20px;"/>

### **SELECT ... FROM**

The most basic SQL query selects a single column from a single table. To do this,

- specify the column you want after the word **SELECT**, and then
- specify the table after the word **FROM**.

For instance, to select the `Name` column (from the `pets` table in the `pet_records` database in the `bigquery-public-data` project), our query would appear as follows:
<img src='./images/select_from.png' 
style="float: center; margin-right: 20px;"/>
Note that when writing an SQL query, the argument we pass to **FROM** is not in single or double quotation marks (' or "). It is in backticks (`).

### **WHERE ...**

BigQuery datasets are large, so you'll usually want to return only the rows meeting specific conditions. You can do this using the **WHERE** clause.

The query below returns the entries from the `Name` column that are in rows where the `Animal` column has the text `'Cat'`.
<img src='./images/where.png' 
style="float: center; margin-right: 20px;"/>

### Example: What are all the U.S. cities in the OpenAQ dataset?

Now that you've got the basics down, let's work through an example with a real dataset. We'll use an OpenAQ dataset about air quality.

First, we'll set up everything we need to run queries and take a quick peek at what tables are in our database. (Since you learned how to do this in the previous tutorial, we have hidden the code. But if you'd like to take a peek, you need only click on the "Code" button below.)

```
from google.cloud import bigquery

# Create a "Client" object
client = bigquery.Client()

# Construct a reference to the "openaq" dataset
dataset_ref = client.dataset("openaq", project="bigquery-public-data")

# API request - fetch the dataset
dataset = client.get_dataset(dataset_ref)

# List all the tables in the "openaq" dataset
tables = list(client.list_tables(dataset))

# Print names of all tables in the dataset (there's only one!)
for table in tables:
    print(table.table_id)
```

The dataset contains only one table, called `global_air_quality`. We'll fetch the table and take a peek at the first few rows to see what sort of data it contains.

Everything looks good! So, let's put together a query. Say we want to select all the values from the `city` column that are in rows where the `country` column is `'US'` (for "United States").

```
# Query to select all the items from the "city" column where the "country" column is 'US'
query = """
        SELECT city
        FROM `bigquery-public-data.openaq.global_air_quality`
        WHERE country = 'US'
        """
```

### Submitting the query to the dataset

```
# Create a "Client" object
client = bigquery.Client()

# Set up the query
query_job = client.query(query)

# API request - run the query, and return a pandas DataFrame
us_cities = query_job.to_dataframe()

# What five cities have the most measurements?
us_cities.city.value_counts().head()
```

### More queries

If you want multiple columns, you can select them with a comma between the names:

```
query = """
        SELECT city, country
        FROM `bigquery-public-data.openaq.global_air_quality`
        WHERE country = 'US'
        """
```

You can select all columns with a `*` like this:

```
query = """
        SELECT *
        FROM `bigquery-public-data.openaq.global_air_quality`
        WHERE country = 'US'
        """
```

### Q&A: Notes on formatting

**Question: What's up with the triple quotation marks (""")?**
Answer: These tell Python that everything inside them is a single string, even though we have line breaks in it. The line breaks aren't necessary, but they make it easier to read your query.

**Question: Do you need to capitalize SELECT and FROM?**
Answer: No, SQL doesn't care about capitalization. However, it's customary to capitalize your SQL commands, and it

### Working with big datasets

To begin,you can estimate the size of any query before running it. Here is an example using the (very large!) Hacker News dataset. To see how much data a query will scan, we create a `QueryJobConfig` object and set the `dry_run` parameter to `True`.

```
# Query to get the score column from every row where the type column has value "job"
query = """
        SELECT score, title
        FROM `bigquery-public-data.hacker_news.full`
        WHERE type = "job" 
        """

# Create a QueryJobConfig object to estimate size of query without running it
dry_run_config = bigquery.QueryJobConfig(dry_run=True)

# API request - dry run query to estimate costs
dry_run_query_job = client.query(query, job_config=dry_run_config)

print("This query will process {} bytes.".format(dry_run_query_job.total_bytes_processed))
```

### The `DISTINCT` keyword ensures each column shows up once:

```
SELECT DISTINCT column_name
FROM dataset_name
WHERE condition
```

---

## Course #2 -> Group By, Having & Count