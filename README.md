# Cursor Skills for 1C:Enterprise

Набор скилов, правил и команд для [Cursor IDE](https://cursor.com): агент правит 1С по формату платформы,
а не текстовым поиском по файлам.

[![Лицензия MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Релиз](https://img.shields.io/github/v/release/Desko77/cursor-1c-skills)](https://github.com/Desko77/cursor-1c-skills/releases)
[![Тесты](https://github.com/Desko77/cursor-1c-skills/actions/workflows/tests.yml/badge.svg)](https://github.com/Desko77/cursor-1c-skills/actions/workflows/tests.yml)

Скилы дают агенту готовые операции над исходниками: собрать объект метаданных, форму, роль, схему
компоновки, расширение, внешнюю обработку - и разобрать их обратно. Правила задают, как он это
делает: стандарты BSL, антипаттерны, требования EDT, работа с БСП. Скрипты правят XML-выгрузку
напрямую, поэтому базовые операции идут без запущенной платформы и без Конфигуратора. Вместе с нашим
плагином [AI-EDT](https://github.com/Desko77/ai-edt) агент получает еще и семантическую модель самой
1C:EDT - об этом ниже.

Проверка результата тоже идет по выгрузке, без внешних инструментов. Четырнадцать валидаторов
разбирают XML объекта, формы, роли, схемы компоновки, макета, расширения. Сверх этого
`1c-config-index` собирает индекс конфигурации, и валидаторы сверяют объект с ОСТАЛЬНОЙ
конфигурацией: ссылочные типы реквизитов, пути данных формы, состав подсистемы, права роли, наборы
данных СКД, имена таблиц и полей в запросе, вызовы общих модулей. Заимствованная форма расширения
сверяется с основной конфигурацией тем же способом. Ни платформа, ни EDT, ни запущенная
база для этого не нужны. Границу набор обозначает прямо: это проверка имен и структуры, а типы
выражений, семантика BSL и план запроса без платформы недоступны.

Кому пригодится: разработчику 1С, который держит исходники под git и готов отдать агенту рутину -
scaffold объектов, разбор чужих обработок, выгрузку и загрузку конфигурации, ревью, тесты.

Кроме скилов в наборе есть **справочники API**: агент не подбирает имя метода по памяти, а берет
его из справочника и проверяет вызов до того, как напишет строку кода. **Справочник по БСП**
(`1c-bsp-api`) - 2624 метода в 284 модулях Библиотеки стандартных подсистем 3.1.11 и 3.2.1:
сигнатуры, типы параметров, контексты выполнения, 372 переопределяемых обработчика; команда `check`
отвергает вызов, которого в библиотеке нет. **Справочник по ЗУП** (`zup-hr-api-reference`) - 1С:ЗУП
3.1: кадровый учет, средний заработок и начисления, пособия и обмен с СФР, регламентированная
отчетность.

> Адаптация [claude-code-skills-1c](https://github.com/Desko77/claude-code-skills-1c) для Cursor IDE.
> Скилы и правила там и здесь одни и те же, различается только формат подключения к среде.

## Состав набора

| Группа | Что делает | Скилов |
|--------|------------|--------|
| Маршрутизатор | подсказывает, каким скилом решать задачу | 1 |
| Конфигурация | создать с нуля, изменить свойства, проанализировать, проиндексировать, прочитать состояние поддержки | 8 |
| Объекты метаданных | собрать из JSON, разобрать обратно в JSON, править реквизиты и табличные части | 6 |
| Управляемые формы | собрать из DSL или по метаданным объекта, править, разобрать, проверить | 8 |
| Расширения | создать, заимствовать объект, поставить перехватчик на метод, сверить с основной | 6 |
| Обработки и отчеты | собрать .epf и .erf из исходников и разобрать обратно | 10 |
| Базы данных и хранилище | создать, выгрузить и загрузить .cf, .dt и XML, обновить, запустить, работать с хранилищем конфигурации | 12 |
| Роли, подсистемы, макеты, СКД, XDTO | сборка и разбор своим DSL под каждый вид | 25 |
| Веб-публикация и тестирование | опубликовать базу через Apache, прогнать сценарий в браузере | 5 |
| БСП, 1С 7.7, обмены | справочник API БСП и поиск функций, разработка под 7.7, правила КД 2.0 и 3.1 | 8 |
| Справочные и утилитарные | документация платформы, справочники API типовых, транскрибация, docx и xlsx | 27 |

Полный перечень с описанием каждого скила - в разделе [Скилы](#скилы-116) ниже.

И **40 правил** (.mdc) - подключаются один раз и действуют во всех задачах, без напоминаний в каждом
промпте:

- **код и запросы** - стандарты BSL, антипаттерны вроде запроса в цикле, оптимизация, чеклист ревью;
- **формы, метаданные, EDT** - требования к XML форм, целостность MDO, ограничения экспорта из EDT;
- **БСП и типовые** - вызывать готовые механизмы вместо собственной реализации, размечать правки типовых;
- **процесс и сам агент** - SDD-workflow, выбор модели под задачу, рабочая память, безопасность git.

## Как это выглядит

Справочник с реквизитами и табличной частью. На входе пятнадцать строк JSON:

```json
{
  "type": "Справочник",
  "name": "Контрагенты",
  "codeLength": 11,
  "descriptionLength": 150,
  "hierarchical": true,
  "attributes": [
    "ИНН: Строка(12), index",
    "Покупатель: Булево",
    "ОсновнойМенеджер: СправочникСсылка.Пользователи"
  ],
  "tabularSections": {
    "КонтактныеЛица": ["ФИО: Строка(150), req", "Телефон: Строка(20)"]
  }
}
```

Один вызов скила `1c-meta-compile` - и на диске готовый объект конфигурации:

- `Catalogs/Контрагенты.xml` - 345 строк со всеми обязательными секциями, стандартными реквизитами
  и сгенерированными UUID;
- `Catalogs/Контрагенты/Ext/ObjectModule.bsl` - заготовка модуля объекта;
- запись `<Catalog>Контрагенты</Catalog>` в `Configuration.xml`, то есть объект сразу в составе
  конфигурации, а не сиротой на диске;
- `Validation OK: Catalog.Контрагенты (16 checks)` от валидатора набора.

Структуру файла агент не выдумывает и не подсматривает в чужих выгрузках: формат зашит в скил.

## Установка

Два пути на выбор: попросить агента или сделать руками.

### Вариант 1: попросить агента, без единой команды

Скопируйте текст ниже, подставьте свой каталог вместо `<КАТАЛОГ>` и отдайте агенту в новой сессии.
Дальше он все сделает сам: склонирует репозиторий, покажет план и поставит только то, что вы выберете.

```
Установи мне набор скилов и правил для работы с 1С из репозитория
https://github.com/Desko77/cursor-1c-skills

Порядок:
1. Склонируй репозиторий в <КАТАЛОГ>, если он уже там - выполни git pull.
2. Прочитай в нем skills/claude-env-setup/SKILL.md и references/safety.md.
   Дальше действуй строго по ним.
3. Спроси, куда ставить: в .cursor/ текущего проекта или глобально
   в ~/.cursor/.
4. Покажи план таблицей и дождись моего выбора. Ничего не ставь
   до моего ответа.
5. Поставь выбранное скриптом install.ps1 из корня репозитория.

Ограничения:
- Ничего не перезаписывай молча. Файл, который у меня уже есть
  и отличается, - покажи diff и спроси.
- Не трогай мои правила и скилы, которых нет в наборе.
- Перед первой правкой любого конфига сделай резервную копию
  с меткой времени.

В конце дай отчет: что поставлено, что не встало и почему,
и отдельным списком - что мне нужно сделать руками.
```

Этот же путь годится для обновления: агент доставит недостающее и не тронет ваши правки. Готовые
промпты под остальные случаи - в [`skills/claude-env-setup/PROMPT.md`](skills/claude-env-setup/PROMPT.md).

### Вариант 2: руками, если вы знаете, что делаете

```powershell
.\install.ps1                                         # в .cursor/ текущего проекта
.\install.ps1 -ProjectDir "C:\Projects\my-1c-project" # в конкретный проект
.\install.ps1 -Global                                 # глобально, в ~/.cursor/
.\install.ps1 -RulesOnly                              # только правила
.\install.ps1 -SkillsOnly                             # только скилы
```

Без скрипта - копированием в `.cursor/` вашего проекта:

```powershell
Copy-Item -Path rules\*    -Destination .cursor\rules\    -Recurse -Force
Copy-Item -Path skills\*   -Destination .cursor\skills\   -Recurse -Force
Copy-Item -Path commands\* -Destination .cursor\commands\ -Recurse -Force
```

> Копирование затрет ваши правки, если вы дорабатывали скилы под себя. На такой машине берите
> вариант 1: он показывает diff и спрашивает по каждому расхождению.

## Первая задача

Откройте проект с исходниками 1С в Cursor и скажите задачу обычными словами: "создай справочник
Контрагенты с ИНН и табличной частью контактных лиц". Скил агент подберет сам.

## AI-EDT: агентная разработка прямо в 1C:EDT

Набор раскрывается полностью вместе с **[AI-EDT](https://github.com/Desko77/ai-edt)** - MCP-сервером,
который работает плагином **внутри запущенной 1C:EDT** и дает ассистенту не текст файлов, а
семантическую модель самой среды.

| | |
|---|---|
| Репозиторий | **https://github.com/Desko77/ai-edt** |
| Установка | update site **https://desko77.github.io/ai-edt/** (в EDT: меню Help, пункт Install New Software) |
| Требования | 1C:EDT 2026.2 или 2026.1 |

Ассистент разрешает ссылки на объекты метаданных, строит иерархии вызовов, спрашивает у EDT
выведенный тип в позиции BSL, правит метаданные без ручного XML, ставит точки останова и читает
переменные в приостановленной сессии - а результат тут же проверяет валидаторами самой EDT.
Скил `ai-edt-tools` в этом наборе - каталог его инструментов и типовых сценариев.

## Типовые сценарии

| Задача | Чем решается |
|--------|--------------|
| Добавить в конфигурацию документ, а к нему форму | `1c-meta-compile`, затем `1c-form-compile` |
| Доработать типовую, не снимая с поддержки | `1c-cfe-init`, `1c-cfe-borrow`, `1c-cfe-patch-method` |
| Собрать внешнюю обработку из исходников и загрузить в базу | `1c-epf-build`, `1c-db-load-cf` |
| Разобрать чужую .epf, чтобы понять, что она делает | `1c-epf-dump`, затем `1c-form-info` |
| Выгрузить конфигурацию в XML и положить под git | `1c-db-dump-xml` |
| Не угадывать, каким скилом решать задачу | `1c-config-router` |

## Зависимости

| Компонент | Обязательно | Для чего |
|-----------|-------------|----------|
| PowerShell 5.1+ (Windows) | Да | Скрипты скилов (.ps1) |
| Python 3.8+ | Нет | `v8unpack-cf`, `check-uuid`, `img-grid-analysis`, `transcribe` |
| Python: `google-genai`, `python-dotenv` | Нет | `transcribe` (транскрибация через Gemini API) |
| `ffmpeg`, `ffprobe` | Нет | `transcribe` (извлечение скриншотов, разбивка длинных файлов) |
| [v8unpack](https://pypi.org/project/v8unpack/) | Нет | Распаковка/сборка CF/CFE/EPF без платформы |
| 1C:Enterprise 8.3 | Нет | Скилы группы `db-*`, сборка EPF/ERF |
| `gcomp` | Нет | `1c77-dev` (разбор/сборка .ert и 1Cv7.MD для 1С 7.7) |
| Node.js + `@mermaid-js/mermaid-cli` (mmdc) | Нет | `mermaid-render` (рендер диаграмм в PNG/SVG/PDF) |
| Обработка `MCP_Toolkit.epf` ([ROCTUP](https://github.com/ROCTUP/1c-mcp-toolkit)) | Нет | `1c-mcp-toolkit` (HTTP API к живой запущенной базе 1С) |
| MCP-серверы (EDT, BSP) | Нет | Расширенный анализ кода, валидация запросов |

Скилы спроектированы по слоям - базовые (генерация XML) работают без платформы, продвинутые требуют 1С или MCP.

## Скилы (116)

Начать проще всего с `1c-config-router`: он сам определяет, каким скилом или workflow
решать задачу.

<table>
<thead><tr><th align="left">Скил</th><th align="left">Описание</th></tr></thead>
<tbody>
<tr><th colspan="2" align="left">Маршрутизатор</th></tr>
<tr><td><code>1c-config-router</code></td><td>Определяет нужный workflow или скил для задачи</td></tr>
<tr><th colspan="2" align="left">Конфигурация (cf-*, support-*)</th></tr>
<tr><td><code>1c-cf-init</code></td><td>Создать пустую конфигурацию (scaffold XML), версии формата 2.17-2.21</td></tr>
<tr><td><code>1c-config-index</code></td><td>Индекс всей выгрузки одним JSON: объекты, реквизиты, измерения регистров, экспортные методы общих модулей. Основа проверок, где объект сверяется с остальной конфигурацией</td></tr>
<tr><td><code>1c-cf-info</code></td><td>Анализ структуры конфигурации</td></tr>
<tr><td><code>1c-cf-edit</code></td><td>Изменить свойства конфигурации</td></tr>
<tr><td><code>1c-cf-validate</code></td><td>Валидация конфигурации</td></tr>
<tr><td><code>1c-cf-add-object</code></td><td>Workflow: добавить объект в конфигурацию</td></tr>
<tr><td><code>1c-cf-new-project</code></td><td>Workflow: создать конфигурацию с нуля</td></tr>
<tr><td><code>1c-support-state</code></td><td>Состояние поддержки в XML-выгрузке: чтение, editable/off-support/locked, возможность изменения</td></tr>
<tr><th colspan="2" align="left">Объекты метаданных (meta-*)</th></tr>
<tr><td><code>1c-meta-compile</code></td><td>Создать объект метаданных из JSON DSL (37 типов)</td></tr>
<tr><td><code>1c-meta-decompile</code></td><td>Декомпиляция объекта в JSON-заготовку формата meta-compile - черновик нового объекта по образцу</td></tr>
<tr><td><code>1c-meta-edit</code></td><td>Изменить реквизиты, ТЧ, свойства объекта</td></tr>
<tr><td><code>1c-meta-info</code></td><td>Анализ структуры объекта</td></tr>
<tr><td><code>1c-meta-remove</code></td><td>Удалить объект из конфигурации</td></tr>
<tr><td><code>1c-meta-validate</code></td><td>Валидация объекта метаданных</td></tr>
<tr><th colspan="2" align="left">Формы (form-*)</th></tr>
<tr><td><code>1c-form-compile</code></td><td>Создать форму из JSON DSL</td></tr>
<tr><td><code>1c-form-decompile</code></td><td>Разобрать Form.xml в JSON-черновик DSL</td></tr>
<tr><td><code>1c-form-edit</code></td><td>Добавить элементы, реквизиты, команды в форму</td></tr>
<tr><td><code>1c-form-add</code></td><td>Добавить форму к объекту конфигурации</td></tr>
<tr><td><code>1c-form-info</code></td><td>Анализ структуры формы</td></tr>
<tr><td><code>1c-form-patterns</code></td><td>Паттерны проектирования форм</td></tr>
<tr><td><code>1c-form-remove</code></td><td>Удалить форму</td></tr>
<tr><td><code>1c-form-validate</code></td><td>Валидация формы</td></tr>
<tr><th colspan="2" align="left">Расширения (cfe-*)</th></tr>
<tr><td><code>1c-cfe-init</code></td><td>Создать расширение конфигурации</td></tr>
<tr><td><code>1c-cfe-borrow</code></td><td>Заимствовать объект из конфигурации</td></tr>
<tr><td><code>1c-cfe-patch-method</code></td><td>Перехватить метод (Before/After/ModificationAndControl)</td></tr>
<tr><td><code>1c-cfe-diff</code></td><td>Анализ расширения</td></tr>
<tr><td><code>1c-cfe-validate</code></td><td>Валидация расширения</td></tr>
<tr><td><code>1c-cfe-full-cycle</code></td><td>Workflow: полный цикл создания расширения</td></tr>
<tr><th colspan="2" align="left">Обработки и отчеты (epf-*, erf-*)</th></tr>
<tr><td><code>1c-epf-scaffold</code></td><td>Создать пустую обработку</td></tr>
<tr><td><code>1c-epf-add-form</code></td><td>Добавить форму к обработке</td></tr>
<tr><td><code>1c-epf-build</code></td><td>Собрать EPF из XML-исходников</td></tr>
<tr><td><code>1c-epf-dump</code></td><td>Разобрать EPF в XML-исходники</td></tr>
<tr><td><code>1c-epf-validate</code></td><td>Валидация обработки</td></tr>
<tr><td><code>1c-epf-full-cycle</code></td><td>Workflow: полный цикл создания обработки</td></tr>
<tr><td><code>1c-erf-init</code></td><td>Создать пустой отчет</td></tr>
<tr><td><code>1c-erf-build</code></td><td>Собрать ERF</td></tr>
<tr><td><code>1c-erf-dump</code></td><td>Разобрать ERF</td></tr>
<tr><td><code>1c-erf-validate</code></td><td>Валидация отчета</td></tr>
<tr><th colspan="2" align="left">Пакеты XDTO (xdto-*)</th></tr>
<tr><td><code>1c-xdto-compile</code></td><td>Создать пакет XDTO из XML-схемы (XSD)</td></tr>
<tr><td><code>1c-xdto-decompile</code></td><td>Выгрузить пакет XDTO обратно в XSD</td></tr>
<tr><td><code>1c-xdto-edit</code></td><td>Точечная правка типов и свойств пакета</td></tr>
<tr><td><code>1c-xdto-info</code></td><td>Структура пакета: типы, свойства, точки входа</td></tr>
<tr><td><code>1c-xdto-validate</code></td><td>Валидация пакета XDTO</td></tr>
<tr><th colspan="2" align="left">Подсистемы и интерфейс</th></tr>
<tr><td><code>1c-subsystem-compile</code></td><td>Создать подсистему</td></tr>
<tr><td><code>1c-subsystem-edit</code></td><td>Изменить состав подсистемы</td></tr>
<tr><td><code>1c-subsystem-info</code></td><td>Анализ подсистемы</td></tr>
<tr><td><code>1c-subsystem-validate</code></td><td>Валидация подсистемы</td></tr>
<tr><td><code>1c-interface-edit</code></td><td>Настроить командный интерфейс</td></tr>
<tr><td><code>1c-interface-validate</code></td><td>Валидация интерфейса</td></tr>
<tr><th colspan="2" align="left">Макеты (mxl-*, template-*)</th></tr>
<tr><td><code>1c-mxl-compile</code></td><td>Создать макет из JSON DSL</td></tr>
<tr><td><code>1c-mxl-decompile</code></td><td>Разобрать макет в JSON</td></tr>
<tr><td><code>1c-mxl-info</code></td><td>Анализ макета</td></tr>
<tr><td><code>1c-mxl-validate</code></td><td>Валидация макета</td></tr>
<tr><td><code>1c-template-add</code></td><td>Добавить макет к объекту</td></tr>
<tr><td><code>1c-template-remove</code></td><td>Удалить макет</td></tr>
<tr><th colspan="2" align="left">Роли (role-*)</th></tr>
<tr><td><code>1c-role-compile</code></td><td>Создать роль из описания прав</td></tr>
<tr><td><code>1c-role-info</code></td><td>Анализ роли</td></tr>
<tr><td><code>1c-role-validate</code></td><td>Валидация роли</td></tr>
<tr><th colspan="2" align="left">СКД (skd-*)</th></tr>
<tr><td><code>1c-skd-compile</code></td><td>Создать схему компоновки данных</td></tr>
<tr><td><code>1c-skd-decompile</code></td><td>Разобрать XML СКД в JSON-черновик DSL</td></tr>
<tr><td><code>1c-skd-edit</code></td><td>Изменить существующую СКД</td></tr>
<tr><td><code>1c-skd-info</code></td><td>Анализ СКД</td></tr>
<tr><td><code>1c-skd-validate</code></td><td>Валидация СКД</td></tr>
<tr><th colspan="2" align="left">Базы данных (db-*)</th></tr>
<tr><td><code>1c-db-list</code></td><td>Управление реестром баз</td></tr>
<tr><td><code>1c-db-create</code></td><td>Создать информационную базу</td></tr>
<tr><td><code>1c-db-dump-cf</code></td><td>Выгрузить конфигурацию в CF</td></tr>
<tr><td><code>1c-db-dump-dt</code></td><td>Выгрузить всю ИБ в DT (полный бэкап)</td></tr>
<tr><td><code>1c-db-dump-xml</code></td><td>Выгрузить конфигурацию в XML</td></tr>
<tr><td><code>1c-db-load-cf</code></td><td>Загрузить конфигурацию из CF</td></tr>
<tr><td><code>1c-db-load-dt</code></td><td>Восстановить ИБ из DT (деструктивная)</td></tr>
<tr><td><code>1c-db-load-xml</code></td><td>Загрузить конфигурацию из XML</td></tr>
<tr><td><code>1c-db-load-git</code></td><td>Загрузить изменения из Git</td></tr>
<tr><td><code>1c-db-update</code></td><td>Обновить конфигурацию БД</td></tr>
<tr><td><code>1c-db-run</code></td><td>Запустить 1С:Предприятие</td></tr>
<tr><td><code>1c-storage-ops</code></td><td>Хранилище конфигурации: отчет по версиям, захват и снятие захвата, обновление, помещение, выгрузка, подключение и отключение базы</td></tr>
<tr><th colspan="2" align="left">БСП</th></tr>
<tr><td><code>1c-bsp-registration</code></td><td>Регистрация обработки в БСП</td></tr>
<tr><td><code>1c-bsp-command</code></td><td>Добавить команду БСП</td></tr>
<tr><td><code>1c-bsp-api</code></td><td>Справочник программного интерфейса БСП: модули, экспортные методы, сигнатуры, типы параметров, контексты выполнения, переопределяемые обработчики. Проверяет вызов до того, как он написан</td></tr>
<tr><td><code>1c-ssl-patterns</code></td><td>Паттерны подсистем БСП</td></tr>
<tr><th colspan="2" align="left">Разработка 1С 7.7</th></tr>
<tr><td><code>1c77-dev</code></td><td>Разработка под 1С 7.7: .1s/.ert/.frm, 1Cv7.MD, gcomp, кодировка CP1251</td></tr>
<tr><th colspan="2" align="left">Конвертация данных и интеграции</th></tr>
<tr><td><code>kd2-rules</code></td><td>Правила обмена "Конвертация данных 2.0" через MCP-toolkit</td></tr>
<tr><td><code>kd31-rules</code></td><td>Правила обмена "Конвертация данных 3.1" через MCP-toolkit</td></tr>
<tr><td><code>cleverence-mslx</code></td><td>Mobile SMARTS (Клеверенс): .mslx-алгоритмы ТСД, online-вызов 1С</td></tr>
<tr><th colspan="2" align="left">Справочные и утилитарные</th></tr>
<tr><td><code>ai-edt-tools</code></td><td>Справочник инструментов AI-EDT (MCP-плагин для 1С:EDT): фасады, сценарии, особенности и ограничения</td></tr>
<tr><td><code>1c-naparnik</code></td><td>Справочник инструментов 1С:Напарник (анализ кода, ИТС, документация)</td></tr>
<tr><td><code>1c-mcp-toolkit</code></td><td>Прямой HTTP API к живой запущенной базе 1С (запросы, BSL-код, метаданные, журнал)</td></tr>
<tr><td><code>1c-platform-docs</code></td><td>Поиск по документации API платформы</td></tr>
<tr><td><code>1c-query-validate</code></td><td>Проверка текста запроса по выгрузке: существуют ли таблицы, табличные части, виртуальные таблицы регистров и поля</td></tr>
<tr><td><code>1c-bsl-validate</code></td><td>Проверка вызовов общих модулей в BSL по выгрузке: существует ли модуль и экспортный ли метод</td></tr>
<tr><td><code>1c-query-optimization</code></td><td>Продвинутая оптимизация запросов</td></tr>
<tr><td><code>1c-help-manage</code></td><td>Встроенная справка объектов 1С</td></tr>
<tr><td><code>composing-1c-queries</code></td><td>Руководство по языку запросов 1С</td></tr>
<tr><td><code>1c-vanessa-steps</code></td><td>Реестр 1569 шагов Vanessa Automation: поиск шага по смыслу и валидация сценария .feature перед прогоном</td></tr>
<tr><td><code>claude-env-setup</code></td><td>Установка и обновление окружения: опись -&gt; план -&gt; установка выбранного, без затирания чужого</td></tr>
<tr><td><code>docx-from-sample</code></td><td>Новый DOCX в оформлении готового образца: стили, нумерация, таблицы, колонтитулы из файла-образца</td></tr>
<tr><td><code>meeting-to-tasks</code></td><td>Цикл от записи встречи до планов разработки</td></tr>
<tr><td><code>sync-fork</code></td><td>Синхронизация форка с upstream без потери своих доработок</td></tr>
<tr><td><code>lmstudio-api</code></td><td>Справочник HTTP-API локального сервера моделей</td></tr>
<tr><td><code>v8unpack-cf</code></td><td>Распаковка/сборка CF/CFE/EPF</td></tr>
<tr><td><code>img-grid-analysis</code></td><td>Анализ изображений для макетов</td></tr>
<tr><td><code>md-to-docx</code></td><td>Конвертация Markdown в DOCX</td></tr>
<tr><td><code>transcribe</code></td><td>Транскрибация аудио (локально, faster-whisper + sherpa-onnx) и видео (Gemini API)</td></tr>
<tr><td><code>transcribe-audio-local</code></td><td>Только аудио, только локально, self-contained - для передачи без облака</td></tr>
<tr><td><code>mermaid-diagrams</code></td><td>Генерация диаграмм Mermaid</td></tr>
<tr><td><code>mermaid-render</code></td><td>Рендер Mermaid в PNG/SVG/PDF через локальный mmdc</td></tr>
<tr><td><code>powershell-windows</code></td><td>PowerShell на Windows</td></tr>
<tr><td><code>zup-hr-api-reference</code></td><td>Справочник API 1С:ЗУП 3.1: кадровый учет (физлица, стажи, договоры ГПХ, представления СКД) и расчет (средний заработок, начисления, пособия/СФР, взносы, отчетность)</td></tr>
<tr><td><code>prompt-enhancer</code></td><td>Улучшение и структурирование коротких промптов и постановок задач в подробные ТЗ</td></tr>
<tr><td><code>humanize-ai-text</code></td><td>Переписывание AI-текста в живой человеческий стиль</td></tr>
<tr><td><code>claude-md-bootstrap</code></td><td>Генерация файла-контекста проекта для AI-агента (CLAUDE.md)</td></tr>
<tr><th colspan="2" align="left">Веб-публикация и тестирование (web-*)</th></tr>
<tr><td><code>1c-web-publish</code></td><td>Публикация ИБ на веб-сервере (Apache/IIS)</td></tr>
<tr><td><code>1c-web-unpublish</code></td><td>Отмена публикации ИБ</td></tr>
<tr><td><code>1c-web-info</code></td><td>Информация о веб-публикациях</td></tr>
<tr><td><code>1c-web-stop</code></td><td>Остановка веб-сервера</td></tr>
<tr><td><code>1c-web-test</code></td><td>Тестирование 1С через веб-клиент (автоматизация браузера)</td></tr>
</tbody>
</table>

## Правила (40)

| Файл | Описание |
|------|----------|
| `1c-coding-standards.mdc` | Стандарты кода BSL: именование, форматирование, запросы, коллекции, директива РАЗРЕШЕННЫЕ |
| `anti_patterns.mdc` | Критические антипаттерны: запрос в цикле, точка, O(n^2) |
| `query-optimization-tips.mdc` | Оптимизация запросов: ВЫРАЗИТЬ, ВТ, индексы, СКД |
| `async-methods-1c.mdc` | Асинхронные методы (Асинх/Ждать/Обещание) |
| `1c-extension-patterns.mdc` | Паттерны расширений CFE: перехватчики, маркеры |
| `form_module_rules.mdc` | Клиент-серверное разделение в модулях форм |
| `1c-form-reserved-names.mdc` | Зарезервированные имена свойств элементов в модулях форм |
| `forms_events.mdc` | Привязка обработчиков событий в Form.xml |
| `forms_generation.mdc` | Генерация и модификация форм 1С через 1c-forms-mcp |
| `edt-form-xml-requirements.mdc` | Требования EDT к XML-формам (Form.form): extInfo, дефолты полей |
| `code-review-checklist.mdc` | Чеклист ревью BSL-кода (Critical/High/Medium/Low) |
| `code-exploration-guide.mdc` | Методология исследования кодовой базы 1С |
| `testing-patterns.mdc` | Паттерны тестирования: YaXUnit, Vanessa Automation |
| `1c-mdo-integrity.mdc` | Целостность MDO-файлов: UUID, ссылки, ограничения типов String/Number |
| `external-data-source-mdo.mdc` | Внешние источники данных (ВИД) в формате EDT |
| `v8unpack-source-structure.mdc` | Структура исходников v8unpack |
| `refactoring.mdc` | Правила рефакторинга 1С |
| `routine_assignment_ext_processor.mdc` | Фоновые задания из внешней обработки через БСП |
| `bsp-profile-rights-api.mdc` | Программная работа с профилями групп доступа БСП: ссылки на ИдентификаторыОбъектовМетаданных |
| `model-selection.mdc` | Стратегия выбора моделей: Opus/Sonnet/Haiku по типу задачи |
| `sdd-workflow.mdc` | Specification-Driven Development: 9-фазный workflow разработки |
| `skill-design.mdc` | Проектирование навыков: способы вызова, лестница информации, критерии завершения шага |
| `agent-working-memory.mdc` | Рабочая память агента: файлы плана, передача сессии, правила проекта как журнал инцидентов |
| `mcp-tool-priority.mdc` | Маршрутизатор MCP-серверов под задачу и обязательные проверки |
| `git-safety-rules.mdc` | Безопасная работа с git: деструктивные команды, секреты, политика EOL для 1С |
| `designer-batch-verdict.mdc` | Вердикт пакетного запуска платформы: четыре сигнала вместо кода возврата, классификация журнала, маскирование пароля |
| `1c-transactions-and-locks.mdc` | Транзакции и управляемые блокировки: неявные транзакции платформы, канонический шаблон, порядок захвата |
| `edt-source-format.mdc` | Формат исходников и один владелец развертывания на прогон |
| `1c-cyrillic-windows.mdc` | Кириллица в путях и аргументах на Windows: симптомы, кодировки, молчаливый отказ при удалении |
| `1c-ordinary-forms.mdc` | Обычные формы: отличия от управляемых, порядок операций с видимостью |
| `text-formatting.mdc` | Запрещенные символы в генерируемом тексте |
| `file-edit-efficiency.mdc` | Выбор инструмента чтения и правки файлов по стоимости токенов |
| `1c-role-rights.mdc` | Права ролей 1С: права на объекты, RLS, наследование |
| `1c-report-direct-query.mdc` | Прямой запрос в отчетах (СКД) без схемы компоновки |
| `edt-zip-export-pitfalls.mdc` | Ограничения инкрементального экспорта конфигурации из EDT в ИБ |
| `agent-verification-patterns.mdc` | Паттерны проверки результата работы агента (reconciliation loop) |
| `bsl-ssl.mdc` | Переиспользование БСП, разметка правок типовых модулей, устаревшие объекты |
| `edt-bsl-write-safety.mdc` | Безопасная запись BSL-модулей через EDT MCP |
| `integrations.mdc` | Python-first подход к HTTP-интеграциям 1С |
| `1c-skd-two-pass-preprocessing.mdc` | Двухпроходный СКД: предобработка деталей до свертки |

## Команды (5)

| Файл | Описание |
|------|----------|
| `check-uuid.md` | Проверка уникальности UUID в MDO-файлах |
| `check_uuid_duplicates.py` | Python-скрипт проверки дубликатов UUID |
| `docx-to-md.md` | Конвертация DOCX в Markdown |
| `move-project.md` | Перенос проекта с сохранением памяти, сессий и планов |
| `browser-debug.md` | Запуск браузера с CDP-отладкой для веб-приложений |

## Отличия от Claude Code версии

| Аспект | Claude Code | Cursor |
|--------|-------------|--------|
| Правила | `.md` в `~/.claude/rules/` | `.mdc` с MDC frontmatter в `.cursor/rules/` |
| Скилы frontmatter | `name`, `description`, `allowed-tools`, `argument-hint` | `name`, `description` |
| Evals | `evals/evals.json` для тестирования | Не включены |
| skill-creator | Включен (создание и тестирование скилов) | Не включен (привязан к Claude Code) |
| Количество скилов | 117 | 116 |

## Синхронизация с исходным репо

При обновлении [claude-code-skills-1c](https://github.com/Desko77/claude-code-skills-1c) можно перегенерировать этот репо:

```bash
python tools/convert_from_claude.py --source ../claude-code-skills-1c --target .
```

## Лицензия

MIT

## История версий

Что изменилось по версиям - [CHANGELOG.md](CHANGELOG.md).
