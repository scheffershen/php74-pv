# php74-pv

php docker image for veille pv project

docker-composer.yml

```
version: '3.6'

services:
  maildev:
    image: maildev/maildev
    container_name: maildev
    ports:
      - "1080:80"
    networks:
      - backend              
  yake:
    image: liaad/yake-server:latest
    container_name: yake 
    ports:
      - "5000:5000"     
    networks:
      - backend               
  php:
    image: scheffershen/php74:pv
    container_name: php
    env_file:
      - .env
    working_dir: /var/www/html
    volumes:
      - "./:/var/www/html"
    networks:
      - backend  
  python:
    image: python:3.9-bullseye
    container_name: python
    working_dir: /var/www/html
    volumes:
      - "./:/var/www/html"
    networks:
      - backend         
  caddy:
    image: caddy:2
    container_name: caddy
    environment:
      SERVER_NAME: ${SERVER_NAME:-pv_detector.universalmedica.local, caddy:80}
    ports:
      - 80:80
      - 443:443
    working_dir: /var/www/html
    volumes:
      - "./public:/var/www/html/public"
      - "./docker/caddy/Caddyfile:/etc/caddy/Caddyfile:ro"
      - "./docker/.data/caddy/data:/data"
      - "./docker/.data/caddy/config:/config"
    networks:
      - backend       
      - frontend      
  db:
    image: mysql:5.7
    container_name: db
    environment:
      MYSQL_ROOT_PASSWORD: 9TT4fgq5
    volumes:
      - "./docker/.data/db:/var/lib/mysql"
      - "./_SQL:/home"
    networks:
      - backend
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    restart: always    
    ports:
      - "8080:80"      
    environment:
        PMA_HOST: db
    networks:
      - backend   
      - frontend
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.3
    container_name: elasticsearch
    volumes:
      - ./docker/.data/elasticsearch:/usr/share/elasticsearch/data    
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" # 512mo HEAP
      - "ELASTICSEARCH_USERNAME=adminum"
      - "ELASTICSEARCH_PASSWORD=azerty"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 9200:9200
    networks:
      - backend
      - frontend

  kibana:
    image: docker.elastic.co/kibana/kibana:7.9.3
    container_name: kibana
    environment:
        ELASTICSEARCH_URL: http://elasticsearch:9200
    depends_on:
        - elasticsearch
    ports:
        - 5601:5601
    networks:
      - backend
      - frontend

volumes:
  caddy_data:
  caddy_config:
  database_data:

networks:
    frontend:
        driver: bridge
    backend:
        driver: bridge      

```