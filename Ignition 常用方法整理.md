## Ignition 常用方法整理

## 时间处理：

> #### 获取时间的小时、分钟、秒

```python
end_hour = system.date.getHour24(value.end_time)  # 取时间小时部分
end_minute = system.date.getMinute(value.end_time)  # 取时间分钟部分
end_second = system.date.getSecond(value.end_time)  # 取时间秒部分
```

> #### 时间格式化

```python
pattern = u"yyyy-MM-dd HH:mm:ss"
now = system.date.now()
temp = system.date.format(now, pattern)
```

> #### 获取当天零点、昨天零点等

```python
now = system.date.now()
now_midnight = system.date.midnight(now)  # 当天零点
yesterday_midnight = system.date.addDays(now_midnight, -1)  # 昨天零点
two_yesterday_midnight = system.date.addDays(now_midnight, -2)  # 前天零点

month_date_num = system.date.getDayOfMonth(now_midnight)
month_start_time = system.date.addDays(now_midnight, -month_date_num + 1)  # 当月1日零点

past_month_midnight = system.date.addMonths(now_midnight, -1)  # 上月当日零点
last_month_start_time = system.date.addDays(past_month_midnight, -month_date_num + 1)  # 上月1日零点

two_past_month_midnight = system.date.addMonths(now_midnight, -2)
two_last_month_start_time = system.date.addDays(two_past_month_midnight, -month_date_num + 1)  # 上两月1日零点

year_date_num = system.date.getDayOfYear(now_midnight)
year_start_time = system.date.addDays(now_midnight, -year_date_num + 1)  # 今年1日零点
last_year_start_time = system.date.addYears(year_start_time, -1)  # 去年1日零点
```

> #### 格式化为小时和分钟

```python
formatted_time = system.date.format(system.date.addMinutes(now, -i * 10), "HH:mm")
```

---

## 数据处理
> #### 按照 `timestamp` 键排序字典列表

```python
sorted_data_desc = sorted(data, key=lambda x: x['timestamp'], reverse=True)
```

> #### 排序字典

```python
data_items = sorted(data.items())
```

> #### 排序二维列表

```python
data = sorted(data, key=lambda x: x[3])
```

> #### 计算列表的总和

```python
row_sum_one = sum(int(x) if x is not None else 0 for x in data_items[1:])
```

> #### 自定义排序列表

```python
label_list = ["1D", "1G", "2D", ..., "DP"]
sorted_root_node = sorted(root_node, key=lambda x: label_list.index(x['label']) if x['label'] in label_list else float('inf'))
```

---

## 数据集操作
> #### 转换为 PyDataset

```python
pyDataSet = system.dataset.toPyDataSet(resultdataset)
```

> #### 插入新列到 PyDataset

```python
for index, path in enumerate(paths):
    pydataset = system.dataset.toPyDataSet(
        system.tag.queryTagHistory(
            paths=paths[path], 
            startDate=startTime, 
            endDate=endTime, 
            aggregationMode='LastValue', 
            returnSize=return_size
        )
    )
    if not py_list:  # 时间只插入一次
        py_list = [[row[0]] for row in pydataset]
    sums = [sum((row[i] if row[i] is not None else 0) for i in range(1, len(row))) for row in pydataset]
    for i, sum_value in enumerate(sums):
        py_list[i].append(sum_value)
```

---

## Tag 操作
> #### 读写 Tag

```python
st_tag_path = []
alias = system.tag.readBlocking([st_tag_path])[0].value
```

> #### 创建 Tag

```python
import pandas as pd
from copy import deepcopy

tag_demo = {
    "name": "Device",
    "typeId": "VOC_device",
    "tagType": "UdtInstance",
    "tags": [
        {"name": "Floor", "tagType": "AtomicTag"},
        {"name": "Factory", "tagType": "AtomicTag"},
        {"name": "Reversed", "tagType": "AtomicTag"},
        {"name": "Location", "tagType": "AtomicTag"}
    ]
}

target_path = '[default]VOC_device'
df = []

tag_list = []
for index, row in df.iterrows():
    device_id = row['传感器ID']
    new_device = deepcopy(tag_demo)
    new_device["name"] = "ID_{}".format(device_id)
    tag_list.append(new_device)

system.tag.configure(target_path, tag_list, "m")
```

---

## CSV 文件操作
> #### 读取 CSV 文件

```python
file_path = u"C:\\Users\\Administrator\\Desktop\\path6.csv"
csv_content = system.file.readFileAsString(file_path, "UTF-8")  # 返回为字符串
lines = csv_content.split("\n")
columnNames = lines[0].split(",")
data = [line.split(",") for line in lines[1:] if line.strip()]
dataset = system.dataset.toDataSet(columnNames, data)
```

---

## 自定义排序功能
> #### 按 `label_list` 顺序排序 `root_node`

```python
label_list = [
    "1D", "1G", "2D", "2G", "3D", "4D", "5D", "6D", "7D", 
    "8D", "9D", "10D", "11D", "12D", "13D", "14D", "15D", 
    "16D", "17D", "18D", "19D", "20D", "21D", "22D", "23D", 
    "7DP", "DP"
]

sorted_root_node = sorted(
    root_node, 
    key=lambda x: label_list.index(x['label']) if x['label'] in label_list else float('inf')
)
```

---

## 数据质量检查
> #### 判断 `quality_alias` 是否有效

```python
quality_alias = curryValue.quality
if not (quality_alias.isBad() or "Bad_NotFound" in str(quality_alias)):
    status = quality_alias.isGOOD()
```

---

## PyDataset 数据操作
> #### 按列操作并计算值

```python
for index, path in enumerate(paths):
    pydataset = system.dataset.toPyDataSet(
        system.tag.queryTagHistory(
            paths=paths[path], 
            startDate=startTime, 
            endDate=endTime, 
            aggregationMode='LastValue', 
            returnSize=return_size
        )
    )
    if not py_list:  # 时间只插入一次
        py_list = [[row[0]] for row in pydataset]
    sums = [
        sum((row[i] if row[i] is not None else 0) for i in range(1, len(row))) 
        for row in pydataset
    ]
    for i, sum_value in enumerate(sums):
        py_list[i].append(sum_value)
```

---

## 实例过滤与匹配
> #### 按条件过滤实例和类型

```python
def type_name_InstancesUse():
    third_layer_path = []
    third_layer = system.tag.browse(third_layer_path)
    for third_item in third_layer:
        if str(third_item["tagType"]) == "UdtInstance" and third_item["name"].startswith('LDB'):
            return True
```

## 其他常用方法

> #### 使用 `find` 方法匹配字符串

```python
risk_paths = []
water_system_prefixes = ('LDS', 'LQS', 'YS')
for i in range(len(risk_paths)):
    if any(risk_paths[i].find(prefix) != -1 for prefix in water_system_prefixes):
        return True
```

> #### 使用 `next` 和 `enumerate` 遍历

```python
parts = []
for index, prefixe in enumerate(prefixes):
    id_part = next((part for part in parts if part.startswith('ID=')), None)
return id_part
```

> #### 获取属性菜单

```python
root_items = self.getSibling("Tree").props.items
false_icon = {
    "path": "material/check_box_outline_blank",
    "color": "#FFFFFF99",
    "style": {}
}
# 清除选项的勾选标志
for i in range(len(root_items)):
    if root_items[i]['enable']:
        root_items[i]['icon'] = false_icon
        root_items[i]['enable'] = False
    for index in range(len(root_items[i]['items'])):
        if root_items[i]['items'][index]['enable']:
            root_items[i]['items'][index]['icon'] = false_icon
            root_items[i]['items'][index]['enable'] = False
self.view.custom.key = []
```

> #### 数据清洗并写入历史记录

```python
def transform(self, value, quality, timestamp):
    file_path = u"C:\\Users\\Administrator\\Desktop\\恩华药业数据库清洗1\\path6.csv"  # 文件必须在网关所在系统中
    csv_content = system.file.readFileAsString(file_path, "UTF-8")  # 返回为字符串
    lines = csv_content.split("\n")
    columnNames = lines[0].split(",")
    data = []
    for line in lines[1:]:
        if line.strip():
            data.append(line.split(","))
    dataset = system.dataset.toDataSet(columnNames, data)
    pydataset = system.dataset.toPyDataSet(dataset)

    paths = ["QY/J18/YS/YS_LJLL_HW" for i in range(len(pydataset))]
    timeStamps = []
    values = []
    historyprovider = "History"
    tagprovider = "default"
    for row in pydataset:
        timeStamps.append(int(row[6]))
        values.append(float(row[7]))

    system.tag.storeTagHistory(
        historyprovider=historyprovider,
        tagprovider=tagprovider,
        paths=paths,
        values=values,
        timestamps=timeStamps
    )
```

> #### 转为 JSON 字符串

```python
# 转为 JSON 格式
return {'json': system.util.jsonEncode(res)}
```

> #### 遍历嵌套路径并匹配前缀

```python
risk_paths = []
water_system_prefixes = ('LDS', 'LQS', 'YS')
for i in range(len(risk_paths)):
    if any(risk_paths[i].find(prefix) != -1 for prefix in water_system_prefixes):
        return True
```

> #### 使用 `next` 查找特定元素

```python
parts = []
for index, prefixe in enumerate(prefixes):
    id_part = next((part for part in parts if part.startswith('ID=')), None)
return id_part
```

> #### 遍历文件夹

```python
folders = system.tag.browse(basePath, {"recursive": False}).results
```

---

