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

The Elasticsearch deployment defined below has been validated with the following components:

* Docker on Windows v4.11.1 (Use WSL 2 based engine)
* Docker-compose v2.7.0
* Elasticsearch stack v8.3.2

Even if personal deployment, the following configuration tries to follow security best practices because real configuration needs some complex configuration which are often hard to find.

Certificates used are self-signed but can be easily replaced by enterprises' certificates signed by your enterprise authority.

The full implementation can be found on the ansible inventory github repository in the monitoring folder.

## Architecture

## Prerequisites

### Certificates

Certificates are needed for:

* Elastic cluster nodes
* Kibana
* Fleet server
* APM Server
* Open telemetry collector

1. Generate self signed certificates

    Thanks to one of elasticsearch's utility, it is possible to quickly generate self-signed certificates.
    Generated certificates will be written in a shared volume **./certificates**

    Docker compose file
    ```yaml
    version: '3'
    services:
      setup-certificates:
        image: docker.elastic.co/elasticsearch/elasticsearch:${STACK_VERSION}
        volumes:
          - type: bind
            source: ./certificates
            target: /usr/share/elasticsearch/config/certs

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
              "  - name: fleet-server\n"\
              "    dns:\n"\
              "      - fleet-server\n"\
              "      - localhost\n"\
              "    ip:\n"\
              "      - 127.0.0.1\n"\
              "  - name: apm-server\n"\
              "    dns:\n"\
              "      - apm-server\n"\
              "      - localhost\n"\
              "    ip:\n"\
              "      - 127.0.0.1\n"\
              "  - name: opentelemetry-collector\n"\
              "    dns:\n"\
              "      - opentelemetry-collector\n"\
              "      - localhost\n"\
              "    ip:\n"\
              "      - 127.0.0.1\n"\
              > config/certs/instances.yml;
              bin/elasticsearch-certutil cert --silent --pem -out config/certs/certs.zip --in config/certs/instances.yml --ca-cert config/certs/ca/ca.crt --ca-key config/certs/ca/ca.key;
              unzip config/certs/certs.zip -d config/certs;
            fi;
          '
        healthcheck:
          test: ["CMD-SHELL", "[ -f config/certs/es01/es01.crt ]"]
          interval: 1s
          timeout: 5s
          retries: 120

    ```

2. Use your own certificates signed by you authority

    **TODO**

### Users

**TODO**

### Environments variable for Docker compose

Some configuration are shared in the docker compose file thanks to the environment variables file **.env**

**TODO Retrieve the full file after verification of potential variables in docker compose file**

```ini
# -----------------------------------------------------------------------------
# Common variables
# -----------------------------------------------------------------------------

# Version of Elastic products
STACK_VERSION=8.3.2

# Project namespace (defaults to the current folder name if not set)
#COMPOSE_PROJECT_NAME=myproject

...
```

## Elasticsearch deployment

### Elasticsearch monitoring status

https://localhost:9200/_cluster/health?wait_for_status=yellow

### Elasticsearch resources

* [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html?baymax=rec&rogue=pop-1&elektra=guide)

## Kibana deployment

### Kibana connection to Elastic search

```yaml
kibana:
  image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
  ...
  environment:
    # Elastic search connection
    - ELASTICSEARCH_URL=https://es01:9200
    - ELASTICSEARCH_HOSTS=https://es01:9200
    - ELASTICSEARCH_USERNAME=kibana_system
    - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
    # Elastic search allow ssl connection
    - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
```

### Kibana integration policies configuration

It is possible to create integration policies from the kibana configuration file **/usr/share/kibana/config/kibana.yml**.
This file will be shared from the docker hosts thanks to volumes mount :

```yaml
volumes:
  ...
  - "./configuration/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml"
  ...
```

We will initialize our deployment with 2 required policies:

* Fleet server
* APM

Integration policies are defined under the key **xpack.fleet.agentPolicies**

1. Deploy required packages for these integration policies

    ```yaml
    # https://www.elastic.co/guide/en/fleet/master/create-a-policy-no-ui.html
    xpack.fleet.packages:
      - name: system
        version: latest
      - name: elastic_agent
        version: latest
      - name: fleet_server
        version: latest
      - name: apm
        version: latest
    ```

2. Enable Fleet policy and configure default outputs

    ```yaml
    xpack.fleet.agentPolicies:
      - name: Fleet Server (APM)
        id: fleet-server-apm
        is_default_fleet_server: true
        is_managed: false
        namespace: default
        package_policies:
          - name: fleet_server-apm
            id: default-fleet-server
            package:
              name: fleet_server

    ```

    As for policies, we can also configure the default outputs for agents in the kibana configuration file.
    We will initialize the default outputs to be able to send encrypted data to elastic search

    ```yaml
    xpack.fleet.outputs:
      - id: fleet-default-output
        name: default
        is_default: true
        is_default_monitoring: true
        type: elasticsearch
        hosts: ['https://es01:9200']
        config:
          ssl.certificate_authorities: ["/usr/share/elastic-agent/config/certs/ca/ca.crt"]

    ```

3. Enable APM Policy and configure APM server

    ```yaml
    xpack.fleet.agentPolicies:
      - name: APM Server from elastic agent
        id: apm-1-server
        is_default: true
        unenroll_timeout: 900
        package_policies:
          - name: apm-1-server
            id: apm-1-server
            package:
              name: apm
            inputs:
            - type: apm
              enabled: true
              vars:
              - name: host
                value: apm-server:8200
              - name: url
                value: http://apm-server:8200

    ```

### Kibana enable SSL Listener

The main configuration for SSL will be applied from environment variables in the docker compose file:

1. Enable Server HTTPS Listener

    ```yaml
    kibana:
      image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
      ...
      environment:
      # https://discuss.elastic.co/t/kibana-tls-to-elastic-cluster-in-docker-compose-self-signed-certificate-in-certificate-chain/224538/2
      # enable HTTPS
      - SERVER_SSL_ENABLED=true
      - SERVER_SSL_KEY=config/certs/kibana/kibana.key
      - SERVER_SSL_CERTIFICATE=config/certs/kibana/kibana.crt
      - SERVER_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
    ```

2. Allow SSL Communication to Elastic Search

    ```yaml
    kibana:
      image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
      ...
      environment:
      # Elastic search allow ssl connection
      - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt
    ```

Additional SSL configuration are applied from the the kibana configuration file.

```yaml
# HTTPS
# https://www.elastic.co/guide/en/kibana/master/Security-production-considerations.html
server.securityResponseHeaders.strictTransportSecurity: "max-age=31536000"
server.securityResponseHeaders.disableEmbedding: true
csp.strict: true

```

### Kibana enable Fleet Server integration

```yaml
kibana:
  image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
  ...
  environment:
    # Fleet server integration
    - xpack.security.enabled=true
    - xpack.security.encryptionKey=fhjskloppd678ehkdfdlliverpoolfcr
    - xpack.encryptedSavedObjects.encryptionKey=fhjskloppd678ehkdfdlliverpoolfcr
    - XPACK_FLEET_AGENTS_FLEET_SERVER_HOSTS=["https://fleet-server:8220"]
    - XPACK_FLEET_AGENTS_ELASTICSEARCH_HOSTS=["https://es01:9200"]
```

{: .warning-title }
> Mandatory for APM Integration
>
> this value is mandatory to configure fleet server:
>
> * xpack.security.encryptionKey
> * xpack.encryptedSavedObjects.encryptionKey

### Kibana monitoring status

Kibana provides rest API to obtain its healtch check status:

* api/status

### Kibana full configuration sample

1. Docker compose service

    ```yaml
    kibana:
      image: docker.elastic.co/kibana/kibana:${STACK_VERSION}
      hostname: kibana
      depends_on:
        es01:
          condition: service_healthy
      environment:
        # Elastic search connection
        - ELASTICSEARCH_URL=https://es01:9200
        - ELASTICSEARCH_HOSTS=https://es01:9200
        - ELASTICSEARCH_USERNAME=kibana_system
        - ELASTICSEARCH_PASSWORD=${KIBANA_PASSWORD}
        # Elastic search allow ssl connection
        - ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt

        # Divers
        - MONITORING_UI_CONTAINER_ELASTICSEARCH_ENABLED=true
        - LOGGING_ROOT_LEVEL=all

        # https://discuss.elastic.co/t/kibana-tls-to-elastic-cluster-in-docker-compose-self-signed-certificate-in-certificate-chain/224538/2
        # enable HTTPS
        - SERVERNAME=kibana
        - SERVER_SSL_ENABLED=true
        - SERVER_SSL_KEY=config/certs/kibana/kibana.key
        - SERVER_SSL_CERTIFICATE=config/certs/kibana/kibana.crt
        - SERVER_SSL_CERTIFICATEAUTHORITIES=config/certs/ca/ca.crt

        # Fleet server integration
        - xpack.security.enabled=true
        - xpack.security.encryptionKey=fhjskloppd678ehkdfdlliverpoolfcr
        - xpack.encryptedSavedObjects.encryptionKey=fhjskloppd678ehkdfdlliverpoolfcr
        - XPACK_FLEET_AGENTS_FLEET_SERVER_HOSTS=["https://fleet-server:8220"]
        - XPACK_FLEET_AGENTS_ELASTICSEARCH_HOSTS=["https://es01:9200"]

      mem_limit: ${MEM_LIMIT}
      ports:
      - ${KIBANA_PORT}:5601
      volumes:
        - "./certificates:/usr/share/kibana/config/certs"
        - "kibanadata:/usr/share/kibana/data"
        - "./configuration/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml"
        - "./logs/kibana:/var/log/kibana"
      networks:
      - elastic
      healthcheck:
        interval: 10s
        retries: 300
        test:
          [
            "CMD-SHELL",
            "curl -s -u elastic:${ELASTIC_PASSWORD} --cacert config/certs/ca/ca.crt https://localhost:5601/api/status | grep -q 'All services are available'"
          ]

    ```

2. Kibana configuration file

    ```yaml
    server.host: 0.0.0.0
    status.allowAnonymous: false
    monitoring.ui.container.elasticsearch.enabled: true
    telemetry.enabled: false

    # HTTPS
    # https://www.elastic.co/guide/en/kibana/master/Security-production-considerations.html
    server.securityResponseHeaders.strictTransportSecurity: "max-age=31536000"
    server.securityResponseHeaders.disableEmbedding: true
    csp.strict: true


    # https://www.elastic.co/guide/en/fleet/master/create-a-policy-no-ui.html
    xpack.fleet.packages:
      - name: fleet_server
        version: latest
      - name: apm
        version: latest

    xpack.fleet.agentPolicies:
      - name: Fleet Server (APM)
        id: fleet-server-apm
        is_default_fleet_server: true
        is_managed: false
        namespace: default
        package_policies:
          - name: fleet_server-apm
            id: default-fleet-server
            package:
              name: fleet_server
      - name: APM Server from elastic agent
        id: apm-1-server
        is_default: true
        unenroll_timeout: 900
        package_policies:
          - name: apm-1-server
            id: apm-1-server
            package:
              name: apm
            inputs:
            - type: apm
              enabled: true
              vars:
              - name: host
                value: apm-server:8200
              - name: url
                value: http://apm-server:8200

    xpack.fleet.outputs:
      - id: fleet-default-output
        name: default
        is_default: true
        is_default_monitoring: true
        type: elasticsearch
        hosts: ['https://es01:9200']
        config:
          ssl.certificate_authorities: ["/usr/share/elastic-agent/config/certs/ca/ca.crt"]

    # https://www.elastic.co/guide/en/kibana/master/logging-configuration.html#logging-appenders
    logging:
      appenders:
        rolling-file:
          type: rolling-file
          fileName: /var/logs/kibana.log
          policy:
            type: time-interval
            interval: 10s
            modulate: true
          strategy:
            type: numeric
            pattern: '-%i'
            max: 2
          layout:
            type: pattern
    ```

### Kibana resources

* [Kibana in Docker](https://www.elastic.co/guide/en/kibana/current/docker.html){:target="_blank"}
* [Kibana environment variables](https://www.elastic.co/guide/en/kibana/8.3/docker.html#environment-variable-config){:target="_blank"}
* [Kibana settings](https://www.elastic.co/guide/en/kibana/current/settings.html){:target="_blank"}
* [Define integration policies](https://www.elastic.co/guide/en/fleet/master/create-a-policy-no-ui.html){:target="_blank"}
* [Security production consideration](https://www.elastic.co/guide/en/kibana/master/Security-production-considerations.html){:target="_blank"}
* [Logging configuration](https://www.elastic.co/guide/en/kibana/master/logging-configuration.html#logging-appenders){:target="_blank"}

## Fleet server deployment

Fleet server is a component used to manage agents from a centralized point. It is available from the elastic-agent binary

### Fleet server connection to Elastic search

```yaml
fleet-server:
  image: docker.elastic.co/beats/elastic-agent:${STACK_VERSION}
  ...
  environment:
    # Elastic search connection
    ELASTICSEARCH_HOST: https://es01:9200
    ELASTICSEARCH_USERNAME: "${ES_SUPERUSER_USER:-elastic}"
    ELASTICSEARCH_PASSWORD: "${ELASTIC_PASSWORD:-changeme}"
    ELASTICSEARCH_CA:  /usr/share/elastic-agent/config/certs/ca/ca.crt
    ...
```

### Fleet server connection to Kibana

```yaml
fleet-server:
  image: docker.elastic.co/beats/elastic-agent:${STACK_VERSION}
  ...
  environment:
    # Kibana connection
    KIBANA_HOST: "https://kibana:${KIBANA_PORT}"
    KIBANA_USERNAME: "${ES_SUPERUSER_USER:-elastic}"
    KIBANA_PASSWORD: "${ELASTIC_PASSWORD:-changeme}"
    ...
```

### Fleet server configuration

The configuration is divided into 2 blocsk :

* Enable fleet server function in the elastic agent

    {: .warning-title }
    > Link fleet server to its enrollment policy
    >
    > The policy for fleet server integration has been defined in the Kibana configuration file in **xpack.fleet.agentPolicies** key.
    > The id value is the key used to enroll the server in its policy and is specified with the environment Variable **FLEET_SERVER_POLICY_ID**

* Configure fleet server into Kibana

```yaml
fleet-server:
  image: docker.elastic.co/beats/elastic-agent:${STACK_VERSION}
  ...
  environment:
    ...
    # Bootstrap Fleet server
    FLEET_SERVER_ENABLE: "1"
    # FLEET_SERVER_ELASTICSEARCH_HOST: https://es01:9200
    # FLEET_SERVER_ELASTICSEARCH_CA:  /usr/share/elastic-agent/config/certs/ca/ca.crt
    # FLEET_SERVER_SERVICE_TOKEN: b0RUUmw0SUI1X1FkdnNlM2FMWGs6SmlKYk4xZzJUWjZFM3EzX19NM0NzUQ==
    # FLEET_SERVER_POLICY_ID: "fleet-server-policy"
    FLEET_SERVER_POLICY_ID: "fleet-server-apm"
    FLEET_SERVER_ELASTICSEARCH_USERNAME: "${ES_SUPERUSER_USER:-elastic}"
    FLEET_SERVER_ELASTICSEARCH_PASSWORD: "${ELASTIC_PASSWORD:-changeme}"
    FLEET_URL: https://fleet-server:8220
    FLEET_SERVER_ELASTICSEARCH_INSECURE: true
    FLEET_SERVER_INSECURE_HTTP: true
    FLEET_INSECURE: true

    # Prepare Kibana for Fleet
    KIBANA_FLEET_SETUP: "1"
    KIBANA_FLEET_HOST: "https://kibana:${KIBANA_PORT}"
    # KIBANA_FLEET_USERNAME: "${ES_SUPERUSER_USER:-elastic}"
    # KIBANA_FLEET_PASSWORD: "${ELASTIC_PASSWORD:-changeme}"
    ...
```

### Fleet server enable SSL listener

```yaml
fleet-server:
  image: docker.elastic.co/beats/elastic-agent:${STACK_VERSION}
  ...
  environment:
    ...
    #  Enable fleet server SSL
    FLEET_SERVER_CERT: /usr/share/elastic-agent/config/certs/fleet-server/fleet-server.crt
    FLEET_SERVER_CERT_KEY: /usr/share/elastic-agent/config/certs/fleet-server/fleet-server.key
    CERTIFICATE_AUTHORITIES:  /usr/share/elastic-agent/config/certs/ca/ca.crt
    ...
```

### Fleet server monitoring status

Fleet server provides rest API to obtain its healtch check status:

* api/status

### Fleet server resources

* [Fleet server environment variables](https://www.elastic.co/guide/en/fleet/current/agent-environment-variables.html){:target="_blank"}