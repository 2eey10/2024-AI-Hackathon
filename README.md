2024 Prometheus AI Hackathon 
======================================
### Service Demo Introduction - 방구석 브리핑(BIS)


Members
-------

| Position   | Name                          |
|------------|-------------------------------|
| **Front, PM**  | Kim Jongmin [@miniwa00](https://github.com/miniwa00) |
| **Back**       | Kim Haeeun [@hyeesw](https://github.com/hyeesw)     |
| **Back, AI**       | Lee Yongryull [@2eey10](https://github.com/2eey10) |
| **AI**         | Youn Taeyang [@youn-sun](https://github.com/youn-sun) |

Index
-----

1. [Competition Overview](#Competition-Overview)
2. [Service Introduction](#Service-Introduction)
3. [Service Contents & Service Flowchart](#Service-Contents-&-Service-Flowchart)
4. [Web/App Development Stack](#Web/App-Development-Stack)



## Competition Overview


| **Organizer**                | **Prometheus**|
|---------------------------|-----------------------------|
| **Theme**                | **_시장성을 고려한 인공지능 활용 서비스 개발_**|
| **Schedule**                | **_2024.01.22 - 2024.01.28 (예선)<br>2024.02.02 - 2024.02.03 (본선)_** |
| **Outcome**                | **`우수상`**                     |


## Service Introduction


* Service Name: 
**방구석 브리핑**
* Service Topic: 
**AI 기술을 사용하여 무인점포 내 객체와 행위에 대한 자연어 캡션을 클라이언트에게 브리핑 해주는 서비스**
* Service Overview:  
**본 서비스는 ‘AI 기술을 사용하여 무인점포 내 객체와 행위에 대한 자연어 캡션을 클라이언트에게 브리핑 해주는 서비스’로써, 무인점포 내 CCTV로 촬영된 영상의 다양한 객체에 대해 Vision AI의 inference 결과를 LLM을 사용해 생성된 자연어 캡션을 클라이언트에게 전송하는 서비스입니다.**

* Service BIZ Strategy:
<img width="746" alt="스크린샷 2024-06-16 오후 8 31 59" src="https://github.com/2eey10/2024-ai-hackathon-backend/assets/133326837/a6514779-805f-4489-9e95-d226f01f7690">


    
## Service Contents & Service Flowchart

* Service Contents
  
| Part | Contents          |
|------|-------------------|
| **AI**   | YOLOv8n 모델링, GPT-4 preview LLM Inference Pipeline 생성             |
| **Back** | Brefing API 구현, DB 구축 및 조작             |
| **Front**| Brefing ReQuest User Interface 구현             |


방구석 브리핑_Backend
================================================

BIS_Backend_Overview
------------------------------------------------

*Implement a functionality using FastAPI to periodically provide clients with briefings on the results of an AI Inference Pipeline. Enable interaction between the client and the server through request and response mechanisms.*

![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![WebSocket](https://img.shields.io/badge/WebSocket-4F4F4F?style=for-the-badge&logo=websocket)
![APScheduler](https://img.shields.io/badge/APScheduler-4285F4?style=for-the-badge&logo=apscheduler)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=for-the-badge&logo=mongodb)

Environment Installation
------------------------
**Please refer to the attached "requirements.txt" for the versions of packages and libraries needed for environment installation.**

### fastapi
```
pip install fastapi
pip install "uvicorn[standard]"
```

### websocket
```
pip install websockets
```

### apscheduler
```
pip install apscheduler
```

### mongodb
```
pip install pymongo
```

Install MongoDB following the instructions on the 

[official MongoDB website](https://www.mongodb.com/try/download/community)



### .env.production
```
pip install python-dotenv
```

Create a separate file named .env.production and add the following content:

```
DATABASE_ID="your_id"
DATABASE_PASSWORD="your_pw"
DATABASE_URI=mongodb+srv://<DATABASE_ID>:<DATABASE_PASSWORD>@"your_dbname".keizkxa.mongodb.net
```

**Ensure that you keep this file secure and do not share sensitive information.**

This file contains configuration details for your production environment, including database credentials. Keep in mind the importance of securing this information and not exposing it publicly.


API List-up
------------

| Index | Method | URI | Description            |
|-------|--------|-----|------------------------|
| 1     | POST   | /login          | 로그인 API              |
| 2    | WebSocket | /ws           | Web socket (Db 변화 감지)  |
| 3    | WebSocket | /reloadBriefing | Web socket (진행 중 briefing 확인) |
| 4     | POST   | /endBriefing    | Briefing 종료          |
| 5     | POST    | /getAllRequest  | 모든 request 조회      |
| 6     | POST    | /getBriefing    | 모든 briefing 조회     |
| 7     | POST    | /aiSummary      | 브리핑 요약            |
| 8     | POST    | /getCustom      | Custom 획득            |
| 9     | POST    | /updateCustom   | Custom 갱신            |


-----------------------

API Specification
-----------------


* ### login

    * description
        
        * Utilize **check_user** in **models.py** to verify new and existing users.

            * In the case of an existing user, return `{"message": "signup", "user_id": <user_id>}`.
            * For a new user, return `{"message": "login", "user_id": <user_id>}`.

| **Endpoint** | **Method** |
|--------------|------------|
| `POST /login`| POST       |


|  |**Request**|
|--------------|------------------|
| **Header**   ||
| *"No special headers required"* ||
| **Body**     ||
| *Parameter*    | *Type* ||
| username    | String || 
  
|**Response** |
|--------------|
| **1) Success Response** |
| **Status Code:** 200 OK |
| **Body:** |
| ```json { "message" : "signup", "user_id" : "user_id_value"} ```|
| or |
| ```json { "message" : "login", "user_id" : "user_id_value"} ```|
| **2) Error Response** |
| **Status Code:** 500 Internal Server Error |
| **Body:** |
| ```json { "detail": "Login Error: error_message"} ```|




* ### websocket_endpoint

    * Description:
        * Handle WebSocket connections for continuous updates.
        * Establish WebSocket connection.
        * Store new request in the database and register with the scheduler.
        * Continuously send updates to the frontend through the WebSocket.
  

| **Endpoint** | **Method** |
|--------------|------------|
| `websocket/ws`| WebSocket |


**Parameters**

| **Name**     | **Type**  | **Description**                   |
|--------------|-----------|-----------------------------------|
| websocket    | WebSocket | The WebSocket connection object.  |
| username     | String    | User's username.                  |
| interval     | String    | Requested interval for updates.   |
| endtime      | String    | Requested end time for updates.   |




* ### websocket_reloadBriefing

    * Description
        * Display the ongoing briefing.
        * Establish WebSocket connection.
        * Send reloaded briefing data to the frontend.
        * Get the latest request ID and continuously watch the database for updates.
    
| **Endpoint** | **Method** |
|--------------|------------|
| `websocket/reloadBriefing`| WebSocket |


    
 **Parameters**
| **Name**     | **Type**  | **Description**                 |
|--------------|-----------|---------------------------------|
| websocket    | WebSocket | The WebSocket connection object. |
| username     | String    | User's username.                 |

* ### endBriefing

    * description
        * Terminate the ongoing briefing.
        * The function **endBriefing** uses the **endBriefing_worker** from **utils.py** and **end_scheduler** from **scheduler.py**.

            * Upon success, it prints a success message and returns a success response with the message `"Briefing successfully terminated"`.
            * In case of failure (e.g., an `exception`), it catches the `"HTTPException"`, logs the error message, and returns an error response with the corresponding status code and details.

| **Endpoint** | **Method** |
|--------------|------------|
| `POST /endBriefing`| POST       |


|  |**Request**|
|--------------|------------------|
| **Header**   ||
| *"No special headers required"* ||
| **Body**     ||
| *Parameter*    | *Type* ||
| username    | String || 
  
|**Response** |
|--------------|
| **1) Success Response** |
| **Status Code:** 200 OK |
| **Body:** |
| ```"message": "브리핑 성공적으로 종료" ```|
| **2)Error Response** |
| **Status Code:** 500 Internal Server Error |
| **Body:** |
| ```Internal Server Error ```|

* ### getAllRequest

    * description
        * Implement a functionality to retrieve all requests for a specific username from the `"request" Collection` within the database.
        * **getAllRequest** utilizes the **get_all_request** function from **utils.py** to retrieve and return the list of requests for a specific username from the **"request" Collection** in the database.

* ### getBriefing

    * description
        * Retrieve briefing information based on the provided `request_id`.
        * Utilize the `get_briefing` function from `utils.py` to perform a functionality that retrieves all briefings for a specific request_id from the `"briefing" Collection` in the database.

* ### aiSummary

    * description
        * Generates an AI summary captions based on the provided `request_id`.
        * This API utilizes two methods: `get_briefing` in `utils.py` for querying all data from the database and `chat_summary` in `utils.py` for generating AI summaries based on the returned briefing data list.

        * To provide additional details on `chat_summary`, it involves using the `OpenAI API key` to generate messages based on a specified format prompt. This process utilizes the `GPT-4-preview model` and performs `prompt engineering` on the briefing data list retrieved earlier, adjusting hyperparameters such as `"tone, writing style, temperature, top_p,"` and others.

* ### getCustom

    * description
        * this API retrieves the `"custom"` data for a specific username. The obtained `custom_list` is then used to generate an HTTP response in JSON format, encoded in UTF-8.
        * The `get_custom` in `models.py` process involves searching for the specified username within the "user" collection in the database and referring to it as a custom list.


* ### updateCustom

    * description
        * Utilizing the `update_custom` function in `models.py`, this API updates the `"custom"` field in the "user" table.
            * If the custom_list is successfully updated, it returns the message `"Success to update Custom."`
            * In case of exceptions or errors, it raises an `HTTPException` with a status code of 500 and the detail `"Fail to update Custom."`




 방구석 브리핑_Service FlowChart
  ===========================
  
![Untitled (2)](https://github.com/2eey10/2024-ai-hackathon-backend/assets/133326837/615010d1-8014-44dd-bbac-561ad3526361)

![Untitled (3)](https://github.com/2eey10/2024-ai-hackathon-backend/assets/133326837/1585f91f-e827-41e8-b2a5-059577d43123)
![Untitled (4)](https://github.com/2eey10/2024-ai-hackathon-backend/assets/133326837/e4990034-9194-4609-a499-49f8c0346d94)

## Web/App Development Stack

+ **Dev Environment**:
  + **Network**: HTTP, WebSocket
   + **Programming Language**:
     + **Python3.9**
       + HTTP, WebSocket
       + FastAPI & extended module
       + ultralytics
       + APscheduler
     + Flutter (Dart) - **Mobile APP**
   + **DB**: MongoDB
   + **OS**: Windows, Mac
   + **Version Control**: Git, GitHub
   + **Web server**: FastAPI
    
+ **Main Functionality**
  + **AI Inference Pipeline (Vision, Language)**
    + Step 1. 카메라가 영상을 촬영하면 영상에 대해 YOLO 가 inference 를 한다.
    + Step 2. YOLO 모델의 inference 결과를 parsing 하여 GPT-4 가 캡션을 생성한다.
  + **Backend (Fast API, MongoDB)**
    + Step 1. client 로부터 요청이 들어오면 server 와 WebSocket 연결을 한다. 연결됨과 동시에 scheduler 에 Inference Pipeline 을 수행하는 작업(브리핑 작업)을 등록한다.
    + Step 2. 작업이 수행될 때마다 DB 에 캡션을 저장하고, DB 의 변화를 감지해 WebSocket 링크를 통해서 새롭게 저장된 캡션(브리핑 작업의 결과물)을 client 에게 전송한다.
  + **Frontend** (Flutter)
    + Step 1. Interval, End Time 값을 WebSocket url 을 통해 서버로 보낸다. 서버가 요청을 수락하면 WebSocket 링크를 통해 캡션을 화면에 출력한다.
    + Step 2. 브리핑을 종료하면 WebSocket 링크를 끊고 홈 화면으로 돌아간다. 브리핑을 종료하지 않고 앱을 종료하면 진행 중인 브리핑 작업의 캡션을 다시 받아 볼 수 있다.
    
+ **Additional Functionality**
  + **과거 브리핑 조회**: client 가 과거에 요청한 모든 브리핑을 확인할 수 있는 탭이다.
  + **AI BIZ 솔루션 요약**: 과거 브리핑 내역을 바탕으로 방문 고객 유형 분석 등과 같은 요약 데이터를 제공해준다.
  + **GPT-4 커스텀**: Inference Pipeline 의 결과를 바탕으로 캡션을 생성하는 과정에서 어떤 형식으로 캡션을 생성할지를 조정하는 기능과 AI 요약 시 어떤 요소에 초점을 맞추어 요약 텍스트를 생성할 지 조정하는 기능을 포함한다.
 
