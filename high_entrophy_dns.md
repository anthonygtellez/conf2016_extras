# Requires URL Toolbox

# parse query into subdomain, domain, tld
`| lookup ut_parse_extended_lookup url AS query`

# get a shannon entrophy score for the subdomain
`| lookup ut_shannon_lookup word AS ut_subdomain`

# find only results with a shannon score > score of 4
`| search ut_shannon > 4`

# filter out cdn providers
`ut_domain_without_tld!="amazonaws"`


# filer out cdn providers
```
| lookup cloud_providers domain AS ut_domain OUTPUT CDN_provider isProvider
| where NOT isProvider="True" AND ut_shannon > 4
```
#  or use lookup to look for bad stuff
```
| lookup ddns dyndns_domains AS ut_domain OUTPUT isBad
| where NOT isBad="True" AND ut_shannon > 4
```
# Full Query for InfoBlox
```
index=infoblox 
| lookup ut_parse_extended_lookup url AS query
| lookup ut_shannon_lookup word AS ut_subdomain
| search ut_shannon > 4 ut_domain_without_tld!="amazonaws" 
| stats sparkline(count) as activity count by query  ut_shannon | sort - ut_shannon
```
# Full Query for Bro
```
index=bro 
| lookup ut_parse_extended_lookup url AS query
| lookup ut_shannon_lookup word AS ut_subdomain
| search ut_shannon > 4 ut_domain_without_tld!="amazonaws" 
| stats sparkline(count) as activity count by query  ut_shannon | sort - ut_shannon
```
# Full Query for Stream
```
index=stream 
| lookup ut_parse_extended_lookup url AS query
| lookup ut_shannon_lookup word AS ut_subdomain
| search ut_shannon > 4 ut_domain_without_tld!="amazonaws" 
| stats sparkline(count) as activity count by query  ut_shannon | sort - ut_shannon
```
# Full Query for Suricata
```
index=suricata event_type=dns
| lookup ut_parse_extended_lookup url AS query
| lookup ut_shannon_lookup word AS ut_subdomain
| search ut_shannon > 4 ut_domain_without_tld!="amazonaws" 
| stats sparkline(count) as activity count by query  ut_shannon | sort - ut_shannon
```

# High Entrophy URLs, example can defeat shannon entrophy scoring by using shorter subdomains, (this example scores 3.943465189601646).
```
index=stream network_interface="/splunk/pcap-malware-traffic/2016-05-13-traffic-analysis-exercise.pcap" src_ip="10.0.21.136" http_method=GET url="http://e7qx9y.he6gnm.top/"
| lookup ut_parse_extended_lookup url
| lookup ut_shannon_lookup word AS url
```

# High Entrophy URLs, example can defeat shannon entrophy scoring by using shorter subdomains, (this example scores 2.584962500721156).
```
index=stream network_interface="/splunk/pcap-malware-traffic/2016-05-13-traffic-analysis-exercise.pcap" src_ip="10.0.21.136" http_method=GET url="http://e7qx9y.he6gnm.top/"
| lookup ut_parse_extended_lookup url
| lookup ut_shannon_lookup word AS ut_subdomain
```

# Full Query for Suricata HTTP
```
index=suricata host=suricata event_type=http  
| lookup ut_parse_extended_lookup url AS dest 
| lookup ut_shannon_lookup word AS ut_subdomain OUTPUT ut_shannon AS ut_shannon_subdomain 
| lookup ut_shannon_lookup word AS dest OUTPUT ut_shannon AS ut_shannon_dest 
| search ut_shannon_dest > 4 OR ut_shannon_subdomain > 4 
| table ut_subdomain ut_shannon_subdomain dest ut_shannon_dest   
| dedup dest ut_subdomain
```
