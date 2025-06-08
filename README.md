# Airflow RabbitMQ Highly Available Cluster Docker Compose example
## Description
This configuration example helps to increase the failover of Airflow by using a highly available RabbitMQ cluster and multiple Airflow schedulers. The entire infrastructure (Airflow, RabbitMQ, and PostgreSQL) will be deployed using Docker Compose.

According to the [RabbitMQ documentation](https://www.rabbitmq.com/docs/clustering#:~:text=(three%2C%20five%2C%20seven%2C%20or%20more)), it's recommended to use at least 3 RabbitMQ cluster nodes. Additionally, there should be an odd number of nodes, with each running on a separate host.

## Features of Multiple Airflow Schedulers Configuration.
The [Airflow documentation](https://airflow.apache.org/docs/apache-airflow/stable/administration-and-deployment/scheduler.html#running-more-than-one-scheduler) describes the possibility of using multiple schedulers. However, it is not explicitly stated that needs to be done in order to launch an Airflow cluster with multiple schedulers.

So! To run multi-schedulers Airflow cluster, it is necessary to specify the same connection to PostgreSQL in the configuration of each scheduler.
An example of an additional scheduler configuration can be found in the `airflow-scheduler/docker-compose.yml` file.


## Features of the RabbitMQ High Availability Cluster Configuration

The `definitions.json` file contains configurations for full cluster mirroring (`"ha-mode": "all"`). This means all queues and messages will be replicated across all nodes. Additionally, the automatic cluster recovery mode is enabled (`"ha-sync-mode": "automatic"`) in case one or more nodes fail.

This configuration ensures that:
- Airflow continues operating without interruption even if only 1 RabbitMQ node remains available
- When failed nodes recover, they automatically rejoin the cluster without manual intervention
- Full message redundancy is maintained across all active nodes

## Setup Steps
0. Prerequisite: Docker Compose should be installed on all hosts, and they should be in the same network.
1. Deploy RabbitMQ nodes on all hosts (3 nodes in this example).
   When configuring RabbitMQ, you need to pay special attention to the RABBITMQ_ERLANG_COOKIE parameter. IT MUST BE IDENTICAL ON ALL NODES. Otherwise, clusterization will not occur. 
3. Set up PostgreSQL on a remote host (Airflow documentation recommends keeping the database separate from the main Airflow host).
4. Configure the Airflow `docker-compose.yml` to connect to PostgreSQL and RabbitMQ (via AMQP):

   | # | Setting | Value |
   |---|---|---|
   | 1 | `AIRFLOW__CELERY__RESULT_BACKEND` | `postgresql://${PG_USER}:${PG_PASSWORD}@${PG_HOST}:5432/airflow` |
   | 2 | `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN` | `postgresql://${PG_USER}:${PG_PASSWORD}@${PG_HOST}:5432/airflow` |
   | 3 | `AIRFLOW__CELERY__BROKER_URL` | `amqp://admin:admin@${RABBITMQ_MASTER:-rabbitmq-master}:5672/;amqp://admin:admin@${RABBITMQ_SLAVE_1:-rabbitmq-slave1}:5672/;amqp://admin:admin@${RABBITMQ_SLAVE_2:-rabbitmq-slave2}:5672/` |
Attention! The environment block must be the same in all airflow nodes.
5. Deploy additional Airflow schedulers on remote hosts using the configuration from the `airflow-schedulers` folder. Airflow documentation recommends having at least 3 schedulers on different servers.
6. Start Airflow on the main host.

## Bonus
This configuration uses Statsd-exporter to collect Airflow metrics, available at:
[http://remote-host:9123/metrics](http://remote-host:9123/metrics)

For RabbitMQ monitoring, refer to:
[https://www.rabbitmq.com/docs/monitoring](https://www.rabbitmq.com/docs/monitoring)
