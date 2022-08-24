# airflow_basics
Some notes on Airflow basics for beginners. Partially based on course [Apache Airflow: The Hands-On Guide](https://udemy.com/course/the-ultimate-hands-on-course-to-master-apache-airflow). 

## Docker basics
**Docker:** a platform for developing, shipping, and running applications that  enables to separate applications from  infrastructure

**Docker compose:** yaml syntax-description of services to run, multi-container Docker apps. If you have Desktop installed then you already have the Compose plugin installed.

### Commands (Command Line Interface, CLI)
`docker build -t airflow-basic .` Build a docker image from the Dockerfile in the current directory (airflow-materials/airflow-basic)  and name it airflow-basic

`docker run --rm -d -p 8080:8080 airflow-basic` Run it

`docker ps` Show running docker containers

`docker exec -it container_id /bin/bash` Execute the command /bin/bash in the container_id to get a shell session

`docker stop <container_id>` Stop the container, ID can be found with `docker ps`

`docker-compose up -d --build` Build images before starting containers

### Small tip:
Just run `airflow tasks test` each time you  create a new task: helps to save a lot of time


## Airflow 
### Refreshing
Both the webserver and scheduler parse your DAGs. You can configure this parsing process with different configuration settings.

**With the Scheduler:**

`min_file_process_interval` - number of seconds after which a DAG file is parsed. The DAG file is parsed every min_file_process_interval number of seconds. Updates to DAGs are reflected after this interval.

`dag_dir_list_interval` - how often (in seconds) to scan the DAGs directory for new files. Default to 5 minutes.
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
