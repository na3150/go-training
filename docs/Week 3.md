# Week 3 : Example 21 - 30

## Example 21 : Simple https-tls server example

1. Install `mkcert`

```bash
$ brew install mkcert
```

```bash
$ mkcert --install
Created a new local CA 💥
Sudo password:
The local CA is now installed in the system trust store! ⚡️
```

2. What is mkcert?

- 로컬 개발 환경에서 안전한 HTTPS 연결을 설정하는 데 사용되는 도구
- mkcert는 로컬에서 유효한 SSL/TLS 인증서를 생성하고 관리하기 위한 간단한 도구
- 이를 사용하면 신뢰할 수 있는 CA로 부터 인증서를 발급받는 대신에**, 로컬에서 자체적으로 개발 및 테스트용 인증서를 생성**할 수 있다. 

3. `main.go`

```go
package main

import (
	"log"
	"net/http"
)

func helloServer(w http.ResponseWriter, req *http.Request) {
	w.Header().Set("Content-Type", "text/plain")
	w.Write([]byte("This is an example server.\n"))
}

func main() {
	log.Println("Server listen in 443 port. Please open https://localhost/hello")
	http.HandleFunc("/hello", helloServer)
	err := http.ListenAndServeTLS(":443", "ssl/localhost.pem", "ssl/localhost-key.pem", nil)
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}
```

- func `helloServer` : HTTP 요청에 대한 응답으로 "This is an example server." 라는 문자열을 포함한 plain text를 반환
- func `main` 
  - https://localhost/hello 엔드포인트에 대한 요청을 처리하는 핸들러를 등록
  - `http.ListenAndServeTLS` 함수를 이용하여 HTTPS 서버를 시작
  - `(port number, SSL path, key path, handler)`

4. mkcert 인증서 생성

```bash
mkcert localhost 
mv localhost-key.pem localhost.pem ssl/
```

5. 서버 실행

``` bash
 go run main.go
```

6. https://localhost/hello 접속

![스크린샷 2024-03-17 오후 10.01.18](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.01.18.png)











