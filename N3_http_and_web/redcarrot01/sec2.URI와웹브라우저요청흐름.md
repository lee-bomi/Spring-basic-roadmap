# 1. URI(Uniform Resource Identifier)

### URI 

로케이터(locator), 이름(name) 또는 둘 다 추가로 분류될 수 있다.

![image](https://user-images.githubusercontent.com/38436013/126944562-7082a43c-299b-4b01-96a8-1c03879e0725.png)



![image](https://user-images.githubusercontent.com/38436013/126944658-b23c65fa-d55b-4df1-9a5d-a655590032e0.png)

- **U**niform: 리소스 식별하는 통일된 방식 
- **R**esource: 자원, URI로 식별할 수 있는 모든 것(제한 없음) 
- **I**dentifier: 다른 항목과 구분하는데 필요한 정보  
- URL: Uniform Resource Locator 
- URN: Uniform Resource Name

### URL, URN

- URL - Locator: 리소스가 있는 위치를 지정, 웹 브라우저에 적는 주소
- URN - Name: 리소스에 이름을 부여 
- 위치는 변할 수 있지만, 이름은 변하지 않는다. 
- urn:isbn:8960777331 (어떤 책의 isbn URN) 
- URN 이름만으로 실제 리소스를 찾을 수 있는 방법이 보편화 되지 않음 
- **앞으로 URI를 URL과 같은 의미로 이야기하겠음**

### URL 전체 문법

- scheme://[userinfo@]host[:port][/path][?query][#fragment] 

- https://www.google.com:443/search?q=hello&hl=ko 

  

- 프로토콜(https)  
- 호스트명(www.google.com) 
- 포트 번호(443)  
- 패스(/search) 
- 쿼리 파라미터(q=hello&hl=ko)

<img src="https://user-images.githubusercontent.com/38436013/126947390-b231969b-b33c-4feb-869b-d341bf246250.png" alt="image" style="zoom: 80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126947408-fafce539-fbfb-4bd2-a433-f63b3b3a778a.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126947425-8a8fa4b5-9e57-4c5e-a77c-59ae146ec533.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126947438-b795c8e3-c7b9-4ffd-a306-b26a78d047c6.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126947458-270e0fc7-111b-4a4f-b6df-21c4324a2435.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126947474-46d7be99-c68a-441d-94c9-5b13cb4ec0c9.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126947500-0e67dd42-c095-4dcd-bcd4-a5cf5e632f38.png" alt="image" style="zoom:80%;" />

# 2. 웹 브라우저 요청 흐름

<img src="https://user-images.githubusercontent.com/38436013/126950435-83de1c42-241e-4f8d-b542-028f4397311c.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126950457-5a85c3bd-f887-4248-8124-9df5a6413196.png" alt="image" style="zoom:80%;" />



### HTTP 메시지 전송

<img src="https://user-images.githubusercontent.com/38436013/126950532-54570a79-f83c-4abb-a06e-f76c3c8ac105.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126950560-907c5e41-ae7c-425e-88ee-9743705302f2.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126950627-279117e1-b03d-4afe-b374-ed330077345e.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126950662-e7ec6d89-8ac0-4f02-9944-684098175dd0.png" alt="image" style="zoom:80%;" />
<img src="https://user-images.githubusercontent.com/38436013/126950684-272907c5-47e3-4cf8-8d9e-c08075839727.png" alt="image" style="zoom:80%;" />