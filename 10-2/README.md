# Домашнее задание к занятию "10.2 Кластеризация" Лазарев Даниил

### Задание 1. 

Главной особенностью систем с архитектурой SMP является наличие общей физической памяти, разделяемой всеми процессорами.
Главная особенность MMP архитектуры состоит в том, что память физически разделена.

### Задание 2.

В сильно связанных системах (SMP, NUMA) вся система работает под управлением одной ОС. В слабо связанных - на каждом узле работает своя ОС.

### Задание 3.

- Масштабируемость
- Отказоустойчивость
- Распределенная загрузка мощностей
- Соотношение цена/производительность ниже, чем у отдельного физического сервера

### Задание 4.

1. Отказоустойчивые кластеры (High-availability clusters, HA, кластеры высокой доступности)
Corosync, Pacemaker, Red Hat Cluster Suite
2. Кластеры с балансировкой нагрузки (Load balancing clusters)
сетевая (DNS), транспортная (HAProxy) и прикладная балансировка (Nginx)
3. вычислительные кластеры (High performance computing clusters, HPC)
Azure, AWS HPC
4. Системы распределенных вычислений (grid)
BOINC

### Задание 5.

Kafka и RabbitMQ — это системы очередей сообщений, которые можно использовать при потоковой обработке. Поток данных — это непрерывные инкрементные данные большого объема, требующие высокоскоростной обработки. Например, это могут быть данные датчиков об окружающей среде, которые необходимо постоянно собирать и обрабатывать для наблюдения за изменениями температуры или давления воздуха в реальном времени. RabbitMQ — это брокер распределенных сообщений, который собирает потоковые данные из нескольких источников и маршрутизирует их в разные пункты назначения для обработки. Apache Kafka — это платформа потоковой трансляции для создания конвейеров данных и приложений потоковой трансляции в реальном времени.
Общий ответ: брокер используется для координации сервисов внутри микросервисной архиетекторы.
Более частный: может использоваться для обработки транзакций в банковских приложениях.
### Задание 6*.
I-II.1. RabbitMQ - брокер сообщений
2. HAProxy - балансировщик нагрузки
3. Docker - среда для контейнеризации
4. Docker compose - оркестрация контейнеров
III.1. Node failures
2. Network partition
3. Big cluster

![Скриншот-1](https://github.com/n123tw/srlb-homework/blob/main/10-2/img/1.jpg)
```
n123@cluster1:~$ docker-compose version
docker-compose version 1.29.2, build unknown
docker-py version: 5.0.3
CPython version: 3.10.6
OpenSSL version: OpenSSL 3.0.2 15 Mar 2022
```
Изменил docker-compose:
```
version: '2'
services:

  composer:
    image: ypereirareis/composer:1.1.3-alpine
    working_dir: /src
    environment:
      COMPOSER_HOME: /tmp/.composer
    volumes:
      - ~/.composer:/tmp/.composer:rw
      - ./sf:/src:rw

  php:
    build: ./docker/php
    environment:
      SYMFONY_ENV:
      CUSTOM_UID: 1000
    volumes:
      - ./var/logs/app:/var/www/html/var/logs:rw
      - ./sf:/var/www/html:rw
    networks:
      - default

  rmq1:
    image: bijukunjummen/rabbitmq-server:3.6.5
    hostname: rmq1
    environment:
      ERLANG_COOKIE: 'ck8FC0uJ9vePIhLr'
      TCP_PORTS: "5672,15672"
    ports:
      - "127.0.0.1:1234:15672"
    networks:
      - rmq_1_2
      - rmq_3_1

  rmq2:
    image: bijukunjummen/rabbitmq-server:3.6.5
    hostname: rmq2
    environment:
      ERLANG_COOKIE: 'ck8FC0uJ9vePIhLr'
      CLUSTERED: 'true'
      CLUSTER_WITH: rmq1
      TCP_PORTS: "5672"
    ports:
      - "127.0.0.1:1235:15672"
    networks:
      - rmq_1_2
      - rmq_2_3

  rmq3:
    image: bijukunjummen/rabbitmq-server:3.6.5
    hostname: rmq3
    environment:
      ERLANG_COOKIE: 'ck8FC0uJ9vePIhLr'
      CLUSTERED: 'true'
      CLUSTER_WITH: rmq1
      TCP_PORTS: "5672"
    ports:
      - "127.0.0.1:1236:15672"
    networks:
      - rmq_3_1
      - rmq_2_3

  rmqN:
    image: bijukunjummen/rabbitmq-server:3.6.5
    environment:
      ERLANG_COOKIE: 'ck8FC0uJ9vePIhLr'
      CLUSTERED: 'true'
      CLUSTER_WITH: rmq1
      TCP_PORTS: "5672"
    networks:
      - rmq_3_1
      - rmq_2_3

  haproxy:
    image: dockercloud/haproxy:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      MODE: tcp
    expose:
      - 5672
    links:
      - rmq1
      - rmq2
      - rmq3
    ports:
      - '80:80'
      - '5672:5672'
    networks:
      - default
      - rmq_1_2
      - rmq_2_3
      - rmq_3_1

networks:
  rmq_1_2:
    external:
      name: rmq_1_2
  rmq_2_3:
    external:
      name: rmq_2_3
  rmq_3_1:
    external:
      name: rmq_3_1
```
И dockerfile:
```
FROM zolweb/php7:2.2-alpine

ENV RABBITMQ_C_VER 0.8.0

RUN set -xe \
  && apk add --update --no-cache \
    wget \
    cmake \
    openssl-dev \
  && rm -rf /var/cache/apk/* \

  # RabbitMQ C client
  && wget -qO- https://github.com/alanxz/rabbitmq-c/releases/download/v${RABBITMQ_C_VER}/rabbitmq-c-${RABBITMQ_C_VER}.tar.gz | tar xz -C /tmp/ \
  && cd /tmp/rabbitmq-c-${RABBITMQ_C_VER} \
  && mkdir -p build \
  && cd build \
  && cmake .. \
    -DCMAKE_INSTALL_PREFIX=/usr \
    -DCMAKE_INSTALL_LIBDIR=lib \
    -DCMAKE_C_FLAGS="$CFLAGS" \
  && cmake --build . --target install \
  && apk del \
    wget \
    cmake \
    openssl-dev

# Install tools...
RUN set -x \
  && docker-php-ext-install \
    bcmath \
    pcntl \
  && pecl channel-update pecl.php.net \
  && pecl install \
    http://pecl.php.net/get/amqp-1.11.0.tgz \
  && docker-php-ext-enable amqp
```
Получаю:
```
root@cluster1:/home/n123/docker-rabbitmq-ha-cluster# make state
== Print state of containers ==
    Name                   Command               State                                   Ports
-------------------------------------------------------------------------------------------------------------------------------
rmq_haproxy_1   /sbin/tini -- dockercloud- ...   Up      1936/tcp, 443/tcp, 0.0.0.0:5672->5672/tcp,:::5672->5672/tcp,
                                                         0.0.0.0:80->80/tcp,:::80->80/tcp
rmq_rmq1_1      /bin/sh -c /opt/rabbit/sta ...   Up      127.0.0.1:1234->15672/tcp, 25672/tcp, 4369/tcp, 5672/tcp, 9100/tcp,
                                                         9101/tcp, 9102/tcp, 9103/tcp, 9104/tcp, 9105/tcp
rmq_rmq2_1      /bin/sh -c /opt/rabbit/sta ...   Up      127.0.0.1:1235->15672/tcp, 25672/tcp, 4369/tcp, 5672/tcp, 9100/tcp,
                                                         9101/tcp, 9102/tcp, 9103/tcp, 9104/tcp, 9105/tcp
rmq_rmq3_1      /bin/sh -c /opt/rabbit/sta ...   Up      127.0.0.1:1236->15672/tcp, 25672/tcp, 4369/tcp, 5672/tcp, 9100/tcp,
                                                         9101/tcp, 9102/tcp, 9103/tcp, 9104/tcp, 9105/tcp

```