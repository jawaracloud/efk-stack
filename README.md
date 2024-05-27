# efk-network Stack
Log Monitoring with Elasticsearch, Fluent-bit, Kibana

![The Flow of EFK Stack !](/assets/flow.gif "The Flow of EFK Stack ")

## Create Docker Network and Volume
```bash
docker network create efk-network --driver bridge
docker volume create nginx_logs --driver local
docker volume create elastic_data --driver local
```

## Run NGINX with Docker Run
```bash
docker run \
-d \
--name nginx \
--network efk-network \
-v ./nginx_logs:/var/log/nginx \
-p 8080:80 \
nginx:1.26.0-alpine
```

## Run Fluent-bit with Docker Run
```bash
docker run -d \
--name fluent_bit \
--restart always \
-v $(pwd)/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf \
-v $(pwd)/parsers.conf:/fluent-bit/parsers.conf \
-v $(pwd)/nginx_logs:/var/log/nginx \
fluent/fluent-bit:3.0.6 \
fluent-bit \
-c /fluent-bit/etc/fluent-bit.conf \
-R /fluent-bit/parsers.conf
```

## Run Elasticsearch with Docker Run
```bash
docker run \
-d \
--name elasticsearch \
--network efk-network \
-v elastic_data:/usr/share/elasticsearch/data \
-e "discovery.type=single-node" \
-e "xpack.security.enabled=false" \
--restart always \
docker.elastic.co/elasticsearch/elasticsearch:8.13.4
```

## Run Kibana with Docker Run
```bash
docker run \
-d \
--name kibana \
--network efk-network \
-p 5601:5601 \
--restart always \
-e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" \ docker.elastic.co/kibana/kibana:8.13.4
```

## Docker Compose Version
```bash
docker-compose up -d .
```

## Access Kibana
- Open http://localhost:5601 in your browser
- Go to Management -> Index Patterns -> Create Index Pattern
- Enter `nginx-log-*` and click Next
- Select `@timestamp` and click Create Index Pattern
- Go to Discover and you should see logs from NGINX
