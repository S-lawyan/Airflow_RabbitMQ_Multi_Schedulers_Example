# Airflow_RabbitMQ_example
## Description
This configuration example helps improve Airflow's fault tolerance by utilizing a highly available RabbitMQ cluster. The entire infrastructure (Airflow, RabbitMQ, and PostgreSQL) will be deployed using Docker Compose.

According to the [RabbitMQ documentation](https://www.rabbitmq.com/docs/clustering#:~:text=(three%2C%20five%2C%20seven%2C%20or%20more)), it's recommended to use at least 3 RabbitMQ cluster nodes. Additionally, there should be an odd number of nodes, with each running on a separate host.

## Setup Steps
0. Prerequisite: Docker Compose should be installed on all hosts, and they should be in the same network.
1. Deploy RabbitMQ nodes on all hosts (3 nodes in this example).
2. Set up PostgreSQL on a remote host (Airflow documentation recommends keeping the database separate from the main Airflow host).
3. Configure the Airflow `docker-compose.yml` to connect to PostgreSQL and RabbitMQ (via AMQP):

   | # | Setting | Value |
   |---|---|---|
   | 1 | `AIRFLOW__CELERY__RESULT_BACKEND` | `postgresql://${PG_USER}:${PG_PASSWORD}@${PG_HOST}:5432/airflow` |
   | 2 | `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN` | `postgresql://${PG_USER}:${PG_PASSWORD}@${PG_HOST}:5432/airflow` |
   | 3 | `AIRFLOW__CELERY__BROKER_URL` | `amqp://admin:admin@${RABBITMQ_MASTER:-rabbitmq-master}:5672/;amqp://admin:admin@${RABBITMQ_SLAVE_1:-rabbitmq-slave1}:5672/;amqp://admin:admin@${RABBITMQ_SLAVE_2:-rabbitmq-slave2}:5672/` |

4. Deploy additional Airflow schedulers on remote hosts using the configuration from the `airflow-schedulers` folder. Airflow documentation recommends having at least 3 schedulers on different servers.
5. Start Airflow on the main host.

## Bonus
This configuration uses Statsd-exporter to collect Airflow metrics, available at:
[http://remote-host:9123/metrics](http://remote-host:9123/metrics)
For RabbitMQ monitoring, refer to:
[https://www.rabbitmq.com/docs/monitoring](https://www.rabbitmq.com/docs/monitoring)
