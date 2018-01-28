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

## Call from Swift3.0+ sample
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
