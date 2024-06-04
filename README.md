# DataLens

[**DataLens**](https://datalens.tech) is a modern business intelligence and data visualization system. It was developed and extensively used as a primary BI tool in Yandex and is also available as a part of [Yandex Cloud](https://datalens.yandex.com) platform. See also [our roadmap](https://github.com/orgs/datalens-tech/projects/1) and [community in telegram](https://t.me/YandexDataLens).

## Мои примечания
С репозитория `https://github.com/datalens-tech/datalens` должны быть скачены:
* datalens-backend
* datalens-ui
* datalens-us (данный модуль изменён и хранится тут `https://github.com/akrasnov87/datalens-us`). Внесены доработки для интергации с RPC на NodeJS

## Getting started

### Installing Docker

DataLens requires Docker to be installed. Follow these instructions depending on the platform you use:

- [macOS](https://docs.docker.com/desktop/install/mac-install/)
- [Linux](https://docs.docker.com/engine/install/)
- [Windows](https://docs.docker.com/desktop/install/windows-install/)

### Running containers

Use the following command to start DataLens containers:

```bash
git clone https://github.com/datalens-tech/datalens && cd datalens

HC=1 docker compose up

# or with an external metadata database
METADATA_POSTGRES_DSN_LIST="postgres://{user}:{password}@{host}:{port}/{database}" HC=1 docker compose up
```

This command will launch all containers required to run DataLens and UI will be available on http://localhost:8080

If you want to use a different port (e.g. `8081`), you can set it using the `UI_PORT` env variable:

```bash
UI_PORT=8081 docker compose up
```

Для подключения к RPC требуется указать аргумент `NODE_RPC_URL`, Данный аргумент позволяет прокидывать авторизованные данные между `datalens` и `rpc node`.

```bash
NODE_RPC_URL=http://localhost:5000/dev/rpc docker compose up
```


<details>
      <summary>Notice on Highcharts usage</summary>

      Highcharts is a proprietary commercial product. If you enable highcharts in your DataLens instance (with `HC=1`` variable), you should comply with Highcharts license (https://github.com/highcharts/highcharts/blob/master/license.txt).

      When Highcharts is disabled in DataLens, we use D3.js instead. However, currently only few visualization types are compatible with D3.js. We are actively working on adding D3 support to additional visualizations and are going to completely replace Highcharts with D3 in DataLens.

</details>

## How to update

Just pull the new `docker-compose.yml` and restart.

```bash
docker compose down
git pull
docker compose up
```

All your user settings will be stored in the `metadata` folder.

## Parts of the project

DataLens consists of the three main parts:

- [**UI**](https://github.com/datalens-tech/datalens-ui) is a SPA application with corresponding Node.js part. It provides user interface, proxies requests from users to backend services and also applies some light data postprocessing for charts.
- [**Backend**](https://github.com/datalens-tech/datalens-backend) is a set of Python applications and libraries. It is responsible for connecting to data sources, generating queries for them and post-processing the data (including formula calculations). The result of this work is an abstract dataset that can be used in UI for charts data request.
- [**UnitedStorage (US)**](https://github.com/datalens-tech/datalens-us) is a Node.js service that uses PostgreSQL to store metadata and configuration of all DataLens objects.

## What's already available

We are releasing DataLens with first minimal set of available connectors (clickhouse, clickhouse over ytsaurus and postgresql) as well as other core functionality such as data processing engine and user interface. However, to kick off this project in a reasonable timeframe we have chosen to drop some of the features out of the first release: this version does not contain middleware and components for user sessions, object ACLs and multitenancy (although code contains entry-points for such extensions). We are planning to add missing features based on our understanding of community priorities and your feedback.

## Cloud Providers
Below is a list of cloud providers offering DataLens as a service:
1. [Yandex Cloud](https://datalens.yandex.com) platform
2. [DoubleCloud](https://double.cloud/services/doublecloud-visualization/) platform

## FAQ

#### Where does DataLens store it's metadata?

We use the `metadata` folder to store PostgreSQL data. If you want to start over, you can delete this folder: it will be recreated with demo objects on the next start of the `datalens-us` container.

#### I use the `METADATA_POSTGRES_DSN_LIST` param for external metadata database and the app doesn't start. What could be the reason?

We use some PostgresSQL extensions for the metadata database and the application checks them at startup and tries to install them if they haven't been already installed. Check your database user's rights for installing extensions by trying to install them manually:

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE EXTENSION IF NOT EXISTS btree_gin;
CREATE EXTENSION IF NOT EXISTS btree_gist;
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

If this attempt is unsuccessful, try to install dependencies by database admin and add param `METADATA_SKIP_INSTALL_DB_EXTENSIONS=1` on startup, this parameter allows the app to skip installing extensions.

If you're using managed database, it's also possible that extensions for your database cluster are controlled by external system and could be changed only using it's UI or API. In such case, consult with documentation for managed database service which you're using. Don't forget to add `METADATA_SKIP_INSTALL_DB_EXTENSIONS=1` after installing extensions this way.

#### My PostgresSQL cluster has multiple hosts, how can I specify them in `METADATA_POSTGRES_DSN_LIST` param?

You can write all cluster hosts separated by commas:

`METADATA_POSTGRES_DSN_LIST="postgres://{user}:{password}@{host_1}:{port}/{database},postgres://{user}:{password}@{host_2}:{port}/{database},postgres://{user}:{password}@{host_3}:{port}/{database}" ...`

#### How can I specify custom certificate for connecting to metadata database?

You can add additional certificates to the database in `./certs/root.crt`, they will be used to connect to the database from the `datalens-us` container.

If `datalens-us` container does not start even though you provided correct certificates, try to change `METADATA_POSTGRES_DSN_LIST` like this:
`METADATA_POSTGRES_DSN_LIST="postgres://{user}:{password}@{host}:{port}/{database}?sslmode=verify-full&sslrootcert=/certs/root.crt"`


#### Why do i see two compose files: docker-compose.yml & docker-compose-dev.yml?

`docker-compose-dev.yml` is a special compose file that is needed only for development purposes. When you run DataLens in production mode, you always need to use `docker-compose.yml`. The `docker-compose up` command uses it by default. 

<<<<<<< HEAD
#### Хочу отключить авторизацию RPC

Просто закомментируйте `NODE_RPC_URL` в `docker-compose.yml`

#### Как передать переменные

Лучше создать в корне проекта файл .env и записать туда:
<pre>
HC="1"
NODE_RPC_URL="http://10.10.6.100:5000/dev/rpc"
</pre>

Для запуска выполнить `docker compose --env-file ./.env up`

#### Почему при запуске через авторизацию система не работает

Лучше вначале запусить создание БД с выключенным параметром `NODE_RPC_URL` для контейнера `datalens-us`, а уже затем только включить этот аргумент. В таком случаи не должно быть ошибок связанных с "версией дадасета" (код ошибок 500).

## Получение последних изменений

<pre>
git remote add upstream https://github.com/datalens-tech/datalens.git
git pull upstream main
</pre>

## Инструкция по разворачиванию локальной копии

- запустить первияный проект
- сделать копию us-db-ci_purgeable
- через pg_admin сделать резервное копирование данных (только данных) - в начтройках использовать INSERT
- применить инструкцию в проекте (нуждно исправить ошибку с функцией naturalsort)

### Запуск для iserv
В корне проекта есть специальный `compose` файл:

<pre>
docker compose -p datalens_dev -f docker-compose-lite.yml --env-file ./.env.iserv up -d
</pre>

Просмотр логов:
<pre>
docker compose -p datalens_dev --env-file ./.env.datalens.dev logs -n 100
</pre>

## Локальное сохранение
<pre>
docker save -o containers/datalens-control-api.tar akrasnov87/datalens-control-api:0.2058.0
docker save -o containers/datalens-data-api.tar ghcr.io/datalens-tech/datalens-data-api:0.2058.0
docker save -o containers/datalens-us.tar akrasnov87/datalens-us:0.189.0
docker save -o containers/datalens-ui.tar akrasnov87/datalens-ui:0.1532.0
</pre>
=======

#### What are the minimum system requirements?

* datalens-ui - 512 MB RAM

* datalens-data-api - 1 GB RAM

* datalens-control-api - 512 MB RAM

* datalens-us - 512 MB RAM

* datalens-pg-compeng - 1 GB RAM

* datalens-pg-us - 512 MB RAM

Summary:

* RAM - 4 GB

* CPU - 2 CORES

This is minimal basic system requirements for OpenSource DataLens installation. Аctual consumption of VM resources depends on the complexity of requests to connections, connections types, the number of users and processing speed at the source level
>>>>>>> e920dbd329391b2129912326323800678d8d1840
