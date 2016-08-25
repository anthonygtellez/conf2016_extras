# Generates summary statistics of all existing fields saving them as new fields

## This search determines lost sales by product name, by creating an averageLoss field
sourcetype=access_combined action=remove
| chart sum(price) as lostSales by product_name
| eventstats avg(lostSales) as averageLoss
| eval delta = (lostSales - avergaeLoss)
| where lostSales > averageLoss
| table product_name, delta
| sort -delta

## BroFiles Outliers for both bytes and duration
index=bro sourcetype=bro_files 
| eventstats avg("duration") as avg_dur stdev("duration") as stdev_dur avg("seen_bytes") as avg_bytes stdev("seen_bytes") as stdev_bytes 
| eval lowerBounddur=(avg_dur-stdev_dur*2), upperBounddur=(avg_dur+stdev_dur*2),lowerBoundbytes=(avg_bytes-stdev_bytes*2), upperBoundbytes=(avg_bytes+stdev_bytes*2)
| eval isOutlierdur=if('duration' > upperBounddur, 1, 0), isOutlierbytes=if('seen_bytes' > upperBoundbytes, 1, 0)
| search isOutlierdur=1 isOutlierbytes=1


## SSH Attempts Ouliers by count
index=suricata event_type=ssh 
| iplocation src_ip prefix=src_ 
| iplocation dest_ip prefix=dest_ 
| stats count by src_ip src_Country ssh_client_sofware dest_ip dest_Country ssh_server_software 
| eventstats  avg(count) AS avg_count, stdev(count) AS stdev_count 
| eval lowerbound = (avg_count-stdev_count*2), upperbound = (avg_count+stdev_count*2) 
| eval isOutlier=if('count' > upperbound, 1, 0) | sort - count