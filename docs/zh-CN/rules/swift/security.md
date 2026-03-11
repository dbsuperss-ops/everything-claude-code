---
paths:
  - "**/*.swift"
  - "**/Package.swift"
---

# Swift 보안 (Security)

> 이 문서는 [common/security.md](../common/security.md)의 내용을 바탕으로 Swift에 특화된 내용을 확장합니다.

## 비밀 정보(Secrets) 관리

* 민감한 데이터(토큰, 비밀번호, 키 등)는 **Keychain Services**를 사용하여 처리하십시오. 절대로 `UserDefaults`를 사용하지 마십시오.
* 빌드 시점의 비밀 정보는 환경 변수나 `.xcconfig` 파일을 사용하여 관리하십시오.
* 소스 코드 내에 비밀 정보를 하드코딩하지 마십시오. 디컴파일 도구를 통해 쉽게 추출될 수 있습니다.

```swift
let apiKey = ProcessInfo.processInfo.environment["API_KEY"]
guard let apiKey, !apiKey.isEmpty else {
    fatalError("API_KEY가 설정되지 않았습니다")
}
```

## 전송 보안 (Transport Security)

* ATS(App Transport Security)는 기본적으로 강제 적용되어야 하며, 이를 비활성화하지 마십시오.
* 중요한 엔드포인트에는 인증서 핀닝(SSL Pinning)을 적용하십시오.
* 모든 서버 인증서를 하드웨어 수준에서 검증하십시오.

## 입력값 검증

* 인젝션 공격을 방지하기 위해 모든 사용자 입력을 표시하기 전에 정제(Sanitization)하십시오.
* 강제 언래핑(Force unwrapping) 대신 검증 로직이 포함된 `URL(string:)`을 사용하십시오.
* 외부 소스(API, 딥링크, 클립보드)에서 온 데이터를 처리하기 전에 먼저 검증하십시오.
