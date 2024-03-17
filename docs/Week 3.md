# Week 3 : Example 21 - 30

<br>

<br>

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

What is mkcert?

- 로컬 개발 환경에서 안전한 HTTPS 연결을 설정하는 데 사용되는 도구
- mkcert는 로컬에서 유효한 SSL/TLS 인증서를 생성하고 관리하기 위한 간단한 도구
- 이를 사용하면 신뢰할 수 있는 CA로 부터 인증서를 발급받는 대신에**, 로컬에서 자체적으로 개발 및 테스트용 인증서를 생성**할 수 있다. 

`main.go`

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

