| **Data Warehouse setup**  **Documentation** |
|---------------------------------------------|


![](media/ccd17ff7260cd5ba34cb5e427b5fbe61.png)

![](media/c7a4753bfcf7293f4a92b973e7a0f55f.png)

1. BigQuery database creation
=============================

-   Go to cloud console - BigQuery service

-   create a dataset “analytics” in EU region

![](media/60de8fa553fdadf723ba5b77ca8a57d2.png)

![](media/45a89167f6c8b7af8173e977fca4c8bc.png)

![](media/6f21b907960f7cbca58585698c03c879.png)

2. Firestore streaming setup
============================

-   Go to cloud console - Firebase extension

![](media/d044d85d7e94ef2af9b3e1dc2f0e4ed3.png)

-   Install extension “Stream firestore to bigquery”

-   Select the environment / project

-   Install the extension with following parameters and naming convention

Add the collection name at the end of the extension to recognize the stream
(example with “users” collection)

![](media/7b27e89e9846a7e6d1d9e7b72c158e3a.png)

Configure the extension with EU settings, project ID of the environment and
collection from FIrestore

![](media/f7e3006b91dea5ccecf99bb9c879baec.png)

List of streams setup

[./media/image9.png](./media/image9.png)
========================================

For each collection, the install operation needs to be completed. For example,
we need one installation for users, POIs…

3. Backfill of collection - data import
=======================================

-   Go to console - shell Terminal

![](media/40a830e846aa9f6d8cf8faf55362d22b.png)

-   Run command

npx \@firebaseextensions/fs-bq-import-collection

Example with users

Firestore collection name: users

Dataset BigQuery name: analytics

Table BigQuery name: users

Region: EU

![](media/e701a02a17a52efa5bd68e0ea280dea4.png)

4. Setup Bigquery views
=======================

-   Create views for each collection with required fields

create or replace view \`bsc-dev-7a548.analytics.users\` as

SELECT

JSON_EXTRACT(data, "\$.display_name") AS display_name

FROM \`bsc-dev-7a548.analytics.users_raw_latest\`
