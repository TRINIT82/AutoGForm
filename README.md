# 🤖 AutoGForm — автоматизація обробки вхідних заявок

> Production-ready no-code/low-code воркфлоу, що замінює ручну обробку заявок повністю автоматизованим пайплайном: форма → класифікація → сповіщення → трекінг статусу.

![n8n](https://img.shields.io/badge/n8n-self--hosted-EA4B71?logo=n8n&logoColor=white)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-API-34A853?logo=googlesheets&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Bot_API-26A5E4?logo=telegram&logoColor=white)
![Status](https://img.shields.io/badge/status-working-brightgreen)

---

## 📌 Проблема

Компанія отримує заявки через Google Form. До автоматизації обробка виглядала так:

1. Хтось вручну перевіряє нові відповіді у формі
2. Копіює дані в таблицю чи месенджер
3. Визначає, чи заявка термінова
4. Сповіщає відповідальну людину

**Результат:** затримки 15–30 хвилин на заявку, ризик пропустити критичну заявку, нуль трекінгу — хто вже обробив запит, а хто ні.

## ✅ Рішення

Повністю автоматизований workflow на n8n, розгорнутий self-hosted на Raspberry Pi 5 (Docker + Traefik + Cloudflare Tunnel):

- **Реакція на нову заявку** — за секунди після заповнення форми, без ручного втручання
- **Автоматична класифікація** — критичні заявки візуально відрізняються від звичайних ще на етапі сповіщення
- **Миттєве сповіщення** відповідальної людини в Telegram з усіма деталями заявки
- **Трекінг статусу** прямо в Google Sheets — видно, яка заявка вже опрацьована
- **Error handling** — окремий workflow, що моніторить збої і сповіщає адміна з деталями помилки (execution error, workflow name, retry count)

## 📊 Бізнес-результат

| Метрика | До | Після |
|---|---|---|
| Час реакції на заявку | 15–30 хв | < 1 хв |
| Пропущені критичні заявки | можливі | 0 |
| Трекінг статусу обробки | відсутній | автоматичний |
| Ручна робота на заявку | ~3–5 хв | 0 хв |

## 🏗️ Архітектура

```
Google Form (нова відповідь)
        ↓
Google Sheets Trigger (n8n)
        ↓
   IF: critical? = Yes
    ↓              ↓
Telegram      Telegram
(🔴 critical) (✅ звичайна)
    ↓              ↓
        Update Status → Google Sheets
        
[паралельно] Error Trigger workflow →
      моніторить всі помилки → Telegram-алерт адміну
```

## 🛠️ Стек

- **Оркестрація:** [n8n](https://n8n.io/) (self-hosted на Raspberry Pi 5)
- **Джерело даних:** Google Forms + Google Sheets API (OAuth2)
- **Сповіщення:** Telegram Bot API
- **Інфраструктура:** Docker, Traefik (reverse proxy), Cloudflare Tunnel

## 📁 Файли в репозиторії

| Файл | Опис |
|---|---|
| `AutoGForm.json` | Основний workflow: тригер → класифікація → сповіщення → оновлення статусу |
| `AutoGForm_Error.json` | Error-handling workflow: перехоплює будь-які збої основного workflow і сповіщає адміна |

## 🚀 Як розгорнути у себе

1. Імпортуй обидва `.json`-файли у свій n8n-інстанс (Workflows → Import from File)
2. Створи Google Form з полями: `name`, `email`, `contacts`, `short Message`, `critical ?` (обов'язкові поля)
3. Підключи форму до Google Sheets, додай колонку `Status`
4. Налаштуй OAuth2-креденшали для Google Sheets API в n8n
5. Створи Telegram-бота через [@BotFather](https://t.me/BotFather), встав токен у credentials
6. У нодах Telegram встав свій `chatId` (дізнатись можна через [@userinfobot](https://t.me/userinfobot))
7. У Google Sheets Trigger та Update-ноді вкажи свій `documentId`
8. В налаштуваннях основного workflow (Settings → Error Workflow) підв'яжи `AutoGForm_Error`
9. Активуй обидва workflow

**Автор:** [TRINIT / trinit82](https://github.com/trinit82) · 📧 TRINTIS@protonmail.com · [raydev.me](https://raydev.me)
