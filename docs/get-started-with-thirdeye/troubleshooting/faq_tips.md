## Alert
<details>
<summary>My datasource historical data can be updated. What is the impact for ThirdEye?
</summary>

If your historical data can be updated, ThirdEye will run detection pipelines on data that can change later.     
This has 2 impacts:
- any historical anomalies created by a detection pipeline run can be invalid after a data update. 
- true anomalies caused by a data update can be missed 
  
To mitigate this, you should think in terms of [watermark](https://www.oreilly.com/library/view/streaming-systems/9781491983867/ch03.html):
- what is the expected time for my dataset to be relevant for alerts?
- what is the expected time for my dataset to be immutable? 
  
Example: your data is supposed to arrive in 3 hours, but the data can frequently change within the last 5 days. You could set up:
1. one alert pipeline that runs with a delay of 3 hours.  
   This alert will be useful to quickly catch problems. It may have false positives.
2. one pipeline that runs with a delay of 5 days.  
   This alert will not be impacted by the data updates. An anomaly raised by this alert is important.

If data updates don't happen often, you can also reset the alert and replay it on a timeframe. 
This can be done with the API.
</details>

<details>
<summary>When I create an alert, do I need to wait for the first detection job run to see anomalies?</summary>

No need! When an alert is created, ThirdEye will replay the alert on past data and create anomalies. 
Depending on the historical data volume and alert monitoring configuration, this could take between 1 second and 30 minutes. 
</details>

<details>
<summary>What is the timezone used by ThirdEye?</summary>

The UI uses the timezone of your browser.   
The CRON in subscription and alert configurations is in UTC.  
The notifications messages timezone is configurable. If you use ThirdEye as SaaS, reach out to the StarTree team to change the timezone.  
The alert timezone can be configured in [metadata$timezone](../../get-started-with-thirdeye/alert-configuration.md#timezone)
</details>


<details>
<summary style="display:block;">What is the alert <code>lastTimestamp</code>? When a detection cron job is run, what is the detection timeframe?</summary>

`lastTimestamp` is the detection watermark. It corresponds to the last detection upper bound (excluded) of the alert.
When an alert runs for the first time, `lastTimestamp` is 0.  

When a detection cron job runs, the detection timeframe is computed as follows:  
- The base timeframe is `[lastTimestamp, cronScheduleTime]`.  
  `cronScheduleTime` is the exact cron time. For instance, if you schedule the cron at 3am, cronScheduleTime = 3:00, even if the processing runtime is slightly different, like 3:01.
- The timeframe is corrected with [completenessDelay](../../get-started-with-thirdeye/alert-configuration.md#completenessdelay): `[lastTimestamp, cronScheduleTime-completenessDelay]` 
- The timeframe is corrected with [monitoringGranularity](../../get-started-with-thirdeye/alert-configuration.md#granularity): `[lastTimestamp, floorByGranularity(cronScheduleTime-completenessDelay)]` 
  The goal is to avoid reading buckets that are incomplete.

completenessDelay and granularity are optional but highly recommended.
</details>
