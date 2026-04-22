# Отчет и инструкция по ДЗ №5 Neo4j

Источник задания: `/Users/gantulga/Downloads/ДЗ-2025-2026-2.docx`.

В документе ДЗ №5 требует развернуть Neo4j в Docker, загрузить датасет Spotify Artist Feature Collaboration Network, самостоятельно спроектировать графовую модель, создать индексы и constraints, выполнить обязательные Cypher-запросы и выбрать любые 2 задания из 4 дополнительных.

## Использованные ссылки из задания

| Ссылка | Для чего используется |
| --- | --- |
| [Spotify Artist Feature Collaboration Network](https://www.kaggle.com/datasets/jfreyberg/spotify-artist-feature-collaboration-network) | основной датасет, файлы `nodes.csv` и `edges.csv` |
| [Google Drive dataset link](https://drive.google.com/open?id=1fBshyKl66cEyB2Ofohf0YoYO0sXrFpZL) | альтернативная ссылка на датасет из документа |
| [Neo4j Graph Data Science](https://neo4j.com/product/graph-data-science/) | библиотека GDS для графовых алгоритмов |
| [GDS overview](https://bigdataschool.ru/blog/graph-data-science-library-in-neo4j-overview/) | обзор GDS на русском |
| [Neo4j GDS Docker docs](https://neo4j.com/docs/graph-data-science/current/installation/installation-docker/) | установка GDS как Docker-плагина |

По описанию Kaggle, датасет содержит данные примерно о 20 тыс. артистов из weekly charts Spotify и дополнительных артистах, которые участвовали в featuring-коллаборациях. Всего получается сеть более чем из 135 тыс. артистов и 300 тыс. связей коллабораций.

## Структура проекта

```text
hw5_neo4j
├── docker-compose.yml
├── requirements.txt
├── import
│   └── .gitkeep
├── output
│   └── .gitkeep
├── cypher
│   ├── 01_import_artists.cypher
│   ├── 02_profile_before_schema.cypher
│   ├── 03_schema.cypher
│   ├── 04_profile_after_schema.cypher
│   ├── 05_import_genres.cypher
│   ├── 06_import_collaborations.cypher
│   ├── 07_required_queries.cypher
│   ├── 08_task1_bridges_gds.cypher
│   ├── 09_task2_recommendations.cypher
│   ├── 10_task3_genre_ecosystem.cypher
│   └── 11_task4_shortest_path.cypher
└── scripts
    ├── prepare_data.py
    └── spotify_graph_report.py
```

## Модель графа

Я выбрал модель, близкую к рекомендованной в задании.

Узлы:

| Узел | Свойства |
| --- | --- |
| `Artist` | `spotify_id`, `name`, `followers`, `popularity`, `genres_raw`, `chart_hits` |
| `Genre` | `name` |

Связи:

| Связь | Смысл |
| --- | --- |
| `(:Artist)-[:HAS_GENRE]->(:Genre)` | артист относится к жанру |
| `(:Artist)-[:COLLABORATED_WITH]-(:Artist)` | два артиста участвовали в одной feature-коллаборации |

Связь `COLLABORATED_WITH` импортируется как ненаправленная логическая связь. В Neo4j физически связь всегда имеет направление, но в запросах используется шаблон `-[:COLLABORATED_WITH]-`, а в GDS-проекции связь дополнительно задаётся как undirected.

## Задание 1. Развернуть Neo4j в Docker

Перейти в проект:

```shell
cd /Users/gantulga/ganbold_db_hometasks/hw5_neo4j
```

Запустить Neo4j:

```shell
docker compose up -d
```

Проверить контейнер:

```shell
docker compose ps
```

Открыть Neo4j Browser:

```text
http://localhost:7474
```

Данные для входа:

```text
login: neo4j
password: spotify123
```

В `docker-compose.yml` подключен плагин `graph-data-science`, потому что в задании рекомендуется GDS.

Проверка GDS:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 "RETURN gds.version();"
```

## Задание 2. Скачать и подготовить данные

Скачать датасет по одной из ссылок:

1. [Kaggle Spotify Artist Feature Collaboration Network](https://www.kaggle.com/datasets/jfreyberg/spotify-artist-feature-collaboration-network)
2. [Google Drive dataset link](https://drive.google.com/open?id=1fBshyKl66cEyB2Ofohf0YoYO0sXrFpZL)

Из архива взять файлы:

```text
nodes.csv
edges.csv
```

Положить их сюда:

```text
/Users/gantulga/ganbold_db_hometasks/hw5_neo4j/import/nodes.csv
/Users/gantulga/ganbold_db_hometasks/hw5_neo4j/import/edges.csv
```

Установить зависимости и подготовить чистые CSV для Neo4j:

```shell
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
python scripts/prepare_data.py
```

Скрипт создаст:

| Файл | Назначение |
| --- | --- |
| `import/artists.csv` | очищенные артисты |
| `import/artist_genres.csv` | нормализованная таблица артист-жанр |
| `import/collaborations.csv` | пары артистов и количество коллабораций |

## Задание 3. Загрузить данные в Neo4j

Запускать строго по шагам:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/01_import_artists.cypher
```

После этого сделать скриншот результата импорта артистов.

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/02_profile_before_schema.cypher
```

После этого сделать скриншот `PROFILE` до создания индексов и constraints.

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/03_schema.cypher
```

После этого сделать скриншот создания schema.

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/04_profile_after_schema.cypher
```

После этого сделать скриншот `PROFILE` после создания индексов и constraints.

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/05_import_genres.cypher
```

После этого сделать скриншот импорта жанров.

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/06_import_collaborations.cypher
```

После этого сделать скриншот импорта связей коллабораций.

Файлы `02_profile_before_schema.cypher` и `04_profile_after_schema.cypher` нужны для демонстрации разницы до и после constraints/indexes. В отчёт нужно вставить оба плана выполнения: до индекса должен быть скан по метке, после создания constraint по `spotify_id` должен использоваться индексный поиск.

Созданные ограничения и индексы:

| Объект | Назначение |
| --- | --- |
| `artist_spotify_id_unique` | уникальность `Artist.spotify_id` и быстрый поиск артиста |
| `genre_name_unique` | уникальность `Genre.name` |
| `artist_name_index` | быстрый поиск по имени артиста |
| `artist_popularity_index` | ускорение сортировок и фильтров по popularity |
| `artist_followers_index` | ускорение сортировок и фильтров по followers |

## Задание 4. Обязательные запросы

Запустить:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/07_required_queries.cypher
```

В файле есть:

1. Проверка количества узлов по labels.
2. Проверка количества связей по типам.
3. `SHOW CONSTRAINTS`.
4. `SHOW INDEXES`.
5. Топ-10 артистов по количеству коллабораций.
6. Самые жанрово-разнообразные артисты по жанрам их коллабораторов.
7. Топ жанров по количеству артистов.

## Задания на выбор

Для сдачи я выбираю два задания:

1. Задание 2: рекомендательная система на графах.
2. Задание 3: жанровая экосистема.

Дополнительно подготовлены скрипты для задания 1 и задания 4, чтобы при желании можно было показать больше аналитики.

### Задание 1. Музыкальные мосты

Файл:

```text
cypher/08_task1_bridges_gds.cypher
```

Запуск:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/08_task1_bridges_gds.cypher
```

Подход: строится GDS-проекция подграфа артистов из Hip-Hop/Rap и Pop, затем считается Betweenness Centrality. Артисты с высоким score являются мостами, потому что через них проходит много кратчайших путей между разными частями графа.

### Задание 2. Рекомендательная система

Файл:

```text
cypher/09_task2_recommendations.cypher
```

Запуск:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/09_task2_recommendations.cypher
```

Подход: используется Common Neighbors. Для выбранного артиста ищутся артисты второго круга, с которыми он ещё не сотрудничал напрямую, но с которыми у него много общих коллабораторов.

Интерпретация результата:

| Поле | Значение |
| --- | --- |
| `base_artist` | артист, для которого строится рекомендация |
| `recommended_artist` | рекомендованный артист |
| `common_neighbors` | число общих коллабораторов |
| `genres` | жанры рекомендованного артиста |
| `via_artists` | через каких общих артистов появилась рекомендация |

### Задание 3. Жанровая экосистема

Файл:

```text
cypher/10_task3_genre_ecosystem.cypher
```

Запуск:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/10_task3_genre_ecosystem.cypher
```

Подход: строится логический граф жанров. Два жанра считаются связанными, если артист одного жанра сотрудничал с артистом другого жанра.

Запросы отвечают на вопросы:

1. Какие жанры наиболее центральны.
2. Какие пары жанров имеют самую сильную связь.
3. Есть ли изолированные жанры.

### Задание 4. Шесть шагов до любого артиста

Файл:

```text
cypher/11_task4_shortest_path.cypher
```

Запуск:

```shell
docker exec -i hw5-neo4j cypher-shell -u neo4j -p spotify123 < cypher/11_task4_shortest_path.cypher
```

Подход: используется `shortestPath` между `Taylor Swift` и `Diddy` с глубиной до 6 коллабораций.

## Python-код и визуализации

Для выбранных заданий 2 и 3 добавлен Python-скрипт:

```text
scripts/spotify_graph_report.py
```

Запустить:

```shell
. .venv/bin/activate
python scripts/spotify_graph_report.py
```

Можно выбрать другого артиста:

```shell
TARGET_ARTIST="Billie Eilish" python scripts/spotify_graph_report.py
```

Скрипт создаёт в `output`:

| Файл | Содержимое |
| --- | --- |
| `top_collaborations.csv` | топ артистов по degree |
| `genre_diversity.csv` | артисты с наиболее разнообразными жанрами коллабораторов |
| `recommendations.csv` | рекомендации для выбранного артиста |
| `genre_ecosystem.csv` | сильнейшие пары жанров |
| `top_collaborations.png` | график топа по коллаборациям |
| `genre_diversity.png` | график жанрового разнообразия |
| `recommendations.png` | график рекомендаций |
| `genre_ecosystem.png` | график связей жанров |

## Что вставить в отчёт

Минимальный набор скриншотов:

1. `docker compose ps` с контейнером `hw5-neo4j`.
2. Вход в Neo4j Browser на `http://localhost:7474`.
3. Результат `RETURN gds.version();`.
4. Результат `python scripts/prepare_data.py`.
5. План `PROFILE` до создания schema.
6. План `PROFILE` после создания schema.
7. `SHOW CONSTRAINTS`.
8. `SHOW INDEXES`.
9. Топ-10 артистов по количеству коллабораций.
10. Самый жанрово-разнообразный артист.
11. Результаты задания 2 по рекомендациям.
12. Результаты задания 3 по жанровой экосистеме.
13. PNG-графики из папки `output`.

## Выводы для отчёта

Короткий текст, который можно адаптировать после получения реальных результатов:

```text
В работе была построена графовая модель Spotify Artist Feature Collaboration Network. Узлы Artist хранят идентификатор Spotify, имя, популярность, число подписчиков и исходные жанры. Узлы Genre выделены отдельно, что позволяет анализировать не только связи артистов, но и взаимодействие жанров.

Связь COLLABORATED_WITH показывает факт feature-коллаборации между двумя артистами и используется как ненаправленная связь. Такая модель удобна для поиска центральных артистов, рекомендаций и кратчайших путей.

Constraint по Artist.spotify_id ускорил импорт связей и точечный поиск артиста. Это видно по PROFILE: до schema Neo4j выполнял скан по Artist, после schema использовал индекс.

Для рекомендаций использован подход Common Neighbors: чем больше общих коллабораторов у двух артистов, тем выше вероятность релевантной рекомендации. Для жанровой экосистемы жанры связывались через коллаборации артистов, что показывает самые активные межжанровые пересечения.
```

## Остановка

Остановить контейнер:

```shell
docker compose down
```

Остановить контейнер и удалить данные Neo4j:

```shell
docker compose down -v
```
