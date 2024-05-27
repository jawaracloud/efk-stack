# efk-network Stack
Log Monitoring with Elasticsearch, Fluent-bit, Kibana

![The Flow of EFK Stack !](/assets/flow.gif "The Flow of EFK Stack ")


## Create Docker Network and Volume
```bash
docker network create efk-network --driver bridge
docker volume create nginx_logs --driver local
docker volume create elastic_data --driver local
```

## Run NGINX with Docker
```bash
docker run \
-d \
--name nginx \
--network efk-network \
-v nginx_logs:/var/log/nginx \
-p 8080:80 \
nginx:alpine
```

## Run Fluent-bit with Docker
```bash
docker run \
-d \
--name fluent-bit \
--network efk-network \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/fluent-bit:/fluent-bit/etc \
-v nginx_logs:/var/log/nginx \
- ./fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf \
fluent/fluent-bit:1.6
```

## Run Elasticsearch with Docker
```bash
docker run \
-d \
--name elasticsearch \
--network efk-network \
-v elastic_data:/usr/share/elasticsearch/data \
-e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.9.3
```

## Run Kibana with Docker
```bash
docker run \
-d \
--name kibana \
--network efk-network \
-p 5601:5601 \
-e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" \ docker.elastic.co/kibana/kibana:7.9.3
```

## Access Kibana
- Open http://localhost:5601 in your browser
- Go to Management -> Index Patterns -> Create Index Pattern
- Enter `fluent-bit-*` and click Next
- Select `@timestamp` and click Create Index Pattern
- Go to Discover and you should see logs from NGINX
