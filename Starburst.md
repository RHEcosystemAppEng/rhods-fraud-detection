# Fraud detection in RHODS using Starburst Enterprise

##### Requirements:

- RHODS working cluster with administrator access
- Startburst Enterprise licence [optional when using demo.redhat.com [workshop](# TODO: create and publish RHDP cluster)]
- Write access to your own Amazon S3 bucket [optional when using demo.redhat.com [workshop](# TODO: create and publish RHDP cluster)]
- Access to a PostgreSQL instance and user with SELECT, INSERT, and CREATE permissions [optional when using demo.redhat.com [workshop](# TODO create and publish RHDP cluster)]
- Access to the [original dataset](# TODO: upload features/transactions csvs) [optional when using demo.redhat.com [workshop](# TODO: create and publish RHDP cluster)]

#### What is Trino and Starburst Enterprise?

Trino (formerly Presto® SQL) is the fastest open source, massively parallel
processing SQL query engine designed for analytics of large datasets distributed
over one or more data sources in object storage, databases and other systems.

Starburst Enterprise platform (SEP) is a fully supported, enterprise-grade
distribution of Trino. It adds integrations, improves performance, provides
security, and makes it easy to deploy, configure and manage your clusters.

### Goal

In this document we will cover how to work with the data of the fraud-detection
example with SQL-like queries and an instance of SEP running in the Red Hat
Openshift Data Science cluster.

We will take advantage of several powerful features of Starburst. We will cover reading from/writing to AWS S3, executing federated queries from multiple data sources, in addition to creating materialized views to aid in our data science workflow.

### Steps:

1. Store the features data in S3 [optional when using demo.redhat.com [workshop](# TODO: create and deploy RHDP cluster)]

    Upload the
file [features.csv](# TODO: upload features.csv)
to a bucket called `<PASTE_HERE_YOUR_BUCKET_NAME>/data`. In this example, we will name it `rhods-fraud-detection`.
Your AWS credentials **must** have read and write access to this bucket.

2. Store the transactions data in Postgres [optional when using demo.redhat.com [workshop](# TODO: create and deploy RHDP cluster)]

    With the file [transactions.csv](# TODO: upload transactions.csv) downloaded, execute the following queries in the psql command line tool

    ```sql
    CREATE TABLE transactions (
      id SERIAL,
      Time INTEGER,
      Amount NUMERIC(10,2),
      Class INTEGER,
      PRIMARY KEY (id)
    );
    ```
    ```sql
    COPY transactions(id, Time, Amount, Class)
      FROM '/path/to/transactions.csv'
      DELIMITER ','
      CSV HEADER;
    ```

3. Set credentials and configure Starburst Enterprise [optional when using demo.redhat.com [workshop](# TODO: create and publish RHDP cluster)]


    1. Go to the configs directory `cd ./configs`
    2. Update the [01_starburst_licence.yaml](configs/01_starburst_licence.yaml) file with your own Startbust Enterprise license. 
    3. Update the [02_aws_credentials.yaml](configs/02_aws_credentials.yaml) file with your own Amazon credentials.
    4. Update the [03_postgres_credentials.yaml](configs/03_postgres_credentials.yaml) file with your own PostgreSQL server and user details.
    5. Apply the configuration files `cat *.yaml | oc apply -f -`
        <details>
            <summary>Expected output</summary>
        
        ```bash
        $: cat *.yaml | oc apply -f -
        secret/starburstdata created
        secret/aws-credentials created
        secret/postgres-credentials created
        starburstenterprise.charts.starburstdata.com/starburstenterprise created
        starbursthive.charts.starburstdata.com/starbursthive created
        route.route.openshift.io/starburst-web created
        ```
        </details>


    *This next step can be done in one of two ways\*:*
    1. *Follow steps 4 and 5 to process the data in the Starburst WebUI*
    2. *Skip steps 4 and 5 then execute all data processing in the Jupyter Notebook*

4. (Optional\*) Working from the Starburst Web UI

    If you are using demo.redhat.com [workshop](# TODO: create and publish RHDP cluster) then the UI link is in the email which you receive after provisioning.

    Otherwise, log into the Web console of your Starburst Enterprise instance. After exposing it
    with a route, it should be available through the URL `http://starburst-web.<cluster_url>/ui/insights/ide`
    and the configured credentials (`default user: admin`). At this point, you should see the query editor in the web ui

<!-- # TODO: update webui.png to include postgres catalog inc luster explorer -->
![SEP Web UI](./images/sep_webui.png)

5. (Optional\*) Queries to read and write using Starburst Enterprise
> **Note:** 
> 1. Please click the arrow &#x25B6; below to view and/or copy the query
> 2. Please put your own s3 bucket name in the below queries by changing the placeholder "CHANGE-THIS-BUCKET-NAME". If you are using demo.redhat.com [workshop](# TODO: create and publish RHDP cluster) then the bucket name is in the email which you received after provisioning.

<details>
    <summary>Create the schema</summary>

```SQL
CREATE SCHEMA s3.fraud WITH (location = 's3a://CHANGE-THIS-BUCKET-NAME/data');
```
</details>

<details>
    <summary>Create a table reading the original dataset from S3</summary>

```SQL
CREATE TABLE IF NOT EXISTS s3.fraud.features
(
    id       VARCHAR,
    v1       VARCHAR,
    v2       VARCHAR,
    v3       VARCHAR,
    v4       VARCHAR,
    v5       VARCHAR,
    v6       VARCHAR,
    v7       VARCHAR,
    v8       VARCHAR,
    v9       VARCHAR,
    v10      VARCHAR,
    v11      VARCHAR,
    v12      VARCHAR,
    v13      VARCHAR,
    v14      VARCHAR,
    v15      VARCHAR,
    v16      VARCHAR,
    v17      VARCHAR,
    v18      VARCHAR,
    v19      VARCHAR,
    v20      VARCHAR,
    v21      VARCHAR,
    v22      VARCHAR,
    v23      VARCHAR,
    v24      VARCHAR,
    v25      VARCHAR,
    v26      VARCHAR,
    v27      VARCHAR,
    v28      VARCHAR,
) WITH ( 
    external_location = 's3a://CHANGE-THIS-BUCKET-NAME/data/',
    skip_header_line_count = 1,
    format = 'csv'
);
```

</details>

<details>
    <summary> Verify you can read the original data loaded into SEP</summary>

```SQL
SELECT * FROM s3.fraud.features;
```

<!-- # TODO: update reading.png with postgres catalog -->
![SEP Web UI](./images/sep_webui_reading.png)
</details>

<details>
    <summary>Create a materialized view with only the filtered rows using a federated query</summary>

```SQL
CREATE MATERIALIZED VIEW s3.fraud.data AS
WITH
  t1 AS (
      SELECT * FROM s3.fraud.features
          WHERE v1  != ''
            AND v2  != ''
            AND v3  != ''
            AND v4  != ''
            AND v5  != ''
            AND v6  != ''
            AND v7  != ''
            AND v8  != ''
            AND v9  != ''
            AND v10 != ''
            AND v11 != ''
            AND v12 != ''
            AND v13 != ''
            AND v14 != ''
            AND v15 != ''
            AND v16 != ''
            AND v17 != ''
            AND v18 != ''
            AND v19 != ''
            AND v20 != ''
            AND v21 != ''
            AND v22 != ''
            AND v23 != ''
            AND v24 != ''
            AND v25 != ''
            AND v26 != ''
            AND v27 != ''
            AND v28 != ''
    ),
  t2 AS (
      SELECT id, time, amount, class FROM postgres.public.transactions
      WHERE amount IS NOT NULL AND class IS NOT NULL
      )
SELECT t1.*, t2.time, t2.amount, t2.class
FROM t1
JOIN t2 ON t1.id = CAST(t2.id AS varchar);
```

<!-- # TODO: add option to write to S3 from the Materialized View -->
<!-- ![SEP Web UI](./images/sep_webui_writing.png)

> **Note:** This query might take some minutes depending on the network between
> RHODS and the AWS S3 bucket. -->

</details>

<!-- **Result:** Now you can verify that the S3 bucket `rhods-fraud-detection/clean`, 
contains a new file with fewer rows than the original source.

![S3 Clean csv file](./images/s3_clean_csv_file.png) -->

---

#### Go back to the [Notebook.ipynb](./Notebook.ipynb) and continue with the process.
