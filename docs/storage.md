Mainflux supports various storage databases in which messages are stored:

- CassandraDB
- MongoDB
- InfluxDB
- PostgreSQL

These storages are activated via docker-compose add-ons.

The `<project_root>/docker` folder contains an `addons` directory. This directory is used for various services that are not core to the Mainflux platform but could be used for providing additional features.

In order to run these services, core services, as well as the network from the core composition, should be already running.

## Writers

Writers provide an implementation of various `message writers`. Message writers are services that consume Mainflux messages, transform them to `SenML` format and store them in specific data store. Supported message payload formats are SenML+CBOR and SenML+JSON. They are configurable over environment variables in each writer (`MF_CASSANDRA_WRITER_CONTENT_TYPE`, `MF_POSTGRES_WRITER_CONTENT_TYPE`, `MF_INFLUX_WRITER_CONTENT_TYPE` and `MF_MONGO_WRITER_CONTENT_TYPE`) and expect `application/senml+json` or `application/senml+cbor` formats.

Each writer can filter messages based on subjects list that is set in `subjects.toml` configuration file. If you want to listen on all subjects, just pass one element ["channels.>"], otherwise pass the list of subjects. Here is an example:

```toml
[subjects]
filter = ["channels.>"]
```

Regarding the [Subtopics Section](messaging.md#subtopics) in the messaging page, the example `channels/<channel_id>/messages/bedroom/temperature` can be filtered as `"channels.*.bedroom.temperature"`. The formatting of this filtering list is determined by the NATS format ([Subject-Based Messaging](https://docs.nats.io/nats-concepts/subjects) & [Wildcards](https://docs.nats.io/nats-concepts/subjects#wildcards)).

### InfluxDB, InfluxDB Writer and Grafana

From the project root execute the following command:

```bash
docker-compose -f docker/addons/influxdb-writer/docker-compose.yml up -d
```

This will install and start:

- [InfluxDB](https://docs.influxdata.com/influxdb) - time series database
- InfluxDB writer - message repository implementation for InfluxDB
- [Grafana](https://grafana.com) - tool for database exploration and data visualization and analytics

Those new services will take some additional ports:

- 8086 by InfluxDB
- 8900 by InfluxDB writer service
- 3001 by Grafana

To access Grafana, navigate to `http://localhost:3001` and login with: `admin`, password: `admin`

### Cassandra and Cassandra Writer

```bash
./docker/addons/cassandra-writer/init.sh
```

_Please note that Cassandra may not be suitable for your testing environment because of its high system requirements._

### MongoDB and MongoDB Writer

```bash
docker-compose -f docker/addons/mongodb-writer/docker-compose.yml up -d
```

MongoDB default port (27017) is exposed, so you can use various tools for database inspection and data visualization.

### PostgreSQL and PostgreSQL Writer

```bash
docker-compose -f docker/addons/postgres-writer/docker-compose.yml up -d
```

Postgres default port (5432) is exposed, so you can use various tools for database inspection and data visualization.

## Readers

Readers provide an implementation of various `message readers`.
Message readers are services that consume normalized (in `SenML` format) Mainflux messages from data storage and opens HTTP API for message consumption.
Installing corresponding writer before reader is implied.

Each of the Reader services exposes the same [HTTP API](https://github.com/mainflux/mainflux/blob/master/readers/openapi.yml) for fetching messages on its default port.

To read sent messages on channel with id `channel_id` you should send `GET` request to `/channels/<channel_id>/messages` with thing access token in `Authorization` header. That thing must be connected to  channel with `channel_id`

Response should look like this:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 18 Sep 2018 18:56:19 GMT
Content-Length: 228

{
    "messages": [
        {
            "Channel": 1,
            "Publisher": 2,
            "Protocol": "mqtt",
            "Name": "name:voltage",
            "Unit": "V",
            "Value": 5.6,
            "Time": 48.56
        },
        {
            "Channel": 1,
            "Publisher": 2,
            "Protocol": "mqtt",
            "Name": "name:temperature",
            "Unit": "C",
            "Value": 24.3,
            "Time": 48.56
        }
    ]
}
```

Note that you will receive only those messages that were sent by authorization token's owner.
You can specify `offset` and `limit` parameters in order to fetch specific group of messages. An example of HTTP request looks like:

```bash
curl -s -S -i  -H "Authorization: <thing_token>" http://localhost:<service_port>/channels/<channel_id>/messages?offset=0&limit=5
```

If you don't provide these parameters, default values will be used instead: 0 for `offset` and 10 for `limit`.

### InfluxDB Reader

To start InfluxDB reader, execute the following command:

```bash
docker-compose -f docker/addons/influxdb-reader/docker-compose.yml up -d
```

### Cassandra Reader

To start Cassandra reader, execute the following command:

```bash
docker-compose -f docker/addons/cassandra-reader/docker-compose.yml up -d
```

### MongoDB Reader

To start MongoDB reader, execute the following command:

```bash
docker-compose -f docker/addons/mongodb-reader/docker-compose.yml up -d
```

### PostgreSQL Reader

To start PostgreSQL reader, execute the following command:

```bash
docker-compose -f docker/addons/postgres-reader/docker-compose.yml up -d
```
