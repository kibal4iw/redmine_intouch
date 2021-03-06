# redmine_intouch

Плагин разработан [Centos-admin.ru](http://centos-admin.ru/).

Плагин предназначен для рассылки уведомлений пользователям Redmine через Telegram или E-mail.

Пожалуйста помогите нам сделать этот плагин лучше, сообщая во вкладке [Issues](https://github.com/centosadmin/redmine_intouch/issues) обо всех проблемах, с которыми Вы столкнётесь при его использовании. Мы готовы ответить на Все ваши вопросы, касающиеся этого плагина.

### Обновление с 0.2 на 0.3+

Начиная с версии 0.3 это плагин использует [redmine_telegram_common](https://github
.com/centosadmin/redmine_telegram_common).

Перед обновлением установите [этот](https://github.com/centosadmin/redmine_telegram_common) плагин.

После обновления запустите `bundle exec rake intouch:common:migrate` для миграции пользоватльских данных в новую таблицу. 

В версии 0.4 модель `TelegramUser` будет упразднена, в месте с ней будет удалена старая таблица `telegram_users`.

## Requirements

**Redmine 3.1.x**

Для работы палагина необходимо установить плагин [redmine_sidekiq](https://github.com/ogom/redmine_sidekiq)

[Запуск Sidekiq как демона](https://github.com/mperham/sidekiq/wiki/Deployment#daemonization)

* примеры конфигурационного файла и скрипта для `init.d` находятся в папке `tools`

# Настройка плагина

## Общие настройки

В секции "Протоколы" указываются требуемые протоколы уведомлений. В настоящий момент доступны - telegram и email.

В секции "Рабочие дни" указываются:
* время начала и завершения рабочего дня
* какие дни недели являются рабочими

В секции "Срочные задачи" указываются приоритеты задач, для которых необходимо **всегда** отправлять уведомления,
независимо от времени суток и дня недели

Плагин содержит функционал периодических уведомлений о задачах "В работе" или со статусом "Обратная связь".
Для правильной интерпретации этих статусов плагином, укажите их в соответствующих секциях.

## Telegram

На этой вкладке необходимо указать токен бота Telegram.

### Создание бота Telegram

Бота необходимо зарегистрировать и получить его токен. Для этого в Telegram существует специальный бот — @BotFather.

Пишем ему `/start` и получаем список всех его команд.
Первая и главная — `/newbot` — отправляем ему и бот просит придумать имя нашему новому боту.
Единственное ограничение на имя —
в конце оно должно оканчиваться на «bot».
В случае успеха BotFather возвращает токен бота и ссылку для быстрого добавления бота в контакты,
иначе придется поломать голову над именем.

Полученный токен нужно ввести на странце настройки плагина.

### Запуск бота

Перед запуском бота на странице настройки плагина нужно указать:

* токен бота Telegram (как его получить описано ниже)
* рабочее время - в это время отправляются уведомления по не срочным задачам
* указать какие приоритеты считать срочными
* указать какие статусы считать *в работе* и *обратной связью*

После этого необходимо запустить бота командой:

```shell
bundle exec rake intouch:telegram:bot PID_DIR='/pid/dir'
```

* пример скрипта для `init.d` в папке `tools`

Этот процесс добавляет пользователей Telegram в Redmine, а также создает в Redmine группы Telegram, в которые добавили бота.

### Добавление аккаунта Telegram к пользователю

После того как бот запущен и пользователь поприветствовал его командой `/start`, бот предложит ввести команду `/connect e@mail.com`.

После выполнения команды пользователь получит письмо со ссылкой. Переход по ссылке свяжет аккаунты пользователя и он сможет получать одноразовые пароли от бота.

#### Если бота обновили

Если у вас поменялся бот, то каждому пользователю нужно с ним лично поздороваться.

То есть через поиск найти @YourTelegramBot и написать ему `/start`

### Добавление группы Telegram

Группы добавятся в Redmine автоматически, если в них будет добавлен бот.

Название группы сохраняется сразу при добавлении. Если, какое-то время спустя, вы изменили название группы и хотите,
чтобы в Redmine название также обновилось - выполните команду `/rename` в групповом чате.

## Шаблоны настроек

Шаблоны настроек позволяют один раз задать все требуемые настройки для проектов, а потом в каждом проекте выбрать нужный шаблон.
Подробней о настройках плагина внутри проекта читайте ниже.


## Расписание регулярных уведомлений

В плагине предусмотрены

* Уведомления о задачах со статусом "В работе"
* Уведомления о задачах со статусом "Обратная связь"
* Уведомления о не назначенных задачах
* Уведомления о просроченных задачах

Периодичность и получатели этих уведомлений, настраиваются в каждом проекте индивидуально, либо с использованием шаблонов.

Расписание регулярных уведомлений настраивается на странице настройки плагина, на вкладке **Расписание периодических задач**.

При первой установке плагина, нужно инициализировать периодические задачи.

Для этого нужно нажать ссылку **Инициализировать периодические задачи** на вкладке **Расписание периодических задач** в настройках плагина.

После этого можно настроить удобное вам расписание периодических уведомлений.

Расписание настраивается используя синтаксис CRON.

Важно отметить, что на этой вкладке настраивается то, как часто проверять наличие задач, по которым требуется отправить уведомления.
Периодичность самих уведомлений указывается в каждом проекте индивидуально, либо с использованием шаблонов.


# Настройка модуля внутри проекта

В настройках проекта на вкладке "Модули" нужно выбрать модуль Intouch.
В результате в настройках появится вкладка "Intouch".

На этой вкладке есть три секции:

* Мгновенные уведомления при смене статуса/приоритета задачи
* Периодические уведомления
* Группы исполнителей - уведомления, адресованные Исполнителю, будут отправлены только, если Исполнитель входит в одну из отмеченных в этой секции групп.

## Мгновенные уведомления при смене статуса/приоритета задачи

В этой секции настраиваются мгновенные уведомления для следующих получателей:

* автор
* исполнитель - уведомления, адресованные Исполнителю, будут отправлены только, если Исполнитель входит в одну из групп отмеченных в секции "Группы исполнителей".
* наблюдатели за задачей
* группы Telegram

**Важное замечание: для того, чтобы пользователь Telegram получал сообщения,
нужно чтобы он предварительно написал команду `/start` боту**

## Периодические уведомления

### Общие настройки

В общих настройках указываются интервалы периодических уведомлений для различных приоритетов.

### В работе / Обратная связь

На этих вкладках указываются получатели периодических уведомлений о задачах со статусами "В работе" и "Обратная связь"

### Не назначенные / Назначенные на группу

На этой вкладке указываются получатели периодических уведомлений о задачах
* без назначенного исполнителя
* назначенные на группу

### Просроченные / Без даты завершения

На этой вкладке указываются получатели периодических уведомлений о задачах
* дата завершения которых находится в прошлом
* с неуказанной датой завершения

# FAQ
## Из-за чего бот может не слать сообщения в канал?
Возможно у вас не инициализировано **Расписание периодических задач**.

В настройках плагина зайдите на вкладку **Расписание периодических задач** и нажмите ссылку **Инициализировать периодические задачи**.

# Автор плагина

Плагин разработан [Centos-admin.ru](http://centos-admin.ru/).
