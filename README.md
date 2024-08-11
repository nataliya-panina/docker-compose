# Docker. Часть 2 - Панина Наталия
## Задание 1
Напишите ответ в свободной форме, не больше одного абзаца текста.  
Установите Docker Compose и опишите, для чего он нужен и как может улучшить вашу жизнь.
## Решение 1
Docker compose - это решение для запуска и управления несколькими контейнерами, вместе составляющими одно приложение, одновременно, при помощи файла конфигурации YAML.
Один раз написанный и протестированный файл конфигурации исключает ошибки, которые можно допустить при запуске контейнеров вручную.

## Задание 2
Выполните действия и приложите текст конфига на этом этапе.  
- Создайте файл docker-compose.yml и внесите туда первичные настройки:  
  - version;  
  - services;  
  - volumes;  
  - networks.  
- При выполнении задания используйте подсеть 10.5.0.0/16. Ваша подсеть должна называться: <ваши фамилия и инициалы>-my-   netology-hw. Все приложения из последующих заданий должны находиться в этой конфигурации.
## Решение 2
```
https://hub.docker.com/r/prom/prometheus
mkdir prometheus
cd prometheus
nano compose.yml
```
compose.yml:
```
version: '3'
services:

volumes:

networks:
  paninang-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
        gateway: 10.5.0.1
```
## Задание 3
Выполните действия:  
- Создайте конфигурацию docker-compose для Prometheus с именем контейнера <ваши фамилия и инициалы>-netology-prometheus.  
- Добавьте необходимые тома с данными и конфигурацией (конфигурация лежит в репозитории в директории 6-04/prometheus).  
- Обеспечьте внешний доступ к порту 9090 c докер-сервера.  
## Решение 3
compose.yml
```
services:
  prometheus:
    image: prom/prometheus:v2.53.1
    container_name: paninang-prometheus
    command: --web.enable-lifecycle --config.file=/etc/prometheus/prometheus.yml
    ports:
      - 9090:9090
    volumes:
      - ./:/etc/prometheus
      - prometheus-data:/prometheus
    networks:
      - paninang-netology-hw
    restart: always
volumes:
  prometheus-data:
networks:
  paninang-netology-hw:
    driver: bridge
    ipam:
      config:
        - subnet: 10.5.0.0/16
        gateway: 10.5.0.1
```
```
docker compose up -d
```
![](https://github.com/nataliya-panina/docker-compose/blob/main/img/compose.v1.png)
![prometheus](https://github.com/nataliya-panina/docker-compose/blob/main/img/prometheusv1.png)

## Задание 4
Выполните действия:  
- Создайте конфигурацию docker-compose для Pushgateway с именем контейнера <ваши фамилия и инициалы>-netology-pushgateway.  
- Обеспечьте внешний доступ к порту 9091 c докер-сервера.  
## Решение 4
nano compose.yml
```
services:
  pushgateway:
    image: prom/pushgateway:v1.6.2
    container_name: paninang-pushgateway
    ports:
      - 9091:9091
    networks:
      - paninang-netology-hw
    depends_on:
      - prometheus
    restart: unless-stopped
```
![pushgateway](https://github.com/nataliya-panina/docker-compose/blob/main/img/pushgateway-v1.png)

## Задание 5
Выполните действия:  
- Создайте конфигурацию docker-compose для Grafana с именем контейнера <ваши фамилия и инициалы>-netology-grafana.  
- Добавьте необходимые тома с данными и конфигурацией (конфигурация лежит в репозитории в директории 6-04/grafana).  
- Добавьте переменную окружения с путем до файла с кастомными настройками (должен быть в томе), в самом файле пропишите логин=<ваши фамилия и инициалы> пароль=netology.  
- Обеспечьте внешний доступ к порту 3000 c порта 80 докер-сервера.  
## Решение 5
Добавляю описание контейнера grafana в файл compose.yml
```
grafana:
  image: grafana/grafana
  container_name: paninang-grafana
  environment:
    GF_PATHS_CONFIG: /etc/grafana/custom.ini
      ports:
        - 80:3000
        volumes:
  	  - ./grafana:/etc/grafana
  	  - grafana-data:/var/lib/grafana
	networks:
  	  - paninang-netology-hw
	depends_on:
  	  - pushgateway
	restart: unless-stopped
volumes:
  grafana-data:
```
Создаю в каталоге grafana файл custom.ini:
```
[security]

admin_user=paninang
admin_password=netology
```
```
docker compose up -d
```
![grafana](https://github.com/nataliya-panina/docker-compose/blob/main/img/grafana-v1.png)

## Задание 6
Выполните действия.  
- Настройте поочередность запуска контейнеров.  
- Настройте режимы перезапуска для контейнеров.  
- Настройте использование контейнерами одной сети.  
- Запустите сценарий в detached режиме.  
## Решение 6
```
services:
  prometheus:
    restart: always
    networks:
paninang-netology-hw
  pushgateway:
    depends_on: prometheus
    restart: unless-stopped
   networks: paninang-netology-hw
   grafana:
    depends_on: pushgateway
    restart: unless-stopped
    networks: paninang-netology-hw
```
запуск контейнера с node_exporter:
```
docker run -d --net=”host” --name node_exporter -v “./:/host:ro,rslave”  \ quay.io/prometheus/node-exporter:latest --path.rootfs=/host
```
![docker compose up -d](https://github.com/nataliya-panina/docker-compose/blob/main/img/sequence.png)

## Задание 7
Выполните действия.  
- Выполните запрос в Pushgateway для помещения метрики <ваши фамилия и инициалы> со значением 5 в Prometheus: echo "<ваши фамилия и инициалы> 5" | curl --data-binary @- http://localhost:<внешний порт выбранный вами в задании 4>/metrics/job/netology.  
- Залогиньтесь в Grafana с помощью логина и пароля из предыдущего задания.  
- Создайте Data Source Prometheus (Home -> Connections -> Data sources -> Add data source -> Prometheus -> указать "Prometheus server URL = http://prometheus:9090" -> Save & Test).  
- Создайте график на основе добавленной в пункте 5 метрики (Build a dashboard -> Add visualization -> Prometheus -> Select metric -> Metric explorer -> <ваши фамилия и инициалы -> Apply.  
В качестве решения приложите:  
- docker-compose.yml целиком;  
- скриншот команды docker ps после запуске docker-compose.yml;  
- скриншот графика, построенного на основе вашей метрики.  
## Решение 7
```
echo "paninang 5" | curl --data-binary @- http://localhost:9091/metrics/job/netology
```
![compose.yml](https://github.com/nataliya-panina/docker-compose/blob/main/compose.yml)
![docker ps](https://github.com/nataliya-panina/docker-compose/blob/main/img/docker-ps.png)
![grafana](https://github.com/nataliya-panina/docker-compose/blob/main/img/my_metric.png)

## Задание 8
Выполните действия:  
- Остановите и удалите все контейнеры одной командой.  
В качестве решения приложите скриншот консоли с проделанными действиями.  
## Решение 8
```
docker rm -f $(docker ps -aq)
```
![Docker rm](https://github.com/nataliya-panina/docker-compose/blob/main/img/docker-rm.png)

## Задание 9*
Выполните действия:  
- Создайте конфигурацию docker-compose для Alertmanager с именем контейнера <ваши фамилия и инициалы>-netology-alertmanager.  
- Добавьте необходимые тома с данными и конфигурацией, сеть, режим и очередность запуска.  
- Обновите конфигурацию Prometheus (необходимые изменения ищите в презентации или документации) и перезапустите его.  
- Обеспечьте внешний доступ к порту 9093 c докер-сервера.  
В качестве решения приложите скриншот с событием из Alertmanager.  
## Решение 9
![Alertmanager](https://github.com/nataliya-panina/docker-compose/blob/main/alertmanager.yml)
![Alert](https://github.com/nataliya-panina/docker-compose/blob/main/img/instance-down.png)

## Задание 10*
Запустите свой сценарий на чистом железе без предзагруженных образов.  
Ответьте на вопросы в свободной форме:  
Опишите выполненный вами процесс развертывания сценария.  
Как вы думаете зачем может понадобиться такой способ развертывания?  
## Решение 10
```
docker image rm -f $(docker image ls)
```
![Загрузка образов](https://github.com/nataliya-panina/docker-compose/blob/main/img/pull_images.png)
![Загрузка контейнеров](https://github.com/nataliya-panina/docker-compose/blob/main/img/pull-2.png)
![Script](https://github.com/nataliya-panina/docker-compose/blob/main/script.sh)
