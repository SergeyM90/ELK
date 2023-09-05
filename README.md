# ELK

# Домашнее задание по теме "ELK" Сергей Миронов SYS-20


Задание 1. Elasticsearch
Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный.  

Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name.  

![image](https://github.com/SergeyM90/ELK/assets/84016375/4b8dce8d-751b-4887-add2-556a8a3c6d96)

Cluster_name передаётся в переменных окружения докер компоузом  

    environment:  
      - xpack.security.enabled=false  
      - discovery.type=single-node  
      - cluster.name=my-cluster  

В докер компоуз добавлен мониторинг состояния эластика:  

    healthcheck:  
      test: curl -s http://elasticsearch:9200/_cluster/health?pretty | grep -q -e 'green' -e 'yellow'  
      interval: 10s  
      timeout: 10s  
      retries: 50  


Задание 2. Kibana  
Установите и запустите Kibana.  

Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty.  

![image](https://github.com/SergeyM90/ELK/assets/84016375/5b59de02-1dcf-4091-81a7-7a1db15459c8)

В докер компоуз добавлен мониторинг доступности кибаны:

    healthcheck:
     test: curl -s -I  http://kibana:5601/app/home | grep -q 'HTTP/1.1 200 OK'
     interval: 10s
     timeout: 10s
     retries: 50  

    

Задание 3. Logstash  
Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch.  

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.  

Решил сделать следующим образом:
На хосте монтируется папка /projects/test/ingest_data/ в которую мапятся логи nginx. Эта же папка мапится в контейнеры с логсташем и файлбитом
![image](https://github.com/SergeyM90/ELK/assets/84016375/80d51845-7680-46ba-b147-a89027e29a4d)

![image](https://github.com/SergeyM90/ELK/assets/84016375/c23939d2-2012-48b6-91cb-8bb1fa80cd44)




Задание 4. Filebeat.
Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.

Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.

Для обработки логов Nginx решил использовать встроенный модуль
Конфигурация Filebeat:

filebeat.config:
  modules:
    enabled: true
    path: /usr/share/filebeat/modules.d/*.yml 
    reload.enabled: false

setup.kibana:
  host: ${KIBANA_HOSTS}

output.elasticsearch:
  hosts: ${ELASTIC_HOSTS}

http.enabled: true
http.port: 5066
monitoring.enabled: false
monitoring.cluster_uuid: "FzMrNr_fS5eCO250wLtsaw"
http.host: 0.0.0.0  


Модуль обработки логов nginx:


- module: nginx

  access:
    enabled: true
    var.paths: 
      - '/usr/share/filebeat/ingest_data/access.log'

  error:
    enabled: false

  ingress_controller:
    enabled: false


  ![image](https://github.com/SergeyM90/ELK/assets/84016375/d6649ffb-fe23-4e0d-b39a-08f56b1f71d2)


  ![image](https://github.com/SergeyM90/ELK/assets/84016375/5f5541de-6628-4d79-b7c5-217cca55934b)

