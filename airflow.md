# Install

1. Install Python 3.
2. Install MySQL.
3. Install libmysqlclient-dev.
4. Create virtual env:
```
$ virtualenv -p $LOCAL/bin/python3 $LOCAL/python-envs/AIRFLOW-ENV
```
5. Install airflow with MySQL support, and useful packages.
```
$ source $LOCAL/python-envs/AIRFLOW-ENV/bin/activate
$ pip install cryptography apache-airflow[mysql]
```
6. Set `AIRFLOW_HOME`.
```
$ export AIRFLOW_HOME=$LOCAL/apps/airflow
```
7. Try starting airflow so `airflow.cfg` gets created.
```
$ airflow scheduler
```
8. Edit `airflow.cfg` to configure executor, MySQL, SMTP, and example dags.
```
$ emacs $AIRFLOW_HOME/airflow.cfg
```

```
...
load_examples = False
...
executor = LocalExecutor
...
#sql_alchemy_conn = sqlite:////Users/dnikolov/Local/apps/airflow/airflow.db
sql_alchemy_conn = mysql://airflow:<enter password>@localhost/airflowdb?unix_socket=/tmp/mysql.sock
...
```

```
...
[smtp]
smtp_host = mail-relay.iu.edu
smtp_starttls = True
smtp_ssl = False
smtp_user = dnikolov
smtp_password = <enter password>
smtp_port = 587
smtp_mail_from = dnikolov@iu.edu
...
```
9. Init db
```
$ airflow initdb
```
10. Start the scheduler and webserver
```
$ airflow scheduler
$ ariflow webserver
```
11. Access the Web UI at https://localhost:8080
