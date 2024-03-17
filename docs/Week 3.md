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

<br>

## Example 22 : Go module in go

1. tree 구조

```bash
$ tree .
.
├── go.mod
├── go.sum
├── main.go
└── vendor
    ├── github.com
    │   └── appleboy
    │       └── com
    │           ├── LICENSE
    │           └── random
    │               └── random.go
    └── vendor.json

6 directories, 6 files
```

- `go.mod`  : go module 파일로, 프로젝트의 의존성 관리와 버전 관리를 담당
- `go.sum`  : go module의 의존성에 대한 체크섬 정보를 포함하는 파일
- `vendor` : 외부 의존성 패키지들이 저장되는 디렉토리, 일반적으로 해당 디렉토리 아래에 외부 패키지들이 복제되어 저장

2. main 함수 및 패키지

`main.go`

```go
package main

import (
	"fmt"

	"github.com/appleboy/com/random"
)

func main() {
	fmt.Println(random.String(10))
}
```

- `random.String(10)` 함수를. 호출하여 길이가 10인 랜덤 문자열을 생성

`ramdon.go`

```go
package random

import (
	"math/rand"
	"time"
)

type (
	// Charset is string type
	Charset string
)

const ( 
  // Alphanumeric contain Alphabetic and Numeric : 알파벳과 숫자를 포함하는 문자열
	Alphanumeric Charset = Alphabetic + Numeric
  // Alphabetic is \w+ \W : 알파벳 문자열
	Alphabetic Charset = "abcdefghijklmnopqrstuvwxyz" + "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
	// Numeric is number list
	Numeric Charset = "0123456789"
  // Hex is Hexadecimal : 16진수 문자열 
	Hex Charset = Numeric + "abcdef"
)

var seededRand = rand.New(
	rand.NewSource(time.Now().UnixNano()))

// StringWithCharset support rand string you defined
func StringWithCharset(length int, charset Charset) string {
	b := make([]byte, length)
	for i := range b {
		b[i] = charset[seededRand.Intn(len(charset))]
	}
	return string(b)
}

// String supply rand string
func String(length int) string {
	return StringWithCharset(length, Alphanumeric) 
}
```

- 주어진 길이의 랜덤 문자열을 생성할 수 있으며, 알파벳/숫자/16진수 등 다양한 문자열 유형을 생성
- var 변수 `seededRand` : `rand.NewSource(time.Now().UnixNano())` 를 사용하여 시드를 설정한 새로운 rand.Rand 인스턴스를 생성

3. main 함수 실행

```bash
$ go run main.go
go: downloading github.com/appleboy/com v0.0.0-20180410030638-c0b5901f9622
9oJ9X6SptM
```

<br>

## Example 23 : Deploy go application with up

1. `main.go`

```go
package main

import (
	"os"

	"github.com/gin-gonic/gin"
)

func main() {
	port := ":" + os.Getenv("PORT")
	stage := os.Getenv("UP_STAGE")

	r := gin.Default() //Gin의 기본 라우터를 생성
	r.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong " + stage,
		})
	})

	r.GET("/v1", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong " + stage + " v1 ++ drone",
		})
	})

	r.GET("/v2", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong " + stage + " v2 ++ production",
		})
	})

	r.GET("/v3", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong " + stage + " v3 ++ production",
		})
	})

	r.Run(port)
}
```

- `gin-gonic/gin` 패키지를 임포트하여 Gin 웹 프레임워크를 사용
  - Gin 웹 프레임워크란? 
    - go 언어로 작성된 웹 프레임워크 중 하나고, 빠르고 경량화된 특징을 가지고 있으며, 높은 성능과 생산성 제공
    - HTTP 라우팅 및 미들웨어 기능을 제공하여 웹 애플리케이션을 쉽게 구축할 수 있도록 도와줌
- Gin의 기본 라우터를 생성하고, 루트(`/`) 엔드포인트를 설정합니다. 이 엔드포인트는 클라이언트로 "pong" 메시지를 반환합니다.
- `/v1`, `/v2`, `/v3` 경로에 대한 핸들러를 설정하고, 각각 다른 메시지를 반환
- `r.Run(port)`를 호출하여 서버를 지정된 포트에서 실행

2. 환경 변수 세팅

```bash
export PORT=8080
export UP_STAGE=development
```

3. 서버 실행 및 접속

```bash
go run main.go
```

![스크린샷 2024-03-17 오후 10.35.02](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.35.02.png)

![스크린샷 2024-03-17 오후 10.35.23](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.35.23.png)

![스크린샷 2024-03-17 오후 10.35.30](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.35.30.png)

![스크린샷 2024-03-17 오후 10.35.38](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2010.35.38.png)

<br>

## Example 24 : Debug go code using vs code

1. `main.go`

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func dostuff(wg *sync.WaitGroup, i int) {
	fmt.Printf("goroutine id %d\n", i)
	time.Sleep(60 * time.Second)
	fmt.Printf("goroutine id %d\n", i)
	wg.Done() //wg.Done()을 호출하여 해당 고루틴이 완료되었음을 WaitGroup에 알린다
}

func main() {
	var wg sync.WaitGroup // Go언어에서 동시성 프로그래밍을 지원하기 위한 동기화 메커니즘 중 하나인 WaitGroup을 선언하는 부분
	workers := 10

	wg.Add(workers) //wg.Add(workers)를 호출하여 WaitGroup에 고루틴 개수를 추가
	for i := 0; i < workers; i++ {
		go dostuff(&wg, i)
	}
	wg.Wait()
}
```

- 10개의 고루틴을 생성
- 각각의 고루틴이 `dostuff` 함수를 실행하고 있으며, 이 함수는 60초 동안 대기한 다음 작업을 완료
  - 고루틴(Goroutine) : golang에서 사용되는 경량 쓰레드(Lightweight Thread), 동시에 여러 작업을 처리하는 데 사용되며, golang 특징 중 하나로 병행성(Concurrency)을 구현하는 데 중요한 역할을 한다.
  - 고루틴은 `go` 키워드를 사용하여 생성된다. 예를 들어, `go dostuff()`와 같이 함수를 호출할 때 앞에 "go" 키워드를 붙여 고루틴으로 실행할 수 있다.
- WaitGroup은 고루틴이 모두 실행을 완료할 때까지 대기할 수 있도록 하는 동기화 기능을 제공

2. Go 실행

```bash
$ go run main.go
goroutine id 0
goroutine id 1
goroutine id 5
goroutine id 3
goroutine id 4
goroutine id 7
goroutine id 2
goroutine id 8
goroutine id 6
goroutine id 9
goroutine id 9
goroutine id 8
goroutine id 7
goroutine id 6
goroutine id 2
goroutine id 0
goroutine id 4
goroutine id 1
goroutine id 5
goroutine id 3
```

- 고루틴이 병렬적으로 실행되기 때문에, 고루틴이 생성된 순서와 실제 실행되는 순서가 일치하지 않을 수 있다.
- 이는 고루틴이 생성되면서, 스케줄러에 의해 실행되는 시점이 어떤 고루틴 먼저 실행될 지 결정되기 때문이다.
- 즉, **고루틴이 생성된 순서와 실행되는 순서는 보장되지 않으며**, 고루틴들간의 실행 순서는 스케줄러의 동작에 따라 달라질 수 있다. 
- 고루틴 간의 실행 순서를 보장하기 위해서는 동기화 메커니즘을 사용해야 한다. 

> 고루틴 간의 실행 순서를 보장하기 위해서는 동기화 메커니즘을 사용해야 한다고 너가 아까 얘기했고, 지금 WaitGroup이라는 동기화 메커니즘을 사용하고 있으면 고루틴의 순서는 보장되어야하는거 아닌가?
>
> WaitGroup은 고루틴 간의 실행 순서를 보장하기 위한 동기화 메커니즘 중 하나이나,  주의할 점은 WaitGroup이 고루틴의 **실행 순서를 제어하는 것이 아니라, 고루틴이 모두 종료될 때까지 기다리도록 하는 동기화 메커니즘**이라는 점이다. 만약 특정 작업을 순차적으로 실행해야 한다면 고루틴 내부에서 이를 보장하는 추가적인 동기화 메커니즘이 필요하다. 

<br>

## Example 25 : Traefik golang app lets encrypt

1. `main.go`

```go
package main

import (
	"flag"
	"fmt"
	"log"
	"net/http"
	"os"
	"time"
)

// HelloWorld for hello world
func HelloWorld() string {
	return "Hello World, traefik workshop!"
}

func handler(w http.ResponseWriter, r *http.Request) {
	log.Printf("Got http request. time: %v", time.Now())
	fmt.Fprintf(w, "I love %s!, %s", r.URL.Path[1:], HelloWorld())
}

func pinger(port string) error { //서버 상태 확인
	resp, err := http.Get("http://localhost:" + port)
	if err != nil {
		return err
	}
	defer resp.Body.Close()
	if resp.StatusCode != 200 {
		return fmt.Errorf("server returned not-200 status code")
	}

	return nil
}

func main() {
	var port string
	var ping bool
	flag.StringVar(&port, "port", "8080", "server port") //서버가 실행될 포트
	flag.StringVar(&port, "p", "8080", "server port")
	flag.BoolVar(&ping, "ping", false, "check server live") //서버 상태를 확인할 지 여부
	flag.Parse() //명령줄 플래그를 분석하고 지정된 변수에 값을 할당

	if p, ok := os.LookupEnv("PORT"); ok { //환경 변수에서 PORT 값을 읽어와서 port 변수에 할당
		port = p
	}

	if ping { //만약 ping 변수가 true이면, pinger() 함수를 호출하여 서버가 정상적으로 응답하는지 확인
		if err := pinger(port); err != nil {
			log.Printf("ping server error: %v\n", err)
		}

		return
	}

	http.HandleFunc("/", handler)
	log.Println("http server run on " + port + " port")
	log.Fatal(http.ListenAndServe(":"+port, nil)) //오류 처리 -> 오류가 발생하면 해당 오류 메세지 출력
}
```

- func `handler()` : HTTP 요청을 처리하는 핸들러 함수로, 요청이 들어오면 현재 시간을 로깅하고, 요청된 경로에 따라 적절한 응답을 반환
  - `fmt.Fprintf(w, "I love %s!, %s", r.URL.Path[1:], HelloWorld())` : 경로에 입력한 문자열
- func `pinger()` : 서버의 상태를 확인하는 함수로, 지정된 포트로 HTTP GET 요청을 보내고, 응답이 정상적으로 돌아오는지 확인

3. 서버 실행 및 접속 확인

```bash
go run main.go
```

![스크린샷 2024-03-17 오후 11.09.28](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.09.28.png)

![스크린샷 2024-03-17 오후 11.16.34](https://raw.githubusercontent.com/na3150/typora-img/main/uPic/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202024-03-17%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%2011.16.34.png)

4. 로그 확인

```bash
2024/03/17 23:08:47 http server run on 8080 port
2024/03/17 23:09:18 Got http request. time: 2024-03-17 23:09:18.640041 +0900 KST m=+30.751460084
2024/03/17 23:09:19 Got http request. time: 2024-03-17 23:09:19.09654 +0900 KST m=+31.207967459
```

<br>

## Example 26 : Go trace

1. `main.go`

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/trace"
)

func Sum(numbers ...int) int {

	total := 0
	for _, n := range numbers {
		total += n
	}
	return total
}

func SumParallel(numbers ...int) int {
	mid := len(numbers) / 2

	ch := make(chan int)
	go func() { ch <- Sum(numbers[:mid]...) }()
	go func() { ch <- Sum(numbers[mid:]...) }()

	total := <-ch + <-ch
	return total
}

var foo = []int{4, 5, 6, 1, 3, 2, 8, 7, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20}

func main() {
	trace.Start(os.Stderr)
	defer trace.Stop()
	fmt.Println(runtime.NumCPU())
	// fmt.Println(Sum(foo...))
	fmt.Println(SumParallel(foo...))
}
```

2. `main_test.go`

```go
package main

import (
	"fmt"
	"os"
	"runtime"
	"runtime/trace"
)

func Sum(numbers ...int) int {

	total := 0
	for _, n := range numbers {
		total += n
	}
	return total
}

func SumParallel(numbers ...int) int {
	mid := len(numbers) / 2

	ch := make(chan int)
	go func() { ch <- Sum(numbers[:mid]...) }()
	go func() { ch <- Sum(numbers[mid:]...) }()

	total := <-ch + <-ch
	return total
}

var foo = []int{4, 5, 6, 1, 3, 2, 8, 7, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20}

func main() {
	trace.Start(os.Stderr)
	defer trace.Stop()
	fmt.Println(runtime.NumCPU())
	// fmt.Println(Sum(foo...))
	fmt.Println(SumParallel(foo...))
}
```












