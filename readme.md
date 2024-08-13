# 1. Railway 을 활용한 프로젝트 진행 기본 실습

Railway는 웹 애플리케이션을 쉽게 배포하고 관리할 수 있는 플랫폼으로, 서버리스 함수와 데이터베이스 관리를 지원합니다. 여기에 대한 자세한 설명과 예제 코드를 제공하겠습니다. 이 가이드는 Railway에서 애플리케이션을 생성하고, 개발하며, 테스트하고 배포하는 과정을 상세히 설명합니다. 또한, 서버리스 함수와 PostgreSQL 데이터베이스와의 연동 방법도 포함됩니다.

<br>

## 1-1. Railway 가입 및 프로젝트 생성

### 1-1-1. Railway 계정 생성

Railway의 웹사이트 (https://railway.app)로 이동하여 계정을 생성합니다.

<br><br>

### 1-1-2. 새 프로젝트 생성

1. 로그인 후 대시보드에서 "New Project" 버튼을 클릭합니다.
2. 원하는 템플릿을 선택하거나 빈 프로젝트를 생성할 수 있습니다.

<br><br><br>

## 1-2. 프로젝트 설정 및 파일 트리 구조

여기서는 Node.js를 예로 들어 설명하겠습니다. 프로젝트 설정에 따라 다른 프레임워크나 언어를 사용할 수도 있습니다.

<br>

### 1-2-1. 파일 트리 구조

```lua
my-railway-app/
│
├── src/
│   ├── index.js         # 서버 코드
│   ├── routes.js        # 라우팅 설정
│   └── db.js            # 데이터베이스 연결 설정
│
├── .env                 # 환경 변수 파일
├── package.json         # Node.js 패키지 관리 파일
└── railway.json         # Railway 설정 파일
```

<br><br><br>

## 1-3. Node.js 애플리케이션 설정

### 1-3-1. Node.js 프로젝트 초기화

```bash
mkdir my-railway-app
cd my-railway-app
npm init -y
```

<br><br>

### 1-3-2. 필요한 패키지 설치

```bash
npm install express pg dotenv
```

<br><br>

### 1-3-3. index.js 파일 작성

```js
// src/index.js
const express = require('express');
const app = express();
const port = process.env.PORT || 3000;

app.use(express.json());

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

<br><br>

### 1-3-4. db.js 파일 작성

```js
// src/db.js
const { Client } = require('pg');
require('dotenv').config();

const client = new Client({
  connectionString: process.env.DATABASE_URL,
  ssl: {
    rejectUnauthorized: false
  }
});

client.connect();
module.exports = client;
```

### 1-3-4. 환경 변수 설정 (.env 파일)

```env
PORT=3000
DATABASE_URL=your_postgres_database_url
```

<br><br>

### 1-3-5. package.json에 start 스크립트 추가

```json
"scripts": {
  "start": "node src/index.js"
}
```

<br><br><br>

## 1-4. Railway에서 배포

### 1-4-1. Railway CLI 설치

```bash
npm install -g railway
```

<br><br>

### 1-4-2. Railway 프로젝트에 로그인 및 초기화

```bash
railway login
railway init
```

<br><br>

### 1-4-3. Railway에 애플리케이션 배포

```bash
railway up
```

Railway CLI는 .env 파일을 자동으로 관리하며, railway up 명령으로 애플리케이션을 배포합니다.

<br><br><br>

## 1-5. 서버리스 함수 설정

### 1-5-1. 서버리스 함수 코드 작성

```js
// src/functions/hello.js
module.exports = async (req, res) => {
  res.json({ message: 'Hello from serverless function!' });
};
```

<br><br>

### 1-5-2. 함수 배포

Railway에서 서버리스 함수를 배포하려면 Railway Dashboard에서 Functions 메뉴를 통해 새 함수를 추가할 수 있습니다.

<br><br><br>

## 1-6. PostgreSQL 데이터베이스 연동

### 1-6-1. Railway에서 PostgreSQL 데이터베이스 추가

Railway Dashboard에서 "New Service"를 클릭하고 PostgreSQL을 선택하여 데이터베이스를 추가합니다.

<br><br>

### 1-6-2. 데이터베이스 URL 확인

Railway Dashboard의 PostgreSQL 서비스에서 연결 문자열을 확인하고 .env 파일의 DATABASE_URL 값으로 사용합니다.

<br><br>

### 1-6-3. 데이터베이스 쿼리 테스트

애플리케이션이 실행 중인 상태에서 데이터베이스 쿼리를 실행해보세요.

```js
// src/index.js에 추가
const client = require('./db');

app.get('/data', async (req, res) => {
  try {
    const result = await client.query('SELECT * FROM your_table');
    res.json(result.rows);
  } catch (err) {
    res.status(500).send('Error querying the database');
  }
});
```

<br><br><br>

## 1-7. 프로젝트 테스트

### 1-7-1. 로컬 테스트

```bash
npm start
```

<br><br>

### 1-7-2. 단위 테스트 작성 (선택사항)

jest나 mocha와 같은 테스트 프레임워크를 사용할 수 있습니다.

```bash
npm install --save-dev jest
```

<br><br>

### 1-7-3. package.json에 테스트 스크립트 추가

```json
"scripts": {
  "test": "jest"
}
```

<br><br>

### 1-7-4. 테스트 코드 작성

```js
// test/app.test.js
const request = require('supertest');
const app = require('../src/index');

test('GET / should return Hello World!', async () => {
  const response = await request(app).get('/');
  expect(response.statusCode).toBe(200);
  expect(response.text).toBe('Hello World!');
});
```

<br><br><br>

## 1-8. 프로젝트 관리 및 모니터링

Railway Dashboard에서 로그와 메트릭스를 모니터링할 수 있습니다.
애플리케이션 상태와 서버리스 함수의 실행 결과를 실시간으로 확인할 수 있습니다