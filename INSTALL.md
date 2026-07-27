# JK위드미 홈페이지 설치 가이드

이 문서는 **정적 프론트(HTTrack 캡처본) + PHP/MySQL 백엔드**를 실제 웹호스팅에
올려서 동작시키는 방법을 설명합니다. 국내 일반 웹호스팅(카페24·가비아·닷홈 등,
Apache+PHP+MySQL 환경) 기준입니다.

---

## 0. 준비물

- 웹호스팅 계정 (PHP 7.4+ / MySQL 5.7+ 또는 MariaDB 지원)
- 호스팅에서 발급받은 **MySQL DB 정보**: DB이름 / 아이디 / 비밀번호 / 호스트
- FTP 접속 정보 (파일 업로드용) 또는 호스팅 파일매니저
- 정적 사이트 폴더 (HTTrack 로 받은 `www.jk-withme.com` 폴더)
- 이 저장소의 `backend/`, `tools/` 폴더

---

## 1. 정적 사이트에 백엔드 연결 (통합 스크립트 실행)

정적 사이트의 모든 상담 폼을 백엔드로 연결하고, 절대주소를 정리합니다.
PHP가 설치된 PC(또는 호스팅 SSH)에서 실행하세요.

```bash
php tools/integrate.php  ./www.jk-withme.com  --rewrite-domain
```

- 상담 폼이 있는 모든 페이지에 `<script src="/backend/submit-endpoint.js">`가 삽입됩니다.
- `--rewrite-domain` : `https://www.jk-withme.com/...` 절대주소를 `/...` 상대주소로
  바꿔 새 도메인/로컬에서도 이미지·링크가 깨지지 않게 합니다. (권장)

> PC에 PHP가 없다면, 호스팅에 파일을 올린 뒤 SSH에서 실행해도 됩니다.

그런 다음 **정적 사이트 폴더 안에 `backend/` 폴더를 통째로 복사**해 넣습니다.
최종 업로드 구조는 아래와 같습니다.

```
(웹 루트)
├── index.html                ← 정적 사이트 파일들
├── nose/ , about/ , skin/ …  ← 정적 사이트 하위 페이지들
├── wp-content/ …             ← 정적 리소스(이미지·CSS·JS)
└── backend/                  ← 이 저장소의 backend 폴더
    ├── submit.php
    ├── submit-endpoint.js
    ├── db.php
    ├── config.php            ← (2단계에서 직접 생성)
    ├── schema.sql
    └── admin/
```

---

## 2. 데이터베이스 만들기

1. 호스팅 관리자에서 MySQL DB를 하나 생성합니다. (예: `jkwithme`)
2. **phpMyAdmin** 에 접속 → 만든 DB 선택 → **가져오기(Import)** 또는 **SQL** 탭에서
   `backend/schema.sql` 내용을 붙여넣고 실행합니다.
   - `inquiries`(문의) 와 `admin_users`(관리자) 테이블이 생성됩니다.
   - 기본 관리자 계정이 함께 만들어집니다.

   | 아이디 | 비밀번호 |
   |--------|----------|
   | `admin` | `jkwithme!2026` |

   > ⚠️ **로그인 후 반드시 비밀번호를 변경하세요.** (관리자 → 비밀번호 변경)

---

## 3. DB 접속 정보 입력 (config.php)

`backend/config.sample.php` 를 복사해서 `backend/config.php` 를 만들고,
호스팅에서 받은 DB 정보를 채웁니다.

```bash
cp backend/config.sample.php backend/config.php
```

```php
'db' => [
    'host' => 'localhost',   // 호스팅이 알려준 DB 호스트
    'port' => 3306,
    'name' => 'jkwithme',    // DB 이름
    'user' => '내DB아이디',
    'pass' => '내DB비밀번호',
    'charset' => 'utf8mb4',
],
```

> `config.php` 에는 비밀번호가 들어있으므로 외부에 노출되면 안 됩니다.
> 함께 들어있는 `backend/.htaccess` 가 이 파일의 직접 접근을 막아줍니다.

---

## 4. 업로드 & 확인

1. 전체 폴더(정적 사이트 + `backend/`)를 FTP로 웹 루트에 업로드합니다.
2. 브라우저에서 확인:
   - **홈페이지**: `https://내도메인/`
   - **관리자**: `https://내도메인/backend/admin/`  (admin / jkwithme!2026 로 로그인)
3. 홈페이지에서 상담신청 폼을 한 번 테스트로 제출 → 관리자 목록에 뜨는지 확인합니다.

---

## 5. 관리자 사용법

- **문의 목록**: 접수된 상담신청을 최신순으로 확인. 상태(신규/처리중/완료) 필터, 이름·연락처·부위 검색.
- **상세 보기**: 접수 내용 확인, 처리 상태 변경, 내부 메모 저장, 삭제.
- **비밀번호 변경**: 상단 메뉴에서.

---

## 자주 묻는 질문

**Q. 폼을 제출해도 관리자에 안 떠요.**
- `config.php` 의 DB 정보가 맞는지, `schema.sql` 을 실제로 실행했는지 확인하세요.
- 브라우저 개발자도구(F12) → 콘솔/네트워크에서 `/backend/submit.php` 응답을 확인하세요.

**Q. 이미지가 깨져요.**
- 1단계에서 `--rewrite-domain` 옵션을 넣었는지 확인하세요. 안 넣었다면 다시 실행하면 됩니다.

**Q. 관리자 주소를 바꾸고 싶어요.**
- `backend/admin` 폴더 이름을 바꾸면 접속 주소도 그에 맞춰 바뀝니다. (예: `backend/manage`)

**Q. 문의가 오면 이메일/문자로 알림 받을 수 있나요?**
- 현재는 DB 저장 + 관리자 조회까지 구현되어 있습니다. 이메일/문자 알림은 추가 개발로 붙일 수 있습니다.
