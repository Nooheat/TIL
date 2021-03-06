# 웹 서버 만들기
## 웹 서버의 동작 원리
1. 웹 서버가 연결을 대기한다. 이때 포트를 지정한다. **Listen**
2. 클라이언트(웹 브라우저)에서 연결을 요청한다. **Connect**
3. 웹 서버가 연결 요청을 받아들인다. **Accept**
4. 상호간에 데이터를 요청하고 받는다. **Request, Response**
5. 연결을 종료한다 **Close**

## 간단한 웹 서버 만들기

1. http모듈을 로딩한다.
```js
var http = require('http');
```

2. 웹 서버 객체를 만든다.  
```js
var server = http.createServer();
```

3. 웹 서버 포트를 지정하고 시작해 요청을 대기한다.  
```js
var port = 8080;
server.listen(port, function(){
  console.log('웹 서버가 시작되었습니다. : %d', port);
  });
```

4. 서버 이벤트를 등록한다.  
  - 서버 이벤트 목록

  이벤트 이름 | 설명
  ---|---
  connection | 클라이언트가 접속하여 연결이 만들어질 때 발생하는 이벤트
  request | 클라이언트가 요청할 때 발생하는 이벤트
  close | 서버를 종료할 때 발생하는 이벤트

```js
// 클라이언트 연결 이벤트 처리
server.on('connection', function(socket){
  var addr = socket.address();
  console.log('클라이언트가 접속했습니다. : %s, %d', addr.address, addr.port);
  });

// 클라이언트 요청 이벤트 처리
server.on('request', function(req, res){
  console.log('클라이언트 요청이 들어왔습니다.');
  // 이 밖에 처리할 코드 작성
  });

// 서버 종료 이벤트 처리
server.on('close', function(){
  console.log('서버가 종료됩니다.');
  // 이 밖에 처리할 코드 작성
  });
```

5. 종합
```js
// http 모듈 로딩
var http = require('http');

// 웹 서버 객체를 생성
var server = http.createServer();

// 웹 서버 포트를 지정하고 시작해 요청을 대기
var port = 8080;
server.listen(port, function(){
  console.log('웹 서버가 시작되었습니다. : %d', port);
  });

// 클라이언트 연결 이벤트 처리
server.on('connection', function(socket){
  var addr = socket.address();
  console.log('클라이언트가 접속했습니다. : %s, %d', addr.address, addr.port);
  });

// 클라이언트 요청 이벤트 처리
server.on('request', function(req, res){
  console.log('클라이언트 요청이 들어왔습니다.');
  // 이 밖에 처리할 코드 작성
  });

// 서버 종료 이벤트 처리
server.on('close', function(){
  console.log('서버가 종료됩니다.');
  // 이 밖에 처리할 코드 작성
  });
```

## 응답을 보낼 때 사용하는 주요 메소드들
response객체(보통은 res라고 씀)를 이용해 클라이언트로 응답을 전송할 때에는 주로 writeHead, write, end 세 가지 메소드들을 사용한다.  

메소드 이름 | 설명
---|---
writeHead(statusCode[, statusMessage][, headers]) | 응답으로 보낼 헤더를 만든다.
write(chunk[, encoding][, callback]) | 응답 본문(body)데이터를 만든다. 여러 번 호출될 수 있음.
end([data],[, encoding],[, callback]) | 클라이언트로 응답을 전송한다. 파라미터에 데이터가 들어 있다면 이 데이터를 포함시켜 응답을 전송한다. 클라이언트의 요청이 있을 때 한 번은 호출되어야 응답을 보내며, 콜백 함수가 지정되면 응답이 전송된 후 콜백 함수가 호출된다.

## 파일을 스트림으로 읽어 응답 보내기
파일을 스트림 객체로 읽어 들인 후 **pipe() 메소드**로 응답 객체와 연결하면 별다른 코드 없이도 파일에 응답을 보낼 수 있다.  
### 예제코드
```js
const http = require('http');
const fs = require('fs');

const port = 8080;
server = http.createServer();

server.listen(port, function(){
  console.log('웹 서버가 시작되었습니다. : %d', port);
});

server.on('connection', function(socket){
  var addr = socket.address();
  console.log('클라이언트가 접속했습니다. :%s, %d',addr.address, addr.port);
});

// 서버에 요청이 들어왔을 때 발생하는 이벤트 'request'
server.on('request', function(req, res){
  console.log('클라이언트 요청이 들어왔습니다.');

  var fileName = 'nodejs.png'; // 이미지 이름.

  fs.readFile(fileName, function(err, data){
    if(err) console.log('sex');
    res.writeHead(200, {"Content-Type" : "image/png"});
    res.write(data);
    res.end();
  });
  
});
``` 
### 예제 리소스
**요청이 왔을 때 보낼 이미지, 아래 이미지를 예제 소스 파일이 있는 디렉토리에 다운로드 받으세요**  
![NodeJS](./http_resources/nodejs.png)

## 클라이언트에서 요청이 있을 때 파일 읽어 응답하기

### Content-Type 헤더
**서버에서는 응답하면서 일반 데이터 이외의 다른 형식의 파일들 또한 전송할 수 있음**  
**-> Content-Type 헤더의 값을 전송하고자 하는 파일 형식에 맞는 MIME TYPE으로 지정해주면 됨**  

### MIME TYPE
* MIME TYPE : **M**ultipurpose **I**nternet **M**ail **E**xtensions의 약어로 메시지의 내용이 어떤 형식인지 알려주기 위해 정의한 인터넷 표준  

* 대표적인 MIME Type  

Content Type의 값(MIME Type) | 설명
---|---
text/plain | 일반 텍스트 문서
text/html | HTML 문서
text/css | CSS 문서
text/xml | XML 문서
image/jpeg, image/png | JPEG 파일, PNG 파일
video/mpeg, audio/mp3 | MPEG 비디오 파일, MP3 음악 파일
application/zip | ZIP 압축 파일



