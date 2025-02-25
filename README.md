# Web-Server-Log-Analysis

## Project Overview
In this hands on exercise we are analyzing server log data with Hive. 
Hive is a database query tool built on distributed cloud systems like Hadoop. 
It makes  complex queries easier, and uses strategies like paritioning to optimize queries.

## Implementation Approach

### Total Web Requests
`select count(*) from hands_on_hive.server_log;`
```
100
```

### Status Code Analysis
```
select status, count(*) as number_of_requests 
from server_log 
group by status;
```
```
status 	number_of_requests
200	    57
404	    21
500	    22
```

### Most Visited Pages
```
select url, count(*) as number_of_visits 
from server_log 
group by url;
```
```
url 	    number_of_visits
/about	    15
/checkout	17
/contact	24
/home	    20
/products	24
```

### Traffic Source Analysis
```
select user_agent, count(*) as user_agent_used 
from server_log
group by user_agent;
```
```
user_agent 	    user_agent_used
Chrome/90.0	    23
Edge/88.0	    23
Mozilla/5.0	    16
Opera/74.0	    21
Safari/13.1	    17
```

### Suspicious IP Adresses
```
select ip, count(*) as failed_requests 
from server_log
where not status = 200
group by ip
having count(*) >= 3;
```
```
ip 	            failed_requests
192.168.1.1	    5
192.168.1.10	3
192.168.1.15	7
192.168.1.2	    6
192.168.1.20	6
192.168.1.25	4
192.168.1.3	    9
192.168.1.30	3
```

### Traffic Trend Over Time
```
select 
from_unixtime(unix_timestamp(time_stamp), 'yyyy-MM-dd HH:mm') as event_minute, 
count(*) as requests
from server_log
group by from_unixtime(unix_timestamp(time_stamp), 'yyyy-MM-dd HH:mm');
```
```
event_minute 	    requests
2024-02-01 10:15	7
2024-02-01 10:16	2
2024-02-01 10:17	3
2024-02-01 10:18	2
2024-02-01 10:19	1
2024-02-01 10:20	4
2024-02-01 10:21	1
2024-02-01 10:22	2
2024-02-01 10:23	2
2024-02-01 10:24	1
2024-02-01 10:25	1
2024-02-01 10:26	6
2024-02-01 10:27	3
2024-02-01 10:29	1
2024-02-01 10:30	4
2024-02-01 10:31	3
2024-02-01 10:32	3
2024-02-01 10:34	2
2024-02-01 10:35	5
2024-02-01 10:36	1
2024-02-01 10:37	1
2024-02-01 10:38	7
2024-02-01 10:39	2
2024-02-01 10:40	3
2024-02-01 10:41	2
2024-02-01 10:42	1
2024-02-01 10:43	3
2024-02-01 10:46	1
2024-02-01 10:47	1
2024-02-01 10:49	1
2024-02-01 10:50	1
2024-02-01 10:51	4
2024-02-01 10:52	1
2024-02-01 10:53	5
2024-02-01 10:55	3
2024-02-01 10:56	3
2024-02-01 10:57	1
2024-02-01 10:58	3
2024-02-01 10:59	3
```

## Execution Steps: Provide commands used to set up and run the project.
### Creating the table
```
CREATE TABLE IF NOT EXISTS hands_on_hive.server_log ( 
ip string, 
time_stamp timestamp, 
url string, 
status int, 
user_agent string) 
PARTITIONED BY (status INT)
COMMENT 'Web Server Log Table' 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY ','; 
 
LOAD DATA INPATH '/user/example/web_server_logs.csv' INTO TABLE hands_on_hive.server_log; 
```

```
ALTER TABLE server_log ADD PARTITION (status=200);
ALTER TABLE server_log ADD PARTITION (status=404);
ALTER TABLE server_log ADD PARTITION (status=500);

show partitions server_log;
```

## Challenges Faced: Document any issues and how they were resolved.
Hive is very similar to SQL in its query language, but the partitioning part is very new.
Since the tool is so old, it is hard to find many examples or proper documentation on the subject.
I did not know what to partition on, how to split it up, etc.
Navigating the UI is quite confusing as well, creating tables with it does not work, instead you must create them using commands.
