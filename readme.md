# Splunk Queries and Reporting 
- These are example so of queries I've run on Apache and Window Servers Logs as part of a 2022 course on cybersecurity where I was using Splunk for SOC monitoring

## Apache HTTP Response Values Observed 
- This query will report the distinct HTTP responses observed in a data set. This was the basis for a table of the different HTTP methods (GET, POST, HEAD, etc). This provided insight into the type of HTTP activity being requested against a web server.
`source="apache_logs.txt" host="aa919eeffb10" sourcetype="access_combined" | stats values(method)`


## Apache HTTP Response Counts
- This query provide insight into any suspicious levels of HTTP responses.
`source="apache_logs.txt" host="aa919eeffb10" sourcetype="access_combined" | stats count by method`

![](ApacheCounts.png)

## Apache Server Post requesets baselining 
- Query to establish baseline for post requests alerts 
`source="apache_logs.txt" host="aa919eeffb10" sourcetype="access_combined"  method=POST |stats count by _time | stats avg(count) median(count) min(count) max(count)`

## Apache Server IP Locations Outside of the United States
- This report establishes the baseline to inform an alert around hour counts of the POST method 
`source="apache_logs.txt" | iplocation clientip | where Country!="United States" |stats count by _time | stats avg(count) median(count) min(count) max(count)`

### Apache Server Alert for IP Locations Outside of the USA observed 
- This is the hourly alert monitor in Splunk to report when non-US hits to the Apache server is above baseline 

`source="apache_logs.txt" | iplocation clientip | where Country!="United States" |  bucket_time span=1h | stats count by _time | stats avg(count) median(count) min(count) max(count)` 

### Apache Server iplocations mapped by country 
- This query returns the data used to map the country location based on the `clientip` field.

`source="apache_logs.txt" | iplocation clientip | geostats count by Country` 
![](ApacheMap.png)

## Apache Server count of top 10 country traffic volume 
- Query that returns the data to visualize via a bar, column, or pie chart that displays the counts of the top 10 countries.

`source="apache_logs.txt" | iplocation clientip | top limit=10 Country`

### Apache Server count of distinct URIs 
-  Counts total URIs observed 
`source="apache_logs.txt"| stats count values(uri)` 

### Apache Server count of top 15 URIs by volume
- This query returns the data that is passed to the chart that displays the number of different URIs.
`source="apache_logs.txt"| top uri limit=15` 
![](ApacheCountUri.png)


## Windows Server Logs Severity Read Out
- This query returns a count of incidents by severity 
`source="windows_server_logs.csv" host="aa919eeffb10" sourcetype="csv"  | stats count by severity | eventstats sum(count) as perc | eval perc=round(count*100/perc,2)` 

## Windows Server Logs Success and Failure Report
- Count of failure/success 
`source="windows_server_attack_logs.csv"  | stats count by status | eventstats sum(count) as perc | eval perc=round(count*100/perc,2)`

## Windows Alerts for High Failure Status Baselininng
- Stats to trigger alert thresholds for failure rates
`source="windows_server_logs.csv" host="aa919eeffb10" sourcetype="csv" status="failure" | bucket_time span=1h | stats count by _time | stats avg(count) median(count) min(count) max(count)` 

## Windows Server Logs Alerts for High Successful login Baselining
- Stats
`source="windows_server_logs.csv" host="aa919eeffb10" sourcetype="csv" signature="An account was successfully logged on" | bucket_time span=1h | stats count by _time | stats avg(count) median(count) min(count) max(count)` 

signature IN "An account was successfully logged on", "A user account was locked out", "An account was successfully logged on", "An attempt was made to reset an accounts password"

# Cluster Map Query - Map POST requests by Source IP 
- Data to visualize a cluster map 
`source="radialgauge.csv" http_method="POST"  | iplocation src_ip | geostats count`

# Splunk Query to select multiple fields/values in a variable
- This is the equivalent of WHERE IN( ) in SQL
`source="windows_server_attack_logs.csv"  signature IN ("An account was successfully logged on", "A user account was locked out", "An attempt was made to reset an accounts password") | stats count by user`