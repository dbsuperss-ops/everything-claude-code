---
name: visa-doc-translate
description: 비자 신청 서류(이미지)를 영어로 번역하고, 원본 이미지와 번역문이 포함된 이중 언어 PDF를 생성합니다.
origin: ECC
---

# 비자 서류 번역 스킬 (Visa Doc Translate)

이 스킬은 비자 신청에 필요한 다양한 증명서(이미지 파일)를 자동으로 분석하고 영어로 번역하여 공식 제출용 PDF를 생성합니다.

## 적용 시점

사용자가 비자 신청용 서류 이미지(PNG, JPG, HEIC 등) 경로를 제공했을 때 다음 단계가 **자동으로** 진행됩니다:

1. **이미지 변환 및 보정**:
   * HEIC 파일은 PNG로 자동 변환합니다.
   * EXIF 데이터를 분석하여 이미지를 올바른 방향으로 자동 회전시킵니다.
2. **OCR 텍스트 추출**:
   * macOS Vision, EasyOCR, Tesseract 등을 활용하여 이미지 내 텍스트를 정확히 추출합니다.
   * 서류 유형(예: 예금잔액증명서, 재직증명서, 경력증명서 등)을 자동 식별합니다.
3. **전문 영문 번역**:
   * 비자 신청용 전문 용어를 사용하여 모든 내용을 영어로 번역합니다.
   * 원본 서류의 구조와 형식을 최대한 유지합니다.
   * 고유 명사는 원어와 영어를 병기합니다 (예: 홍길동(HONG Gildong)).
   * 숫자, 날짜, 금액 정보를 정확하게 보존합니다.
4. **PDF 보고서 생성**:
   * **1페이지**: 회전 보정된 원본 이미지 배치 (A4 사이즈에 맞게 크기 조정)
   * **2페이지**: 전문적인 영문 번역본 배치 (가독성 높은 레이아웃)
   * 하단에 인증 문구 추가: "This is a certified English translation of the original document"
5. **결과물 저장**: 동일 디렉터리에 `<파일명>_Translated.pdf` 파일을 생성합니다.

## 지원 서류 목록

* 예금잔액증명서 (Deposit Certificate)
* 소득금액증명원 (Income Certificate)
* 재직증명서 / 경력증명서 (Employment Certificate / Experience Certificate)
* 퇴직증명서 (Retirement Certificate)
* 부동산 등기사항전부증명서 (Real Estate Registration)
* 사업자등록증 (Business License)
* 주민등록증 / 여권 복사본
* 기타 공식 문서

## 기술 사양 및 요구 사항

* **필수 라이브러리**: `pillow`, `reportlab`
* **OCR 도구**: `easyocr` (권장), `pytesseract`
* **macOS 전용**: `pyobjc-framework-Vision` (최고성능 OCR)

**핵심**: 비자 신청용 서류 번역은 정확성이 생명입니다. 모든 숫자와 날짜가 원본과 일치하는지 확인하십시오. 사용자 승인 과정을 최소화하여 파일 경로 제공만으로 최종 결과물(PDF)을 받아볼 수 있게 설계되었습니다.
