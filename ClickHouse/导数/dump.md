```bash
#!/bin/bash

OUTDIR=.

while read -r db ; do
  while read -r table ; do

  if [ "$db" == "system" ]; then
     echo "skip system db"
     continue 2;
  fi

  if [[ "$table" == ".inner."* ]]; then
     echo "skip materialized view $table ($db)"
     continue;
  fi

  echo "export table $table from database $db"

    # dump schema
    clickhouse-client -q "SHOW CREATE TABLE ${db}.${table}" > "${OUTDIR}/${db}_${table}_schema.sql"

    # dump 
    clickhouse-client -q "SELECT * FROM ${db}.${table} FORMAT TabSeparated" | pigz > "${OUTDIR}/${db}_${table}_data.tsv.gz"

  done < <(clickhouse-client -q "SHOW TABLES FROM $db") 
done < <(clickhouse-client -q "SHOW DATABASES")
```

