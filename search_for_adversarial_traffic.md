## Search for  SSL connections with insecure cipher to adversaries 
index=bro sourcetype=bro_ssl | lookup insecure_ciphers cipher OUTPUT reason_insecure | search reason_insecure!="" 
| iplocation src_ip prefix=src_ | iplocation dest_ip prefix=dest_ | lookup adversaries country AS dest_Country OUTPUT isAdversary 
| search isAdversary=TRUE | stats sparkline(count) as activity count by src_ip dest_ip dest_Country | sort - count