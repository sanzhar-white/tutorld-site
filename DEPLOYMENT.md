# 🚀 SECURE DEPLOYMENT GUIDE - TUTORLD

Этот документ содержит инструкции по безопасному развертыванию сайта Tutorld на различных платформах.

## 📋 Содержание

1. [GitHub Pages](#github-pages)
2. [Cloudflare Pages](#cloudflare-pages)
3. [Netlify](#netlify)
4. [Vercel](#vercel)
5. [Nginx (Self-hosted)](#nginx-self-hosted)
6. [Apache (Self-hosted)](#apache-self-hosted)
7. [Post-Deployment Checklist](#post-deployment-checklist)

---

## 🐙 GitHub Pages

### Преимущества:
- ✅ Бесплатный
- ✅ SSL из коробки
- ✅ Git-based deployment
- ❌ Ограниченная настройка headers

### Развертывание:

1. **Создайте репозиторий** (если еще не создан):
   ```bash
   git init
   git add .
   git commit -m "Initial commit with security hardening"
   git branch -M main
   git remote add origin https://github.com/yourusername/tutorld-site.git
   git push -u origin main
   ```

2. **Настройте GitHub Pages**:
   - Перейдите в Settings → Pages
   - Source: Deploy from a branch
   - Branch: main / (root)
   - Сохраните

3. **Настройте кастомный домен**:
   - Settings → Pages → Custom domain
   - Введите: tutorld.com
   - Отметьте: Enforce HTTPS

4. **Настройте DNS**:
   ```
   # A records
   185.199.108.153
   185.199.109.153
   185.199.110.153
   185.199.111.153

   # CNAME (для www)
   www.tutorld.com → yourusername.github.io
   ```

### ⚠️ Ограничения GitHub Pages:

- `_headers` файл не работает (нужны meta теги в HTML)
- `.htaccess` не поддерживается
- `nginx.conf` не используется

### Решение: Использовать meta теги

Добавьте в `<head>` каждого HTML файла:

```html
<!-- Security Headers -->
<meta http-equiv="Content-Security-Policy" content="default-src 'none'; style-src 'unsafe-inline' 'self'; img-src 'self' data:;">
<meta http-equiv="X-Content-Type-Options" content="nosniff">
<meta http-equiv="X-Frame-Options" content="DENY">
<meta name="referrer" content="strict-origin-when-cross-origin">
```

### Улучшение через Cloudflare:

1. Используйте Cloudflare перед GitHub Pages
2. Настройте Page Rules для headers
3. Включите WAF

---

## ☁️ Cloudflare Pages

### Преимущества:
- ✅ Бесплатный SSL
- ✅ Глобальный CDN
- ✅ Поддержка _headers
- ✅ Встроенный WAF
- ✅ DDoS protection

### Развертывание:

1. **Подключите Git репозиторий**:
   - Перейдите на dash.cloudflare.com
   - Pages → Create a project
   - Подключите GitHub/GitLab
   - Выберите tutorld-site репозиторий

2. **Настройте build**:
   ```
   Build command: (оставить пустым)
   Build output directory: /
   Root directory: /
   ```

3. **Настройте кастомный домен**:
   - Pages → Custom domains
   - Добавьте tutorld.com
   - Cloudflare автоматически настроит SSL

4. **Файл `_headers` автоматически применится!**

### Дополнительная настройка:

5. **Включите WAF**:
   - Security → WAF → Managed rules → Enable

6. **Настройте Page Rules**:
   - Rules → Page Rules → Create Page Rule
   - URL: `https://tutorld.com/*`
   - Settings:
     - Always Use HTTPS: On
     - SSL: Full (strict)
     - Security Level: High

7. **CAA DNS Records**:
   ```
   tutorld.com CAA 0 issue "letsencrypt.org"
   tutorld.com CAA 0 issuewild ";"
   tutorld.com CAA 0 iodef "mailto:security@tutorld.com"
   ```

---

## 🌐 Netlify

### Преимущества:
- ✅ Бесплатный SSL
- ✅ Поддержка _headers
- ✅ Git-based deployment
- ✅ Automatic HTTPS

### Развертывание:

1. **Создайте сайт**:
   - app.netlify.com → New site from Git
   - Подключите репозиторий
   - Deploy settings:
     ```
     Build command: (пусто)
     Publish directory: /
     ```

2. **Настройте домен**:
   - Site settings → Domain management
   - Add custom domain: tutorld.com
   - Netlify автоматически выдаст SSL

3. **Файл `_headers` автоматически применится!**

### Netlify-специфичные настройки:

4. **Создайте `netlify.toml`** (опционально):
   ```toml
   [build]
     publish = "/"

   [[headers]]
     for = "/*"
     [headers.values]
       X-Frame-Options = "DENY"
       X-Content-Type-Options = "nosniff"
       Referrer-Policy = "strict-origin-when-cross-origin"
       Permissions-Policy = "camera=(), microphone=(), geolocation=()"

   [[redirects]]
     from = "http://tutorld.com/*"
     to = "https://tutorld.com/:splat"
     status = 301
     force = true

   [[redirects]]
     from = "https://www.tutorld.com/*"
     to = "https://tutorld.com/:splat"
     status = 301
     force = true
   ```

---

## ▲ Vercel

### Преимущества:
- ✅ Бесплатный SSL
- ✅ Глобальный CDN
- ✅ Git-based deployment

### Развертывание:

1. **Создайте проект**:
   - vercel.com → New Project
   - Import Git Repository
   - Framework Preset: Other

2. **Настройте домен**:
   - Settings → Domains
   - Добавьте tutorld.com

3. **Создайте `vercel.json`**:
   ```json
   {
     "headers": [
       {
         "source": "/(.*)",
         "headers": [
           {
             "key": "X-Frame-Options",
             "value": "DENY"
           },
           {
             "key": "X-Content-Type-Options",
             "value": "nosniff"
           },
           {
             "key": "Strict-Transport-Security",
             "value": "max-age=63072000; includeSubDomains; preload"
           },
           {
             "key": "Referrer-Policy",
             "value": "strict-origin-when-cross-origin"
           },
           {
             "key": "Content-Security-Policy",
             "value": "default-src 'none'; style-src 'unsafe-inline' 'self'; img-src 'self' data:; font-src 'self'; frame-ancestors 'none'; base-uri 'self'; upgrade-insecure-requests"
           },
           {
             "key": "Permissions-Policy",
             "value": "camera=(), microphone=(), geolocation=()"
           }
         ]
       }
     ]
   }
   ```

---

## 🔧 Nginx (Self-hosted)

### Требования:
- Ubuntu/Debian server
- Root доступ
- Домен с DNS настройками

### Установка и настройка:

1. **Установите Nginx**:
   ```bash
   sudo apt update
   sudo apt install nginx
   ```

2. **Установите Certbot для SSL**:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   ```

3. **Скопируйте файлы сайта**:
   ```bash
   sudo mkdir -p /var/www/tutorld.com
   sudo chown -R $USER:$USER /var/www/tutorld.com

   # Загрузите файлы (через git, scp, или rsync)
   git clone https://github.com/yourusername/tutorld-site.git /var/www/tutorld.com
   ```

4. **Настройте Nginx**:
   ```bash
   # Скопируйте конфигурацию
   sudo cp /var/www/tutorld.com/nginx.conf /etc/nginx/sites-available/tutorld.com

   # Отредактируйте пути в конфигурации
   sudo nano /etc/nginx/sites-available/tutorld.com

   # Создайте symlink
   sudo ln -s /etc/nginx/sites-available/tutorld.com /etc/nginx/sites-enabled/

   # Удалите default конфигурацию
   sudo rm /etc/nginx/sites-enabled/default

   # Проверьте конфигурацию
   sudo nginx -t

   # Перезапустите Nginx
   sudo systemctl restart nginx
   ```

5. **Получите SSL сертификат**:
   ```bash
   sudo certbot --nginx -d tutorld.com -d www.tutorld.com
   ```

6. **Настройте автообновление сертификата**:
   ```bash
   # Certbot автоматически создаст cron job
   sudo certbot renew --dry-run
   ```

### Дополнительная защита:

7. **Установите Fail2ban**:
   ```bash
   sudo apt install fail2ban

   # Настройте правила
   sudo nano /etc/fail2ban/jail.local
   ```

   ```ini
   [DEFAULT]
   bantime = 3600
   findtime = 600
   maxretry = 5

   [nginx-http-auth]
   enabled = true

   [nginx-botsearch]
   enabled = true

   [nginx-badbots]
   enabled = true
   ```

8. **Настройте UFW firewall**:
   ```bash
   sudo ufw allow 'Nginx Full'
   sudo ufw allow OpenSSH
   sudo ufw enable
   ```

9. **Регулярные обновления**:
   ```bash
   # Создайте скрипт для автообновления
   sudo nano /etc/cron.weekly/tutorld-update
   ```

   ```bash
   #!/bin/bash
   cd /var/www/tutorld.com
   git pull origin main
   sudo systemctl reload nginx
   ```

   ```bash
   sudo chmod +x /etc/cron.weekly/tutorld-update
   ```

---

## 🔨 Apache (Self-hosted)

### Требования:
- Ubuntu/Debian server
- Root доступ
- Домен с DNS настройками

### Установка и настройка:

1. **Установите Apache**:
   ```bash
   sudo apt update
   sudo apt install apache2
   ```

2. **Включите необходимые модули**:
   ```bash
   sudo a2enmod ssl
   sudo a2enmod rewrite
   sudo a2enmod headers
   sudo a2enmod deflate
   sudo a2enmod expires
   ```

3. **Скопируйте файлы сайта**:
   ```bash
   sudo mkdir -p /var/www/tutorld.com/public_html
   sudo chown -R $USER:$USER /var/www/tutorld.com

   git clone https://github.com/yourusername/tutorld-site.git /var/www/tutorld.com/public_html
   ```

4. **Файл `.htaccess` уже на месте!**

5. **Создайте Virtual Host**:
   ```bash
   sudo nano /etc/apache2/sites-available/tutorld.com.conf
   ```

   ```apache
   <VirtualHost *:80>
       ServerName tutorld.com
       ServerAlias www.tutorld.com
       DocumentRoot /var/www/tutorld.com/public_html

       ErrorLog ${APACHE_LOG_DIR}/tutorld.com-error.log
       CustomLog ${APACHE_LOG_DIR}/tutorld.com-access.log combined

       <Directory /var/www/tutorld.com/public_html>
           Options -Indexes +FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
   ```

6. **Включите сайт**:
   ```bash
   sudo a2ensite tutorld.com.conf
   sudo a2dissite 000-default.conf
   sudo apache2ctl configtest
   sudo systemctl restart apache2
   ```

7. **Установите Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-apache
   sudo certbot --apache -d tutorld.com -d www.tutorld.com
   ```

---

## ✅ Post-Deployment Checklist

### Сразу после деплоя:

- [ ] **HTTPS работает** → https://tutorld.com
- [ ] **WWW редирект работает** → www.tutorld.com → tutorld.com
- [ ] **HTTP редирект работает** → http://tutorld.com → https://tutorld.com

### Проверка безопасности:

- [ ] **SecurityHeaders.com** → https://securityheaders.com/?q=tutorld.com
  - Цель: A+ rating

- [ ] **SSL Labs** → https://www.ssllabs.com/ssltest/analyze.html?d=tutorld.com
  - Цель: A+ rating

- [ ] **Mozilla Observatory** → https://observatory.mozilla.org/analyze/tutorld.com
  - Цель: A+ rating

- [ ] **CSP Evaluator** → https://csp-evaluator.withgoogle.com/
  - Проверить CSP policy

### Проверка headers (curl):

```bash
curl -I https://tutorld.com
```

Должны быть:
- ✅ `Strict-Transport-Security`
- ✅ `Content-Security-Policy`
- ✅ `X-Frame-Options`
- ✅ `X-Content-Type-Options`
- ✅ `Referrer-Policy`

### Функциональные проверки:

- [ ] Все страницы загружаются:
  - [ ] / (главная)
  - [ ] /terms/ (условия)
  - [ ] /support/ (приватность)
  - [ ] /download/ (скачать)

- [ ] Error pages работают:
  - [ ] /404-test → показывает 404.html
  - [ ] /forbidden-test → показывает 403.html (если настроено)

- [ ] Security files доступны:
  - [ ] /.well-known/security.txt
  - [ ] /robots.txt
  - [ ] /humans.txt
  - [ ] /sitemap.xml

### DNS Проверки:

```bash
# A records
dig tutorld.com A

# AAAA records (если есть IPv6)
dig tutorld.com AAAA

# CAA records
dig tutorld.com CAA

# MX records (для email)
dig tutorld.com MX

# TXT records (SPF, DMARC)
dig tutorld.com TXT
```

### HSTS Preload:

- [ ] Добавить домен на https://hstspreload.org/
- [ ] Проверить статус через несколько дней

### Мониторинг:

- [ ] Настроить UptimeRobot или аналог
- [ ] Настроить Google Search Console
- [ ] Настроить Cloudflare Analytics (если используется)

---

## 🔄 Continuous Deployment

### GitHub Actions (для автодеплоя):

Создайте `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_HOST }}
        username: ${{ secrets.SERVER_USER }}
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /var/www/tutorld.com
          git pull origin main
          sudo systemctl reload nginx
```

---

## 🆘 Troubleshooting

### Headers не применяются (GitHub Pages):

**Решение**: Используйте meta теги в HTML или поставьте Cloudflare перед GitHub Pages

### SSL сертификат не обновляется:

```bash
# Проверьте статус certbot
sudo certbot certificates

# Принудительное обновление
sudo certbot renew --force-renewal
```

### 403 Forbidden на все файлы:

```bash
# Проверьте права доступа
sudo chown -R www-data:www-data /var/www/tutorld.com
sudo chmod -R 755 /var/www/tutorld.com
```

### CSP блокирует стили:

Проверьте, что CSP включает:
```
style-src 'unsafe-inline' 'self';
```

---

## 📞 Поддержка

**Вопросы по деплою**: support@tutorld.com

**Security вопросы**: security@tutorld.com

**Документация**: См. SECURITY.md

---

**Последнее обновление:** 14 февраля 2026
