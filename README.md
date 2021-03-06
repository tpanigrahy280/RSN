# RSN
R interface for ServiceNow REST API

If your company leverages R for your data needs and uses ServiceNow, this repository can become useful to quickly extract data from ServiceNow into R Data Frames.

Currently there are 2 quick functions for ServiceNow Table API and Aggregate API:
 - Table API - obtain raw table data from ServiceNow - rsn_table_api().
 - Aggregate API - use SN native function to aggregate data before returning it into R - rsn_aggr_api().

# News

### 2017-07-06:

2 new arguments added:
- `retry` (logical): in case if API returns an empty result or an error, RSN will retry multiple times before exiting (default: `TRUE`). Set it to `FALSE` to speed up certain batches, but it has its downsides.
- `inital_check` (logical): for `rsn_table_api()` only. If set to `TRUE`, RSN will first attempt to extract 1 record based on your query (limit 1) to see if data exists based on your filtering condition. It will only proceed with full data extraction if it finds at least 1 record (default: `FALSE`).


### 2017-04-05:
Apparently, the access to demoXXX instances has been discontinued by ServiceNow. But you can create your own developer instance. Here is  a video I found on how to do that: https://www.youtube.com/watch?v=VEGedFeIHnQ.

# Requirements
 - access to a ServiceNow instance with REST API access;
 - R>=3.2.0;
 - R packages: RCurl, jsonlite, lubridate, utils, gsubfn;

# Example

This example uses ServiceNow demo instance:
https://demo016.service-now.com
 - login: itil
 - password: itil

Note: this demo instance might not be available. Last checked: 2017-02-07. Demo instances might be available demo016 through demo022.

### Table API:
```
source(file=url("https://raw.githubusercontent.com/kirillsavine/RSN/master/rsn.r"))

rsn_table_api(
	domain="https://demo016.service-now.com",  
	user_pwd="itil:itil",   
	table_name="incident",		
	sysparm_query="active%3Dtrue",	 
	sysparm_fields=c("number","opened_at","short_description"),  
	sysparm_limit=10   
)
```
		
### Aggregate API:
```
source(file=url("https://raw.githubusercontent.com/kirillsavine/RSN/master/rsn.r"))

rsn_aggr_api(
	domain="https://demo016.service-now.com",	
	user_pwd="itil:itil",				
	table_name="incident",				
	sysparm_query="numberSTARTSWITHINC&state=2&approval=not requested",	
 sysparm_group_by=c("assigned_to.name","state"), 
	sysparm_aggregate_fields=list("avg"="reassignment_count","sum"="business_duration")			
)		
```		
		
# Arguments

 - *domain* - character string, path to your SN instance.
 - *user_pwd* - character string, user:password to your SN instance.
 - *table_name* - character string, name of SN table you wish to query.
 - *sysparm_query* - character string, url filtering query, arguments sperated by '&' sign.

### rsn_table_api():
 - *sysparm_fields* - character array, column names you wish to return.
 - *sysparm_limit* - integer, limits the size of rsulting data.frame. returns top N rows, st to a very large number to return all records

### rsn_aggr_api():
 - *sysparm_group_by* - character array, table variables you wish to group by.
 - *sysparm_aggregate_fields* - named list or a named array, table variables you wish to group by.
 - ... other arguments to be passed to the REST API call.

# notes
1.	in table API when setting the list of columns to return (the sysparm_fields argument), be aware that certain field names are looked up in other tables, although it might not look like the case when you browse the  table in a browser using the user interface. Therefore, for situations like this you have to specify the column you need to return from the looked-up table. For example, at the time of writing this, the demo021 instance had a table named 'incident' with a column 'assigned_to', however when you just include 'assigned_to' in sysparm_fields list, the function will return a list of assigned_to 'assigned_to.link' and 'assigned_to.value'. None of those contain the name of the 'assigned_to' person. To return the name, include assigned_to.name' in sysparm_fields because the name of the 'assigned_to' person is located in the 'sys_user' tablewhich we can tell by looking at URL api call found in the 'assigned_to.link' column value.
2.	Thank you to [Gobii Vivagananda](https://www.linkedin.com/in/gobii/) who initially referred me to the SN REST API documentation.


# Learn More:
 - ServiceNow wiki on Table API: http://wiki.servicenow.com/index.php?title=Table_API 
 - ServiceNow wiki on Aggregate API: http://wiki.servicenow.com/index.php?title=Aggregate_API

