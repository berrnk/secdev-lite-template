# S03 - Шаблон реестра NFR (Given-When-Then)

Этот файл - рабочий **реестр NFR** на семинаре.  
Заполняйте его у себя в репозитории в: `SEMINARS/S03/S03_register.md`.

---

## Поля реестра (data dictionary)

* **ID** - короткий идентификатор, например `NFR-001`.
* **User Story / Feature** - к какой истории/фиче относится требование (ссылка допустима).
* **Category** - выберите из банка (напр.: `Performance`, `Security-AuthZ/RBAC`, `RateLimiting`, `Privacy/PII`, `Observability/Logging`, …).
* **Requirement (NFR)** - *измеримое* требование (числа/пороги/границы действия).
* **Rationale / Risk** - зачем это нужно, какой риск/ценность покрываем.
* **Acceptance (G-W-T)** - проверяемая формулировка: *Given … When … Then …*.
* **Evidence (test/log/scan/policy)** - чем подтвердим выполнение (тест, лог-шаблон, сканер, политика).
* **Trace (issue/link)** - ссылка на задачу, обсуждение, артефакт.
* **Owner** - ответственный.
* **Status** - `Draft` | `Proposed` | `Approved` | `Implemented` | `Verified`.
* **Priority** - `P1 - High` | `P2 - Medium` | `P3 - Low`.
* **Severity** - `S1 - Critical` | `S2 - Major` | `S3 - Minor`.
* **Tags** - произвольные метки (через запятую).

---

## Таблица реестра

| ID      | User Story / Feature      | Category                 | Requirement (NFR)                                                   | Rationale / Risk                     | Acceptance (G-W-T)                                                                                                    | Evidence (test/log/scan/policy)               | Trace (issue/link) | Owner  | Status     | Priority      | Severity     | Tags                          |
|---------|---------------------------|--------------------------|---------------------------------------------------------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------------------|-----------------------------------------------|--------------------|--------|------------|---------------|--------------|-------------------------------|
| NFR-001 | As a user I upload avatar | Security-InputValidation | Reject payloads >1 MiB; MIME из allowlist; **extra** поля запрещены | Защита от DoS/грязных входных данных | **Given** тело 2 MiB и неизвестные поля<br>**When** POST `/api/files/avatar`<br>**Then** 413 с RFC7807 + запрет extra | test: `e2e-upload-limit`; policy: schema/size | #123               | team-a | Proposed   | P2 - Medium   | S2 - Major   | limits,validation               |
| NFR-002 | As a client I call /api/* | Performance              | P95 ≤ 300 ms при 50 RPS в течение 5 мин                             | UX/SLO                               | **Given** сервис здоров<br>**When** 50 RPS на `/api/*` 5 минут<br>**Then** P95 ≤ 300 ms; error rate ≤ 1%              | test: `load-50rps`; log: latency quantiles    | #124               | team-a | Draft      | P1 - High     | S2 - Major   | perf,slo                        |

> Продолжайте ниже добавлять строки до достижения **8-10 NFR** (или больше, если нужно):

| ID      | User Story / Feature                     | Category                     | Requirement (NFR)                                                                 | Rationale / Risk                                                                 | Acceptance (G-W-T)                                                                                                                              | Evidence (test/log/scan/policy)                              | Trace (issue/link) | Owner   | Status     | Priority      | Severity     | Tags                          |
|---------|------------------------------------------|------------------------------|------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------|--------------------|---------|------------|---------------|--------------|-------------------------------|
| NFR-003 | US-002 - Вход в систему (Login)          | Security-AuthN               | Аутентификация: пароль проверяется за ≤ 1 сек; используется bcrypt (cost=12)      | Защита от перебора и утечки хешей; соответствие best practices                    | **Given** валидный логин и пароль<br>**When** POST `/api/auth/login`<br>**Then** успешная аутентификация за ≤1s; пароль хешируется bcrypt cost=12 | test: `auth-bcrypt-integration`; policy: password hashing    | #203               | team-b  | Proposed   | P1 - High     | S1 - Critical | auth,security,hashing         |
| NFR-004 | US-002 - Вход в систему (Login)          | RateLimiting                 | Ограничение: макс. 5 попыток входа с IP за 5 минут; блокировка на 15 мин при превышении | Защита от брутфорса и DoS-атак на аутентификацию                                 | **Given** 5 неудачных попыток с одного IP<br>**When** 6-я попытка входа<br>**Then** 429 Too Many Requests + блокировка на 15 мин                   | test: `rate-limit-login`; policy: rate-limiting rules        | #204               | team-b  | Proposed   | P1 - High     | S1 - Critical | rate-limit,security,bruteforce |
| NFR-005 | US-002 - Вход в систему (Login)          | Security-AuthN               | При 10 неудачных попыток с учётной записи — блокировка аккаунта на 1 час           | Защита от targeted-атак на конкретные аккаунты                                   | **Given** 10 неудачных попыток подряд для одного логина<br>**When** новая попытка<br>**Then** 403 Forbidden + сообщение "Account locked"         | test: `account-lockout`; log: "user_locked"                 | #205               | team-b  | Proposed   | P1 - High     | S1 - Critical | auth,account-lock              |
| NFR-006 | US-002 - Вход в систему (Login)          | Observability/Logging        | Логировать все попытки входа: успех/неудача, IP, timestamp, user ID (если известен) | Аудит, расследование инцидентов, обнаружение атак                                 | **Given** попытка входа<br>**When** POST `/api/auth/login`<br>**Then** запись в лог с уровнем INFO/ERROR и полями: `ip`, `success`, `user_id?`   | log: `auth-attempt`; SIEM: correlation rules                 | #206               | team-b  | Proposed   | P2 - Medium   | S2 - Major    | logging,audit,observability    |
| NFR-007 | US-002 - Вход в систему (Login)          | Privacy/PII                  | Не логировать пароли, даже в debug/trace; маскировать в дампах и трейсах           | Защита персональных данных; соответствие GDPR/privacy-требованиям                 | **Given** ошибка валидации или логирование<br>**When** обработка запроса на вход<br>**Then** пароль отсутствует или замаскирован в логах/трейсах  | log-scan: `no-plaintext-password`; policy: PII masking       | #207               | team-b  | Proposed   | P1 - High     | S1 - Critical | privacy,pii,security           |
| NFR-008 | US-002 - Вход в систему (Login)          | Performance                  | P95 времени ответа ≤ 800 ms при нагрузке 20 R
