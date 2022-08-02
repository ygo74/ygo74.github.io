---
layout: default
title: Elasticsearch implementation
parent: Observability platforms
grand_parent: Platforms support
nav_order: 1
has_children: false
---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Docker Implementation

```yaml
version: '3'
services:
  setup:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
    networks:
      - elastic
    user: "0"
    command: >
      bash -c '
        if [ x${ELASTIC_PASSWORD} == x ]; then
          echo "Set the ELASTIC_PASSWORD environment variable in the .env file";
          exit 1;
        elif [ x${KIBANA_PASSWORD} == x ]; then
          echo "Set the KIBANA_PASSWORD environment variable in the .env file";
          exit 1;
        fi;
        if [ ! -f config/certs/ca.zip ]; then
          echo "Creating CA";
          bin/elasticsearch-certutil ca --silent --pem -out config/certs/ca.zip;
          unzip config/certs/ca.zip -d config/certs;
        fi;
        if [ ! -f config/certs/certs.zip ]; then
          echo "Creating certs";
          echo -ne \
          "instances:\n"\
          "  - name: es01\n"\
          "    dns:\n"\
          "      - es01\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          "  - name: es02\n"\
          "    dns:\n"\
          "      - es02\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          "  - name: es03\n"\
          "    dns:\n"\
          "      - es03\n"\
          "      - localhost\n"\
          "    ip:\n"\
          "      - 127.0.0.1\n"\
          > config/certs/instances.yml;
          bin/elasticsearch-certutil cert --silent --pem -out config/certs/certs.zip --in config/certs/instances.yml --ca-cert config/certs/ca/ca.crt --ca-key config/certs/ca/ca.key;
          unzip config/certs/certs.zip -d config/certs;
        fi;
        echo "Setting file permissions"
        chown -R root:root config/certs;
        find . -type d -exec chmod 750 \{\} \;;
        find . -type f -exec chmod 640 \{\} \;;
        echo "Setting CA file permissions for logstash"
        find config -type d -exec chmod 755 \{\} \;;
        find config/certs/ca -type f -exec chmod 644 \{\} \;;
        echo "Waiting for Elasticsearch availability";
        until curl -v -s --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} https://es01:9200 | grep -q "cluster_name"; do sleep 30; done;
        echo "Setting kibana_system password";
        until curl -s -X POST --cacert config/certs/ca/ca.crt -u elastic:${ELASTIC_PASSWORD} -H "Content-Type: application/json" https://es01:9200/_security/user/kibana_system/_password -d "{\"password\":\"${KIBANA_PASSWORD}\"}" | grep -q "^{}"; do sleep 10; done;
        echo "All done!";
      '
    healthcheck:
      test: ["CMD-SHELL", "[ -f config/certs/es01/es01.crt ]"]
      interval: 1s
      timeout: 5s
      retries: 120

  apm-server:
    image: docker.elastic.co/apm/apm-server:${STACK_VERSION}
    hostname: apm-server
    depends_on:
      es01:
        condition: service_healthy
      kibana:
        condition: service_healthy
    cap_add: ["CHOWN", "DAC_OVERRIDE", "SETGID", "SETUID"]
    cap_drop: ["ALL"]
    ports:
    - 8200:8200
    networks:
    - elastic
    command: >
       apm-server -e
         -E apm-server.rum.enabled=true
         -E setup.kibana.host=kibana:5601
         -E setup.template.settings.index.number_of_replicas=0
         -E output.elasticsearch.hosts=["es01:9200"]
         -E output.elasticsearch.username=elastic
         -E output.elasticsearch.password=${ELASTIC_PASSWORD}
         -E apm-server.kibana.enabled=true
         -E apm-server.kibana.host=kibana:5601
         -E apm-server.kibana.username=kibana_system
         -E apm-server.kibana.password=${KIBANA_PASSWORD}
    healthcheck:
      interval: 10s
      retries: 80
      test: curl --write-out 'HTTP %{http_code}' --fail --silent --output /dev/null http://localhost:8200/

  opentelemetry-collector:
    container_name: opentelemetry-collector
    hostname: opentelemetry-collector
    image: otel/opentelemetry-collector:0.54.0
    command: [ "--config=/etc/otel-collector-config.yml" ]
    volumes:
      - ./otel-collector-config.yml:/etc/otel-collector-config.yml
    ports:
      - "14250:14250"
      - "55680:55680"
      - "55690:55690"
    depends_on:
      es01:
        condition: service_healthy
      apm-server:
        condition: service_healthy
    networks:
      - elastic

  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    hostname: es01
    restart: on-failure
    # container_name: es01
    environment:
      - node.name=es01
      # - cluster.name=${CLUSTER_NAME}
      #- discovery.seed_hosts=es02
      #- cluster.initial_master_nodes=es01,es02
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=certs/es01/es01.key
      - xpack.security.http.ssl.certificate=certs/es01/es01.crt
      - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.http.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.key=certs/es01/es01.key
      - xpack.security.transport.ssl.certificate=certs/es01/es01.crt
      - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.license.self_generated.type=${LICENSE}
    mem_limit: ${MEM_LIMIT}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - certs:/usr/share/elasticsearch/config/certs
      - esdata01:/usr/share/elasticsearch/data
    ports:
      - ${ES_PORT}:9200
    networks:
      - elastic
    healthcheck:
      test:
        [
          "CMD-SHELL",
          "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        ]
      interval: 30s
      timeout: 10s
      retries: 50
    depends_on:
      setup:
        condition: service_healthy

  # es02:
    # image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
    # hostname: es02
    # restart: on-failure
    # # container_name: es02
    # environment:
      # - node.name=es02
      # - cluster.name=${CLUSTER_NAME}
      # - discovery.seed_hosts=es01
      # - cluster.initial_master_nodes=es01,es02
      # - bootstrap.memory_lock=true
      # - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # - xpack.security.enabled=true
      # - xpack.security.http.ssl.enabled=true
      # - xpack.security.http.ssl.key=certs/es02/es02.key
      # - xpack.security.http.ssl.certificate=certs/es02/es02.crt
      # - xpack.security.http.ssl.certificate_authorities=certs/ca/ca.crt
      # - xpack.security.http.ssl.verification_mode=certificate
      # - xpack.security.transport.ssl.enabled=true
      # - xpack.security.transport.ssl.key=certs/es02/es02.key
      # - xpack.security.transport.ssl.certificate=certs/es02/es02.crt
      # - xpack.security.transport.ssl.certificate_authorities=certs/ca/ca.crt
      # - xpack.security.transport.ssl.verification_mode=certificate
      # - xpack.license.self_generated.type=${LICENSE}
    # mem_limit: ${MEM_LIMIT}
    # ulimits:
      # memlock:
        # soft: -1
        # hard: -1
    # volumes:
      # - certs:/usr/share/elasticsearch/config/certs
      # - esdata02:/usr/share/elasticsearch/data
    # networks:
      # - elastic
    # depends_on:
      # es01:
        # condition: service_healthy
    # healthcheck:
      # test:
        # [
          # "CMD-SHELL",
          # "curl -s --cacert config/certs/ca/ca.crt https://localhost:9200 | grep -q 'missing authentication credentials'",
        # ]
      # interval: 30s
      # timeout: 10s
      # retries: 50

  kibana:
    image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
    hostname: kibana
    depends_on:
      es01:
        condition: service_healthy
      # es02:
        # condition: service_healthy
    environment:
      - ELASTICSEARCH_URL=https://es01:9200
      - ELASTICSEARCH_HOSTS=https://es01:9200
      - SERVERNAME=kibana
      - ELASTICSEARCH_USERNAME=kibana_system
      - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
      - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
      - MONITORING_UI_CONTAINER_ELASTICSEARCH_ENABLED=true
      - LOGGING_ROOT_LEVEL=all
    mem_limit: ${MEM_LIMIT}
    ports:
    - ${KIBANA_PORT}:5601
    volumes:
      - certs:/usr/share/kibana/config/certs
      - kibanadata:/usr/share/kibana/data
    networks:
    - elastic
    healthcheck:
      interval: 10s
      retries: 80
      # test: curl --write-out 'HTTP %{http_code}' --fail --silent --output /dev/null http://localhost:5601/api/status
      test:
        [
          "CMD-SHELL",
          "curl -s -I http://localhost:5601 | grep -q 'HTTP/1.1 302 Found'",
        ]
  logstash:
    image: docker.elastic.co/logstash/logstash:${STACK_VERSION}
    hostname: logstash
    volumes:
      # - type: bind
      #   source: ./logstash/logstash.yml
      #   target: /usr/share/logstash/config/logstash.yml
      #   read_only: true
      - type: bind
        source: ./logstash
        target: /usr/share/logstash/pipeline
        read_only: true
      - type: bind
        source: ./logstash.yml
        target: /usr/share/logstash/config/logstash.yml
        read_only: true
      - certs:/etc/logstash/config/certs
    ports:
      - "5500:5500/tcp"
      - "5500:5500/udp"
      - 5501:5501
    environment:
      - "LS_JAVA_OPTS=-Xms512m -Xmx512m"
      - ELASTICSEARCH_URL=https://es01:9200
      - ELASTICSEARCH_HOSTS='["https://es01:9200"]'
      - ELASTICSEARCH_USERNAME=elastic
      - ELASTICSEARCH_PASSWORD=${ELASTIC_PASSWORD}
      - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
      - xpack.security.enabled=true
      - MONITORING_ELASTICSEARCH_HOSTS='["https://es01:9200"]'

    networks:
      - elastic
    depends_on:
      es01:
        condition: service_healthy
    # user: "0"
    # command: >
      # bash -c '
        # ls -altr /etc/logstash/config/certs/ca
      # '

volumes:
  certs:
  esdata01:
  esdata02:
  kibanadata:

networks:
  elastic:
    driver: bridge

```

### Sources

* <https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html?baymax=rec&rogue=pop-1&elektra=guide>
* <https://github.com/damikun/trouble-training>
* <https://www.luminis.eu/blog/search-en/robust-logstash-configuration/>
