## Use a lookup to review various services running on well-known ports:

| lookup well_known_ports port AS dest_port OUTPUT service | search service!=NULL | timechart span=1d count by service