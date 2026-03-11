---
description: "공통 규칙을 확장하는 Swift 보안"
globs: ["**/*.swift", "**/Package.swift"]
alwaysApply: false
---
# Swift 보안 (Security)

> 이 문서는 공통 보안 규칙을 기반으로 Swift에 특화된 내용을 확장합니다.

## 비밀 정보(Secret) 관리

- 민감한 데이터(토큰, 비밀번호, 키 등)에는 **Keychain Services**를 사용하십시오. 절대 `UserDefaults`를 사용하지 마십시오.
- 빌드 타임의 비밀 정보에는 환경 변수나 `.xcconfig` 파일을 사용하십시오.
- 소스 코드에 비밀 정보를 절대 하드코딩하지 마십시오. 디컴파일 도구를 통해 쉽게 추출될 수 있습니다.

```swift
let apiKey = ProcessInfo.processInfo.environment["API_KEY"]
guard let apiKey, !apiKey.isEmpty else {
    fatalError("API_KEY가 구성되지 않았습니다")
}
```

## 전송 보안 (Transport Security)

- 앱 전송 보안(ATS)은 기본적으로 강제됩니다. 이를 비활성화하지 마십시오.
- 중요한 엔드포인트에 대해서는 인증서 핀닝(Certificate pinning)을 사용하십시오.
- 모든 서버 인증서를 검증하십시오.

## 입력값 검증

- 인젝션 공격을 방지하기 위해 모든 사용자 입력값을 표시하기 전에 정제(Sanitize)하십시오.
- 강제 언래핑(Force-unwrapping) 대신 유효성 검사가 포함된 `URL(string:)`을 사용하십시오.
- 외부 소스(API, 딥 링크, 페이스트보드)로부터 오는 데이터는 처리하기 전에 유효성을 검증하십시오.
