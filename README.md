# vpsssr_library_parser

Для работы необходимы **docker windows**, **git windows**, необязательные программы postman, dbeaver

1. Установить Docker для Windows https://docs.docker.com/desktop/install/windows-install/

2. Установить git для windows https://git-scm.com/downloads При установке можно оставить все по умолчанию

3. Установить программу Postman https://www.postman.com/downloads/

4. Установить программу DBeaver Community, необходима для просмотра данные postgres БД в графическом интерфейсе
   https://dbeaver.io/

После установки необходимых программ, нажмите меню пуск, найдите git консоль Git CMD, запустите

Выберите или создайте папку с проектами, например c:\terratensor,

Для создания наберите в консоли Git CMD
```
mkdir ./terratensor
```

Выбрать созданную папку, набрать в консоли Git CMD:
```
cd ./terratensor
```

Для клонирования репозитория terratensor/vpsssr_library_parser, наберите в консоли Git CMD:

```
git clone https://github.com/terratensor/vpsssr_library_parser.git
```

Увидите сообщение в консоли об успешном клонировании:
```
Cloning into 'vpsssr_library_parser'...
remote: Enumerating objects: 71, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (62/62), done.
remote: Total 71 (delta 3), reused 67 (delta 2), pack-reused 0
Unpacking objects: 100% (71/71), 22.11 MiB | 3.24 MiB/s, done.
```

После наберите в консоли Git CMD

```
cd ./vpsssr_library_parser
```

Затем наберите в консоли
```
ls -la
```

Увидите следующую структуру папки:
```
total 20
drwxrwxrwx 1 username username  4096 Jun 25 20:54 .
drwxrwxrwx 1 username username  4096 Jun 25 20:54 ..
drwxrwxrwx 1 username username  4096 Jun 25 20:54 .git
-rwxrwxrwx 1 username username  1526 Jun 25 20:54 LICENSE
-rwxrwxrwx 1 username username 11675 Jun 25 20:54 README.md
drwxrwxrwx 1 username username  4096 Jun 25 20:54 books
drwxrwxrwx 1 username username  4096 Jun 25 20:54 docker
-rwxrwxrwx 1 username username   950 Jun 25 20:54 docker-compose.yml
```

Скачайте последнюю версию парсера book-parser-vpsssr.exe.
Ссылка на файл находится в секции Assets страницы описания релиза

https://github.com/terratensor/book-parser/releases/latest

Сохраните book-parser-vpsssr.exe в папке с проектом .terratensor/vpsssr_library_parser

Далее запустите windows docker (через меню пуск)

После запуска необходимо проверить, что нет запущенных контейнеров с manticoresearch и postgres (меню containers), если есть остановить их или удалить, иначе будет конфликт доступа к портам.

В консоли Git CMD, где ранее создали папку и клонировали репозиторий для запуска контейнера с парсером vpsssr_library_parser набрать:

```
docker-compose down --remove-orphans
```

```
docker compose up --build -d
```

запустится база данных postgres и manticore search

- Будет создана локальная БД с именем vpsssr-library
- Имя пользователя app
- Пароль secret
- Порт для подключения к БД 54323

Эти настройки можно увидеть или изменить в файле docker-compose.yml
```
    ports:
      - "54323:5432"
    environment:
      APP_ENV: dev
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: vpsssr-library
```

В папке `books` по умолчанию размещены 50 толстых книг ВП СССР, при необходимости вы можете изменить путь до папки с книгами. С помощью параметра `-o` можно указать любой другой путь к нужной папке, в конце пути обязательно поставить слэш

Запустите book-parser-vpsssr.exe:

```
./book-parser-vpsssr.exe
```

Или запустите book-parser-common.exe с параметром -o и укажите нужную папку, где расположены файлы для обработки, например, для другой своей папки `/books/VPSSSR/DOCX/`:

```
./book-parser-vpsssr -o ./books/VPSSSR/DOCX/
```

Будет произведена обработка docx файлов и запись их в таблицы БД:
```
db_vpsssr_books
    id, name, filename, created_at, updated_at, deleted_at

db_vpsssr_paragraphs
    id, book_id, book_name, text, position, length, created_at, updated_at, deleted_at 
```

При обработке файлов в консоли после каждой успешно обработанной книги будет появляться сообщение с наименованием и номером файла:
```
2023/06/25 21:19:46 Об иcкоренении глобальной угрозы «международного терроризма».docx #26 done
2023/06/25 21:19:46 Об имитационно-провокационной деятельности.docx #27 done
```
Иногда в процессе на экран могут вылетать большие «портянки» текста с параграфами - это лог медленного запроса, который выполнялся в БД более 200 мс,
не обращайте на это внимание, это информационное сообщение, книга и параграфы в этом случае все равно сохранены в БД.
```
[211.031ms] [rows:749] INSERT INTO "db_vpsssr_paragraphs" ("book_id","book_name","text","position","length","created_at","updated_at","deleted_at") VALUES (1,'Вера и Мера','<p>ВНУТРЕННИЙ ПРЕДИКТОР СССР</p>',1,
32,'2023-06-25 20:37:25.581','2023-06-25 20:37:25.581',NULL),(1,'Вера и Мера','<p>Вhра и Мhра</p>',2,18,'2023-06-25 20:37:25.581','2023-06-25 20:37:25.581',NULL),(1,'Вера и Мера','<p>Санкт-Петербург</p>',3,22,
'2023-06-25 20:37:25.581','2023-06-25 20:37:25.581',NULL),(1,'Вера и Мера','<p>1999 г.</p>',4,14,'2023-06-25 20:37:25.581','2023-06-25 20:37:25.581',NULL),(1,'Вера и Мера','<p>Страница, зарезервированная для в
ыходных типографских данных</p>',5,67,'2023-06-25 20:37:25.581','2023-06-25 20:37:25.581',NULL)
...
,(1,'Вера и Мера','<p>Уточнения: 21 — 26 февраля, 9 — 21 марта 1999 г.</p>',748,55,'2023-06-25 20:37:25.5
81','2023-06-25 20:37:25.581',NULL),(1,'Вера и Мера','<h1>Примечания</h1>',749,19,'2023-06-25 20:37:25.581','2023-06-25 20:37:25.581',NULL) RETURNING "id"
```


После обработки всех файлов в консоли будет завершающее сообщение:

```
...
2023/06/22 11:11:53 all files done
```

Для запуска индексации данных из БД в мантикору запустите manticore indexer:

```
docker exec -it book-parser-manticore indexer vpsssr_library
```

После того как данный будут проиндексированы необходимо **обязательно** перезапустить контейнеры vpsssr_library_parser, сделать это можно командами:

```
docker compose down
```

```
docker compose up -d
```

Теперь можно запустить программу postman

В программе Postman сделать вкладку и в адресе указать:
```
POST localhost:9308/search
```
![image](https://github.com/audetv/book-parser/assets/129882753/aad0a4c2-e213-46ac-a615-4ac98a8a7f82)


Ниже выбрать Body, ниже raw, рядом JSON

В поле ниже скопировать запрос:

```
{
    "index": "vpsssr_library",
    "highlight": {
        "fields": [
            "text"
        ],
        "limit": 0,
        "no_match_size": 0
    },
    "max_matches": 353545,
    "limit": 50,
    "offset": 0,
    "query": {
        "bool": {
            "must": [
                {
                    "query_string": "мера"
                }
            ]
        }
    }
}
```
В match_phrase, вместо "мера", можно набрать любой поисковый запрос в кавычках, например "концепция"

"limit" - кол-во записей на странице
"offset" - смещение от начала.

Если надо посмотреть следующие 100 записей, изменить offset, например:
```
"limit": 100
"offset": 100
```

--------------------------
### Дополнительные сведения:

**Остановка контейнеров book-parser-vpsssr**

```
docker-compose down --remove-orphans
```

**Запуск контейнеров book-parser-vpsssr из папки проекта**

```
docker compose up --build -d
```

### Первый запуск manticore indexer, если еще не создана таблица common_library

```
docker exec -it book-parser-manticore indexer vpsssr_library
```

### Повторный запуск manticore indexer, если таблица существует, для переиндексации

```
docker exec -it book-parser-manticore indexer vpsssr_library --rotate
```

#### Бэкап postgres БД:
```
docker exec -it book-parser-postgres bash
```

```
pg_dump --dbname=vpsssr-library --username=app --host=postgres-book-parser | gzip -9 > vpsssr-library_backup-filename.gz
```

Скопировать файл бэкапа в контейнер докера для восстановления БД, распаковать из gz, запустить загрузку бэкапа в БД

```
cp vpsssr-library_backup-filename.gz book-parser-postgres:app/vpsssr-library_backup-filename.gz
```
```
gzip -d vpsssr-library_backup-filename.gz
```
```
psql -U app -d lib < vpsssr-library_backup-filename.sql
```

#### Остановка и удаление БД. ВНИМАНИЕ ДАННЫЕ БУДУТ УДАЛЕНЫ.

`docker compose down -v --remove-orphans`

### Опции командной строки

```
-h --help помощь
-b, --batchSize int   размер пакета по умолчанию (default batch size) (default 3000)
-o, --output string   путь хранения файлов для обработки (default "./books/")
```
