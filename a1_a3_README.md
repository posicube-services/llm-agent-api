# a1 ~ a3 API (단일 호출형) 가이드 문서

## 1. 개요
- 목적: 다양한 LLM(GPT, Claude, Gemini)을 단일 REST API로 제공
- 시스템명: Postech AI API
- 통신 방식: HTTPS + JSON

## 빠른 시작 (Quick Start)

아래 API를 통해 각 모델을 바로 호출할 수 있습니다.

- GPT: `POST https://genai.postech.ac.kr/agent/api/a1/gpt`
- Gemini: `POST https://genai.postech.ac.kr/agent/api/a2/gemini`
- Claude: `POST https://genai.postech.ac.kr/agent/api/a3/claude`

### 예시 (GPT)

```bash
curl -X POST 'https://genai.postech.ac.kr/agent/api/a1/gpt' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "안녕하세요"
  }'
```

## 2. 인증
### 2.1 인증 방식
- 방식: API Key 기반 인증
- 발급 위치: 서비스 웹 콘솔 `설정 > API Key 관리`
- 적용 대상: GPT / Claude / Gemini 전체
- 인증 위치: HTTP Header

### 2.2 헤더 규칙
#### GPT
```http
x-api-key: {YOUR_API_KEY}
Content-Type: application/json
```
- `Authorization` 헤더 사용하지 않음

#### Claude / Gemini
```http
x-api-key: {YOUR_API_KEY}
Authorization: {YOUR_API_KEY}
Content-Type: application/json
```
- `Authorization`에 `Bearer`를 붙이지 않음
- `x-api-key`와 `Authorization` 값은 동일해야 함

### 2.3 인증 실패 예시
#### 유효하지 않은 API Key
```json
{
  "code": "00012",
  "message": "Gateway authentication failed - args : INVALID_API_KEY",
  "data": null
}
```

#### x-api-key 누락 (예시)
```json
{
  "code": "20099",
  "message": "Gateway internal server Error - 404 NOT_FOUND \"No static resource agent/api/a1/gpt.\"",
  "data": null
}
```

#### Authorization 누락 (Gemini)
```json
{
  "message": "{\"detail\":\"Required headers missing\"}"
}
```

#### Authorization 누락 (Claude)
```json
{
  "message": "{\"detail\":\"Authorization header is required\"}"
}
```

## 3. 공통 Request / Response 스키마
### 3.1 Request Body
```json
{
  "message": "string",
  "stream": false,
  "files": [
    {
      "id": "string",
      "name": "string",
      "url": "string"
    }
  ]
}
```

### 3.2 Request 필드 설명
| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `message` | string | Y | 사용자 입력 메시지 |
| `stream` | boolean | N | 스트리밍 응답 여부 (기본값: `false`) |
| `files` | array | N | 질의에 사용될 파일 목록 |

#### files 객체
| 필드명 | 타입 | 필수 | 설명 |
|---|---|---|---|
| `id` | string | N | 파일 식별자 |
| `name` | string | N | 파일 이름 |
| `url` | string | N | 파일 접근 URL |

- `files`는 File Search / Code Interpreter Task에서만 사용

### 3.3 Response Body
```json
{
  "message": "string"
}
```

| 필드명 | 타입 | 설명 |
|---|---|---|
| `message` | string | LLM 응답 결과 |

## 4. 모델별 API Endpoint
### 4.1 GPT
- Endpoint: `POST https://genai.postech.ac.kr/agent/api/a1/gpt`
- Description: GPT 모델 기반 LLM 호출
- Headers:
  - `x-api-key: {API_KEY}`
  - `Content-Type: application/json`

### 4.2 Gemini
- Endpoint: `POST https://genai.postech.ac.kr/agent/api/a2/gemini`
- Description: Gemini 모델 기반 LLM 호출
- Headers:
  - `x-api-key: {API_KEY}`
  - `Authorization: {API_KEY}`
  - `Content-Type: application/json`

### 4.3 Claude
- Endpoint: `POST https://genai.postech.ac.kr/agent/api/a3/claude`
- Description: Claude 모델 기반 LLM 호출
- Headers:
  - `x-api-key: {API_KEY}`
  - `Authorization: {API_KEY}`
  - `Content-Type: application/json`

## 5. 지원 기능 매트릭스
| 모델 | 일반 질의 | Code Interpreter | File Search | Web Search | Image Generation |
|---|---|---|---|---|---|
| GPT | ✅ | ⚠ | ❌ | ✅ | ❌ |
| Claude | ✅ | ⚠ | ❌ | ✅ | ❌ |
| Gemini | ✅ | ⚠ | ❌ | ✅ | ✅ |

- ✅: 완전 지원
- ⚠: 제한적 지원
- ❌: 미지원

## 6. 기능별 상세
### 6.1 일반 질의 / Web Search
- 세 모델 모두 지원
- GPT는 응답에 URL 형태 `citation`이 포함될 수 있으나, 실제 접근 가능한 웹 URL이 아닐 수 있음
- Claude/Gemini는 citation 또는 출처 URL 정보가 노출되지 않음

### 6.2 Code Interpreter (제한적 지원)
- 코드 작성 가능
- 그래프/시각화 결과를 이미지 형태로 API 응답 반환 불가

### 6.3 File Search (미지원)
- 클라우드 스토리지 외부 URL 기반 파일은 File Search 대상 불가
- 현재 API에서 File Search 기능 미제공

### 6.4 Image Generation
- GPT / Claude: 이미지 생성 요청은 가능하나 결과 이미지를 API 응답으로 반환하지 않음
- Gemini: 이미지 생성 결과를 URL로 반환하며, URL은 일정 시간 후 만료

#### Gemini 이미지 생성 응답 예시
```json
{
  "message": "https://genai-minio.postech.ac.kr/postech/<image-object>?X-Amz-Expires=900&..."
}
```

## 7. 호출 예시
### 7.1 GPT 호출
```bash
curl -X POST 'https://genai.postech.ac.kr/agent/api/a1/gpt' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "안녕하세요",
    "stream": false
  }'
```

### 7.2 Gemini 호출
```bash
curl -X POST 'https://genai.postech.ac.kr/agent/api/a2/gemini' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Authorization: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "해당 주제로 이미지를 생성해줘",
    "stream": false
  }'
```
