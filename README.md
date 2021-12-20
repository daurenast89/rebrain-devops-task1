# Проверка CI/CD
# Configurator

### Описание:
Configurator - это сервис, который с помощью своего  web UI помогает настроить торговую конфигурацию для торгов

### Использование в проекте:
Данный сервис предоставляет файл конфигурации для торговых системи посылает через REST запрос новую конфигурацию для [collector-client](ссылка на проект) (на каких биржах, какие пары собирать для данной системы)

Подробное описание интерфейса конфигуратора есть в инструкции ["Описание сервиса Configurator"](тут ссылка на инструкцию)

### Зависимости в проекте:
Данный сервис зависит от (как именно смотреть в репозитории зависимости):

 - [commmon-libs/misc](https://gitlab.com/darwintradekz/common-libs/misc). Используется как хранилище простых общих примитивов (алгоритмы сортировки, поиска, helper-методы и т.д.)
 
 - [core/exchanger](https://gitlab.com/darwintradekz/core/exchanger/). Библиотека, предоставляющая доступ к API бирж.
 
### Входные данные
### Inputs

##### conf.yml
`conf.yml` содержит общие настройки сервиса darvin-configurator. Пример файла настроек - [conf.yml.example](conf/conf.yml.example)  

- `log_level` - уровень логирования [debug|info|error]
- `system_name` - имя торговой системы, которое будет отображено на главной странице web-интерфейса и будет отправляться на collector-client, чтобы он мог идентифицировать для какой торговой системы он получает конфигурацию. Возможное значение: (crossboarder|triangular|monitoring-triangular|monitoring-crossboarder|test|dynamic-test)
- `port` - на каком порту запускать web-интерфейс
- `collector_client_addr` - адрес collector-client, на котором он слушает новые обновления с конфигуратора
- `coin_market_key` - API ключи ресурса [coinmarketcap](https://coinmarketcap.com/), который используется в основном для получения цен активов и их наименований

#### configSource.db
`configSource.db` - файл, хранящий текущую торговую конфигурацию, обновляется периодически, при обновлении настроек. Используется для того, чтобы при перезапусках darvin-configurator не создавать конфигурацию сначала, а использовать уже существующую.

Таким образом для экономии времени, можно всегда взять готовый файл `configSource.db` с одного конфигуратора (или из хранилища бэкапов конфигураций) и запустить новый darvin-configurator с готовой конфигурацией

Расположение: `static/source/configSource.db`

### Пример использования:
Darvin-configurator может быть запущен и без файла `configSource.db`
Наличие `conf.yml` обязательно

1) Для сборки проекта необходима утилита `bee` пакета `beego`. Установить ее можно по [инструкции](https://beego.me/docs/install/bee.md)

2) Соберем проект:

```shell
bee pack
```

Получим архив `darvin-configurator.tar.gz`

3) Распакуем архив в отдельную директорию `test-configurator`
```shell
mkdir test-configurator
mv configurator.tar.gz test-configurator
cd test-configurator
tar -xzvf test-configurator
cd ..
```

После распаковки архива получим все необходимые для запуска артефакты, включая бинарный файл самого конфигуратора `darvin-configurator`

4) Сформируем конфигурацию `conf.yml` (файл `conf.yml` должен лежать на один уровень выше директории `test-configurator`):
```yaml
log_level: info
system_name: crossboarder
port: 11543
collector_client_addr: http://localhost:8081
coinmarket_key: "033438eb-318b-47bc-8642-e30d448a347d"
```

Если необходимо, то можно поднять [collector-client](https://gitlab.com/darwintradekz/core/collector/-/tree/master/collClient) (см [readme collector-client](https://bitbucket.org/vladimir_savostin/darvin-collector/src/master/collClient/readme.md)) чтобы он слушал новые конфигурации на `http://localhost:8081`

5) запустим darvin-configurator
```shell
cd test-configurator
./darvin-configurator
```

6) если все успешно запустилось, заходим в браузере на `https://localhost:11543/` (Обязательно `https`)  - должна отобразиться главная страница.

Для более упрощенного запуска в директории `tests` есть скрипт [run-configurator.sh](tests/run-configurator.sh) который делает все описанные выше шаги и запускает конфигуратор


### Тестирование:
Darvin-configurator можно протестировать только в ручном режиме. Для запуска использовать скрипт [run-configurator.sh](tests/run-configurator.sh), который запустит конфигуратор на `https://localhost:11543/`. 

### Выходные данные:

1) Конфигурация для коллектора (какие пары и на каких биржах собирать). Данная конфигурация находится в файле [confExport.go](controllers/confExport.go) структура `NewConfRequest`    


2) `config.db`  - файл торговой конфигурации, который используется на [handler](https://gitlab.com/darwintradekz/core/crypto-darwin-b)

Расположение: `tests/test-configurator/static/export/config.db` - появляется/обновляется после нажатия кнопки `Export` (см ["Описание сервиса Configurator"](ссылка на описание проекта))

`config.db` - это файловая база данных (используется библиотека  [storm](https://github.com/asdine/storm)), в которой каждая сущность заключена в одну из структур в файле [dbmodels/config.go](dbmodels/config.go)

Опишем основные сущности, хранимые в `config.db`

|Название                      |Структура в [config.go](dbmodels/config.go)|Вкладка в web-interface           | Исходные файлы, участвующие в настройке                                   |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Глобальные настройки          |GlobalStaticConfig                         |[Common settings](ссылка на инструкцию)    |[confGlobalController.go](controllers/confGlobalController.go)             |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confGlobal.html](views/confGlobal.html)                                     |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confGlobal.js](static/js/confGlobal.js)                                   |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Валюты на глобальном уровне   |DarvinCurrencies                           |[Currencies](ссылка на инструкцию)         |[confDarvinCurrController.go](controllers/confDarvinCurrController.go)     |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confDarvinCurr.html](views/confDarvinCurr.html)                             |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confDarvinCurr.js](static/js/confDarvinCurr.js)                           |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Базовые валюты                |DarvinBaseCurrencies                       |[Base currencies](ссылка на инструкцию)    |[confDarvinBCurrController.go](controllers/confDarvinBCurrController.go)   |
|на глобальном уровне          |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confDarvinBCurr.html](views/confDarvinBCurr.html)                           |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confDarvinBCurr.js](static/js/confDarvinBCurr.js)                         |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Пары на глобальном уровне     |DarvinPairs                                |[Pairs](ссылка на инструкцию)              |[confDarvinPairController.go](controllers/confDarvinPairController.go)     |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confDarvinPairs.html](views/confDarvinPairs.html)                           |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confDarvinPairs.js](static/js/confDarvinPairs.js)                         |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Перформеры                    |Performer                                  |[Performers](ссылка на инструкцию)         |[confPerformers.go](controllers/confPerformers.go)                         |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confPerformers.html](views/confPerformers.html)                             |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confPerformers.js](static/js/confPerformers.js)                           |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Общие настройки биржи         |ConfigExchanges                            |[Common](ссылка на инструкцию)             |[confExchangeController.go](controllers/confExchangeController.go)         |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confExchange.html](views/confExchange.html)                                 |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confExchange.js](static/js/confExchange.js)                               |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Настройки валют на бирже       |ExCurrency                                 |[Currencies](ссылка на инструкцию)          |[confExchangeController.go](controllers/confExchangeController.go)         |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confExchange.html](views/confExchange.html)                                 |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confExchange.js](static/js/confExchange.js)                               |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Настройки пар на бирже         |ExPair                                     |[Pairs](ссылка на инструкцию)              |[confDarvinCurrController.go](controllers/confDarvinCurrController.go)     |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confExchange.html](views/confExchange.html)                                 |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confExchange.js](static/js/confExchange.js)                               |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|
|Настройки валют на бирже       |ExCurrencyAcc                              |[Accounts](ссылка на инструкцию)           |[confAccountController.go](controllers/confAccountController.go)           |
|с привязкой к аккаунту        |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confAccounts.html](views/confAccounts.html)                                 |
|                              |                                           |                                                                                                                                                                                                                                                                                                                                                                                                                                   |[confAccounts.js](static/js/confAccounts.js)                               |
|------------------------------|-------------------------------------------|----------------------------------|---------------------------------------------------------------------------|

!!!
