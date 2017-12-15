---
layout: post
title: Getting Facebook Page Fans data through Graph API
---

สิ่งสำคัญก่อนที่เราจะสามารถวิเคราะห์ข้อมูลได้นั้น เราจำเป็นจะต้องมีแหล่งข้อมูลก่อน ซึ่งวิธีการเก็บข้อมูลนั้นก็มีมากมายหลากหลายวิธี ซึ่งในวันนี้เราจะมาพูดถึงการดึงข้อมูลจากฐานข้อมูลด้วย API หรือ Application Program Interface 

> APIs are commonly used to retrieve data from remote websites. Sites like Reddit, Twitter, and Facebook all offer certain data through their APIs. To use an API, you make a request to a remote web server, and retrieve the data you need.

### Why API?

เหตุผลที่เราเลือกใช้ API แทนที่จะ download static data ก็เพราะว่า API นั้นมีประโยชน์ในหลายๆกรณี เช่น
- กรณีที่ข้อมูลเปลี่ยนแปลงอย่างรวดเร็วและตลอดเวลา อย่างเช่นข้อมูลของราคาหุ้น คงไม่มีใครมานั่งดาว์นโหลดข้อมูลราคาหุ้นทุกๆนาทีเป็นแน่ 
- กรณีที่เราต้องการเพียงเศษเสี้ยวของฐานข้อมูลขนาดใหญ่ เช่น เราต้องการข้อมูล fanpage บน facebook แค่เฉพาะระยะเวลาหนึ่ง ไม่ใช่ทั้งหมด หากเราต้องทำการดาว์นโหลดทั้งฐานข้อมูลของ facebook แล้วค่อยมา filter ทีหลัง ก็ดูจะไม่ใช่วิธีที่ดีนัก

### Facebook Graph API

สำหรับตัวอย่างในวันนี้ ก็จะเป็นการใช้ Graph API ของทาง Facebook ซึ่งจะอนุญาตให้เราดึงข้อมูล ผู้ใช้ซึ่งกดไลค์เพจต่างๆ โดยแบ่งตามประเทศที่ผู้ใช้นั้นอาศัยอยู่

Facebook Graph API ที่เปิดให้บุคคลทั่วไปได้เรียกใช้นั้น ค่อนข้างจำกัด สำหรับข้อมูลเพิ่มเติม สามารถอ่านต่อได้ที่  https://developers.facebook.com/docs/graph-api/reference/v2.10/insights ซึ่งเวอร์ชั่นที่เราจะใช้กันในวันนี้ก็คือ `v2.10`

### Getting Started

```python
import json
import urllib.request
import numpy as np
import pandas as pd
import time
```

```python
# get access token from graph api @ https://developers.facebook.com/tools/explorer/
page_access_token = "EAACEdEose0cBALMl1uTfThIb9NBwzw8f5Ac6c7cAJo2YdmoluTnAVh5vzbXehRidZA0NMhZAsxqBSRISkf1ZAVgThjbZBTPtFAu8EVkGdHs2qJObX6IKmMNw8hXJTWLPlZCwtfGv43GZANKBJFglofcGwl2n24aCrcBkLod8GL1cbrLMeu0Vk73gqPtwZCm7SwZD"
```

```python
page_id = 'StrangerThingsTV' # Get page fans by country
```


```python
def getFacebookPageData(page_id, access_token, start_date, end_date):
    # URL query
    base = "https://graph.facebook.com/v2.10"
    node = "/" + page_id
    parameter1 = "/page_fans_country" 
    parameter2 = "/lifetime?&since="+start_date+"&until="+end_date+"&"
    access_param = "access_token=%s" % page_access_token
    url = base + node + "/insights" + parameter1 + parameter2 + access_param
    # retrieve data
    req = urllib.request.Request(url)
    response = urllib.request.urlopen(req)
    data = json.loads(response.read().decode('utf-8'))
    return data
```



```python
# Specify input (max 3 months)
start_date = "2016-06-01"
end_date = "2016-09-01"
mydata = getFacebookPageData(page_id, access_token, start_date, end_date)
```



```python
# test run
dt = pd.DataFrame.from_dict(mydata['data'][0]['values'])
dt2 = pd.read_json(dt['value'].to_json(), orient="index")
dt3 = dt2.join(dt['end_time'])
dt3.tail(3)
```



```python
df = pd.DataFrame()
## Write a for loop to go through from 2016 to 2017
for i in range(2016,2018):
    start_date_1 = str(i)+"-01-01"
    end_date_1 = str(i)+"-04-01"
    start_date_2 = str(i)+"-03-31"
    end_date_2 = str(i)+"-07-01"
    start_date_3 = str(i)+"-06-30"
    end_date_3 = str(i)+"-10-01"
    start_date_4 = str(i)+"-09-30"
    end_date_4 = str(i)+"-12-31"
    # Jan-March
    mydata1 = getFacebookPageData(page_id, access_token,start_date_1, end_date_1)
    dt = pd.DataFrame.from_dict(mydata1['data'][0]['values'])
    dt2 = pd.read_json(dt['value'].to_json(), orient="index")
    dt3 = dt2.join(dt['end_time'])
    df = df.append(dt3)
    print(i,"Q1")
    time.sleep(5)
    # April-June
    mydata2 = getFacebookPageData(page_id, access_token,start_date_2, end_date_2)
    dt = pd.DataFrame.from_dict(mydata2['data'][0]['values'])
    dt2 = pd.read_json(dt['value'].to_json(), orient="index")
    dt3 = dt2.join(dt['end_time'])
    df = df.append(dt3)
    time.sleep(5)
    print(i,"Q2")
    # July-Sep
    mydata3 = getFacebookPageData(page_id, access_token,start_date_3, end_date_3)
    dt = pd.DataFrame.from_dict(mydata3['data'][0]['values'])
    dt2 = pd.read_json(dt['value'].to_json(), orient="index")
    dt3 = dt2.join(dt['end_time'])
    df = df.append(dt3)
    time.sleep(5)
    print(i,"Q3")
    # Oct-Dec
    mydata4 = getFacebookPageData(page_id, access_token,start_date_4, end_date_4)
    dt = pd.DataFrame.from_dict(mydata4['data'][0]['values']
    dt2 = pd.read_json(dt['value'].to_json(), orient="index")
    dt3 = dt2.join(dt['end_time'])
    df = df.append(dt3)
    time.sleep(5)
    print(i,"Q4")
```

```python
# clean date/time - only take the date
df['end_day'] = df['end_time'].str[:10] 
# fill NA with 0
df = df.fillna(0)
```



```python
# melt columns to rows
df2 = pd.melt(df, id_vars = ["end_time", "end_day"],
                  var_name = "Country", value_name = "Likes")
```

```python
df2.tail()

```



```python
# Total check
sum(df2['Likes'][df2['end_day'] == '2017-11-30'])
```



```python
# export to csv
df2.to_csv("Strangerthings_FB.csv")
```

