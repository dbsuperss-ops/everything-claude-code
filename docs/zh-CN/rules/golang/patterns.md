---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---

# Go 패턴 (Patterns)

> 이 문서는 [common/patterns.md](../common/patterns.md)의 내용을 바탕으로 Go 언어에 특화된 내용을 확장합니다.

## 함수형 옵션 (Functional Options) 패턴

```go
type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) { s.port = port }
}

func NewServer(opts ...Option) *Server {
    s := &Server{port: 8080}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

## 인터페이스 최소화

인터페이스는 구현되는 곳이 아니라 **사용되는 곳**에서 정의하십시오.

## 의존성 주입 (Dependency Injection)

생성자를 사용하여 의존성을 주입하십시오:

```go
func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}
```

## 참고 자료

동시성, 에러 처리, 패키지 구성 등을 포함한 Go 언어의 종합적인 패턴에 대해서는 `golang-patterns` 스킬을 참조하십시오.
