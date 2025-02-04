---
sidebar_label: startree-remote-http-percentile
position: 57
---
# startree-remote-http-percentile 
## Description 
Remote HTTP Detector Template
## Flowchart 
```mermaid
graph TD
impactProcessor>"impactProcessor<br>IMPACT "]--->root
eventProcessor>"eventProcessor<br>EVENTS "]--->impactProcessor
offsetAggregationProcessor>"offsetAggregationProcessor<br>OFFSET_AGGREGATION "]--->eventProcessor
guardrailThresholdProcessor>"guardrailThresholdProcessor<br>THRESHOLD "]--->offsetAggregationProcessor
metricThresholdProcessor>"metricThresholdProcessor<br>THRESHOLD "]--->guardrailThresholdProcessor
timeOfWeekProcessor>"timeOfWeekProcessor<br>TIME_OF_WEEK "]--->metricThresholdProcessor
coldStartProcessor>"coldStartProcessor<br>COLD_START "]--->timeOfWeekProcessor
anomalyDetector[["anomalyDetector<br>REMOTE_HTTP "]]--->coldStartProcessor
missingDataPointsFiller["missingDataPointsFiller<br>TimeIndexFiller "]--->anomalyDetector
currentDataWithHistoryFetcher[("currentDataWithHistoryFetcher<br>DataFetcher ")]--->missingDataPointsFiller
root>"root<br>ANOMALY_MERGER "]
eventsDataFetcher[("eventsDataFetcher<br>EventFetcher ")]--->anomalyDetector
root ---> END((END))
```
## Parameters 
### DATA
| name | description | default value |
| --- | --- | --- |
|aggregationColumn|The column to aggregate. Can be a derived metric.|-|
|aggregationFunction|The aggregation function to apply on the aggregationColumn. Example: `AVG`.|-|
|dataSource|The Pinot datasource to use.|-|
|dataset|The dataset to query.|-|
|monitoringGranularity|The period of aggregation of the timeseries. In ISO-8601 format. Example: `PT1H`.|-|
|completenessDelay|The time for your data to be considered complete and ready for anomaly detection. In ISO-8601 format. Example: `PT2H`. [Learn more](https://dev.startree.ai/docs/startree-enterprise-edition/startree-thirdeye/concepts/alert-configuration#completenessdelay).|P0D|
|timezone|Timezone used to group by time. In [TZ-identifier](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) format.<br></br>For instance, `UTC` or `US/Pacific`.|UTC|
|queryFilters|Filters to apply when fetching data. Prefix with `AND`. Example: `AND country='US'`||
|timeColumnFormat|Required if timeColumn is not AUTO. [Learn more](https://dev.startree.ai/docs/startree-enterprise-edition/startree-thirdeye/reference/operators/data-fetcher#timeformat-strings).||
|timeColumn|TimeColumn used to group by time. If set to AUTO (the default value), the Pinot primary time column is used.|AUTO|
|queryLimit|Maximum number of timeseries point to fetch.|100000000|
|aggregationParameter|The second argument of the aggregationFunction. Example: for `PERCENTILETDIGEST`: `95`.|-|

### DETECTION
| name | description | default value |
| --- | --- | --- |
|lookback|Historical time period to use to train the model. In ISO-8601 format. Example: `P21D`.|-|
|pattern|Whether to detect an anomaly if it's a drop, a spike or any of the two.|UP_OR_DOWN|
|seasonalityPeriod|Biggest seasonality period to learn. In ISO-8601 format. Example: `P7D`.|P7D|

### FILTER
#### Time of week
| name | description | default value |
| --- | --- | --- |
|daysOfWeek|Used to ignore anomalies that happen at specific time periods. A list of days. Anomalies happening on these days are ignored if timeOfWeekIgnore is true. Example: `["MONDAY", "SUNDAY"]`.|[]|
|hoursOfDay|Used to ignore anomalies that happen at specific time periods. A list of hours. Anomalies happening on these hours are ignored. Example: `[0,1,2,23]`|[]|
|dayHoursOfWeek|Used to ignore anomalies that happen at specific time periods. A mapping of `{DAY: [hours]}`. Anomalies happening on these timeframes are ignored if timeOfWeekIgnore is true. Example: `{"FRIDAY": [22, 23], "SATURDAY": [0, 1, 2]}`|{}|
#### Threshold
| name | description | default value |
| --- | --- | --- |
|thresholdFilterMin|Used to ignore anomalies that don't meet the thresholdFilter min and max. Example: set `thresholdFilterMin = 10` to ignore anomalies when the metric is smaller than 10. Can help ignore anomalies happening in low data regimes. Filter threshold minimum. If `-1`, no minimum threshold is applied.|-1|
|thresholdFilterMax|Used to ignore anomalies that don't meet the thresholdFilter min and max. Example: set `thresholdFilterMin = 10` to ignore anomalies when the metric is smaller than 10. Can help ignore anomalies happening in low data regimes. Filter threshold maximum. If `-1`, no maximum threshold is applied.|-1|
#### Guardrail metric
| name | description | default value |
| --- | --- | --- |
|guardrailMetricMin|Used to ignore anomalies that don't meet the guardrail threshold. Minimum threshold of the guardrail metric. If `-1`, no minimum threshold is applied.|-1|
|guardrailMetricMax|Used to ignore anomalies that don't meet the guardrail threshold. Maximum threshold of guardrailMetric. If `-1`, no maximum threshold is applied.|-1|
|guardrailMetric|Used to ignore anomalies that don't meet the guardrail threshold. Metric to use as a threshold guardrail. Example: `COUNT(*)` and set `guardrailMetricMin = 100` to ignore anomalies detected when there is less than 100 observations in the period.|COUNT(*)|
#### Simple baseline
| name | description | default value |
| --- | --- | --- |
|offsetBaselineFilterPattern|Used to ignore anomalies that are not detected as anomalies by a simple model. Whether to detect an anomaly if it's a drop, a spike or any of the two.|UP_OR_DOWN|
|offsetBaselineFilterSensitivity|Used to ignore anomalies that are not detected as anomalies by a simple model. Detection sensitivity. For instance with `offsetBaselineFilterIntervalsMethod=PERCENTAGE`, set 50 for a 50% percentage change threshold. With `offsetBaselineFilterIntervalsMethod=ABSOLUTE`, set 200 for a 200 absolute difference threshold between the metric and the baseline.|-1|
|offsetBaselineFilterIntervalsMethod|Used to ignore anomalies that are not detected as anomalies by a simple model. Method to compute intervals. `PERCENTAGE` or `ABSOLUTE`.|ABSOLUTE|
|offsetBaselineFilterModelOffsets|Used to ignore anomalies that are not detected as anomalies by a simple model. A list of offsets in ISO-8601 format to use as baseline. Eg `[P7D, P14D]` will compare the current value to the aggregation of the values of the 2 previous weeks.|['P7D']|
|offsetBaselineFilterModelAggregation|Used to ignore anomalies that are not detected as anomalies by a simple model. The aggregation function to use to combine historical values. In `MEDIAN`, `AVERAGE`, `MIN`, `MAX` and any of `PCTXXXXX` eg `PCT05` (5th percentile), `PCT95`, `PCT999` (99.9th percentile).|MEDIAN|
#### Special events
| name | description | default value |
| --- | --- | --- |
|eventFilterSqlFilter|Used to ignore anomalies that happen during events. Sql filter to apply on the events. [Learn more](https://dev.startree.ai/docs/startree-enterprise-edition/startree-thirdeye/reference/operators/event-fetcher#sql-filter)||
|eventFilterLookaround|Used to ignore anomalies that happen during events. Offset to apply on startTime and endTime to look around the timeframe. In ISO-8601 format. Example: `P1D`.|P2D|
|eventFilterTypes|Used to ignore anomalies that happen during events. List of event types to fetch by. Example: `["HOLIDAY", "DEPLOYMENT"]`. `[]` fetches all events. Use `["__NO_EVENTS"]` to disable.|['__NO_EVENTS']|
|eventFilterBeforeEventMargin|Used to ignore anomalies that happen during events. A period in ISO-8601 format that corresponds to a period that is also impacted by the event. Example: if beforeEventMargin is `P1D`, if event happens on `[Dec 24 0:00, Dec 25 0:00[`, the label will be applied to anomalies happening on `[Dec 23 0:00 and Dec 25 0:00[`|P0D|
|eventFilterAfterEventMargin|Used to ignore anomalies that happen during events. Same as eventFilterBeforeEventMargin at the end of the event.|P0D|
#### Impact
| name | description | default value |
| --- | --- | --- |
|impactThreshold|Used to ignore anomalies that don't meet the impact threshold. Impact filter threshold.|-1|

### POSTPROCESS
#### Data mutability
| name | description | default value |
| --- | --- | --- |
|mutabilityPeriod|Use if your data is mutable. ThirdEye will maintain the detection results up to date on the mutable period. For instance, if your last 10 days of data is mutable, set `P10D`. At each cron detection job, the detection results for the last 10 days will be updated.|P0D|
|reNotifyPercentageThreshold|For detection replay when data is mutable. If the percentage difference between an existing anomaly and a new anomaly on the same time frame is above this threshold, renotify. Combined with `reNotifyAbsoluteThreshold`. Both thresholds must pass to be re-notified. If zero, always renotify. If null or negative, never re-notifies.|-1|
|reNotifyAbsoluteThreshold|For detection replay when data is mutable. If the absolute difference between an existing anomaly and a new anomaly on the same time frame is above this threshold, renotify. Combined with `reNotifyPercentageThreshold`. Both thresholds must pass to be re-notified. If zero, always renotify. If null or negative, never re-notifies.|-1|
#### Anomaly merger
| name | description | default value |
| --- | --- | --- |
|mergeMaxDuration|Maximum duration of an anomaly merger. At merge time, if an anomaly merger would get bigger than this limit, the anomalies are not merged. In ISO-8601 format. Example: `P7D`.||
|mergeMaxGap|Maximum gap between 2 anomalies for anomalies to be merged. In ISO-8601 format. Example: `PT2H`. The default behavior is to merge consecutive anomalies only. To disable anomaly merging entirely, set this value to `P0D`.||

### RCA
| name | description | default value |
| --- | --- | --- |
|rcaExcludedDimensions|List of dimensions (columns in the dataset) to ignore in RCA drill-downs. If not set or empty, all dimensions of the table are used. rcaExcludedDimensions and rcaIncludedDimensions cannot be used at the same time.|[]|
|rcaIncludedDimensions|List of the dimensions (columns in the dataset) to use in RCA drill-downs. If not set or empty, all dimensions of the table are used. [Learn more](https://dev.startree.ai/docs/startree-enterprise-edition/startree-thirdeye/concepts/alert-configuration#dimensions).|[]|
|rcaAggregationFunction|The aggregation function to use for RCA. If the detection metric name is known to ThirdEye, this parameter is optional.||
|rcaEventTypes|A list of type to filter on for RCA. Only events that match such types will be shown in the RCA related events tab. [Learn more](https://dev.startree.ai/docs/startree-enterprise-edition/startree-thirdeye/concepts/alert-configuration#types).|[]|
|rcaEventSqlFilter|A Sql filter for RCA events. Only events that match the filter will be shown in the RCA related events tab. [Learn more](https://dev.startree.ai/docs/startree-enterprise-edition/startree-thirdeye/concepts/alert-configuration#sqlfilter).||

### OTHER
| name | description | default value |
| --- | --- | --- |
|url|-|-|
|eventSqlFilter|-||
|eventTypes|-|['__NO_EVENTS']|
|eventLookaround|-|P1D|

