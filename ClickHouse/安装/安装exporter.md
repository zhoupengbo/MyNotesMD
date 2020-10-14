```bash
#!/bin/bash
export CLICKHOUSE_USER=default
export CLICKHOUSE_PASSWORD=xxxxx
nohup ./clickhouse_exporter -scrape_uri=http://localhost:8123/ &
```
