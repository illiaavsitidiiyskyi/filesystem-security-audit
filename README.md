# filesystem-security-audit

 gpt вот наш  проект: Автоматический аудит безопасности файловой системы
Команда:
  - Илья — специалист по SQLite (Ubuntu)
  - Назар — специалист по кибербезопасности (Kali Linux)
Стек: Python, SQLite, watchdog, hashlib, psutil, Flask, smtplib, requests
============================
ЗАДАЧИ ИЛ (SQLite / Ubuntu)
============================
1. Спроектировать схему БД:
   - Таблица events (id, timestamp, event_type, file_path, old_hash, new_hash, severity)
   - Таблица alerts (id, event_id, reason, sent_at)
   - Таблица known_hashes (id, file_path, hash, last_checked)
2. Написать модуль database.py:
   - Функции insert_event(), get_events_by_severity(), get_recent_alerts()
   - Подключение через context manager (with sqlite3.connect(...))
   - Индексы на timestamp и severity для быстрых выборок
3. Написать модуль reporter.py:
   - Экспорт событий за период в CSV
   - Генерация HTML-отчёта по данным из БД
   - Запрос топ-10 подозрительных файлов за неделю
4. Flask-дашборд (dashboard/app.py):
   - Страница /events — таблица всех событий
   - Страница /alerts — только критические алерты
   - Фильтрация по дате и severity
=====================================
ЗАДАЧИ Н (Кибербезопасность / Kali)
=====================================
1. Написать модуль watcher.py:
   - Мониторинг заданной директории через watchdog
   - Обработка событий: on_created, on_modified, on_deleted
   - Передача события в database.py для логирования
2. Написать модуль detector.py:
   - Проверка на опасные расширения (.sh, .elf, .py с правами 777)
   - Детект SUID/SGID файлов через os.stat()
   - Сравнение хэша файла с таблицей known_hashes
   - Присвоение severity: info / warning / critical
3. Написать модуль virustotal.py:
   - Отправка SHA256-хэша файла в VirusTotal API v3
   - Парсинг ответа — сколько антивирусов детектят угрозу
   - Если detections > 0 → severity = critical → запись в alerts
4. Написать модуль mailer.py:
   - Отправка алерта на почту при critical-событии через smtplib
   - HTML-письмо с деталями: файл, тип угрозы, время, хэш
============================
ОБЩИЕ ФАЙЛЫ (оба участника)
============================
- config.py — пути для мониторинга, email, VirusTotal API-ключ
- main.py   — запуск watcher + scheduler для ежедневного отчёта
============================
КАК ВЗАИМОДЕЙСТВУЮТ МОДУЛИ
============================
watcher.py → detector.py → database.py → mailer.py
                                       → reporter.py → dashboard