# Install and configure Apache Airflow on Ubuntu Server

[Apache Airflow](https://airflow.apache.org/) is one of the most widely used data orchestration and workflow management tools.
This open-source tool was started in Airbnb in October 2014 and written in Python.
Airflow allows data engineers to orchestrate ETL/ELT data pipelines by authoring, scheduling, logging, 
monitoring and troubleshooting DAG runs. If you want to see an example using Apache Airflow DAG to orchestrate a data 
ingestion (ELT) process, [click here](https://github.com/fingersfun/data-integration-with-apache-airflow).

This how-to guide will walk you through the sequential steps required to install and configure Apache Airflow 
on Ubuntu 20.04 server.

![Apache Airflow logo](docs/images/apache_airflow_logo.png) &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ![Ubuntu logo](docs/images/ubuntu_logo.png)

### Prerequisite:

* Ubuntu 20.04 (LTS) [download and install here](https://releases.ubuntu.com/focal/)

The recommended minimum system requirements for Airflow are:

* 64 bit Operating System
* 4GB RAM or more
* 2 vCPUs or more

Following these instruction should take about 30 minutes.

1. Open ubuntu terminal
    
    This is the recommended command line interface to install and configure Airflow on Ubuntu

2. Install and update packages

    `sudo apt update` downloads package information from all configured sources
    
    `sudo apt install software-properties-common` provides an abstraction of the used apt repositories allowing you to 
    easily manage your distribution and independent vendor software sources
    
    `sudo apt list --upgradable` list available updates but do not install them
    
    `sudo apt-get update` install newer version of existing packages
    
3. Install python 3.8

    `sudo apt update`
    
    `sudo apt install python3.8` download and install python version 3.8
    
    `python3 --version` check your ubuntu server has correctly installed python version 3.8
    
4. Update pip to the latest version

    pip is the default package-management system used to install and manage software packages 
    written in Python programming language
    
    `sudo apt-get install python-setuptools` python library to facilitate the packaging of Python 
    projects by enhancing the Python standard library distutils
    
    `sudo apt-add-repository universe` community-maintained free and open-source software
    
    `sudo apt-get update`
    
    `sudo apt install python3-pip` install pip
    
    `sudo pip3 install --upgrade pip` upgrade pip to latest version
    
5. Install PostgreSQL for Airflow

    `sudo apt-get install postgresql postgresql-contrib`
    
6. Grant access to root user (using sudo) to run the command line tool for PostgreSQL psql

    `sudo -u postgres psql`
    
7. Create database and user and provide privileges to it using PostgreSQL psql

    `CREATE USER airflow password '<POSTGRES-USER-PASSWORD-HERE>';` create airflow user.
    
    `CREATE DATABASE airflow;` create airflow database
    
    `GRANT ALL PRIVILEGES on database airflow to airflow;` Grant airflow user all privileges to airflow database
    
    `ALTER ROLE airflow SUPERUSER;` grant superuser level access to airflow user
    
    `ALTER ROLE airflow CREATEDB;` grant airflow user the access to create new databases
    
    `GRANTR ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO airflow;` grant airflow user access to all tables in public schema
    
    `\du` or `du+` list all user accounts (or roles) in the PostgreSQL database server
    
    `\c airflow` connect to airflow database and get connection information. After successful connection, 
    prompt will be changed to `airflow-#`
    
    `\conninfo` verify connection to airflow database by fetching connection information.
    
    `\q` exit psql command line shell
    
8. Restart PostgreSQL to load new changes

    `sudo service postgresql restart`
    
9. Install Ubuntu dependencies required for Airflow (Optional)

    All installations here are optional. You can skip to the next step if you do not require any of these packages
    
    `sudo apt-get install libmysqlclient-dev` for Airflow MySQL
    
    `sudo apt-get install libssl-dev` for Airflow cryptographic package 
    
    `sudo apt-get install libkrb5-dev` for Airflow Kerbero package
    
    `sudo apt-get install libsasl2-dev` for Airflow Hive package
    
10. Set up Airflow Default Home environment variable (Airflow Installation)

    `export AIRFLOW_HOME=~/airflow`
    
11. Install airflow, its packages and other subpackages

    `sudo install "apache-airflow[postgresql, celery, rabbitmq]"` install together
    
    OR install separately
    
    `sudo install apache-airflow`
    
    `sudo install apache-airflow[postgres]`
    
    `sudo install apache-airflow[celery]`
    
    `sudo install apache-airflow[rabbitmq]`
    
12. Initialise Airflow's database backend

    `airflow initdb`
    
13. Setup and configure Airflow configuration file (i.e. airflow.cfg)

    Airflow airflow.cfg config file should now be generated in airflow home directory. 
    
    `nano ~/airflow/airflow.cfg` open Airflow config file
    
    Update the config file with the below
    
    `executor = CeleryExecutor` useful for scaling out the number of worker nodes
    
    `sql_alchemy_conn = postgres+psycopg2://airflow:<POSTGRES-USER-PASSWORD-HERE>@localhost:5432/airflow` 
    PostgreSQL connection string for airflow database.
    
    `load_examples = False` removes default Airflow examples. True is recommended in Development and Staging. False is recommended in Production
    
    `default_timezone = Europe/London` update IANA timezone
    
    `default_ui_timezone = Europe/London` update IANA timezone
    
    `broker_url = amqp://guest:guest@localhost:5672//` celery settings for broker url.
    This connection string is not recommended for Production
    
    `result_backend = amqp://guest:guest@localhost:5672//` celery setting for result backend.
    This connection string is not recommended for Production
    
14. Load updated made to Airflow config file

    `airflow initdb`
    
15. Install and configure RabbitMQ

    RabbitMQ is a message broker that is required to run Airflow DAGs when using Celery
    
    `sudo apt install rabbitmq-server` install RabbitMQ
    
    `nano /etc/rabbitmq/rabbit-env-conf` open RabbitMQ config file
    
    Update the config file with the below
    
    `NODE_IP_ADDRESS=0.0.0.0` update IPv4 Address
    
    `NODE_PORT=5672` update port
    
16. Start RabbitMQ service

    `sudo service rabbitmq-server start`
    
17. Install Celery

    Celery is an open or asynchronous task or job queue which is based on distributed message passing
    
    `sudo pip3 install celery==4.4.7` Recommended as this is currently the most compatible version
    
18. Start Airflow

    Go through step 10. (i.e. `export AIRFLOW_HOME=~/airflow`) to ensure Airflow's default home environment variable is setup
    
    `mkdir ~/airflow/dags/` This is the folder Production Airflow DAGs will be stored
    
    Start all Airflow services 
    
    `airflow webserver -D` Start airflow web server. This starts a python flask app and -D runs them as a daemon or background process

    `airflow scheduler -D` Start to ensure scheduler is running for Airflow jobs and tasks
    
    `airflow worker -D` create multiple Airflow worker processes
    
    `http://localhost:8080` or `http://<IP-ADDRESS>:8080` Opens Airflow Web UI
    
19. Stop Airflow

    It is tricky to stop Airflow when running it as a daemon.
    
    `cat $AIRFLOW_HOME/airflow-webserver.pid` gets AIRFLOW-PROCESS-ID
    
    `sudo kill -9 <AIRFLOW-PROCESS-ID>` kill Airflow process using printed AIRFLOW-PROCESS-ID
    
