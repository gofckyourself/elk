# Elasticsearch and Kibana with Docker Compose

Этот проект позволяет развернуть Elasticsearch и Kibana с использованием Docker Compose, с включенной безопасностью через SSL и базовой аутентификацией. Включены также инструкции по сбросу пароля для пользователя `kibana_system` и обходу блокировки для скачивания Docker-образов.

## Требования

Перед использованием убедитесь, что у вас установлены следующие компоненты:

- Docker: [Инструкция по установке Docker](https://docs.docker.com/get-docker/)
- Docker Compose: [Инструкция по установке Docker Compose](https://docs.docker.com/compose/install/)

## Установка

1. Клонируйте репозиторий:

    ```bash
    git clone https://github.com/gofckyourself/elk.git
    cd elk
    ```

2. Скопируйте файл `.env-template` в корне проекта и измените следующие переменные:
    
    ```bash
    cp .env-temlate .env
    ```

    - `STACK_VER`: версия Elastic Stack (например, `8.15.0`).
    - `ELASTIC_PASSWORD`: пароль для пользователя `elastic`.
    - `KIBANA_PWD`: пароль для пользователя `kibana_system`, пароль назначается с помощью curl после запуска контейнера elastic, способ сброса пароля ниже. 
    - `KIBANA_URL`: адрес по которому будет доступна кибана, нужно для корректной работы сертификата. Имя должно быть прописано на вашем DNS сервере
    


3. Запустите контейнеры с помощью Docker Compose:

    ```bash
    docker-compose up -d
    ```

    Эта команда развернет Elasticsearch и Kibana в фоновом режиме. Контейнеры будут доступны на портах:
    
    - Elasticsearch: `9200` (HTTP), `9300` (TCP)
    - Kibana: `5601` (HTTP)

## Сброс пароля пользователя `kibana_system`

Нужно сбросить пароль для пользователя `kibana_system` и указать этот пароля в .env, используйте команду с `curl`:

```bash
curl -X POST -u elastic 'https://localhost:9200/_security/user/kibana_system/_password' \
-H 'Content-Type: application/json' \
-d '{"password": "new_password"}'

## Конфигурация SSL

    Все соединения с Elasticsearch и Kibana защищены SSL-сертификатами. Сертификаты должны быть размещены в папке ./certs и включать:

    privkey.pem — приватный ключ.
    fullchain.pem — сертификат.

## Обход блокировки для скачивания Docker-образов в РФ
    Из-за ограничений на доступ к Docker-образам в России, вам возможно придется использовать VPN или другие способы для скачивания Docker-образов. Вот два варианта:

    1. Скачивание через Docker Save
      Убедитесь, что у вас настроен VPN или доступ через сервер за пределами России.
      
      Используйте команду для скачивания и сохранения образов на локальной машине (укажите нужную версию из .env вместо STACK_VER):
      
      ```bash
      docker pull --platform=linux/amd64 docker.elastic.co/elasticsearch/elasticsearch:STACK_VER
      docker save -o elasticsearch.tar docker.elastic.co/elasticsearch/elasticsearch:STACK_VER
      docker pull --platform=linux/amd64  docker.elastic.co/kibana/kibana:STACK_VER
      docker save -o kibana.tar docker.elastic.co/kibana/kibana:STACK_VER
      ```
    
    Перенесите .tar файлы на сервер и загрузите их с помощью команды:
    ```bash
    docker load -i /path/to/elasticsearch.tar
    docker load -i /path/to/kibana.tar
    ```
    
    2. Скачивание через Skopeo
      На macos может возникнуть баг при котором не удастся выгрузить скачанные образы в файл, для решения проблемы можно воспользоваться skopeo (можно установить через brew)

      ```bash
      skopeo copy --override-arch amd64 --override-os linux docker://docker.elastic.co/kibana/kibana:STACK_VER docker-archive:kibana.tar
      skopeo copy --override-arch amd64 --override-os linux docker://docker.elastic.co/elasticsearch/elasticsearch:STACK_VER docker-archive:elastic.tar
      ```

      Перенесите .tar файлы на сервер и загрузите их с помощью команды:
    ```bash
    docker load -i /path/to/elasticsearch.tar
    docker load -i /path/to/kibana.tar
