# 与微信集成的天气查询会话机器人
## 需要安装以下python包
+ `requests`，用于爬取天气相关的数据
```
pip install requests
```
+ `rasa`, `spacy` 用于nlp, ner  
[rasa安装](https://rasa.com/docs/rasa/user-guide/installation/)  
[spacy安装](https://spacy.io/usage)  
**要注意版本问题否则不能兼容**
+ `wxpy` 用于在微信上的交互对话
```
pip install -U wxpy
```
## 如何初始化机器人并且实现消息处理
[wxpy: 用 Python 玩微信](https://wxpy.readthedocs.io/zh/latest/)
## Core Method	
**1 天气数据查询API**  
[Open Weather Map](https://rapidapi.com/community/api/open-weather-map/endpoints)  
+ 查询实时天气
```python
#define the current_weather to get the data about weather from certain website
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
+ 查询地区温度
```python
#define the temperature_search to get the data about weather from certain website
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
+ 查询天气预报
```python
#define the weather_forecast to get the data about weather from certain website
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
**2 使用rasa训练数据**
```python
# Import necessary modules
from rasa_nlu.training_data import load_data
from rasa_nlu.config import RasaNLUModelConfig
from rasa_nlu.model import Trainer
from rasa_nlu import config

# Create a trainer that uses this config
trainer = Trainer(config.load("/home/jcw/rasa_nlu/sample_configs/config_spacy.yml"))

# Load the training data
training_data = load_data('/home/jcw/rasa_nlu/data/examples/rasa/weather-rasa.json')

# Create an interpreter by training the model
interpreter = trainer.train(training_data)
```
**3 状态机**
```python
#Define the states
INIT=0
AUTHED=1
ASK_ABOUT_WEATHER=2
# Define the policy rules
policy_rules = {
    (INIT,"none"):[(INIT, "you'll have to log in first, what's your phone number?", AUTHED)],
    (AUTHED,"none"):[(AUTHED,"what do you mean?",None),(AUTHED,"Sorry, I can't help you.",None),(AUTHED,"You can ask me something about weather.",None)],
    (ASK_ABOUT_WEATHER,"none"):[(ASK_ABOUT_WEATHER,"what do you mean?",None),(ASK_ABOUT_WEATHER,"Sorry, I can't help you",None),(ASK_ABOUT_WEATHER,"You can ask me something about weather.",None)],
    (INIT,"ask_explanation"):[(INIT, "I'm a clever weather robot and I can provide you with a lot of data about weather, also, I can tell you the temperature of any city.", None)],
    (AUTHED,"ask_explanation"):[(AUTHED, "I'm a clever weather bot and I can provide you with a lot of data about weather, also, I can tell you the temperature of any city.", None)],
    (ASK_ABOUT_WEATHER,"ask_explanation"):[(ASK_ABOUT_WEATHER, "I'm a clever weather bot and I can provide you with a lot of data about weather, also, I can tell you the temperature of any city.", None)],
    (INIT,"ask_usage"):[(INIT, "1.I can search weather for you. Try to ask me in this way:\nI want to know the current weather in Wuhan\nor\nTell me the weather in wuhan tomorrow.\n2.I can tell you the temperature of any city,but you have to tell me where do you want to search.", None)],
    (AUTHED,"ask_usage"):[(AUTHED,  "1.I can search weather for you. Try to ask me in this way:\nI want to know the current weather in Wuhan\nor\nTell me the weather in wuhan tomorrow.\n2.I can tell you the temperature of any city,but you have to tell me where do you want to search.", None)],
    (ASK_ABOUT_WEATHER,"ask_usage"):[(ASK_ABOUT_WEATHER, "1.I can search weather for you. Try to ask me in this way:\nI want to know the current weather in Wuhan\nor\nTell me the weather in wuhan tomorrow.\n2.I can tell you the temperature of any city,but you have to tell me where do you want to search.", None)],
    (INIT, "number"):[(AUTHED, "perfect, welcome back!", None)],
    (AUTHED,"number"):[(AUTHED,"Sorry, I can't help you.",None),(AUTHED,"You can ask me something about weather.",None)],
    (ASK_ABOUT_WEATHER,"number"):[(ASK_ABOUT_WEATHER,"Sorry, I can't help you.",None),(AUTHED,"You can ask me something about weather.",None)],
    (INIT, "number"):[(AUTHED, "perfect, welcome back!", None)],
    (AUTHED,"number"):[(AUTHED,"Sorry, I can't help you.",None),(AUTHED,"You can ask me something about weather.",None)],
    (ASK_ABOUT_WEATHER,"number"):[(ASK_ABOUT_WEATHER,"Sorry, I can't help you.",None),(AUTHED,"You can ask me something about weather.",None)],
    (INIT, "service_require"):[(INIT, "I can help you, but you have to log in first, so what's your phone number?", None)],
    (AUTHED,"service_require"):[(AUTHED, "Well, please tell me exactly what do you want to search?", None)],
    (ASK_ABOUT_WEATHER,"service_require"):[(ASK_ABOUT_WEATHER,"Well, please tell me exactly what do you want to search?",None)],    
    (INIT, "current_weather"):[(INIT, "you'll have to log in first, what's your phone number?", AUTHED)],
    (INIT, "weather_forecast"):[(INIT, "you'll have to log in first, what's your phone number?", AUTHED)],
    (INIT, "temperature_search"):[(INIT, "you'll have to log in first, what's your phone number?", AUTHED)],
    (AUTHED,"current_weather"):[(ASK_ABOUT_WEATHER, "OK, The current weather in {} is mainly {}, with the force-{} wind.", None)],
    (ASK_ABOUT_WEATHER,"current_weather"):[(ASK_ABOUT_WEATHER,"OK, The current weather in {} is mainly {}, with the force-{} wind.",None)],    
    (AUTHED,"current_weather"):[(ASK_ABOUT_WEATHER, "OK, The current weather in {} is mainly {}, with the force-{} wind.", None)],
    (ASK_ABOUT_WEATHER,"current_weather"):[(ASK_ABOUT_WEATHER,"OK, The current weather in {} is mainly {}, with the force-{} wind.",None)], 
    (AUTHED,"weather_forecast"):[(ASK_ABOUT_WEATHER, "n", None)],
    (ASK_ABOUT_WEATHER,"weather_forecast"):[(ASK_ABOUT_WEATHER,"n",None)], 
    (AUTHED,"temperature_search"):[(ASK_ABOUT_WEATHER, "In {}, the max temperature is {}℃, the min temperature is {}℃ and the average temperature is {}℃.", None)],
    (ASK_ABOUT_WEATHER,"temperature_search"):[(ASK_ABOUT_WEATHER,"In {}, the max temperature is {}℃, the min temperature is {}℃ and the average temperature is {}℃.", None)],
    (INIT, "goodbye"):[(INIT,"goodbye!",None),(INIT,"See you next time!",None)],
    (AUTHED,"goodbye"):[(INIT,"goodbye!",None),(INIT,"See you next time!",None)],
    (ASK_ABOUT_WEATHER,"goodbye"):[(INIT,"goodbye!",None),(INIT,"See you next time!",None)],
    (INIT, "greet"):[(INIT,"Hello!",None),(INIT,"I,m glad to see you.",None),(INIT,"Hi!",None),(INIT,"Welcome here!",None)],
    (AUTHED,"greet"):[(AUTHED,"Hello!",None),(AUTHED,"I,m glad to see you.",None),(AUTHED,"Hi!",None),(AUTHED,"Welcome here!",None)],
    (ASK_ABOUT_WEATHER,"greet"):[(ASK_ABOUT_WEATHER,"Hello!",None),(ASK_ABOUT_WEATHER,"I,m glad to see you.",None),(ASK_ABOUT_WEATHER,"Hi!",None),(ASK_ABOUT_WEATHER,"Welcome here!",None)],
    (INIT, "affirm"):[(INIT,"Thanks for your affirmation!",None),(INIT,"I,m glad to hear that!",None),(INIT,"Yeah!",None)],
    (AUTHED,"affirm"):[(AUTHED,"Thanks for your affirmation!",None),(AUTHED,"I,m glad to hear that!",None),(AUTHED,"Yeah!",None)],
    (ASK_ABOUT_WEATHER,"affirm"):[(ASK_ABOUT_WEATHER,"Thanks for your affirmation!",None),(ASK_ABOUT_WEATHER,"I,m glad to hear that!",None),(ASK_ABOUT_WEATHER,"Yeah!",None)]
}
```
**4 意图识别**
```python
#recognise the intent
def interpret(message):
    msg = message.lower()
    if any([d in msg for d in string.digits]):
        return 'number'
    else:
        intent = interpreter.parse(msg)["intent"]["name"]
        if intent is '':
            return 'none'
        else:
            return intent
```
**5 生成回答**
```python
state = INIT
pending = None
import random
# Define send_message()
def send_message(state, pending, message,answer):
    print("USER : {}".format(message))
    response = chitchat_response(message)
    if response is not None:
        answer=response
        print(answer)
        return state, None, answer
    
    # Calculate the new_state, response, and pending_state
    new_state, response, pending_state = random.choice(policy_rules[(state, interpret(message))])
    if interpret(message) == 'current_weather':
        city=interpreter.parse(message)['entities'][0]['value']
        (a,b,c)=(current_weather(city)[0],current_weather(city)[1],current_weather(city)[2])
        answer=response.format(a,b,c)
    elif interpret(message) == 'weather_forecast':
        city=interpreter.parse(message)['entities'][0]['value']
        answer=weather_forecast(city)
    elif interpret(message) == 'temperature_search':
        city=interpreter.parse(message)['entities'][0]['value']
        (a,b,c,d)=(temperature_search(city)[0],temperature_search(city)[1],temperature_search(city)[2],temperature_search(city)[3])
        answer=response.format(a,b,c,d)
    else:
        answer=response
    
    if pending is not None and new_state == pending[0]:
        new_state, response, pending_state = random.choice(policy_rules[pending])
        answer=response    
        print(answer) 
    if pending_state is not None:
        pending = (pending_state, interpret(message))
    print(answer)
    return new_state, pending, answer
```
**6 运行wechat bot**  
-用一个帐号登录wechat bot作为机器人  
-用编译器打开文档Weather_chatbot.ipynb  
-给一名用户授权  *即将###改成该用户的昵称*  
-进行多轮对话  
