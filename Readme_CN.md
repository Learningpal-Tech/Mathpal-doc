# 简介
欢迎使用MathPal的数学方程式手写体识别工具，该引擎是采用先进的神经网络算法所搭建的手写体图像识别技术，在这里我们将会提供详尽的API测试与使用的指导。
#### API测试包地址 ([GitHub](https://github.com/TomNong/learningpal_api) )
```
git clone https://github.com/TomNong/learningpal_api.git
cd learningpal_api
```
所需环境
```
pip install requests 
pip install json
```
## 单张图片API请求
![](https://github.com/TomNong/learningpal_api/blob/master/single_equation.png?raw=true)

用http提交POST请求：
```
python image_sender.py 
```
详细代码包括两个POST请求，第一个POST请求服务器会返回task_id以确认任务接受成功，第二个请求返回最终结果：
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
终端控制台中返回的结果（包括两个JSON）：
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
### 请求参数与结果
###### 请求一

| 参数 | 详细描述 |
|------------|------------|
| content_type | application/json |
| url | 字符串 "http://api.learningpal.com/math/upload" |
| files | 图片文件 |

 ###### 结果一

| 内容 | 详细描述 |
|------------|------------|
| cur_num | 图片的序号，单张图片请求永远是0 |
| task_ID | 字符串，代表当前任务的ID |
| filename | 由系统创建的文件名 |

###### 请求二

| 参数 | 详细描述 |
|------------|------------|
| task_ID | 由结果一中所得到的当前任务ID |
| password | 空字符串 "" |
| content_type | application/json |
| url | 字符串 "http://api.learningpal.com/math/result" |

###### 结果二

| 内容 | 详细描述 |
| --------| ----------- |
| confidence | 信心值，代表机器对本次识别结果是否准确的信心 |
| isResult | 任务成功完成时，该结果返回"true"(字符串) |
| result | 本次识别的最终结果，LaTeX格式(字符串) |

### 信心值 
返回结果中的confidence信心值，代表该识别引擎对于本次识别的结果的准确性的判断。信心值越接近1.0就代表本次识别正确的可能性越高。
在程序设计时，建议根据返回的信心值设定输出壁垒（建议基准线在10%左右）

## 更多的识别样例
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

## 通过Swift语言发送请求样例（Swift3.0+）
```swift
import UIKit
import Foundation

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        let img = #imageLiteral(resourceName: "test_img")
        let url_str = "http://api.learningpal.com/math/upload"
        let url_str_res = "http://api.learningpal.com/math/result"

        data_request_multipart(url_str, img, completion: { respond in
            let taskID = respond
            self.Get_result_request(taskID as! String, url_str_res, fulfill: { (respond) in
                print("res: ", respond)
            }, reject:{ error in
                print("error: ", error)
            })
        })
    }
    
    func data_request_multipart(_ url:String, _ data:UIImage, completion: @escaping (_ result: Any) -> Void) {
        var task_ID = String()
        let post_url = NSURL(string: url)
        var request = URLRequest(url: post_url! as URL)
        let boundary = "Boundary-\(NSUUID().uuidString)"
        let image_data = UIImageJPEGRepresentation(data, 0.8)
        let fname = "name.jpg"
        let mimetype = "image/png"
        let body = NSMutableData()
        body.append("--\(boundary)\r\n".data(using: String.Encoding.utf8)!)
        body.append("Content-Disposition:form-data;name=\"photo\"\r\n\r\n".data(using: String.Encoding.utf8)!)
        body.append("Incoming\r\n".data(using: String.Encoding.utf8)!)
        body.append("--\(boundary)\r\n".data(using: String.Encoding.utf8)!)
        body.append("Content-Disposition:form-data; name=\"file\";filename=\"\(fname)\"\r\n".data(using: String.Encoding.utf8)!)
        body.append("Content-Type: \(mimetype)\r\n\r\n".data(using:
            String.Encoding.utf8)!)
        body.append(image_data!)
        body.append("\r\n".data(using: String.Encoding.utf8)!)
        body.append("--\(boundary)--\r\n".data(using:
            String.Encoding.utf8)!)
        
        request.httpMethod = "POST"
        request.httpBody = body as Data
        request.setValue("multipart/form-data; boundary=\(boundary)", forHTTPHeaderField: "Content-Type")
        request.cachePolicy = .reloadIgnoringLocalCacheData
        request.httpShouldHandleCookies = false
        request.timeoutInterval = 60
        
        let session = URLSession.shared
        let task = session.dataTask(with: request as URLRequest) {
            (
            data, response, error) in
            guard let _:Data = data, let _:URLResponse = response , error
                == nil else {
                    print("error", error)
                    return
            }
            if let res_json = try! JSONSerialization.jsonObject(with: data!, options: []) as? [String: Any] {
                task_ID = res_json["task_ID"] as! String
                completion(task_ID)
            }
        }
        task.resume()
    }
    
    func Get_result_request(_ taskID: String, _ url: String, maxRetries: Int = 10, attempt: Int=0, fulfill: @escaping (_ result:Any)->Void, reject: @escaping (Error)->Void){
        guard attempt < maxRetries else {
            reject(RetryError.tooManyRetries)
            return
        }
        var final_res = [String:Any]()
        var isResult = "false"
        let json: [String: Any] = ["task_ID": taskID, "password": "nopassword"]
        let jsonData = try? JSONSerialization.data(withJSONObject: json)
        let post_url = URL(string: url)!
        var request = URLRequest(url: post_url)
        request.httpMethod = "POST"
        request.httpBody = jsonData
        request.addValue("application/json", forHTTPHeaderField: "Content-Type")
        request.addValue("application/json", forHTTPHeaderField: "Accept")
        if taskID.count == 0 {return}
        let task = URLSession.shared.dataTask(with: request) { data, response, error in
            guard let data = data, error == nil else {
                print(error?.localizedDescription ?? "No data")
                return
            }
            if let result_json = try! JSONSerialization.jsonObject(with: data, options: []) as? [String: Any] {
                final_res = result_json
                isResult = result_json["isResult"] as! String
                if isResult == "true"{
                    fulfill(final_res)
                } else {
                    self.Get_result_request(taskID, url, maxRetries: maxRetries, attempt: attempt+1, fulfill: fulfill, reject: reject)
                }
            }
        }
        
        task.resume()
    }
    
    enum RetryError: Error {
        case tooManyRetries
        case taskFailed
    }
}

```
