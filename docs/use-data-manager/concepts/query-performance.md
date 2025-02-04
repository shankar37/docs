# Query performance 

Query performance may be affected by a dataset's indexes and encodings, which are crucial for boosting query performance. 

Here's a few things to consider for query performance:

- Streamline column settings
- Default configuration for efficiency
- Enhance performance with the star-tree index

## Streamline column settings

Efficient query performance is dependent upon the proper configuration of indexes and encodings when creating a dataset. You can effortlessly configure these settings for relevant columns via a dropdown menu with options such as Dictionary, On-heap Dictionary, or Variable Length Dictionary. Similarly, indexing options like Sorted, Inverted, Bloom Filter, Range, and JSON cater to diverse querying needs, allowing for optimized data access. Explanation is provided below on when to utilize a specific encoding or indexing type.

## Default configuration for efficiency

StarTree simplifies the setup process by automatically applying encodings and indexes based on the pre-defined data types configured during the data modeling phase. This default configuration ensures initial efficiency. However, you have the flexibility to adjust encodings and indexes using an intuitive widget or JSON configuration for more intricate and advanced modeling. You can fine-tuning encodings and indexes based on your specific requirements.

## Example dataset configuration

You can configure the kind of Pinot table and a few properties like replication factor, retention period, primary keys, dataset type, and tenants. A sample configuration is shown below:

```json

{  "replication": 1,
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


## Enhance performance with the star-tree index

Pinot offers the option to configure star-tree indexes explicitly for single-value columns, presenting an opportunity for further optimization. Leveraging star-tree indexes provides an additional performance boost, especially for queries involving hierarchical or categorical data, enhancing the efficiency of data retrieval and analysis for specific use cases.
These functionalities within StarTree enable you to fine-tune their datasets through proper indexing and encoding strategies, optimizing query performance and enhancing overall data accessibility and analysis capabilities. You can find more information on the star-tree index [here](https://docs.pinot.apache.org/basics/indexing/star-tree-index).

## Column encoding types

We recommend the following column encoding type for certain use cases shown in the following table.

|Type | Description|
|:---|:---|
|Dictionary |Use this when there are duplicate values in a column.|
|Variable, Length, or Dictionary  | Only applicable to STRING and BYTE columns. Columns of these types will usually be stored in fixed-size blocks equal to the length of the largest value, but with this config, values will be variable length. Use this if you have outlier values. |
|On heap dictionary | Avoids doing string deserialization and therefore gives faster lookup times. But this is an advanced setting that can lead to out-of-memory exceptions, so use it with caution.  |
|None | Use this when a column has high cardinality/most rows are unique. Don't use this encoding if you want to apply an inverted index to the column as this index type isn't supported without a dictionary encoding. |

## Index types

We recommend the following index types for certain use cases shown in the following table.

|Type | Description|
|:---|:---|
|Sorted |When enabled, Pinot uses a sorted forward index with run-length encoding on top of the dictionary encoding. Instead of saving dictionary IDs for each document ID, Pinot will store a pair of start and end document IDs for each value.|
|Inverted| When enabled, Pinot maintains a map from each value to a bitmap of rows, which makes value lookup take constant time. If you have a column that is frequently used for an exact lookup or filtering, adding an inverted index will improve performance greatly.|
|Range| Better performance for queries that involve filtering over a range. For example, timestamp-based queries will likely find the records less than, equal to, or between dates, so a range index works best.|
|Bloom filter| Prunes segments that do not contain any record matching an EQUALITY predicate.|
|Text| Used on STRING columns where doing standard filter operations (EQUALITY, RANGE, BETWEEN) doesn't fit the bill because each column value is a large blob of text or contains a free text whose length varies a lot across columns.|
|JSON |Can be applied to JSON string columns to accelerate the value lookup and filtering for the column.|

## Sample configuration with indexing and column type encoding

An example of a JSON configuration that includes both indexing and column type encoding:

```json
{
  "fieldConfigs": [
    {
      "name": "extracurricular",
      "encodingType": "dictionary",
      "indexes": {
        "dictionary": {
          "useVarLengthDictionary": true
        },
        "inverted": {}
      }
    },
    {
      "name": "name",
      "encodingType": "dictionary",
      "indexes": {
        "dictionary": {}
      }
    },
    {
      "name": "created_ts",
      "encodingType": "dictionary",
      "indexes": {
        "dictionary": {}
      }
    }
  ],
  "starTreeIndexConfigs": [
    {
      "dimensionsSplitOrder": [
        "firstName",
        "lastName",
        "gender"
      ],
      "functionColumnPairs": [
        "COUNT__*"
      ],
      "skipStarNodeCreationForDimensions": [],
      "maxLeadRecords": 10000
    }
  ]
}
```
