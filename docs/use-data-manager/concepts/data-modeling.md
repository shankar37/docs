# Data modeling

Data modeling helps you organize and structure the data fetched from the selected data source into Pinot tables. 
To improve your data model, consider making adjustments to customize your columns, transform your data, or add or remove columns.

## Customize columns

When organizing your data in Pinot tables, consider the column data type, for example, as a Dimension, Metric, or DateTime, you can adjust these attributes to suit the nature of the data fetched. Additionally, if your data is structured in arrays, it enables you to configure columns capable of handling multiple values. Moreover, you can set default values for columns in instances where the value is absent. This customization lets you align your data more effectively with your analytical needs.

## Transform data

Modify columns through a variety of transformation functions to manipulate and refine your data. Test transformations to ensure they meet your expectations using the “Data” section before applying changes. See all supported mapping functions. making the date-time column more or less granular. To modify the existing DateTime field type, click the Edit icon. This capability enhances the quality and usability of your datasets.

## Add new columns

Introducing new columns to your Pinot table schema is a seamless process. Whether it's adding dimensions or metrics, mandatory mapping functions can be assigned to these newly created columns. Moreover, DateTime columns can be generated from an existing column, simplifying the management of date formats or varying levels of granularity within your datasets. This feature empowers you to enhance the richness of their data by incorporating new elements efficiently.

## Clean up your tables

Keeping your Pinot table schema concise and relevant is crucial for efficient data management. Pinot lets you remove unnecessary columns by using the delete function in the schema interface. Retain only pertinent data columns in your dataset structures.

## Advanced options with JSON configuration

Pinot supports using JSON configurations for detailed specification of column properties such as name, type, data format, and transformations. By using JSON configurations, you can exercise a higher level of control over the structure and behavior of your data tables tailored to specific use cases.

### Examples of schema in JSON format

The following examples show how different types of data are represented in columns.

#### Single value dimension column

```json
{
    "name": "gender",
    "fieldType": "dimension",
    "dataType": "string"
}
```

#### Multi-value dimension column

For columns with multiple values (like `courses`), set the type as `dimension` with `mv(string)` for handling arrays.

```json
{
    "name": "courses",
    "fieldType": "dimension",
    "dataType": "mv(string)"
}
```

#### Metric column
   
```json
{
    "name": "studentID",
    "fieldType": "metric",
    "dataType": "long "
}
```

#### Timestamp column with epoch format

Create timestamp columns like `created_ts` with the format `EPOCH` and granularity `SECONDS`.

```json
{
    "name": "created_ts",
    "fieldType": "datetime",
    "dataType": "long",
    "format": "EPOCH|SECONDS|1",
    "granularity": "SECONDS|1"
}
```

### Examples of transformation functions in JSON format

Transformation functions in JSON have the following 3 components:
- transforms
- timeTransforms
- filter

```json
{
  "timeTransforms": [],
  "transforms": [],
  "filter": null
}
```

#### Transforming epoch seconds to epoch milliseconds

Easily convert timestamps from seconds to milliseconds using transformation functions.

```json
{
  "timeTransforms": [
      {
        "inputFieldName": "timestamp",
        "inputFormat": "EPOCH",
        "inputGranularity": "SECONDS",
        "outputFieldName": "created_ts",
        "outputFormat": "EPOCH",
        "outputGranularity": "MILLISECONDS"
      }
}
```

#### Creating a new string column by concatenation

Combine columns (like `firstName` and `lastName`) into a new column called `name`.

```json
{
  "transforms": [
    {
      "outputFieldName": "name",
      "transformFunction": "concat("firstName", "lastName")"
    }
}
```
