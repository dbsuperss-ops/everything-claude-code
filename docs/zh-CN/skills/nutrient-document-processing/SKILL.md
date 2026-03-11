---
name: nutrient-document-processing
description: Nutrient DWS API를 사용하여 문서를 처리, 변환, OCR 인식, 추출, 수정(Redact), 서명 및 입력합니다. PDF, DOCX, XLSX, PPTX, HTML 및 이미지 형식을 지원합니다.
origin: ECC
---

# 문서 처리 (Document Processing)

[Nutrient DWS Processor API](https://www.nutrient.io/api/)를 사용하여 문서를 처리합니다. 형식 변환, 텍스트 및 테이블 추출, 스캔된 문서의 OCR, 개인정보(PII) 수정, 워터마크 추가, 디지털 서명 및 PDF 양식 작성을 지원합니다.

## 설정

**[nutrient.io](https://dashboard.nutrient.io/sign_up/?product=processor)**에서 무료 API 키를 발급받으십시오.

```bash
export NUTRIENT_API_KEY="pdf_live_..."
```

모든 요청은 `https://api.nutrient.io/build` 주소로 `instructions` JSON 필드와 함께 multipart POST 방식으로 전송됩니다.

## 주요 작업 (Operations)

### 문서 변환 (Convert)

```bash
# DOCX를 PDF로 변환
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.docx=@document.docx" \
  -F 'instructions={"parts":[{"file":"document.docx"}]}' \
  -o output.pdf

# PDF를 DOCX로 변환
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"docx"}}' \
  -o output.docx

# HTML을 PDF로 변환
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "index.html=@index.html" \
  -F 'instructions={"parts":[{"html":"index.html"}]}' \
  -o output.pdf
```

지원되는 입력 형식: PDF, DOCX, XLSX, PPTX, DOC, XLS, PPT, PPS, PPSX, ODT, RTF, HTML, JPG, PNG, TIFF, HEIC, GIF, WebP, SVG, TGA, EPS.

### 텍스트 및 데이터 추출 (Extract)

```bash
# 일반 텍스트 추출
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"text"}}' \
  -o output.txt

# 테이블을 Excel로 추출
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"output":{"type":"xlsx"}}' \
  -o tables.xlsx
```

### 스캔 문서 OCR 처리

```bash
# OCR을 통해 검색 가능한 PDF로 변환 (100개 이상의 언어 지원)
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "scanned.pdf=@scanned.pdf" \
  -F 'instructions={"parts":[{"file":"scanned.pdf"}],"actions":[{"type":"ocr","language":"kor"}]}' \
  -o searchable.pdf
```

지원 언터: ISO 639-2 코드를 통해 100개 이상의 언어를 지원합니다 (예: `kor`, `eng`, `deu`, `fra`, `spa`, `jpn`, `chi_sim`, `chi_tra`, `ara`, `hin`, `rus`). `korean` 또는 `english`와 같은 전체 언어 이름도 사용 가능합니다. [전체 OCR 언어 표](https://www.nutrient.io/guides/document-engine/ocr/language-support/)에서 지원되는 모든 코드를 확인할 수 있습니다.

### 민감 정보 수정 (Redaction)

```bash
# 프리셋 기반 (주민등록번호, 이메일 등)
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"redaction","strategy":"preset","strategyOptions":{"preset":"social-security-number"}},{"type":"redaction","strategy":"preset","strategyOptions":{"preset":"email-address"}}]}' \
  -o redacted.pdf

# 정규표현식(Regex) 기반
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"redaction","strategy":"regex","strategyOptions":{"regex":"\\b[A-Z]{2}\\d{6}\\b"}}]}' \
  -o redacted.pdf
```

지원 프리셋: `social-security-number`, `email-address`, `credit-card-number`, `international-phone-number`, `north-american-phone-number`, `date`, `time`, `url`, `ipv4`, `ipv6`, `mac-address`, `us-zip-code`, `vin`.

### 워터마크 추가

```bash
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"watermark","text":"대외비(CONFIDENTIAL)","fontSize":72,"opacity":0.3,"rotation":-45}]}' \
  -o watermarked.pdf
```

### 디지털 서명

```bash
# 자체 서명된 CMS 서명
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "document.pdf=@document.pdf" \
  -F 'instructions={"parts":[{"file":"document.pdf"}],"actions":[{"type":"sign","signatureType":"cms"}]}' \
  -o signed.pdf
```

### PDF 양식 작성 (Fill Form)

```bash
curl -X POST https://api.nutrient.io/build \
  -H "Authorization: Bearer $NUTRIENT_API_KEY" \
  -F "form.pdf=@form.pdf" \
  -F 'instructions={"parts":[{"file":"form.pdf"}],"actions":[{"type":"fillForm","formFields":{"name":"홍길동","email":"kildong@example.com","date":"2026-02-06"}}]}' \
  -o filled.pdf
```

## MCP 서버 (대안)

네이티브 도구 연동을 위해 curl 대신 MCP 서버를 사용할 수 있습니다:

```json
{
  "mcpServers": {
    "nutrient-dws": {
      "command": "npx",
      "args": ["-y", "@nutrient-sdk/dws-mcp-server"],
      "env": {
        "NUTRIENT_DWS_API_KEY": "YOUR_API_KEY",
        "SANDBOX_PATH": "/작업/디렉토리/경로"
      }
    }
  }
}
```

## 사용 사례

* 문서 형식 간 변환 (PDF, DOCX, XLSX, PPTX, HTML, 이미지)
* PDF에서 텍스트, 테이블 또는 키-값 쌍(Key-value pairs) 추출
* 스캔된 문서나 이미지에 대한 OCR 처리
* 문서 공유 전 개인정보(PII) 비식별화 처리
* 초안이나 기밀 문서에 워터마크 삽입
* 계약서나 합의서에 디지털 서명 추가
* 프로그래밍 방식으로 PDF 양식(Form) 자동 작성

## 관련 링크

* [API 플레이그라운드](https://dashboard.nutrient.io/processor-api/playground/)
* [전체 API 문서](https://www.nutrient.io/guides/dws-processor/)
* [npm MCP 서버 패키지](https://www.npmjs.com/package/@nutrient-sdk/dws-mcp-server)
