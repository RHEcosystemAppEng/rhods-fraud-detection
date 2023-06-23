# Fraud detection in RHODS using Starburst Enterprise

##### Requirements:

- RHODS working cluster with administrator access
- Startburst Enterprise licence [optional when using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link)]
- Write access to your own Amazon S3 bucket [optional when using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link)]
- Access to the [original dataset](https://drive.google.com/file/d/1YhmV3vPbFe-JXU_biwvaizV0WGhAegH1/view) [optional when using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link)]

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

### Steps:

1. Store the original data in S3 [optional when using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link)]

    Upload the
file [creditcard_with_empty_values.csv](https://drive.google.com/file/d/1YhmV3vPbFe-JXU_biwvaizV0WGhAegH1/view)
to a bucket called `<PASTE_HERE_YOUR_BUCKET_NAME>/data`. In this example, we will name it `rhods-fraud-detection`.
Your AWS credentials **must** have read and write access to this bucket.

2. Set credentials and configure Starburst Enterprise [optional when using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link)]


    1. Go to the configs directory `cd ./configs`
    2. Update the [01_starburst_licence.yaml](configs/01_starburst_licence.yaml) file with your own Startbust Enterprise license. 
    3. Update the [02_aws_credentials.yaml](configs/02_aws_credentials.yaml) file with your own Amazon credentials.
    4. Apply the configuration files `cat *.yaml | oc apply -f -`
        <details>
            <summary>Expected output</summary>
        
        ```bash
        $: cat *.yaml | oc apply -f -
        secret/starburstdata created
        secret/aws-credentials created
        starburstenterprise.charts.starburstdata.com/starburstenterprise created
        starbursthive.charts.starburstdata.com/starbursthive created
        route.route.openshift.io/starburst-web created
        ```
        </details>

3. Working from the Starburst Web UI

    If you are using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link) then the UI link is in the email which you receive after provisioning.

    Otherwise, log into the Web console of your Starburst Enterprise instance. After exposing it
    with a route, it should be available through the URL `http://starburst-web.<cluster_url>/ui/insights/ide`
    and the configured credentials (`default user: admin`). At this point, you should see the query editor in the web ui

![SEP Web UI](./images/sep_webui.png)

4. Queries to read and write using Starburst Enterprise
> **Note:** 
> 1. Please click the arrow &#x25B6; below to view and/or copy the query
> 2. Please put your own s3 bucket name in the below queries by changing the placeholder "CHANGE-THIS-BUCKET-NAME". If you are using demo.redhat.com [workshop](https://demo.redhat.com/catalog?item=babylon-catalog-prod/sandboxes-gpte.ocp4-workshop-fraud-detection.prod&utm_source=webapp&utm_medium=share-link) then the bucket name is in the email which you received after provisioning.

<details>
    <summary>Create the schema</summary>

```SQL
CREATE SCHEMA s3.fraud WITH (location = 's3a://CHANGE-THIS-BUCKET-NAME/data');
```
</details>

<details>
    <summary>Create a table reading the original dataset from S3</summary>

```SQL
CREATE TABLE IF NOT EXISTS s3.fraud.original
(
    time     VARCHAR,
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
    amount   VARCHAR,
    class VARCHAR
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
SELECT * FROM s3.fraud.original;
```

![SEP Web UI](./images/sep_webui_reading.png)
</details>

<details>
    <summary>Set session variable</summary>

```SQL
SHOW SESSION LIKE 'writer_%';
SET SESSION writer_min_size = '160MB';
```

</details>

<details>
    <summary>Create a second table with only the filtered rows</summary>

```SQL
CREATE TABLE IF NOT EXISTS s3.fraud.clean
    WITH (
        external_location = 's3a://CHANGE-THIS-BUCKET-NAME/clean/',
        format = 'csv',
        skip_header_line_count=1
        ) AS (
    SELECT *
    FROM s3.fraud.original
    WHERE v1 != ''
    AND v2 != ''
    AND v3 != ''
    AND v4 != ''
    AND v5 != ''
    AND v6 != ''
    AND v7 != ''
    AND v8 != ''
    AND v9 != ''
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
    AND amount != ''
    AND class != '');
```

![SEP Web UI](./images/sep_webui_writing.png)

> **Note:** This query might take some minutes depending on the network between
> RHODS and the AWS S3 bucket.

</details>

**Result:** Now you can verify that the S3 bucket `rhods-fraud-detection/clean`, 
contains a new file with fewer rows than the original source.

![S3 Clean csv file](./images/s3_clean_csv_file.png)

#### Go back to the [Notebook.ipynb](./Notebook.ipynb) and continue with the process.
