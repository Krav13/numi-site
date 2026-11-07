# Сайт Numi

Статичный сайт приложения Numi (iPhone, учёт финансов) на GitHub Pages: HTML + CSS, без сборщиков и внешних скриптов, системные шрифты.
Сейчас опубликован по адресу https://krav13.github.io/numi-site/, после покупки домена переедет на https://numi.space/.

## Страницы

| Файл | Что это | Где используется |
|---|---|---|
| `index.html` | Лендинг | — |
| `privacy.html` | Политика конфиденциальности | URL вписан в App Store Connect, **путь менять нельзя** |
| `support.html` | Поддержка (через GitHub Issues) | URL вписан в App Store Connect, **путь менять нельзя** |
| `style.css` | Общие стили, светлая и тёмная тема по настройке системы | все страницы |
| `icon.png`, `favicon.png`, `apple-touch-icon.png` | Иконка приложения (1024, 64, 180 px) | `<head>` всех страниц |
| `img/` | Скриншоты в WebP для страниц, `og.png` для превью ссылок | `index.html` |
| `screenshots/`, `screenshots63/` | Оригиналы скриншотов App Store (1320×2868 и 1206×2622) | в вёрстке не используются, лежат для истории |
| `robots.txt`, `sitemap.xml` | Для поисковиков | — |
| `.nojekyll` | Отключает Jekyll на GitHub Pages | — |

Все ссылки в HTML относительные (без ведущего `/`), поэтому сайт одинаково работает и в подпапке `/numi-site/`, и в корне домена.

## Деплой

Публикуется ветка `main` целиком (GitHub Pages, legacy build). После `git push` сборка занимает около минуты:

```bash
gh api repos/Krav13/numi-site/pages/builds/latest --jq '{status,created_at,error}'
```

## Домен numi.space — что сделать после покупки

Домен ещё не куплен (на 2026-11-07 у `numi.space` нет NS-записей), поэтому `CNAME` в репозиторий **пока не добавлен**: как только GitHub Pages получает custom domain, он начинает отдавать редирект с `krav13.github.io/numi-site/` на этот домен, а пока домен не резолвится, перестали бы открываться `privacy.html` и `support.html`, которые проверяет Apple.

Порядок действий после покупки домена:

1. **DNS у регистратора.** Завести записи:

   | Тип | Имя | Значение |
   |---|---|---|
   | A | `@` | `185.199.108.153` |
   | A | `@` | `185.199.109.153` |
   | A | `@` | `185.199.110.153` |
   | A | `@` | `185.199.111.153` |
   | CNAME | `www` | `krav13.github.io` |

   По желанию ещё AAAA для `@`: `2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`.
   Проверка: `dig +short numi.space A` должен вернуть четыре адреса выше, `dig +short www.numi.space CNAME` — `krav13.github.io.`

2. **Привязать домен к Pages** (после того как DNS разошёлся, обычно 10–60 минут):

   ```bash
   echo numi.space > CNAME && git add CNAME && git commit -m "Custom domain numi.space" && git push
   ```

   Файл `CNAME` в корне репозитория — это и есть настройка custom domain для GitHub Pages; при каждой сборке она берётся из него. Либо то же через API:

   ```bash
   gh api -X PUT repos/Krav13/numi-site/pages -f cname=numi.space
   ```

3. **Включить HTTPS.** Сертификат GitHub выпускает сам через несколько минут после того, как увидит DNS. Потом:

   ```bash
   gh api -X PUT repos/Krav13/numi-site/pages -F https_enforced=true
   ```

   Если API ответит ошибкой (сертификат ещё не готов), включить галочку «Enforce HTTPS» в Settings → Pages репозитория чуть позже. Состояние:

   ```bash
   gh api repos/Krav13/numi-site/pages --jq '{cname,https_enforced,protected_domain_state}'
   ```

4. **Переключить адреса в файлах сайта** с `krav13.github.io/numi-site/` на `numi.space/` (canonical, Open Graph, sitemap, robots):

   ```bash
   sed -i '' 's#https://krav13.github.io/numi-site/#https://numi.space/#g' index.html privacy.html support.html robots.txt sitemap.xml && git commit -am "Site URLs: numi.space" && git push
   ```

5. Проверить: `curl -I https://numi.space/` → `200`, `curl -I https://krav13.github.io/numi-site/privacy.html` → `301` на `https://numi.space/privacy.html`. Старые URL политики и поддержки в App Store Connect продолжат работать через этот редирект, но лучше заменить их на `https://numi.space/privacy.html` и `https://numi.space/support.html` в следующей версии карточки.

## После публикации в App Store

Приложение (Apple ID 6826416374, версия 1.0, сборка 75) отправлено на проверку 2026-11-04. Кнопки «Скачать в App Store» уже ведут на `https://apps.apple.com/app/id6826416374`, но страница отдаёт 404 до публикации. Когда приложение выйдет:

- в `index.html` убрать строку с классом `review` («проходит проверку») и поправить текст в последнем блоке;
- раскомментировать `<meta name="apple-itunes-app" content="app-id=6826416374">` в `<head>` — Safari покажет плашку «Открыть в App Store»;
- места помечены в коде комментариями `APP_STORE`.

## Как обновить скриншоты

Оригиналы кладутся в `screenshots/`, для страниц делаются WebP того же имени в `img/` (например, `cwebp -q 82 screenshots/01_overview.png -o img/01_overview.webp`). Подписи к экранам берутся из `docs/appstore/metadata_ru.md` репозитория приложения.
