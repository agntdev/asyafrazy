# AsyaPhrase Audio Publisher — Bot specification

**Archetype:** content

**Voice:** дружелюбный и заботливый — write every user-facing message, button label, error, and empty state in this voice.

Автоматически генерирует и публикует аудиозаписи с фразами в Telegram-канал, включая обязательные тексты о дружбе с Асей. Работает по расписанию с возможностью админ-управления.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- подписчики Telegram-каналов
- любители русской поэзии/фраз
- языковые студенты

## Success criteria

- Регулярные публикации (1 раз/6ч) с аудио и текстом
- Ежедневное появление обязательных фраз о Асе
- Работа админ-команд без ошибок

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Открыть главное меню бота
- **/schedule** (command, actor: owner, command: /schedule) — Показать/изменить расписание публикаций
- **/addphrase** (command, actor: owner, command: /addphrase) — Добавить новую фразу в библиотеку
- **/publishnow** (command, actor: owner, command: /publishnow) — Сразу сгенерировать и опубликовать пост

## Flows

### scheduled_publish
_Trigger:_ cron_schedule

1. Выбор фразы из библиотеки
2. Синтез аудио
3. Публикация в канал с текстом и аудио

_Data touched:_ Phrase, ScheduledJob

### admin_management
_Trigger:_ /schedule

1. Показать текущее расписание
2. Принять изменения от владельца
3. Обновить расписание

_Data touched:_ ScheduledJob

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **Phrase** _(retention: persistent)_ — Фраза для озвучки с метаданными
  - fields: text, theme, required_flag
- **ScheduledJob** _(retention: persistent)_ — Параметры расписания публикаций
  - fields: interval, last_run_time
- **ChannelPost** _(retention: persistent)_ — История публикаций
  - fields: post_text, audio_file_id, publish_time

## Integrations

- **Telegram** (required) — Публикация в канал и админ-управление
- **TTS_Service** (required) — Синтез аудио из текста
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- /schedule
- /addphrase
- /publishnow
- /setmandatory

## Notifications

- Уведомление владельцу при успешной публикации
- Предупреждение при сбое генерации аудио

## Permissions & privacy

- Доступ к каналу ограничен владельцем
- Аудио не содержит личных данных

## Edge cases

- Отсутствие доступа к каналу
- Ошибка TTS-сервиса
- Пустая библиотека фраз

## Required tests

- Проверка ежедневного появления фразы про Асу
- Тестирование команды /publishnow

## Assumptions

- Используется женский нейтральный голос для русского текста
- Минимальный интервал публикации 6 часов
