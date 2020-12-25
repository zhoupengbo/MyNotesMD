cd /etc/clickhouse-server/config.d

vim macros.xml

```xml
<yandex>
    <macros>
        <layer>01</layer>
        <shard>02</shard>
        <replica>ip</replica>
    </macros>
</yandex>
```

