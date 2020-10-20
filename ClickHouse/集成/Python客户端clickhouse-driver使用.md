##### 安装：

```bash
pip install clickhouse-driver
```

##### 示例：

```python
from clickhouse_driver import Client

client = Client(host='localhost', port='9000', database='test', user='readonly', password='')
result = client.execute('SHOW DATABASES')
print(result)
```

官网：https://clickhouse-driver.readthedocs.io/en/latest/quickstart.html