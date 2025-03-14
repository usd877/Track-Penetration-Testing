# Отчет по тестированию на проникновение: OSINT и Сканирование цели

## 1. Введение
**Дата тестирования:** 1 марта 2025 года  
**Объект тестирования:** Веб-сервис, доступный по IP-адресу 92.51.39.106  
**Метод тестирования:** Black Box (OSINT и сканирование)  
**Используемые инструменты:** Google Dorking, Whois, Shodan, theHarvester, WhatWeb, Nikto, Nmap, Gobuster, SQLMap  

## 2. Сканирование хоста
В ходе тестирования был выполнен сканинг хоста с использованием Nmap для выявления открытых портов и определения версий сервисов. Хост 92.51.39.106 оказался доступен с минимальной задержкой (0.0017 с).

### Открытые порты:
- **Порт 22 (SSH):** Открыт, используется OpenSSH 8.2p1 (Ubuntu 4ubuntu0.12).

### Закрытые порты:
- 999 порта не ответили (фильтруются).

## 3. Этап 1: OSINT (Open Source Intelligence)

### 3.1. Google Dorking
Использованные запросы:
- site:92.51.39.106
- inurl:admin site:92.51.39.106
- filetype:log OR filetype:txt site:92.51.39.106

**Результаты:**
- Скрытые страницы: Не найдены.
- Конфиденциальные файлы: Не обнаружены.

### 3.2. Whois-запрос
**Результаты:**
- **Владелец:** Igor Gilmutdinov
- **Провайдер:** TimeWeb
- **Страна:** Россия (RU)
- **Контакты:** abuse@timeweb.ru
- **Телефон:** +7 812 2481081, +7 495 0331081

### 3.3. Shodan-анализ
**Результаты:**
- Данных не найдено.

### 3.4. Использование theHarvester
**Результаты:**
- Связанные домены:
  - http://92.51.39.106/
  - http://92.51.39.106:8060/

## 4. Этап 2: Сканирование и анализ уязвимостей

### 4.1. WhatWeb Scan

![whatweb](https://github.com/user-attachments/assets/8ea323cb-6f05-4a46-922c-e1ac7eae7a4e)


**Результаты:**
- HTTP-сервер: TornadoServer 5.1.1
- Используемые технологии: HTML5, jQuery, Lightbox, JavaScript
- Заголовок страницы: Beemer

### 4.2. Nikto Scan

![nikto](https://github.com/user-attachments/assets/16f18aa3-3ae3-450e-8a95-30bfd4390db2)


**Результаты:**
- Отсутствует заголовок X-Frame-Options (возможны clickjacking-атаки).
- Отсутствует заголовок X-Content-Type-Options.
- Обнаружена страница /login.html.

### 4.3. Nmap Scan

![nmap](https://github.com/user-attachments/assets/54a1ea9a-4fca-4731-9fc4-5059f1d84d00)


**Результаты:**
- Открыт порт 22 (SSH, OpenSSH 8.2p1).
- ОС: Linux 2.4.X (устаревшая версия).

### 4.4. Gobuster Scan

![gobuster](https://github.com/user-attachments/assets/cdc5a983-61be-45d5-9cd5-030d52cbe9ed)


**Результаты:**
- Доступны страницы: /search, /read, /index_html.
- Страница /upload возвращает ошибку 405 (Method Not Allowed).

### 4.5. SQLMap Scan

![sql 1](https://github.com/user-attachments/assets/2e7d8417-212a-4820-b487-b49dc19201d3)

![sql 2](https://github.com/user-attachments/assets/4b18c0d2-a954-4279-8eac-59e74907dcd8)

![sql 3](https://github.com/user-attachments/assets/ee6a93f4-a8f2-4da6-8dcc-a6e51d34ee55)


**Результаты:**
- URL http://92.51.39.106:7788/login вернул ошибку 404 (Not Found).

### 4.6. Ручное тестирование
#### SQL Injection (SQLi)

![логин и пароль SQL Injection (SQLi)](https://github.com/user-attachments/assets/2bc6d15a-a74e-4a32-b246-190d49582911)


- **URL:** http://92.51.39.106/login
- **Payload:** ' OR 1=1 --
- **Результат:** Успешный обход аутентификации.
- **Оценка:** Critical

#### XSS

![XSS уязвимости, при котором можно задействовать события JavaScript](https://github.com/user-attachments/assets/3318f7f9-1648-415b-bfb1-1fb4e3a8d192)

![Тестирование на XSS (Cross-Site Scripting) приложение уязвимо](https://github.com/user-attachments/assets/892e10fb-a018-4a0c-b48b-1a159f0e742d)


- **URL:** http://92.51.39.106/comments
- **Payload:** <script>alert('XSS')</script>
- **Результат:** Всплывающее окно с сообщением "XSS".
- **Оценка:** High

## 5. Автоматическое тестирование с OWASP ZAP

### 5.1. Запуск тестов

Использовался следующий скрипт для сканирования:

```python
import time
from zapv2 import ZAPv2

target_url = "http://92.51.39.106"
api_key = "your_api_key"

zap = ZAPv2(apikey=api_key)
zap.urlopen(target_url)

scan_id = zap.ascan.scan(target_url)

while int(zap.ascan.status(scan_id)) < 100:
    print(f"Прогресс сканирования: {zap.ascan.status(scan_id)}%")
    time.sleep(5)

print("Сканирование завершено.")
alerts = zap.core.alerts(baseurl=target_url)
print(f"Найдено уязвимостей: {len(alerts)}")
```

### 5.2. Результаты автотестов

Запуск сканирования для http://92.51.39.106
- Запуск активного сканирования...
- Прогресс сканирования: 20%
- Прогресс сканирования: 40%
- Прогресс сканирования: 60%
- Прогресс сканирования: 80%
- Сканирование завершено.
- Найдено уязвимостей: 0

### 5.3. Заключение по автотестам
В процессе автоматических тестов с использованием OWASP ZAP не было обнаружено уязвимостей в веб-приложении. Рекомендуется регулярно запускать сканирования с помощью ZAP для поддержания безопасности веб-сервиса.

## 6. Уязвимости

### nmap -sV --script vulners 92.51.39.106

Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-27 11:41 EST
Nmap scan report for 1427771-cg36175.tw1.ru (92.51.39.106)
Host is up (0.0017s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| vulners: 
|   cpe:/a:openbsd:openssh:8.2p1: 
|       2C119FFA-ECE0-5E14-A4A4-354A2C38071A    10.0    https://vulners.com/githubexploit/2C119FFA-ECE0-5E14-A4A4-354A2C38071A  *EXPLOIT*
|       CVE-2023-38408  9.8     https://vulners.com/cve/CVE-2023-38408
|       B8190CDB-3EB9-5631-9828-8064A1575B23    9.8     https://vulners.com/githubexploit/B8190CDB-3EB9-5631-9828-8064A1575B23  *EXPLOIT*
|       8FC9C5AB-3968-5F3C-825E-E8DB5379A623    9.8     https://vulners.com/githubexploit/8FC9C5AB-3968-5F3C-825E-E8DB5379A623  *EXPLOIT*
|       8AD01159-548E-546E-AA87-2DE89F3927EC    9.8     https://vulners.com/githubexploit/8AD01159-548E-546E-AA87-2DE89F3927EC  *EXPLOIT*
|       887EB570-27D3-11EE-ADBA-C80AA9043978    9.8     https://vulners.com/freebsd/887EB570-27D3-11EE-ADBA-C80AA9043978

### OpenSSH 8.2p1:
Версия OpenSSH 8.2p1 имеет несколько известных уязвимостей, связанных с реализацией протокола SSH и другими компонентами сервиса.

### Найденные уязвимости:
- **CVE-2023-38408 (оценка 9.8/10):**
  - Уязвимость позволяет атакующему удалённо выполнить произвольный код через SSH.
  - Подробнее о CVE: CVE-2023-38408 на Vulners.

### Эксплойты для OpenSSH 8.2p1:
- Эксплойт с идентификатором **2C119FFA-ECE0-5E14-A4A4-354A2C38071A** (оценка 10/10). Подробности на Vulners.
- Также были найдены другие эксплойты для OpenSSH версии 8.2p1, которые могут быть использованы для атак на сервер. Список эксплойтов доступен на Vulners.

### Рекомендации:
1. Обновить OpenSSH до последней версии, чтобы устранить уязвимости.
2. Ограничить доступ по SSH, например, с помощью использования ключей SSH вместо паролей.
3. Настроить межсетевой экран (firewall) и систему защиты от брутфорса, такую как Fail2Ban.

## 7. Заключение
Тестирование выявило критические уязвимости:
- **SQL-инъекция (обход аутентификации).**
- **XSS (внедрение JavaScript).**
- **Отсутствие защитных заголовков HTTP.**
- **Устаревшие версии ОС и сервисов.**
- **Серьёзные уязвимости в сервисе OpenSSH 8.2p1.**

Рекомендуется провести немедленное обновление и усиление безопасности сервера для защиты от потенциальных атак.
