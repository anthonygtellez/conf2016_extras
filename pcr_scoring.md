## Create a Ratio of bytes_in to bytes_out
index=suricata event_type=flow 
| eval bytes_total=bytes_in+bytes_out 
| eval bytes_ratio= ((bytes_out-bytes_in)/bytes_total) 
| iplocation dest_ip 
| table src_ip src_port dest_ip dest_port bytes_in bytes_out bytes_total bytes_ratio 
| sort - bytes_ratio


## Apply case logic to determine inboud or outbound

index=suricata event_type=flow 
| eval bytes_total=bytes_in+bytes_out 
| eval bytes_ratio= ((bytes_out-bytes_in)/bytes_total) 
| eval bytes_pcr_range = case(bytes_ratio > 0.4, "Pure Push", bytes_ratio > 0, "70:30 Export", bytes_ratio == 0, "Balanced Exchange", bytes_ratio >= -0.5, "3:1 Import", bytes_ratio > -1, "Pure Pull") 
| stats sparkline(count) AS activity by src_ip src_port dest_ip dest_port bytes_in bytes_out bytes_pcr_range
