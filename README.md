## API 구조 안내

본 API는 URL 경로 기준으로 두 가지 타입으로 구분됩니다.

### a1 ~ a3 API (단일 호출형)
- 예: `https://genai.postech.ac.kr/agent/api/a3/claude`
- JSON 기반 단순 LLM 호출
- 멀티파트(form-data) 미지원

간단한 질의/응답 또는 빠른 연동이 필요한 경우 사용  
가이드 문서: [a1_a3_README.md](./a1_a3_README.md)

---

### a11 ~ a64 API (Agent/OpenAPI 기반)
- 예: `https://genai.postech.ac.kr/agent/api/a64/google/files/{name}`
- multipart/form-data 지원
- 파일 업로드 및 고급 기능 제공

본 README는 해당 API(a11 ~ a64)에 대한 가이드입니다.

---

## LLM Agent API

**OpenAI, Anthropic, Google**의 주요 개발자 API를 Agent OpenAPI 표준으로 통합 제공

- **OpenAPI 스펙**: `spec.json`
- **대상 엔드포인트**: `/openai/*`, `/anthropic/*`, `/google/*`

---

## 빠르게 시작하기

### 1) Base URL 이해하기

`spec.json`의 `servers.url`은 다음 형태입니다.

- `https://{domain}/agent/{agent_alias}/a{idx}`

예를 들어,

- `{domain}` = `api.example.com`
- `{agent_alias}` = `llms`
- `{idx}` = `27`

이면 Base URL은 아래와 같습니다.

- `https://api.example.com/agent/llms/a27`

각 API 호출은 **Base URL + path**로 구성됩니다.

---

### 2) 사용 가능한 모델

본 API는 멀티 LLM을 지원하며, 모델별 사용 가능 범위는 아래와 같습니다.

#### Claude (Anthropic)
- **공식 지원 모델 전체 사용 가능**

#### Gemini (Google)
- **공식 지원 모델 전체 사용 가능**

#### ⚠️ GPT (OpenAI)
- 일부 특수 모델만 사용 가능 (Response 지원 모델)

사용 가능 모델 목록:
```
gpt-4.1-mini-gs-2025-04-14
o3-deep-research-gs-2025-06-26
gpt-5.2-gs-2025-12-11
gpt-5.1-gs-2025-11-13
gpt-5-mini-gs-2025-08-07
gpt-5-gs-2025-08-07
```
> ⚠️ 위 목록 외 GPT 모델은 현재 사용이 제한됩니다.

---

## 문서(스펙) 보는 방법

### Swagger Editor로 보기

- **방법**: Swagger Editor(웹)에서 `spec.json` 내용을 붙여넣거나 파일로 로드
- **링크**: [Swagger Editor](https://editor.swagger.io/)

### Redoc(로컬 프리뷰)

아래는 로컬에서 `spec.json`을 미리보는 예시입니다.

```bash
npx @redocly/cli@latest preview-docs spec.json
```

---

## 인증/헤더

이 API는 모든 요청에 **`x-api-key` 헤더가 필수**입니다. (`spec.json`의 Security Schema 참고)

- **x-api-key (필수)**: `x-api-key: <YOUR_API_KEY>`
- **Authorization (선택)**: `Bearer <TOKEN>` 형태 등 (서버 구성에 따라 필요할 수 있음)
- **추적용 헤더 (선택)**: **x-user-id**, **x-request-id**, **x-request-traces** 등

README의 예제 코드는 `x-api-key`를 포함한 **최소 필수 형태**로 작성되어 있습니다.

---

## 공통 호출 예제 (Python / Node.js)

아래 예제들은 “**Base URL + path**” 형태로 호출하는 기본 패턴을 보여줍니다.

### Python (httpx)

```python
import httpx

BASE_URL = "https://{domain}/agent/a{idx}"
API_KEY = "YOUR_API_KEY"

url = BASE_URL + "/openai/responses"
payload = {
    # TODO: spec.json의 request schema에 맞게 채우세요
}

res = httpx.post(
    url,
    json=payload,
    headers={"x-api-key": API_KEY},
    timeout=60,
)
res.raise_for_status()
print(res.json())
```

### Node.js (node-fetch)

```js
import fetch from "node-fetch";

async function main() {
  const BASE_URL = "https://{domain}/agent/a{idx}";
  const API_KEY = "YOUR_API_KEY";

  const url = BASE_URL + "/openai/responses";
  const payload = {
    // TODO: spec.json의 request schema에 맞게 채우세요
  };

  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json", "x-api-key": API_KEY },
    body: JSON.stringify(payload),
  });

  if (!res.ok) throw new Error(res.status + " " + (await res.text()));
  console.log(await res.json());
}

main();
```

---

## 업로드(multipart/form-data) 예제

일부 API는 파일 업로드를 위해 `multipart/form-data`를 사용합니다. (예: `/anthropic/files`, `/google/files`, `/openai/uploads/{upload_id}/parts` 등)

### Python (httpx)

```python
import httpx

BASE_URL = "https://{domain}/agent/a{idx}"
API_KEY = "YOUR_API_KEY"

url = BASE_URL + "/anthropic/files"
with open("path/to/file.bin", "rb") as f:
    res = httpx.post(
        url,
        files={"file": f},
        headers={"x-api-key": API_KEY},
        timeout=60,
    )

res.raise_for_status()
print(res.json())
```

### Node.js (node-fetch + form-data)

`node-fetch`만으로는 multipart 바디 구성이 번거로워서 보통 `form-data`를 함께 사용합니다.

```js
import fetch from "node-fetch";
import FormData from "form-data";
import fs from "node:fs";

async function main() {
  const BASE_URL = "https://{domain}/agent/a{idx}";
  const API_KEY = "YOUR_API_KEY";
  const url = BASE_URL + "/anthropic/files";

  const form = new FormData();
  form.append("file", fs.createReadStream("path/to/file.bin"));

  const res = await fetch(url, {
    method: "POST",
    body: form,
    headers: { ...form.getHeaders(), "x-api-key": API_KEY },
  });

  if (!res.ok) throw new Error(res.status + " " + (await res.text()));
  console.log(await res.json());
}

main();
```

---

## 스트리밍(SSE) 관련

일부 엔드포인트는 `stream=true` 같은 옵션을 통해 스트리밍을 지원할 수 있습니다(예: `/openai/responses`).  
구체적인 스트리밍 포맷(JSON Lines, SSE 등)은 서버 구현에 따라 달라질 수 있으니, 해당 operation의 `description`에 추가된 예제/설명을 참고하세요.

---

## 참고

- `spec.json`의 각 API `description`에는 **Python(httpx)** / **Node.js(node-fetch)** 기반 요청 예제가 포함되어 있습니다.
- 공개 레포로 배포할 때는 **실제 도메인/토큰/키**가 포함되지 않도록 주의하세요.
