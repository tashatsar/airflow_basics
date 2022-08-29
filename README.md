# airflow_basics
Some notes on Airflow basics for beginners. Partially based on course [Apache Airflow: The Hands-On Guide](https://udemy.com/course/the-ultimate-hands-on-course-to-master-apache-airflow). The followng commands and code worked for Win10, Airflow in Docker. 

## Docker basics
**Docker:** a platform for developing, shipping, and running applications that  enables to separate applications from  infrastructure

**Docker compose:** yaml syntax-description of services to run, multi-container Docker apps. If you have Desktop installed then you already have the Compose plugin installed.

![](https://github.com/tashatsar/airflow_basics/blob/main/photo_2022-08-23_23-40-33.jpg)

### Commands (Command Line Interface, CLI)
`docker build -t airflow-basic .` Build a docker image from the Dockerfile in the current directory (airflow-materials/airflow-basic)  and name it airflow-basic

`docker run --rm -d -p 8080:8080 airflow-basic` Run it

`docker ps` Show running docker containers

`docker exec -it container_id //bin//bash` Open a bash terminal in a container

`docker stop <container_id>` Stop the container, ID can be found with `docker ps`

`docker-compose up -d --build` Build images before starting containers

### Small tip:
Just run `airflow tasks test` each time you  create a new task: helps to save a lot of time
`airflow tasks test <dag_is> <task_id> <execution date in the past>`

## Airflow 

### Commands (Command Line Interface, CLI)

Not the only, but a pretty convinient way to run airflow commands is from bash command line. Let's get into bash command line of a particular docker container with a command `airflow dags backfill --start-date START_DATE --end-date END_DATE dag_id`

- **"Historical" predictions** `airflow dags backfill --start-date START_DATE --end-date END_DATE dag_id`. More params [here](https://airflow.apache.org/docs/apache-airflow/stable/cli-and-env-variables-ref.html#backfill)

### Parameters

- `start_date` can be defined as a datetime object `datetime.datetime(2022.01.01)` in the past or in the future and can be set on a task level (which **is not** recommended). It is not the execution date, execution date equals to `start_date+schedule_interval`. **#bestpractice**: set `start_date` globally!
- `schedule_interval` by default is daily (at 00:00 UTC), can be defined as cron expression or datetime object. Possible presets: `@once, @hourly, @daily, @weekly, @monthly, @yearly`. **#bestpractice**: set `schedule_interval` as cron expression!
- `end_date` it's pretty obvious, isn't it?

Let’s Repeat That: The scheduler runs your job one schedule_interval AFTER the start date, at the END of the period.

- Catchup: The scheduler, by default, will kick off a DAG Run for any data interval that has not been run since the last data interval (or has been cleared). It can be turned off either on the DAG itself with `dag.catchup = False` or by default at the configuration file level with `catchup_by_default = False`. 

### Refreshing
Both the webserver and scheduler parse your DAGs. You can configure this parsing process with different configuration settings. Configurations in general can be set set in `airflow.cfg` file or using environment variables.

**With the Scheduler:**

- `min_file_process_interval` - number of seconds after which a DAG file is parsed. The DAG file is parsed every min_file_process_interval number of seconds. Updates to DAGs are reflected after this interval.
- `dag_dir_list_interval` - how often (in seconds) to scan the DAGs directory for new files. Default to 5 minutes.
Those 2 settings tell you that you have to wait up 5 minutes before your DAG gets detected by the scheduler and then it is parsed every 30 seconds by default.

**With the Webserver:**

`worker_refresh_interval` - number of seconds to wait before refreshing a batch of workers. 30 seconds by default.
This setting tells you that every 30 seconds, the web server parses for new DAG in your DAG folder.

### Operators 
Basics
Operator = Task. Types of operators: 
- action
- transfer
- sensor
