﻿# 1С:ГитКонвертер

Конфигурация 1С, односторонняя синхронизация хранилища конфигураций 1С в репозиторий Git.

Корректное переименование истории объектов метаданных при переименовании их в хранилище 1С по UUID'дам.

### Основные возможности

* Сконвертировать существующее хранилище 1С в репозиторий git
* Обновлять изменения из хранилища в репозиторий
* Параллелизировать загрузку истории хранилища из копий хранилища
* Ограничение нагрузки на сервер с помощью очередей
* Возможно "сращивать" историю в git, если хранилище 1С обрезалось или начиналось заново.
* Сообщение гиту команды ```git mv старый_файл новый_файл``` при переименовании метаданных
* Выгружать только изменения конфигурации. Доступно для Платформы 8.3.10 и выше, требуется использовать "очереди"

### Необходимые компоненты

* Конфигурацию можно запустить, используя 1C:Enterprise Development Tools 1.7 (https://releases.1c.ru/project/DevelopmentTools10)
* Платформа 1С:Предприятия 8.3.11 и выше (https://releases.1c.ru/project/Platform83)
* СУБД поддерживаемая 1С:Предприятием
* OS Windows 7 или выше, ОС Linux и macOS - в бета-режиме.

## Начальная настройка

### Настройка базы ГитКонвертера

1. Разместите базу ГитКонвертера на сервере 1С. Работа в файловом режиме может быть использована только в демонстрационных целях.
2. Заполните константу __"Путь к версиям платформы на сервере"__, где располагаются файлы Конфигуратора 1cv8(.exe) в формате: `C:\Program files (x86)\1cv8\%ВерсияПлатформы%\bin`
%ВерсияПлатформы% - в параметр будет подставлена текущая версия хранилища.
4. Для ограничения производительности можно включить константу __"Использовать очереди выполнения"__

### Настройка сервера 1С

Для ИБ ГитКонвертера на сервере 1С необходимо переключить формат журнала регистрации на старый режим.

Для этого необходимо в каталог журнала регистрации ИБ скопировать пустой файл с именем: __1Cv8.lgf__. 
Для больших проектов рекомендуется настроить удаление или бэкапирование файлов журнала регистрации.

Так же рекомендуется переключить регистрацию событий - только ошибки: __Конфигуратор - Администрирование - Настройка журнала регистрации... = Регистрировать ошибки__ с минимальной периодичностью.


## Настройка конвертации хранилища 1С

Рекомендуется использовать сервер хранилищ конфигураций 1С.

Для оптимальной работы сервера хранилищ настройте __Размер глобального кэша__ в "Администрировании" в 1,5-2 раза больше __количества__ параллельных потоков (если используются "копии хранилища") __получения версий * размер одной версии__.

### Параметры конвертации

* Укажите адрес хранилища. При использовании сервера хранилища рекомендуется настроить в администрировании хранилища параметр "Глобальный кэш версий конфигурации" чтобы количество кэшированных версий было больше параллельно получаемых версий.
* Укажите версию платформы, рекомендуется использовать 8.3.9.1818 и выше.
* Укажите начальную версию в хранилище, если текущее хранилище было обрезано и первая версия больше 1.
* Если указана версия окончания - не будет выполняться запрос новых версий.
* Укажите расписание запусков
* Укажите __Каталог выгрузки версий__, в котором будут создаваться временные каталоги с номерами версий и выгрузкой данных.
* Желательно ограничить количество подготавливаемых (выгружаемых) версий - рекомендуется установить значение исходя из __Размер базы с версией + Размер выгрузки в xml__ и размера жесткого диска.
* Укажите __Минимальное количество метаданных__ - число файлов и каталогов выгрузки в xml - необходимо для контроля, что все файлы выгружены. Рекомендуется устанавливать 90-95% от текущей версии, чтобы учесть возможность удаления метаданных (т.е. сокращения количества файлов)
* Не рекомендуется устанавливать __Удаление конфигураций поставщиков__, если планируется загружать конфигурацию из файлов и обновлять конфигурации поставщиков. Опция позволяет оптимизировать размер хранилища git и не хранить объемные файлы *.cf.
* Установите __Удалять временные данные версии после коммита__ - рекомендуется на реальных проектах.
* __Выгружать изменения__ - позволяет на Платформе 8.3.10 и выше выгружать только изменения. Доступно при использовании __"Очередей"__
* __Локальный каталог git__ рекомендуется указывать на одном логическом диске с __Каталогом выгрузки версий__ - будет использовано перемещение версий возможностями ОС, иначе будет выполняться копирование.
* __Каталог выгрузки в репозитории__ - относительный путь к каталогу выгрузки внутри репозитория. Рекомендуется указывать имя проекта для будущей совместимости с рабочим пространством 1C:EDT или оставить пустым.
* Установите флаг __Выполнять коммиты__ для выполнения коммитов. Отключение может быть необходимо с целью временно приостановить работу конвертера.
* Установите флаг __Обрабатывать все очереди__, если используются очереди в ИБ.

Нажмите кнопку "Создать гит репозиторий" перед конвертацией - команда выполняет инициализацию репозитория и начальную настройку, специфичную для конфигураций 1С.

### Дополнительная настройка репозитория Git

По умолчанию создается файл исключений __.gitignore__, в который добавляются файлы `DumpFilesIndex.txt` и `ConfigDumpInfo.xml` - не требуемые для работы с исходными файлами конфигурации 1С.

Для увеличения быстродействия репозитория git можно использовать расширение __git lfs__ (https://git-lfs.github.com)

Если используется сервер репозиториев Git, необходимо убедиться, что он поддерживает это расширение и включить настройки для проекта. Например, GitLab, GitHub, BitBucket - поддерживают.

Выполнить начальную настройку репозитория до выполнения первого коммита:

```bash
git lfs install
```

Включить отслеживание бинарных файлов конфигурации
```bash
git lfs track "*.cf"
git lfs track "*.bin"
git lfs track "*.png"
git lfs track "*.gif"
git lfs track "*.bmp"
git lfs track "*.jpg"
git lfs track "*.zip"
```
В этом примере - все файлы конфигураций поставщиков, файлы макетов с "Двоичными данными" и картинки из конфигурации попадут в lfs.

### Копии хранилища

Копии хранилища используются для ускорения получения версий из хранилища.

* Возможно использовать тот же адрес серверного хранилища конфигураций, но с разными пользователями.  Количество "копий" влияет на размер создаваемого глобального кэша версий на сервере хранилища 1С. Желательно установить кэш в настройках сервера хранилищ 1С в __полтора раза__ больше, чем количество копий в ГитКонвертере.

* Укажите другой адрес архивной копии хранилища, если в текущем хранилище выполнялось сокращение версий, и установите ограничение номеров версий в этой копии.
* Укажите расписание получения версий из этой копии. Если в "копии" указан адрес основного хранилища, необходимо в расписании учесть возможность работы разработчиков с хранилищем - запуски на получение выполнять с промежутками, обеспечивающими комфортную работу разработчиков.

### Очереди выполнения

Если включена константа "Использовать очереди выполнения", то для каждого хранилища конфигураций необходимо указать 2 очереди:

* Выгрузка метаданных. Начиная с версии Платформы 8.3.10 возможно использовать __выгрузку изменений__ - для этого необходимо выгружать версии строго последовательно и не рекомендуется создавать более одной очереди на выгрузку.
* Загрузка метаданных

Возможно указать диапазоны количества версий для каждой очереди для разграничения "рабочей зоны". 

Укажите ограничение количества версий обрабатываемых очередью за один запуск и расписание запусков.

Очередь может быть общей на всю базу или привязанной к конкретному хранилищу. Для очереди общего типа выбор версий для обработки выполняется по дате версии - это следует учитывать при конвертации проектов с длинной историей и более "молодых" проектов в одной базе ГитКонвертера.

### Информация пользователей

Хранилище 1С использует для идентификации __Пользователя__, а в репозитории git основным идентификатором является __email__ и имя пользователя. Для этих целей предназначен регистр сведений __Информация пользователей__, позволяющий указать соответствие пользователей хранилищ пользователям репозитория git.

Можно выполнять коммиты анонимно, с потерей информации об авторстве. Пользователь хранилища будет указан в дополнении к комментарию к каждой версии.

Пользователи могут быть указаны общие для всех хранилищ или с уточнением по хранилищам.