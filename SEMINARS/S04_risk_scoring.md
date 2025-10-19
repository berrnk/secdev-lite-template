| Risk ID | Source (DFD/Row) | Consolidated Description | Threat (S/T/R/I/D/E) | NFR link (ID) | L (1-5) | Rationale-L | I (1-5) | Rationale-I | Score (=L×I) | Decision (Top-5?) | ADR candidate |
|--------|------------------|--------------------------|----------------------|---------------|--------|-------------|--------|-------------|---------------|-------------------|---------------|
| R-001 | Edge: Internet → API | Брутфорс входа с одного IP | D | NFR-004 | 5 | Публичный эндпоинт, типовая атака, слабая детекция | 4 | Утечка учётных данных, компрометация аккаунтов | 20 | ✅ Top-1 | Rate Limit на /auth/* |
| R-002 | Node: API / Controller | Утечка PII (email/пароль) в логах | I | NFR-006, NFR-007 | 4 | Частые ошибки валидации, логи доступны | 5 | Массовая утечка PII, нарушение GDPR | 20 | ✅ Top-2 | Маскирование PII в логах |
| R-003 | Edge: Internet → API | Подделка JWT (spoofing) | S | NFR-003 | 4 | Известные уязвимости в JWT-библиотеках | 5 | Полный доступ к аккаунту без аутентификации | 20 | ✅ Top-3 | JWT TTL + Refresh + strict validation |
| R-004 | Node: API / Controller | Обход блокировки аккаунта через смену IP | E | NFR-005 | 3 | Требует знания логина, но реально | 4 | Целевые атаки на VIP-пользователей | 12 | ✅ Top-4 | Dual lockout (user + IP) |
| R-005 | Edge: Service → SMTP | Утечка email через verification link в логах | I | NFR-007 | 3 | Логи могут попасть в SIEM/мониторинг | 4 | PII в открытом виде, риск утечки | 12 | ✅ Top-5 | Redaction PII in external call logs |

### Сводка топ 5:
- **Top-1 (R-001):** Брутфорс входа → **ADR: Rate Limit на /auth/***  
- **Top-2 (R-002):** Утечка PII в логах → **ADR: Маскирование PII в логах**  
- **Top-3 (R-003):** Подделка JWT → **ADR: JWT TTL + Refresh + strict validation**  
- **Top-4 (R-004):** Обход блокировки аккаунта → **ADR: Dual lockout (user + IP)**  
- **Top-5 (R-005):** Утечка PII через SMTP → **ADR: Redaction PII in external call logs**
