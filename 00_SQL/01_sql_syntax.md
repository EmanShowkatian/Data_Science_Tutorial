# **Kaggle SQL**
**Structured Query Language**
## Course #1 ->  **Getting Started With SQL and BigQuery**

We'll build our SQL skills using BigQuery, a web service that lets you apply SQL to huge datasets.

<pre><code>from google.cloud import bigquery</code></pre>

- The first step in the workflow is to create a <code>Client</code> object. As you'll soon see, this <code>Client</code> object will play a central role in retrieving information from BigQuery datasets.

<pre><code>client = bigquery.Client()</code></pre>

- In BigQuery, each dataset is contained in a corresponding project. In this case, our <code>hacker_news</code> dataset is contained in the <code>bigquery-public-data</code> project. To access the dataset,
  - We begin by constructing a reference to the  dataset with the <code>dataset()</code> method.
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

In this example, we'll investigate the ```full``` table that we fetched above.

<pre>
# Print information on all the columns in the "full" table in the "hacker_news" dataset
<code>table.schema</code>
</pre>

Each ```SchemaField``` tells us about a specific column (which we also refer to as a **field**). In order, the information is:

- The **name** of the column
- The **field** type (or datatype) in the column
- The **mode** of the column (```'NULLABLE'``` means that a column allows NULL values, and is the default)
- A **description** of the data in that column

Viewing ```table```:

```
# Preview the first five lines of the "full" table
client.list_rows(table, max_results=5).to_dataframe()
```

Viewing especific ```columns```:

```
# Preview the first five entries in the "by" column of the "full" table
client.list_rows(table, selected_fields=table.schema[:1], max_results=5).to_dataframe()
```

***

## Course #2 -> Select, From & Where


