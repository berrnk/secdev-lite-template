| Element | Data/Boundary | Threat (S/T/R/I/D/E) | Description | NFR link (ID) | Mitigation idea (ADR later) |
|--------|---------------|----------------------|-------------|---------------|-----------------------------|
| Edge: Internet → API | email/password [PII] | I | Утечка пароля в логах при неудачной аутентификации | NFR-007 | Маскирование PII в логах, RFC7807-ошибки |
| Edge: Internet → API | /api/auth/login | D | Брутфорс входа через частые запросы с одного IP | NFR-004 | Rate limit: 5 попыток/5 мин |
| Edge: Internet → API | /api/auth/login | E | Обход авторизации через IDOR (если user_id в теле) | NFR-005 | Проверка владения ресурсом, server-side authZ |
| Node: API / Controller | JWT | S | Подделка или повтор использования истёкшего JWT | NFR-003 | JWT с коротким TTL + refresh, проверка exp/iss |
| Node: API / Controller | Логи | I | PII (email) в логах при неудачном входе | NFR-006, NFR-007 | Структурные логи без паролей, маскирование email |
| Node: API / Controller | /login | R | Нет трассировки попыток входа (нет correlation_id) | NFR-006 | Добавить correlation_id, логировать user_id, IP |
| Node: Service → DB | SQL | T | SQL-инъекция при слабой валидации ввода | NFR-003 | Параметризованные запросы, ORM |
| Node: Service → DB | DB Credentials | I | Учётные данные БД в коде или .env | need NFR | Использовать secret store (Vault, GitHub Secrets) |
| Edge: Service → SMTP | verification link [PII] | I | Утечка email через ссылку в логах или мониторинге | NFR-007 | Не логировать полные ссылки, маскировать PII |
| Node: API / Controller | /login | E | Обход блокировки аккаунта через смену IP | NFR-005 | Блокировка по user_id + IP (dual lockout) |
