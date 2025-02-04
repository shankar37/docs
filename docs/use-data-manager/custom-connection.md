# Create a custom connection
 
- On the StarTree Data Manager overview page, click **Create a dataset**, you'll be prompted to complete the following steps:
   
   1. **Select the connection type**
   2. **Enter dataset details**
   3. **Perform data modeling**. 
   4. **Add indexes and additional configuration**
   5. **Review and create your dataset**

**This document covers steps 1-2 above.** For information about steps 3-5, see the following sections in the Ingest data doc:
  - [Perform data modeling](/docs/use-data-manager/ingest-data#perform-data-modeling)
  - [Add indexes and additional configuration](/docs/use-data-manager/ingest-data#add-indexes-and-perform-additional-configuration)
  - [Review and submit](#/docs/use-data-manager/ingest-data#review-and-submit)

## Select the custom connection type

- Select **Custom Connection** as the connection type.

## Enter dataset details

1. Under **Dataset Details**, enter the dataset name and optional description of the dataset.
2. Under **Datasource Details**, the GCS data source template is selected by default. Do one of the following:
   - If you do not want to use the GPC template, click **Clear Template**, and then enter the connection details to connect to your data source.
   - If you’re connecting to a Google Cloud Storage data source and want to use the template, enter the following connection and input configuration details in JSON format:

      |**Configure**|<div style="width:70px">**Option** </div> | <div style="width:80px">**json key** </div> | **Desciption**|
      |:--|:----------------|:--------- |:--- |
      |**Connection configuration details** |Custom connection name |`name`| Provide a name for your custom connection. |
      ||GCP project ID| `projectId` | Enter your Google project ID. Find your project ID in the [Google Cloud console](https://console.cloud.google.com/)|
      ||GPC credentials| `gcpKey` <br> `jsonKey` <br> `jsonNodeKey` | To obtain a string-encoded service account key, see Google documentation on how to [create a service account key](https://cloud.google.com/iam/docs/keys-create-delete#creating). Include the approprate format for your selected json key.| 
      |**Input configuration details**| Input file format  | `format` | Specify your input file format: `csv`, `json`, `avro`, `parquet`, or `orc`| 
      ||   | `inputUri` | Enter the path to input file(s)|
      ||   | <font size=2>`includeFilePatternMatch`</font>| Optional. Use to only fetch data from files in the `inputUri` path that match this pattern.| 
      ||   | <font size=2>`excludeFilePatternMatch`</font> | Optional. Use to only fetch data from files in the `inputUri` path that *do not match* this pattern.|
      ||   | `recordReaderConfig`: <br> {`key`: ... }| Provide additional details about the source schema. For more information, see supported [input formats](https://docs.pinot.apache.org/basics/data-import/pinot-input-formats).|

3. Click **Check Connection & Sample Data** to validate the connection is successful and fetch sample data, and then click **Next** to configure the schema.

## Update dataset configuration

You can configure the kind of Pinot table and a few properties like replication factor, retention period, primary keys, 
dataset type, and tenants. A sample configuration is shown below:

```json
    {
        "replication": 1,
        "retentionInDays": 180,
        "primaryKey": [],
        "primaryTimeColumn": "timestamp",
        "datasetType": "standard",
        "tenants": {
            "brokerTenant": "DefaultTenant",
            "serverTenant": "DefaultTenant"
        }
    }
```  

## Configure ingestion
Set the **Mode* to **append** or **sync**, and then specify the ingestion schedule as a cron expression. 
The following sample configuration appends ingested data on the 20th minute of every hour, every day:

```json
{
  "mode": "append",
  "schedule": "0 20 * ? * * *"  
}
```

## Review and verify the dataset

1. Review and verify the Pinot schema and table configuration, and make edits as needed. 
2. When ready, click **Create Dataset**.
   