# 비자 서류 번역기 (Visa Document Translator)

이미지 형태의 비자 신청 서류를 자동으로 분석하여 전문적인 영문 PDF로 번역해 주는 도구입니다.

## 주요 기능

* 🔄 **자동 OCR**: 여러 OCR 방식(macOS Vision, EasyOCR, Tesseract)을 순차적으로 시도하여 텍스트를 추출합니다.
* 📄 **이중 언어 PDF**: 원본 이미지(1p) + 전문 영문 번역본(2p)이 포함된 최종 결과물을 생성합니다.
* 🌍 **다국어 지원**: 한국어 및 기타 언어로 된 서류를 지원합니다.
* 📋 **전문 양식**: 공식 비자 신청에 적합한 신뢰도 높은 레이아웃을 제공합니다.
* 🚀 **완전 자동화**: 파일 경로 입력만으로 모든 과정을 자동으로 완료합니다.

## 지원 서류 유형

* 예금잔액증명서 (Bank Statement)
* 재직증명서 (Employment Certificate)
* 퇴직증명서 (Retirement Certificate)
* 소득금액증명원 (Income Certificate)
* 부동산 등기사항전부증명서 (Real Estate Registration)
* 사업자등록증 (Business License)
* 주민등록증 및 여권

## 사용 방법

```bash
/visa-doc-translate <이미지-파일-경로>
```

### 실행 예시

```bash
/visa-doc-translate RetirementCertificate.PNG
/visa-doc-translate BankStatement.HEIC
/visa-doc-translate EmploymentLetter.jpg
```

## 결과물 구성

`<파일명>_Translated.pdf` 파일이 생성되며 다음과 같은 내용을 포함합니다:

* **1페이지**: 원본 서류 이미지 (A4 중앙 배치)
* **2페이지**: 전문 영문 번역본

## 기술 요구 사항 (Python 라이브러리)

* **이미지/PDF 처리**: `pillow`, `reportlab`
* **macOS 권장 (최고속도)**: `pyobjc-framework-Vision`, `pyobjc-framework-Quartz`
* **범용 OCR**: `easyocr` 또는 `pytesseract`

**핵심**: 호주, 미국, 캐나다, 영국 등 주요 국가의 비자 신청을 위한 서류 번역 및 공증 준비용으로 최적화되어 있습니다.
