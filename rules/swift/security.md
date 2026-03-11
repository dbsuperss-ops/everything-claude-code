---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---
# Swift 보안 (Swift Security)

> 이 파일은 [common/security.md](../common/security.md)을 Swift 전용 내용으로 확장합니다.

## 비밀 정보 관리

- 민감한 데이터(토큰, 비밀번호, 키)에는 **Keychain Services**를 사용하십시오 — `UserDefaults`는 절대 사용하지 마십시오.
- 빌드 시점의 비밀 정보에는 환경 변수나 `.xcconfig` 파일을 사용하십시오.
- 소스 코드에 비밀 정보를 하드코딩하지 마십시오 — 디컴파일 도구를 통해 쉽게 추출될 수 있습니다.

```swift
let apiKey = ProcessInfo.processInfo.environment["API_KEY"]
guard let apiKey, !apiKey.isEmpty else {
    fatalError("API_KEY가 설정되지 않았습니다.")
}
```

## 통신 보안 (Transport Security)

- 앱 통신 보안(ATS)은 기본적으로 강제됩니다 — 이를 비활성화하지 마십시오.
- 핵심 엔드포인트에 대해서는 인증서 피닝(Certificate pinning)을 사용하십시오.
- 모든 서버 인증서를 검증하십시오.

## 입력값 검증

- 인젝션 방지를 위해 표시 전 모든 사용자 입력을 새니타이징하십시오.
- 강제 언래핑 대신 검증 절차가 포함된 `URL(string:)`을 사용하십시오.
- 처리 전 외부 소스(API, 딥링크, 붙여넣기 보드 등)에서 온 데이터를 검증하십시오.
