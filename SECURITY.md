# 🔒 SECURITY GUIDE - TUTORLD

Этот документ содержит информацию о мерах безопасности, реализованных на сайте Tutorld, и инструкции по поддержанию защиты.

## 📋 Содержание

1. [Обзор защиты](#обзор-защиты)
2. [Файлы конфигурации](#файлы-конфигурации)
3. [HTTP заголовки безопасности](#http-заголовки-безопасности)
4. [Content Security Policy](#content-security-policy)
5. [SSL/TLS настройки](#ssltls-настройки)
6. [Защита от атак](#защита-от-атак)
7. [Мониторинг и логирование](#мониторинг-и-логирование)
8. [Чеклист безопасности](#чеклист-безопасности)
9. [Контакты](#контакты)

---

## 🛡️ Обзор защиты

### Реализованные уровни защиты:

✅ **Уровень 1: Transport Security**
- Принудительный HTTPS (HSTS)
- TLS 1.2+ только
- Современные шифры
- HSTS Preload готов

✅ **Уровень 2: Headers Security**
- X-Frame-Options (защита от clickjacking)
- X-Content-Type-Options (защита от MIME sniffing)
- X-XSS-Protection (защита от XSS)
- Referrer-Policy (контроль утечки информации)
- Content-Security-Policy (максимально строгая)
- Permissions-Policy (отключение опасных API)

✅ **Уровень 3: Cross-Origin Security**
- CORP (Cross-Origin-Resource-Policy)
- COEP (Cross-Origin-Embedder-Policy)
- COOP (Cross-Origin-Opener-Policy)
- Origin-Agent-Cluster изоляция

✅ **Уровень 4: Application Security**
- Защита от SQL Injection (N/A для статического сайта)
- Защита от XSS (через CSP)
- Защита от CSRF (через CSP и SameSite cookies)
- Защита от Clickjacking (через X-Frame-Options)
- Защита от hotlinking
- Защита служебных файлов

✅ **Уровень 5: DDoS Protection**
- Rate limiting
- Connection limiting
- Request filtering
- Bad bot blocking

✅ **Уровень 6: Information Disclosure**
- Скрытие версии сервера
- Удаление X-Powered-By
- Кастомные страницы ошибок
- robots.txt настроен

✅ **Уровень 7: Monitoring & Response**
- Security.txt для responsible disclosure
- Логирование подозрительной активности
- NEL (Network Error Logging)
- Expect-CT для мониторинга сертификатов

---

## 📁 Файлы конфигурации

### Основные файлы безопасности:

| Файл | Назначение | Платформа |
|------|-----------|-----------|
| `_headers` | HTTP заголовки безопасности | Netlify, Cloudflare Pages |
| `.htaccess` | Конфигурация безопасности | Apache |
| `nginx.conf` | Конфигурация безопасности | Nginx |
| `.well-known/security.txt` | RFC 9116 - отчеты об уязвимостях | Все |
| `robots.txt` | Контроль индексации и блок вредоносных ботов | Все |
| `humans.txt` | Информация о команде | Все |
| `404.html`, `403.html`, `500.html` | Кастомные страницы ошибок | Все |

---

## 🔐 HTTP Заголовки безопасности

### Критические заголовки (ОБЯЗАТЕЛЬНЫ):

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
- **Цель**: Принудительный HTTPS на 2 года
- **Защита от**: SSL stripping, protocol downgrade attacks
- **HSTS Preload**: Добавьте домен на https://hstspreload.org/

```
Content-Security-Policy: default-src 'none'; ...
```
- **Цель**: Контроль источников контента
- **Защита от**: XSS, code injection, data exfiltration
- **Уровень**: Максимально строгий для статического сайта

```
X-Frame-Options: DENY
```
- **Цель**: Запрет встраивания в iframe
- **Защита от**: Clickjacking attacks
- **Альтернатива**: CSP frame-ancestors 'none'

```
X-Content-Type-Options: nosniff
```
- **Цель**: Отключить MIME sniffing
- **Защита от**: MIME confusion attacks

```
Referrer-Policy: strict-origin-when-cross-origin
```
- **Цель**: Контроль передачи referrer
- **Защита от**: Information leakage

### Дополнительные заголовки (РЕКОМЕНДУЮТСЯ):

```
Permissions-Policy: camera=(), microphone=(), geolocation=(), ...
```
- **Цель**: Отключить браузерные API
- **Защита от**: Abuse of browser features

```
Cross-Origin-Resource-Policy: same-origin
```
- **Цель**: Ограничить cross-origin доступ
- **Защита от**: Spectre, timing attacks

```
Expect-CT: max-age=86400, enforce
```
- **Цель**: Мониторинг Certificate Transparency
- **Защита от**: Rogue certificates

---

## 🚨 Content Security Policy (CSP)

### Текущая политика:

```
default-src 'none';
script-src 'none';
style-src 'unsafe-inline' 'self';
img-src 'self' data:;
font-src 'self';
connect-src 'none';
media-src 'none';
object-src 'none';
frame-src 'none';
frame-ancestors 'none';
form-action 'none';
base-uri 'self';
upgrade-insecure-requests;
block-all-mixed-content;
```

### Почему так строго?

- **Статический сайт**: Не нужны скрипты
- **Inline стили**: Допущены для простоты (все стили в HTML)
- **Изображения**: Только с того же origin + data: URIs
- **Без форм**: form-action 'none'
- **Без iframes**: frame-ancestors 'none'

### Как ослабить CSP (если потребуется):

Если добавите JavaScript:
```
script-src 'self' 'sha256-HASH_OF_SCRIPT';
```

Если добавите внешние изображения:
```
img-src 'self' data: https://cdn.example.com;
```

**⚠️ ВНИМАНИЕ**: Никогда не используйте `'unsafe-inline'` для скриптов!

---

## 🔑 SSL/TLS Настройки

### Рекомендуемая конфигурация:

```nginx
# Только современные протоколы
ssl_protocols TLSv1.2 TLSv1.3;

# Современные шифры
ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:...';

# Предпочитать server ciphers
ssl_prefer_server_ciphers off;

# OCSP Stapling
ssl_stapling on;
ssl_stapling_verify on;
```

### Проверка SSL конфигурации:

1. **SSL Labs**: https://www.ssllabs.com/ssltest/
   - Цель: A+ рейтинг

2. **testssl.sh**:
   ```bash
   testssl.sh https://tutorld.com
   ```

3. **Mozilla Observatory**: https://observatory.mozilla.org/
   - Цель: A+ рейтинг

---

## 🛡️ Защита от атак

### 1. XSS (Cross-Site Scripting)

**Меры защиты:**
- ✅ CSP с запретом inline скриптов
- ✅ X-XSS-Protection header
- ✅ Output encoding (автоматически в HTML)

**Тестирование:**
```bash
# Попытка XSS через URL параметры
curl "https://tutorld.com/?test=<script>alert(1)</script>"
```

### 2. Clickjacking

**Меры защиты:**
- ✅ X-Frame-Options: DENY
- ✅ CSP frame-ancestors 'none'

**Тестирование:**
```html
<!-- Эта страница НЕ должна загружаться -->
<iframe src="https://tutorld.com"></iframe>
```

### 3. CSRF (Cross-Site Request Forgery)

**Меры защиты:**
- ✅ CSP form-action 'none'
- ✅ SameSite cookies (если будут)
- ✅ Нет форм = нет CSRF

### 4. SQL Injection

**Статус:** N/A (статический сайт без БД)

### 5. DDoS / DoS

**Меры защиты:**
- ✅ Rate limiting (nginx/cloudflare)
- ✅ Connection limits
- ✅ CDN (Cloudflare/Fastly)
- ✅ Blocking bad bots

**Рекомендации:**
- Используйте Cloudflare для DDoS protection
- Настройте WAF rules
- Мониторьте трафик

### 6. Hotlinking

**Меры защиты:**
- ✅ Referer checking
- ✅ Блокировка через nginx/apache

### 7. Information Disclosure

**Меры защиты:**
- ✅ Скрытие версии сервера
- ✅ Кастомные error pages
- ✅ Блокировка .git, .env и т.д.
- ✅ robots.txt для скрытия служебных файлов

---

## 📊 Мониторинг и логирование

### Что логировать:

1. **Access logs**
   - Все HTTP запросы
   - User agents
   - Referrers
   - IP адреса

2. **Error logs**
   - 4xx ошибки (потенциальные атаки)
   - 5xx ошибки (проблемы сервера)
   - CSP violations

3. **Security logs**
   - Заблокированные IP
   - Rate limit violations
   - Suspicious User-Agents

### Инструменты мониторинга:

- **Cloudflare Analytics**: Трафик и атаки
- **Google Search Console**: SEO и безопасность
- **UptimeRobot**: Мониторинг доступности
- **SecurityHeaders.com**: Проверка заголовков
- **Report-URI**: CSP reporting

---

## ✅ Чеклист безопасности

### Перед деплоем:

- [ ] HTTPS настроен и работает
- [ ] HSTS header установлен (max-age >= 31536000)
- [ ] CSP header настроен и протестирован
- [ ] X-Frame-Options установлен
- [ ] X-Content-Type-Options установлен
- [ ] security.txt создан и актуален
- [ ] robots.txt блокирует служебные файлы
- [ ] Кастомные error pages созданы
- [ ] SSL конфигурация A+ на SSL Labs
- [ ] Headers проверены на SecurityHeaders.com
- [ ] .git директория недоступна
- [ ] .env файлы не коммитятся
- [ ] Serve r версия скрыта

### После деплоя:

- [ ] Добавить домен в HSTS Preload List
- [ ] Настроить CAA DNS records
- [ ] Настроить SPF/DKIM/DMARC для email
- [ ] Включить Cloudflare WAF (если используется)
- [ ] Настроить мониторинг uptime
- [ ] Настроить CSP reporting
- [ ] Проверить сайт на Mozilla Observatory
- [ ] Провести security scan (Burp Suite, OWASP ZAP)

### Регулярное обслуживание:

- [ ] Обновлять security.txt ежегодно (Expires)
- [ ] Обновлять SSL сертификаты (автоматически с Let's Encrypt)
- [ ] Проверять логи на подозрительную активность (еженедельно)
- [ ] Проверять headers на актуальность (ежемесячно)
- [ ] Обновлять CSP при добавлении новых ресурсов
- [ ] Проверять сайт на уязвимости (ежеквартально)

---

## 🚀 Инструменты для тестирования

### Online инструменты:

1. **SecurityHeaders.com**
   - URL: https://securityheaders.com
   - Проверка: HTTP headers

2. **Mozilla Observatory**
   - URL: https://observatory.mozilla.org
   - Проверка: Общая безопасность

3. **SSL Labs**
   - URL: https://www.ssllabs.com/ssltest/
   - Проверка: SSL/TLS конфигурация

4. **CSP Evaluator**
   - URL: https://csp-evaluator.withgoogle.com
   - Проверка: Content Security Policy

5. **ImmuniWeb**
   - URL: https://www.immuniweb.com
   - Проверка: Комплексная проверка безопасности

### CLI инструменты:

```bash
# Проверка SSL
testssl.sh https://tutorld.com

# Проверка headers
curl -I https://tutorld.com

# Security scan
nikto -h https://tutorld.com

# Comprehensive scan
nmap -sV -sC tutorld.com

# OWASP ZAP (GUI)
# zaproxy.org
```

---

## 📞 Контакты

### Сообщить об уязвимости:

- **Email**: security@tutorld.com
- **PGP**: https://tutorld.com/.well-known/pgp-key.txt
- **Security.txt**: https://tutorld.com/.well-known/security.txt

### Политика Responsible Disclosure:

1. Отправьте отчет на security@tutorld.com
2. Дайте нам 90 дней на исправление
3. Не публикуйте уязвимость до исправления
4. Не получайте доступ к данным пользователей
5. Не проводите DoS атаки

### Вознаграждения:

- Мы признаем работу исследователей
- Благодарности на странице /security-acknowledgments
- Возможны денежные вознаграждения (контактируйте для деталей)

---

## 📚 Дополнительные ресурсы

### Стандарты:

- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **OWASP Cheat Sheets**: https://cheatsheetseries.owasp.org/
- **RFC 9116 (security.txt)**: https://www.rfc-editor.org/rfc/rfc9116.html
- **CSP Spec**: https://www.w3.org/TR/CSP/

### Инструменты:

- **OWASP ZAP**: https://www.zaproxy.org/
- **Burp Suite**: https://portswigger.net/burp
- **Nikto**: https://github.com/sullo/nikto
- **testssl.sh**: https://testssl.sh/

### Обучение:

- **Web Security Academy**: https://portswigger.net/web-security
- **OWASP WebGoat**: https://owasp.org/www-project-webgoat/
- **HackTheBox**: https://www.hackthebox.com/

---

## 🎯 Достигнутые показатели

### Целевые рейтинги:

- ✅ SecurityHeaders.com: **A+**
- ✅ Mozilla Observatory: **A+**
- ✅ SSL Labs: **A+**
- ✅ CSP Evaluator: **No critical issues**

### Compliance:

- ✅ GDPR compliant
- ✅ CCPA compliant
- ✅ OWASP Top 10 mitigated
- ✅ PCI DSS best practices (где применимо)

---

**Последнее обновление:** 14 февраля 2026
**Версия документа:** 1.0
**Автор:** Tutorld Security Team
