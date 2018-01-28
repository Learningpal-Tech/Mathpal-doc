# Intro
Welcome to Mathpal handwritten math equation recognition API. The recognition engine integrates the most advanced neural network algorithm to achieve variable type of handwritten with high accurate.  
#### To get started with testing (repo location: [learningpal api](https://github.com/TomNong/learningpal_api) )
```
git clone https://github.com/TomNong/learningpal_api.git
cd learningpal_api
```
## Call API with single image 
![](https://github.com/TomNong/learningpal_api/blob/master/single_equation.png?raw=true)

Sending the image with http post request as following:
```
python image_sender.py 
```
Detail code (two post requests needed to finish the whole task):
```python
import requests
import json, time

IMG_NAME = "single_equation.png"
url = 'http://api.learningpal.com/math/upload'
files = {'file': open('./' + IMG_NAME, 'rb')}
headers = {'content-type': 'application/json'}
try:
  r = requests.post(url, files=files)
  print r.text
  response = json.loads(r.text)
  task_ID = response['task_ID']
  payload = {'task_ID' : task_ID, 'password' : ""}
  url2 = 'http://api.learningpal.com/math/result'
  while True:
    response = requests.post(url2, data=json.dumps(payload), headers=headers)
    if response.text and "processing" not in response.text:
    	break
except Exception as err :
	print err
print response.text
```
Results on console with two JSONs:
```json
{
  "cur_num": 0, 
  "task_ID": "1517108361.99single_equation__2018-01-28_02:59:21.939329__172.31.31.93.png", 
  "filename": "2018-01-28_02:59:21.991933__single_equation__2018-01-28_02:59:21.939329__172.31.31.93.png"
}
{
  "confidence": 0.9945661408039893, 
  "isResult": "true", 
  "result": "( 3 - x ) ( x - 2 ) = 0 "
}

```
### Parameters and results of two post requests
###### Post#1

| parameter | Description |
|------------|------------|
| content_type | application/json |
| url | string "http://api.learningpal.com/math/upload" |
| files | image file |

 ###### Result#1

| element | Description |
|------------|------------|
| cur_num | index of the image. Always 0 for single post |
| task_ID | string of current task id |
| filename | system created filename |

###### Post#2

| parameter | Description |
|------------|------------|
| task_ID | task ID returned from last post requst |
| password | string "" |
| content_type | application/json |
| url | string "http://api.learningpal.com/math/result" |

###### Result#2

| element | Description |
| --------| ----------- |
| confidence | Confidence of the judgement the machine made on recognizing the image |
| isResult | (string) successful call will return "true" |
| result | The recognition result on post image |

### Confidence 
Confidence value represents the possibility of machine get answer right. The more it close to 1.0, the greater possibility that the recognition result is correct. Create a return barrier for your application with confidence value, 10% is suggested.


## More Examples
![](https://github.com/RobinXL/Mathpal-doc/blob/master/346.png?raw=true)
```JSON
{
    "confidence": 0.49241428279855526, 
    "isResult": "true", 
    "result": "\\frac { a ^ { 2 } + b ^ { 2 } } { a b } "
}
```

![](https://github.com/RobinXL/Mathpal-doc/blob/master/latex2.png?raw=true)
```JSON
{
    "confidence": 0.48371252203373305, 
    "isResult": "true", 
    "result": "{ \\frac { b ^ { 2 } + c ^ { 2 } - a ^ { 2 } } } { 2 b c } = \\frac { 4 } { 5 } "
}
```

![](https://github.com/RobinXL/Mathpal-doc/blob/master/latex20.png?raw=true)
```JSON
{
    "confidence": 0.9923326232870631, 
    "isResult": "true", 
    "result": "\\sin A = \\frac { 3 } { 5 } "
}
```


