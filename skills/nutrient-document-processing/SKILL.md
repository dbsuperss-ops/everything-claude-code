---
name: nutrient-document-processing
description: Nutrient DWS API를 사용하여 문서 처리(변환, OCR, 추출, 민감 정보 마스킹, 서명, 폼 채우기 등)를 수행합니다. PDF, DOCX, XLSX, PPTX, HTML 및 이미지를 지원합니다.
origin: ECC
---

# Nutrient 문서 처리 (Nutrient Document Processing)

[Nutrient DWS Processor API](https://www.nutrient.io/api/)를 사용하여 문서를 처리합니다. 포맷 변환, 텍스트 및 테이블 추출, 스캔된 문서의 OCR, 개인정보(PII) 마스킹, 워터마크 추가, 디지털 서명 및 PDF 폼 채우기 기능을 제공합니다.

## 설정 (Setup)

**[nutrient.io](https://dashboard.nutrient.io/sign_up/?product=processor)**에서 무료 API 키를 발급받으십시오.
전축 환경 변수 `NUTRIENT_API_KEY`에 설정하여 사용합니다.

## 주요 작업

### 1. 문서 변환 (Convert)
- DOCX, XLSX, PPTX, HTML 등을 PDF로 변환하거나 그 반대로 변환할 수 있습니다.
- 이미지(JPG, PNG, TIFF 등) 변환도 지원합니다.

### 2. 텍스트 및 데이터 추출
- PDF에서 일반 텍스트를 추출하거나 테이블을 엑셀(XLSX) 형식으로 추출할 수 있습니다.

### 3. OCR (광학 문자 인식)
- 스캔된 문서나 이미지를 검색 가능한 PDF로 변환합니다.
- `kor`(한국어), `eng`(영어)를 포함한 100개 이상의 언어를 지원합니다.

### 4. 민감 정보 마스킹 (Redaction)
- 프리셋(`social-security-number`, `email-address`, `credit-card-number` 등)이나 정규표현식을 사용하여 문서 내 민감 정보를 자동으로 가릴 수 있습니다.

### 5. 워터마크 추가 및 디지털 서명
- "CONFIDENTIAL"과 같은 텍스트 워터마크를 투명도와 각도를 조절하여 추가할 수 있습니다.
- CMS 서명 방식을 이용한 디지털 서명을 지원합니다.

### 6. PDF 폼 채우기 (Fill Form)
- JSON 데이터를 전달하여 PDF 내의 폼 필드(`name`, `email` 등)를 프로그램 방식으로 채울 수 있습니다.

## MCP 서버 활용
`npx`를 통해 `@nutrient-sdk/dws-mcp-server`를 설치하고 설정하여 네이티브 도구처럼 사용할 수도 있습니다.

## 활성화 시점
- 문서 포맷 간 변환이 필요할 때
- PDF에서 텍스트나 테이블 데이터를 추출해야 할 때
- 스캔된 이미지 문서에 OCR 처리가 필요할 때
- 문서를 공유하기 전 개인정보를 가려야 할 때 (PII Redaction)
- 계약서나 합의서에 디지털 서명을 하거나 PDF 폼을 자동으로 채워야 할 때

**기억하십시오**: Nutrient API는 강력한 문서 처리 기능을 제공합니다. API 요청 시 `instructions` JSON 필드를 통해 세부 동작을 정교하게 제어하십시오.
    
