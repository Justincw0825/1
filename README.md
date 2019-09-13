# 与微信集成的天气查询机器人
## 需要安装以下python包
+ requests，用于爬取天气的数据
+ rasa, spacy 用于nlp, ner
+ wxpy 用于在微信上的交互对话
## 如何初始化机器人并且实现消息处理
[wxpy: 用 Python 玩微信](https://wxpy.readthedocs.io/zh/latest/)
## Core Method	
+ 天气数据查询API  
[Open Weather Map](https://rapidapi.com/community/api/open-weather-map/endpoints)  
1.查询实时天气
```python
def current_weather(city):
		url = "https://community-open-weather-map.p.rapidapi.com/weather"

		querystring = {"callback":"test","id":"2172797","units":"\"metric\" or \"imperial\"","mode":"xml, html","q":"London,uk"}
		querystring['q'] = city

		headers = {
				'x-rapidapi-host': "community-open-weather-map.p.rapidapi.com",
				'x-rapidapi-key': "84db939702msh3255af9f5bf635dp177c06jsn84d6de59e9ba"
				}

		response = requests.request(",,,GET", url, headers=headers, params=querystring)
		response1=response.text
		s=response1[4:]
		b=eval(s)

		return [b['name'],b['weather'][0]['description'],b['wind']['speed']]
```
2.查询地区温度
```python
def temperature_search(city):
		url = "https://community-open-weather-map.p.rapidapi.com/weather"

		querystring = {"callback":"test","id":"2172797","units":"\"metric\" or \"imperial\"","mode":"xml, html","q":"London,uk"}
		querystring['q'] = city

		headers = {
				'x-rapidapi-host': "community-open-weather-map.p.rapidapi.com",
				'x-rapidapi-key': "84db939702msh3255af9f5bf635dp177c06jsn84d6de59e9ba"
				}

		response = requests.request("GET", url, headers=headers, params=querystring)
		response1=response.text
		s=response1[4:]
		b=eval(s)

		return [b['name'],"%.2f"%(b['main']['temp_max']-273.15),"%.2f"%(b['main']['temp_min']-273.15),"%.2f"%(b['main']['temp']-273.15)]
```
3.查询天气预报
```python
def weather_forecast(city):
    url = "https://community-open-weather-map.p.rapidapi.com/forecast"

    querystring = {"q":"london,uk"}
    querystring['q'] = city

    headers = {
        'x-rapidapi-host': "community-open-weather-map.p.rapidapi.com",
        'x-rapidapi-key': "84db939702msh3255af9f5bf635dp177c06jsn84d6de59e9ba"
        }

    response = requests.request("GET", url, headers=headers, params=querystring)
    response1=response.text
    b=eval(response1)
		
    for i in b['list'][4:10]:
        str="The weather in {} at {} is mainly {}".format(b['city']['name'],i['dt_txt'],i['weather'][0]['description'])
        print("BOT:{}".format(str))
```
```python
def forecast(city):
    m = weather_forecast(city)
		
    for i in m['list'][4:10]:
        str="The weather in {} at {} is mainly {}".format(m['city']['name'],i['dt_txt'],i['weather'][0]['description'])
        print("BOT:{}".format(str))
```
