## digest-pull-request.yml

Описаниt

Workflow ищет открытые пулл реквесты, созданные за последние N дней (по умолчанию 7), и отправляет их список в Telegram. Если новых PR за период нет, ничего не отправляется.

Как работает
- Является reusable workflow и вызывается через `workflow_call` из другого workflow.
- Принимает параметры:
  - `days` — глубина поиска в днях;
  - `repositories` — JSON-массив репозиториев вида `[{ "owner": "...", "repo": "..." }]`.
- Читает список открытых PR для каждого репозитория через `github.rest.pulls.list` с `state: 'open'` и фильтрует по дате создания `created_at >= sinceDate`.
- Сортирует PR по дате создания от новых к старым.
- Собирает Markdown-текст со строками вида: `owner/repo#number`, заголовок, автор, сколько дней назад создан, ссылка на PR.
- Отправляет сообщение в Telegram тем же способом, что и `digest-new-features.yml` (Bot API, `sendMessage`).
- Использует секреты `TELEGRAM_CHAT_ID` и `TELEGRAM_BOT_TOKEN`, прокинутые из вызывающего workflow.


## digest-new-features.yml

Описание

Workflow собирает за заданный период все смерженные пулл реквесты и отправляет короткий дайджест в Telegram-чат. Если за период не было ни одного merge, сообщение не отправляется.

Как работает
- Запускается по расписанию и/или вручную.
- Использует `actions/github-script@v7`.
- Через GitHub REST API (`github.rest.pulls.list`) получает список закрытых PR и фильтрует их по полю `merged_at` за последние N дней (по умолчанию 1 день).
- Формирует простой Markdown-текст: репозиторий, номер PR, заголовок, автор, когда был смержен, ссылка на PR.
- Делает POST-запрос к `https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/sendMessage` с этим текстом.
- Использует секреты `TELEGRAM_CHAT_ID` и `TELEGRAM_BOT_TOKEN`.
