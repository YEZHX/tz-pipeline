# Extella: подключение и работа с каноном

Нужен, когда продукт собирается на Extella или ходит в её API.

## Источник правды

Единственный источник — репозиторий [extella-agent-standards](https://github.com/AnvarBakiyev/extella-agent-standards). Публичный extella.ai/api.html неполон: рабочих путей в нём нет, и его молчание ничего не доказывает.

Читай только raw-ссылки, по одному файлу: страница GitHub и полный свод правил обрезаются при чтении. Правила не переписывай себе — читай их из первоисточника каждый раз: канон правится замерами, а копия молча устаревает.

| Что | Адрес |
|---|---|
| Вход: маршрут по задачам и ловушки | https://raw.githubusercontent.com/AnvarBakiyev/extella-agent-standards/main/en/AGENT_START.md |
| Оглавление правил | https://raw.githubusercontent.com/AnvarBakiyev/extella-agent-standards/main/en/rules/INDEX.md |
| Правило по номеру | https://raw.githubusercontent.com/AnvarBakiyev/extella-agent-standards/main/en/rules/H108.md (меняй номер) |
| Не работает — вход по симптому | https://raw.githubusercontent.com/AnvarBakiyev/extella-agent-standards/main/en/SYMPTOMS.md |
| Гайд по сборке агента (большой, читай по разделам) | https://raw.githubusercontent.com/AnvarBakiyev/extella-agent-standards/main/en/AGENT_BUILD_GUIDE.md |

Имена файлов с кириллицей (например, `H5-бис.md`) кодируй в адресе: `H5-%D0%B1%D0%B8%D1%81.md`.

## Перед сборкой

1. Прочитай `AGENT_START.md` и возьми из его таблицы маршрут для своей задачи.
2. Назови человеку номера правил для задачи и по строке о каждом — так видно, что правила прочитаны, а не угаданы.
3. Узнай число обязательных проверок: `python3 tools/stage_gates.py --stage build --json`. Число называет машина; в текст его не переписывай.

## Подключение: в начале каждой сессии

1. **Проверь связь.** Вызови `list_profiles` без аргументов у сервера Extella (обычно `extella_acct_<хеш>`). Ответ с данными аккаунта (`isError: false`, непустое содержимое) — связь есть. Список агентов запрашивай только после этого: на большом аккаунте он переполняет ответ. На старте сессии сервер может ещё подниматься — подожди несколько секунд или загрузи схему через поиск инструментов.
2. **Нет инструментов Extella или ошибка** — подключись скриптом канона `tools/connect_mcp.py` (команды ниже). Он ищет ключ (переменная `EXTELLA_API_TOKEN`, `~/.extella/api_token.txt`, `~/.extella/os_token.txt`), проверяет его вызовом `list_profiles`, ставит плагин-мост `extella-claude-bridge` и заводит связку его регистратором. Ключ в конфиг не попадает: заголовки отдаёт скрипт-помощник.
3. **Ключа нет — одно действие человека.** В Extella открыть приложение Extella Guide и нажать «Подключить Claude Code и Codex»: ключ ляжет в `~/.extella/api_token.txt` сам. Запасной путь — `Library → System → Tokens`, создать токен и сохранить в тот же файл. В облачной среде кнопка не достанет: там создают отдельный токен и задают его в настройках окружения как `EXTELLA_API_TOKEN`.
4. **Открой новую сессию и повтори шаг 1.** Связь доказывает только вызов `list_profiles` из самого клиента. Связка ставится один раз на машину и дальше поднимается в каждой сессии сама.

## Команды

macOS и Linux — Терминал:

```bash
# Claude Code CLI — только если `claude --version` не находит команду
curl -fsSL https://claude.ai/install.sh | bash
git clone --depth 1 https://github.com/AnvarBakiyev/extella-agent-standards.git ~/extella-agent-standards
cd ~/extella-agent-standards
python3 tools/connect_mcp.py
```

Windows — PowerShell:

```powershell
# Claude Code CLI — только если `claude --version` не находит команду
irm https://claude.ai/install.ps1 | iex
git clone --depth 1 https://github.com/AnvarBakiyev/extella-agent-standards.git $HOME\extella-agent-standards
cd $HOME\extella-agent-standards
# Мост вручную: версия скрипта от 22.10.2026 падает на Windows с WinError 2 —
# subprocess не находит claude.cmd по короткому имени (замер 26.10.2026)
claude plugin marketplace add AnvarBakiyev/extella-bridges
claude plugin install extella-claude-bridge@extella-claude
py -3 tools\connect_mcp.py
```

Если репозиторий уже склонирован — `git pull` в его папке вместо `git clone`. Если на машине есть Codex, скрипт пропишет ключ и в `~/.codex/config.toml` открытым текстом — скажи об этом человеку.

Проверка без изменений: `python3 tools/connect_mcp.py --проверить`. Состояние связок моста: `python3 ~/.claude/plugins/cache/extella-claude/extella-claude-bridge/<версия>/skills/extella-connect/scripts/extella_connect.py --json`.

## Заголовки

| Куда | Адрес | Заголовки |
|---|---|---|
| Ядро: агенты, эксперты, MCP | `api.extella.ai` | `X-Auth-Token`; для MCP ещё `X-Profile-Id` и `X-Agent-Id` |
| Магазин | `os.extella.ai` | `X-Extella-Token` |

Ключ один, различаются заголовки (H109). Без `X-Agent-Id` MCP отвечает «Failed to resolve dependency 'token'» и винит ключ, хотя ключ верен (H81).

## Чего не делать

- Не просить ключ в чат и не создавать токены за человека: ключ даёт полный доступ, вплоть до удаления агента.
- Не считать связью наличие файла, рукопожатие `initialize` или ответ `get_current_profile_and_agent`: файл переживает смену аккаунта, рукопожатие проходит с любой строкой вместо ключа, а этот вызов просто возвращает присланные заголовки.
- Не писать «откройте приложение — ключ появится сам»: это неверно, приложение ключ на диск не кладёт (H108, H110).
- Не заводить вторую связку вручную через `claude mcp add`, если мост уже стоит: будет двойник, а ключ ляжет открытым текстом в `~/.claude.json`.
- Не объявлять «платформа не умеет» без замера: вызов, заголовки, код ответа, дословная ошибка. Канон спорит только с замерами.
