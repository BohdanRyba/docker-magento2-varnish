#Magento 2 Docker

##Реєстр готових (pre-build) PHP імеджів
Всі доступні версії PHP контейнерів доступні за посиланням: [`https://web.docker.pp.ua/#!taglist/magento2-php`](https://web.docker.pp.ua/#!taglist/magento2-php)

##E-MAIL
Для листів встановлена локальна загрушка, яка не дозволить відправити лист в Internet, а лише збереже його локально для перегляду.

Переглянути всі отримані листи можна за посиланням: [`http://localhost:8025/`](http://localhost:8025/)

![](https://i.imgur.com/GaarM5O.png)

##Генерація SSL сертифікатів
Для генерації сертифіката потрібно:
- перейти в директорію `ssl/ssl_generator`
- виконати команду `./create.sh sample.test`, де:
    - `sample.test` - хост для якого потрібно згенерувати сертифікат
    - ![](https://i.imgur.com/oxl7utN.png)
- повинна створитись нова директорія з вашим хостом:
    - ![](https://i.imgur.com/Zx4t8qz.png)
- необхідно вказати цей хост у `docker-compose.yml` файлі:
    - ![](https://i.imgur.com/VXdd8JB.png)
- імпортувати кореневий сертифікат в браузер, для того щоб браузер міг довіряти самопідписаним сертифікатам:
    - Chrome - Settings - [Privacy and security] Security - [Advanced] Manage certificates - Authorities
    - натиснути кнопку Import
    - ![](https://i.imgur.com/BjtlZ9X.png)
    - перейти у директорію з проектом `docker-magento2/ssl/ssl_generator`
    - змінити фільтр на `All Files`
    - ![](https://i.imgur.com/jsJOSJR.png)
    - вибрати файл `rootCA.crt`
    - відмітити 3 чекбокси і натиснути ОК
    - ![](https://i.imgur.com/hjMKC8T.png)
- Готово
    - ![](https://i.imgur.com/mBxnMks.png)

##Конфігурація та параметри
Для конфігурування мадженти використовуються наступні параметри:

![](https://i.imgur.com/QAUICmD.png)

- **`VIRTUAL_HOST`** - хостнейм, який буде використовуватись як адреса веб-сайту
- **`MAGE_MODE`** - режим роботи Magento (`default`, `developer`, `production`, `maintenance`)
- **`MAGE_RUN_TYPE`** - тип коду для ініціалізації (`store`, `website`)
- **`MAGE_RUN_CODE`** - код store або website (залежить від параметра `MAGE_RUN_TYPE`), який буде використовуватись для ініціалізації Magento.

Примітка:
- `MAGE_RUN_TYPE =`**`store`**:
    - `MAGE_RUN_CODE` дивитись БД у таблиці `store`
    - ![](https://i.imgur.com/BSjJBXc.png)

- `MAGE_RUN_TYPE =`**`website`**:
    - `MAGE_RUN_CODE` дивитись БД у таблиці `store_website`
    - ![](https://i.imgur.com/GlJaahP.png)

##Права доступу до директорій:
Для правильної роботи Magento необхідно надати право на запис у наступні директорії:
- `var`
- `app/etc`
- `pub/media`
- `pub/static`

Для цього необхідно виконати команду (знаходитись у директорії проекту):
- **`sudo chmod -R 777 code/var code/app/etc code/pub/media code/pub/static`**

##Переглянути активні контейнери:

Команда **`docker ps`**

![](https://i.imgur.com/l5WV11H.png)

- `CONTAINER ID` - ID контейнера
- `IMAGE` - образ контейнера на якому він працює
- `COMMAND` - головна команда контейнера (задається при створенні в `Dockerfile`)
- `CREATED` - дата створення
- `STATUS` - час роботи від старту
- `PORTS` - показує як локальний (доступний на ПК) порт співвідноситься до порту в контейнері Docker
- `NAMES` - ім'я контейнера - використовується у командах по роботі з контейнером

##CLI в контейнері
Як заходити в PHP контейнер, щоб не зіпсувати права доступу:

**`docker exec -ti magento2_php su www-data -s /bin/bash`** - за допомогою цієї команди ви відкриєте CLI контейнера і там вже виконувати усі решту маніпуляції
- `magento2_php` - ім'я контейнера PHP (див. секцію `Переглянути активні контейнери`)

##Імпорт бази даних з backup-файлу:
**`cat backup.sql | docker exec -i CONTAINER_NAME /usr/bin/mysql -u ROOT --password=PASSWORD DATABASE`**:
- `backup.sql` - файл для імпорту
- `CONTAINER_NAME` - контейнер з MySQL (по замовчуванню `magento2_mysql`)
- `ROOT` - ім'я користувача MySQL (по замовчуванню `root`)
- `PASSWORD` - пароль користувача MySQL (по замовчуванню `magento2`)
- `DATABASE` - база данних MySQL (по замовчуванню `magento2`)

##Backup (dump) бази у файл:
**`docker exec CONTAINER_NAME /usr/bin/mysqldump -u ROOT --password=PASSWORD DATABASE > backup.sql`**:
- `CONTAINER_NAME` - контейнер з MySQL (по замовчуванню `magento2_mysql`)
- `ROOT` - ім'я користувача MySQL (по замовчуванню `root`)
- `PASSWORD` - пароль користувача MySQL (по замовчуванню `magento2`)
- `DATABASE` - база данних MySQL (по замовчуванню `magento2`)
- `backup.sql` - файл для імпорту

##Використовуємо докер локально
- Клонуємо GIT-репозиторій на локальний комп'ютер: `git clone https://github.pp.ua/anbis/docker-magento2.git`
- Переходимо у директорію, яка була створена у попередньому пункті `cd docker-magento2`
- Запускаємо контейнери `docker-compose up`
- Перевіряємо статус контейнерів `docker ps` (у новому вікні терміналу): 
    - має бути активно 5 контейнерів (`nginx`, `php`, `mysql`, `mailhog`, `redis`)
    - ![](https://i.imgur.com/sJxjcgs.png)
- переходимо в контейнер PHP `docker exec -ti magento2_php bash`

##Завантаження Magento
- за допомогою **`СOMPOSER`**
    - `composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition` - завантажуємо файли за допомогою Composer
    - `mv project-community-edition/{*,.*} .` - переміщуємо файли на рівень нижче (можуть бути повідомлення про помилку - ігноруємо)
    - `rm -rf project-community-edition` - видаляємо директорію, вона нам більше не потрібна
    - `chown -R www-data:www-data .` - змінюємо права доступу після завантаження на коректні
    - `exit` - виходимо з терміналу контейнера
    
- за допомогою **`GIT`**
    - to be continued

##Встановлення Magento
- `bin/magento` - перевіряємо чи працює консоль Magento
    - ![](https://i.imgur.com/jVwRPRS.png)
- **`bin/magento setup:install --db-host=magento2_mysql --db-name=magento2 --db-user=magento2 --db-password=magento2 --base-url=https://magento2.dev/ --backend-frontname=admin --admin-user=anbis --admin-password=magent0 --admin-email=a@b.com --admin-firstname=a --admin-lastname=b --language=en_US --currency=USD --timezone=America/Chicago --cleanup-database --use-rewrites=1`**
    - `db-host=magento2_mysql` - хост бази даних (БД)
    - `db-user=magento2` - користувач БД
    - `db-password=magento2` - пароль користувача БД
    - `db-name=magento2` - назва БД
    - `base-url=https://magento2.dev/` - URL для доступу Magento з браузера (`magento2.dev`, `magento2.local`, `magento2.test`)
    - `backend-frontname=admin` - URL-постфікс для доступу в адмін-панель
    - `admin-user=anbis` - користувач для адмін-панелі
    - `admin-password=magent0` - пароль користувача адмін-панелі
    - `admin-email=test@example.com` - email користувача адмін-панелі
    - `admin-firstname=name` - ім'я користувача адмін-панелі
    - `admin-lastname=surname` - прізвище користувача адмін-панелі
    - `language=en_US` - локаль магазину
    - `currency=USD` - валюта магазину
    - `timezone=America/Chicago` - часова зона магазину
    - `cleanup-database` - очистити базу перед встановленням Magento
    - `use-rewrites=1` - використовувати SEO-friendly URL для категорій і продуктів