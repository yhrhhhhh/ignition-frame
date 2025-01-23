## 使用`pymysql` 库远程数据库表迁移

### 数据库`alarm.alarm_events`表数据迁移

- **功能：**

> 使用 Python 和 `pymysql` 库将数据从源数据库的 `alarm.alarm_events` 表中经过过滤后插入到目标数据`alarm.alarm_events` 表库中。
>
> 源数据库信息（本地）:
>
> ```makefile
> 'host': '127.0.0.1'
> 'user': 'root'
> 'password': 'dcny123'
> ```
> 目标数据库地址：
>
> ```makefile
>  'host': '10.108.145.1'
>  'user': 'root'
>  'password': '123456'
> ```

- **代码：**

```python
import pymysql

# 源数据库配置
source_db_config = {
    'host': '127.0.0.1',
    'user': 'root',
    'password': 'dcny123',
    'database': 'alarm',
}

# 目标数据库配置
target_db_config = {
    'host': '10.108.145.1',
    'user': 'root',
    'password': '123456',
    'database': 'alarm',
}

# 连接到源数据库
source_conn = pymysql.connect(**source_db_config)
source_cursor = source_conn.cursor()

# 连接到目标数据库
target_conn = pymysql.connect(**target_db_config)
target_cursor = target_conn.cursor()

try:
    select_query = """
    SELECT * 
    FROM alarm_events 
    WHERE id > 500000 AND id < 1000000
    AND source NOT LIKE '%Sensor%' 
    AND source NOT LIKE '%Environment%'
    """
    source_cursor.execute(select_query)
    rows = source_cursor.fetchall()

    insert_query = """
    INSERT INTO alarm_events (id, eventid, source, displaypath, priority, eventtype, eventflags, eventtime)
    VALUES (%s, %s, %s, %s, %s, %s, %s, %s)
    """

    target_cursor.executemany(insert_query, rows)
    target_conn.commit()

    # 提交事务
    target_conn.commit()

finally:
    # 关闭数据库连接
    source_cursor.close()
    source_conn.close()
    target_cursor.close()
    target_conn.close()

```

### 数据库`alarm.alarm_event_data`表数据迁移

- **功能：**

> 使用 Python 和 `pymysql` 库，首先查询目标数据库中的 `alarm.alarm_events` 表中的所有 `id`，然后使用这些 `id` 从源数据库中的 `alarm.alarm_event_data` 表中获取对应的数据，并将这些数据插入到目标数据库中的 `alarm.alarm_event_data` 表中。
>
> 源数据库信息（本地）:
>
> ```makefile
> 'host': '127.0.0.1'
> 'user': 'root'
> 'password': 'dcny123'
> ```
>
> 目标数据库地址：
>
> ```makefile
>  'host': '10.108.145.1'
>  'user': 'root'
>  'password': '123456'
> ```

- **代码：**

```python
import pymysql

# 源数据库配置
source_db_config = {
    'host': '127.0.0.1',
    'user': 'root',
    'password': 'dcny123',
    'database': 'alarm',
}

# 目标数据库配置
target_db_config = {
    'host': '10.108.145.1',
    'user': 'root',
    'password': '123456',
    'database': 'alarm',
}

# 连接到源数据库
source_conn = pymysql.connect(**source_db_config)
source_cursor = source_conn.cursor()

# 连接到目标数据库
target_conn = pymysql.connect(**target_db_config)
target_cursor = target_conn.cursor()

try:
	# 获取目标数据库中的 alarm.alarm_events 表中的所有 id
	target_cursor.execute("SELECT id FROM alarm_events")
	target_ids = target_cursor.fetchall()
	target_ids = [id[0] for id in target_ids]  # 将结果转换为列表

	# 根据 target_ids 查询源数据库中的 alarm.alarm_event_data 表中的数据
	format_strings = ','.join(['%s'] * len(target_ids))
	select_query = f"SELECT * FROM alarm_event_data WHERE id IN ({format_strings})"
	source_cursor.execute(select_query, target_ids)
	rows = source_cursor.fetchall()

	# 插入数据到目标数据库中的 alarm.alarm_event_data 表中
	insert_query = """
    INSERT INTO alarm_event_data (id, propname, dtype, intvalue, floatvalue, strvalue)
    VALUES (%s, %s, %s, %s, %s, %s)
    """

	if rows:
		target_cursor.executemany(insert_query, rows)
		target_conn.commit()

	print(f"{len(rows)} 条数据已插入到目标数据库中的 alarm_event_data 表中。")

finally:
	source_cursor.close()
	source_conn.close()
	target_cursor.close()
	target_conn.close()
```



### 数据库`audit.audit_events`表数据迁移

- **功能：**

> 使用 Python 和 `pymysql` 库将数据从源数据库的 `audit.audit_events` 表中经过过滤后插入到目标数据`audit.audit_events` 表库中。
>
> 源数据库信息（本地）:
>
> ```makefile
> 'host': '127.0.0.1'
> 'user': 'root'
> 'password': 'dcny123'
> ```
>
> 目标数据库地址：
>
> ```makefile
> 'host': '10.108.145.1'
> 'user': 'root'
> 'password': '123456'
> ```

- **代码**

```python
import pymysql

# 源数据库配置
source_db_config = {
    'host': '127.0.0.1',
    'user': 'root',
    'password': 'dcny123',
    'database': 'audit',
}

# 目标数据库配置
target_db_config = {
    'host': '10.108.145.1',
    'user': 'root',
    'password': '123456',
    'database': 'audit',
}

# 连接到源数据库
source_conn = pymysql.connect(**source_db_config)
source_cursor = source_conn.cursor()

# 连接到目标数据库
target_conn = pymysql.connect(**target_db_config)
target_cursor = target_conn.cursor()

try:
    select_query = """
        SELECT * 
        FROM audit_events 
        WHERE id < 100000
        AND actor NOT LIKE '%Unauthenticated%' 
        """
    source_cursor.execute(select_query)
    rows = source_cursor.fetchall()

    insert_query = """
        INSERT INTO aduit_events (audit_events_id, event_timestamp, actor, actor_host, action, action_target, action_value, status_code, originating_system, originating_context)
        VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s)
        """

    target_cursor.executemany(insert_query, rows)
    target_conn.commit()

    # 提交事务
    target_conn.commit()

finally:
    # 关闭数据库连接
    source_cursor.close()
    source_conn.close()
    target_cursor.close()
    target_conn.close()
```

Migration数据库整体迁移

对于大数量数据迁移可采用数据库整体迁移方法,此办法迁移速度更快：

配置Migration

以MySQLBranch为例，Database—>Migration Wizard

配置目标数据库的地址和用户名：

![1722494367427](https://github.com/user-attachments/assets/b0256ba9-592e-4b08-b998-056856f57f1f)

选择要传入的数据库。

![1722495340484](https://github.com/user-attachments/assets/e04a87e5-d011-4117-bca9-bcb5ef6ba9c6)


后面，根据提示点击next即可。
