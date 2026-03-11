---
description: "공통 규칙을 확장하는 Go 패턴"
globs: ["**/*.go", "**/go.mod", "**/go.sum"]
alwaysApply: false
---
# Go 패턴 (Patterns)

> 이 문서는 공통 패턴 규칙을 기반으로 Go 언어에 특화된 내용을 확장합니다.

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

인터페이스는 구현되는 시점이 아니라 사용되는 시점에 정의하십시오.

## 의존성 주입 (Dependency Injection)

생성자 함수를 사용하여 의존성을 주입하십시오:

```go
func NewUserService(repo UserRepository, logger Logger) *UserService {
    return &UserService{repo: repo, logger: logger}
}
```

## 참고 자료

동시성, 에러 처리, 패키지 구성 등을 포함한 Go 언어의 종합적인 패턴에 대해서는 `golang-patterns` 스킬을 참조하십시오.
