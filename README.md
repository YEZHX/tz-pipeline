# ТЗ → готовый продукт

Плагин для Claude Code. Человек присылает ТЗ и получает готовое приложение или AI-агента.

Как работает скилл:

1. **Разбор ТЗ.** Отправляет одно короткое сообщение: понимание задачи с цифрой, вопрос «где работать» с рекомендацией (Extella OS, локально или отдельно), тип решения, до семи вопросов и просьбу прислать реальные примеры.
2. **Договорённость о «готово».** После ответов собирает спецификацию со списком «вне MVP», карточку агента и тест-кейсы, затем ждёт согласования.
3. **Сборка и запуск.** Ведёт архитектуру, прототип, сборку короткими циклами, проверку надёжности и безопасности, приёмку и запуск. Останавливается только перед публичными и необратимыми действиями.

## Установка

В терминале, один раз на машину:

```bash
claude plugin marketplace add YEZHX/tz-pipeline
claude plugin install tz-to-product@tz-pipeline
```

То же внутри сессии Claude Code: `/plugin marketplace add YEZHX/tz-pipeline`, затем `/plugin install tz-to-product@tz-pipeline`.

Репозиторий публичный: аккаунт GitHub и доступы для установки не нужны.

### Обновление

```bash
claude plugin update tz-to-product@tz-pipeline
```

Или включить автообновление: `/plugin` → Marketplaces → tz-pipeline → Enable auto-update.

### Выпуск новой версии

Поднимите `version` в `plugins/tz-to-product/.claude-plugin/plugin.json` и запушьте. Без смены версии пользователи остаются на своей копии и изменений не получат. Файл `tz-to-product.skill` пересоберите из `plugins/tz-to-product/skills/tz-to-product`.

## Как пользоваться

Пришлите ТЗ в сессию Claude Code: «вот ТЗ, собери». Скилл включится сам. Явный вызов — `/tz-to-product:tz-to-product`.

Для Claude на сайте или в приложении есть упакованный файл `tz-to-product.skill`: его загружают в разделе Skills в настройках.

## Состав

```
.claude-plugin/marketplace.json            маркетплейс tz-pipeline
plugins/tz-to-product/
  .claude-plugin/plugin.json               манифест плагина
  skills/tz-to-product/
    SKILL.md                               три фазы и точки остановки
    references/stages.md                   этапы 0–10: зачем, выход, гейт, ошибки
    references/placement.md                Extella OS, локально или отдельно
    references/extella.md                  подключение к Extella на macOS и Windows
    assets/templates.md                    бриф, карточка агента, тест-кейсы, чеклисты
  evals/                                   три тест-кейса со скиллом и без
```

## Проверка

```bash
claude plugin validate .
claude plugin eval plugins/tz-to-product
```

`plugin eval` прогоняет три ТЗ (стоматология, сверка 1С, сайт йога-студии) со скиллом и без него. Прогоны идут на модели и оплачиваются.

## Лицензия

MIT — см. [LICENSE](LICENSE).
