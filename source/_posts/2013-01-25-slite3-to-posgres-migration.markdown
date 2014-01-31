---
author: admin
comments: true
date: 2013-01-25 08:12:16+00:00
layout: post
slug: slite3-to-posgres-migration

permalink: /2013/01/slite3-to-posgres-migration

title: Перенос sqlite3 базы данных в Postgres (Heroku)
wordpress_id: 961
categories:
- db
- программирование
tags:
- db
- heroku
- mysql
- postgre
- sqlite
---

Иногда бывает крайне необходимо конвертировать имеющуюся базу данных в одном формате в формат другой. Сегодня я расскажу вам как это быстро и безболезненно (почти) сделать.<!-- more -->

Для начала нам понадобится замечательный [gem taps](https://github.com/ricardochimal/taps) и запущенный с его помощью локальный сервер с бекендом из Sinatra.

Т.к. часто создавая новый проект, не хочется особо заморачиваться и создавать для этого отдельную базу данных в Postgres, я обычно использую базу данных по умолчанинию, в рельсах это, как вам известно, sqlite. Но бывает иногда необходимо скопировать данные с локальной базы данных в удаленную, к примеру, если вы хостите проект на heroku, где по умолчанию используется postgres.

Итак, поехали, для начала создаем сервер:

[cc]taps server sqlite://db/development.sqlite3 user pass[/cc], где user - это имя пользователя для доступа к нашему серверу (серверу taps), pass соответственно пароль.

Дальше нам нужно скопировать нашу sqlite базу данных в postgres, для этого достаточно запустить одну команду:

[cc]taps pull postgres://zudochkin@localhost/project_development http://user:pass@localhost:5000[/cc], где postgres - это адаптер базы данных, вы можете смело использовать mysql, zudochkin - имя пользователя для postgres, project_development - имя базы данных, куда я собираюсь импортировать данные, http://user:pass@localhost:5000 - коннектимся к нашему серверу, где в качестве имени пользователя указали user, в качестве пароля pass, а крутится все это дело по умолчанию на 5000 порте.

Теперь, если зайти и посмотреть через pgAdmin в нашу postgres базу данных, то мы увидим данные, скопированные из sqlite3.

Дальше необходимо сделать postgres дамп, для того, чтобы импортировать его в heroku базу данных.

[cc]pg_dump -Fc --no-acl --no-owner -h localhost -U zudochkin project_development > project_development.dump[/cc]

Загружаем дамп куда-нибудь в интернет: подойдет любой фтп или что-нибудь наподобие [CloudApp](http://getcloudapp.com), я использую последнее, потому что есть нативный клиент под Mac и он удобен в работе.

Дальше получаем URL для доступа к postgres базе данных heroku, для этого в папке с приложением выполнеяем (или из любой папке, но дописав к команде **--app %app_name%**) [cc]heroku config | grep DATABASE_URL[/cc] и увидим длинный адрес, это и есть ссылка для доступа к БД. Если вы, как я используете cloudApp, то знайте, что, когда вы копируете ссылку на файл, это, не прямая ссылка и поэтому загрузить на heroku дамп у вас не получится, для того, чтобы увидеть реально прямую ссылку необходимо набрать следующее [cc]curl -I http://cl.ly/2h1k1X3Z2j4f/project_development.dump | grep Location[/cc] и вы увидите прямую ссылку после слова **"Location: "**. Длинный и запутанный адрес это ссылка, которую вам дал cloudApp для доступа к файлу.

![sqlite3 to postgres migration](http://vredniy.ru/wp-content/uploads/2013/01/Screen-Shot-2013-01-25-at-12.05.28-PM1-300x146.png)


После этих манипуляций давайте запустим импорт базы данных из папки с проектом [cc]heroku pgbackups:restore DATABASE 'http://f.cl.ly/items/4G2u393d432U3V0u2X1d/project_development.dump'[/cc] если все прошло нормально, то мы увидим
[cc]
Retrieving... done
Restoring... done
[/cc], если же что-то пошло не так и вы увидели сообщение об ошибке, напишите [cc]heroku logs --ps pgbackups[/cc] и вы увидите что именно пошло не так.

Надеюсь, данная заметка вам поможет перенести локальные данные на heroku или просто сделать из sqlite3 базы данных postgres или mysql.
