---
paths:
  - "**/*.go"
  - "**/go.mod"
  - "**/go.sum"
---
# Go 패턴 (Go Patterns)

> 이 파일은 [common/patterns.md](../common/patterns.md)을 Go 전용 내용으로 확장합니다.

## 함수형 옵션 (Functional Options)

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

## 작은 인터페이스 (Small Interfaces)

인터페이스는 구현되는 곳이 아니라 사용되는 곳에서 정의하십시오.

## 의존성 주입 (Dependency Injection)

의존성을 주입하기 위해 생성자 함수를 사용하십시오:

```go
func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}
```

## 참조

동시성, 에러 처리, 패키지 구성 등을 포함한 포괄적인 Go 패턴에 대해서는 `golang-patterns` 스킬을 참조하십시오.
