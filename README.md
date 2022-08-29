# airflow_basics
Some notes on Airflow basics for beginners. Partially based on course [Apache Airflow: The Hands-On Guide](https://udemy.com/course/the-ultimate-hands-on-course-to-master-apache-airflow) and mostly on the [official documentation](https://airflow.apache.org/docs/apache-airflow/stable/index.html). The followng commands and code worked for Win10, Airflow in Docker.

## Table of Contents
1. [Docker basics](#docker-basics):
	1. [Docker basics](#docker-basics)
	1. [Commands](#commands)
2. [Airflow](#airflow):
	1. [Commands](#commands)
	2. [Parameters](#parameters)
	3. [DAGs folder organization](#dags-folder-organization)
	4. [Refreshing](#refreshing)
	5. [Operators](#operators)


## Docker 
### Docker basics
**Docker:** a platform for developing, shipping, and running applications that  enables to separate applications from  infrastructure

**Docker compose:** yaml syntax-description of services to run, multi-container Docker apps. If you have Desktop installed then you already have the Compose plugin installed.

![](https://github.com/tashatsar/airflow_basics/blob/main/photo_2022-08-23_23-40-33.jpg)

### Commands
`docker build -t airflow-basic .` Build a docker image from the Dockerfile in the current directory (airflow-materials/airflow-basic)  and name it airflow-basic

`docker run --rm -d -p 8080:8080 airflow-basic` Run it

`docker ps` Show running docker containers

`docker exec -it container_id //bin//bash` Open a bash terminal in a container

`docker stop <container_id>` Stop the container, ID can be found with `docker ps`

`docker-compose up -d --build` Build images before starting containers

## Airflow 

### Commands (Command Line Interface, CLI)

Not the only, but a pretty convinient way to run airflow commands is from bash command line. Let's get into bash command line of a particular docker container with a command `docker exec -it container_id //bin//bash`.

- **"Historical" predictions** `airflow dags backfill --start-date START_DATE --end-date END_DATE dag_id`. More params [here](https://airflow.apache.org/docs/apache-airflow/stable/cli-and-env-variables-ref.html#backfill).
- For **debug** just run `airflow tasks test <dag_id> <task_id> <execution_date_in_the_past>` each time you  create a new task: helps to save a lot of time of possible debugging. This command runs task instances locally, outputs their log to stdout (on screen), does not bother with dependencies, and does not communicate state (running, success, failed, …) to the database. It simply allows testing a single task instance.
- For **debug on a DAG level**: the same applies to `airflow dags test <dag_id> <execution_date_in_the_past>`, but on a DAG level. It performs a single DAG run of the given DAG id. While it does take task dependencies into account, no state is registered in the database. It is convenient for locally testing a full run of your DAG, given that e.g. if one of your tasks expects data at some location, it is available.

### Parameters

- `start_date` can be defined as a datetime object `datetime.datetime(2022.01.01)` in the past or in the future and can be set on a task level (which **is not** recommended). It is not the execution date, execution date equals to `start_date+schedule_interval`. **#bestpractice**: set `start_date` globally!
- `schedule_interval` by default is daily (at 00:00 UTC), can be defined as cron expression or datetime object. Possible presets: `@once, @hourly, @daily, @weekly, @monthly, @yearly`. **#bestpractice**: set `schedule_interval` as cron expression because of possible issues with timezones in case on Python datetime objects. 
- `end_date` it's pretty obvious, isn't it?
Let’s Repeat That: The scheduler runs your job one schedule_interval AFTER the start date, at the END of the period.
- Catchup: The scheduler, by default, will kick off a DAG Run for any data interval that has not been run since the last data interval (or has been cleared). It can be turned off either on the DAG itself with `dag.catchup = False` or by default at the configuration file level with `catchup_by_default = False`. 
- `depends_on_past` is defined at the **task** level. For example:
1. `DAGRun (depends_on_past=False): task_A (successeed) -> task_B (successeed) -> task_C (successeed)`
2. `DAGRun (depends_on_past=False): task_A (successeed) -> task_B (failed) -> task_C (failed)` task_C failed because task_B failed first
3. `DAGRun (depends_on_past=True): task_A (successeed) -> task_B (failed) -> task_C (not triggered)`
- `wait_for_downstream` is also defined at the **task** level. For giving an example of usage (suppose that depends_on_past=True for all cases):
1. `DAGRun 1 (wait_for_downstream=True): task_A (successeed) -> task_B (successeed) -> task_C (successeed)`
2. `DAGRun 2 (wait_for_downstream=True): task_A (successeed) -> task_B (in process of execution) -> task_C (not triggered yet)` 
3. `DAGRun 3 (wait_for_downstream=True): task_A (not triggered yet) -> task_B (not triggered yet) -> task_C (not triggered yet)` because wait_for_downstream is set to True and this DAG run cannot be started before DAG run 2 is finished. Thus, DAG run 3 is waiting for tasks B and C of DAG run 2 to be executed. 

### DAGs folder organization

**Don't want anyone to see some of your presious DAGs in the UI?** Use `.airflowignore` file with regex!  **#bestpractice**: always to have a `.airflowignore` file even if it's empty. 

**Wanna hide this mess of files and modules?** You can use zip files! Why? In order to:
- combine several dags together to version them together.
- manage them together.
- have an extra module that is not available by default on the system you are running airflow on.

In more detailes there is [documentation](https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html?highlight=zip#packaging-dags)

**Tired of storing everything in the same directory!?** DAGBag can help a lot! A dagbag is a collection of dags, parsed **out of a folder tree**. It makes it easier to run distinct environments for say production and development, tests, or for different teams or security profiles. What would have been system level settings are now dagbag level so that one system can run multiple, independent settings sets. **#bestpractice**: DAGBag should have high level configuration settings, like what database to use as a backend and what executor to use to fire off tasks. 
**Drawbacks: errors and problems of DAGBags won't appear in the Airflow UI, only logs of command line**

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
