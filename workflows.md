# Workflow Management Tools

## Pegasus

* Web: [Docs](https://pegasus.isi.edu/documentation/), [Github](https://github.com/pegasus-isi/pegasus)
* License: [Apache 2](https://tldrlegal.com/license/apache-license-2.0-(apache-2.0))

### Features
* Easy integration with HPC batch systems like **PBS/Torque**.
* Support for **Docker** and **Singularity** containers in the Transformation/Executables catalog.
* Integration with **Jupyter notebooks**: In addition to the DAX API for creating workflows, Pegasus has a Python API for running and monitoring workflows from Jupyter notebooks or other Python code.
* For **short tasks** where the overhead of running them on a cluster is too big, Pegasus has a **task clustering**, which combines the short task together in a bigger job. This feature needs to be tested further to determine how useful in practice it is.

### Configuration and Ease of Use
* Workflows are **defined as code**. Interfaces exist for **Python, R, Java, and Perl**. As such, anyone using HPC infrastructure should be able to define the DAGs without much trouble. This means that **defining the workflow is like writing another script** for your project. This is OK, but it lacks the self-documenting, easy-to-translate from individual commands to an automated workflow, nature that Snakemake has.
* In Pegasus, **the workflows are independent of the underlying physical infrastructure** (location of data, executables, cluster end-points) where they are run. This makes them **portable**, but it results in a more abstract workflow that you then have to configure with the physical infrastructure parameters. There are Site (temp and output storage locations), Transformation (executable locations), and Replica (input locations) "catalogs" (XML files) that need to be configured for a workflow. Thus, **defining the workflow is abstract and cumbersome**, and it results in extra jobs for file management being added to the workflow. This diagram illustrates this nicely: ![Workflow composition with the Site, Transformation, and Replica catalogs.](https://pegasus.isi.edu/documentation/images/tutorial-pegasus-catalogs.png)
* While a bit cumbersome, **the catalogs streamline accessing data on remote hosts**. You declare where your inputs, outputs, and scratch locations are in the catalogs, and Pegasus takes care of any file transfer that need to occur when you deploy the workflow on an HPC system This, however, is not very relevant to us since users will largely be executing these on HPC login nodes, where the same storage is mounted as on the compute nodes.

### Job Monitoring, Logging, and Summary
* A **Web app** allows you to monitor the running of the job or how the job ran. There is **a lot of detail**, perhaps too much. It is difficult to get a picture at a glance of what a specific workflow is doing. For example, a simple `split`, follower by a few `ls`s generates multiple jobs (jobs are the subunits of the workflow), not only for the actual commands, but also for cleaning up, creating directories, registering the workflow. To see what each job does you have to drill down in it, view its tasks (tasks are the subunits of a job), and then you can see the actual command represented by the job. This structure is **a little cumbersome**, but you do get a detailed, transparent view of it.
* There are a number of **command-line tools** that allow you to monitor progress, debug, and view several summaries of the workflow from the command line. These are more streamlined than the Web interface, while still offering enough detail.

### PBS/Torque Integration

Pegasus has direct support for execution on a batch HPC system powered by PBS/Torque, through the Site Catalog.

### Example Workflow
#### Definition
```python
import os
import pwd
import sys
import time
from Pegasus.DAX3 import *

# The name of the DAX file is the first argument
if len(sys.argv) != 2:
        sys.stderr.write("Usage: %s DAXFILE\n" % (sys.argv[0]))
        sys.exit(1)
daxfile = sys.argv[1]

USER = pwd.getpwuid(os.getuid())[0]

# Create a abstract dag
dax = ADAG("split")

# Add some workflow-level metadata
dax.metadata("creator", "%s@%s" % (USER, os.uname()[1]))
dax.metadata("created", time.ctime())

webpage = File("pegasus.html")

# the split job that splits the webpage into smaller chunks
split = Job("split")
split.addArguments("-l","100","-a","1",webpage,"part.")
split.uses(webpage, link=Link.INPUT)
# associate the label with the job. all jobs with same label
# are run with PMC when doing job clustering
split.addProfile( Profile("pegasus","label","p1"))
dax.addJob(split)

# we do a parmeter sweep on the first 4 chunks created
for c in "abcd":
    part = File("part.%s" % c)
    split.uses(part, link=Link.OUTPUT, transfer=False, register=False)

    count = File("count.txt.%s" % c)

    wc = Job("wc")
    wc.addProfile( Profile("pegasus","label","p1"))
    wc.addArguments("-l",part)
    wc.setStdout(count)
    wc.uses(part, link=Link.INPUT)
    wc.uses(count, link=Link.OUTPUT, transfer=True, register=True)
    dax.addJob(wc)

    #adding dependency
    dax.depends(wc, split)

f = open(daxfile, "w")
dax.writeXML(f)
f.close()
print "Generated dax %s" %daxfile
```

#### Execution
```shell
pegasus-plan --conf pegasus.properties \
    --dax $DAXFILE \
    --dir $DIR/submit \
    --input-dir $DIR/input \
    --output-dir $DIR/output \
    --cleanup leaf \
    --force \
    --sites condorpool \
    --submit
```

#### Directory Structure
```
.
├── README.md
├── daxgen.py
├── generate_dax.sh
├── input
│   └── pegasus.html
├── output
│   ├── count.txt.a
│   ├── count.txt.b
│   ├── count.txt.c
│   └── count.txt.d
├── pegasus.properties
├── plan_cluster_dax.sh
├── plan_dax.sh
├── rc.txt
├── scratch
│   └── tutorial
│       └── pegasus
│           └── split
├── sites.xml
├── sites.xml~
├── split.dax
├── submit
│   └── tutorial
│       └── pegasus
│           └── split
│               └── run0001
│                   ├── 00
│                   │   └── 00
│                   │       ├── cleanup_split_0_local.err
│                   │       ├── cleanup_split_0_local.in
│                   │       ├── cleanup_split_0_local.out
│                   │       ├── cleanup_split_0_local.sub
│                   │       ├── create_dir_split_0_local.err.000
│                   │       ├── create_dir_split_0_local.in
│                   │       ├── create_dir_split_0_local.out.000
│                   │       ├── create_dir_split_0_local.sub
│                   │       ├── register_local_1_0.err.000
│                   │       ├── register_local_1_0.in
│                   │       ├── register_local_1_0.out.000
│                   │       ├── register_local_1_0.sub
│                   │       ├── split_ID0000001.err.000
│                   │       ├── split_ID0000001.out.000
│                   │       ├── split_ID0000001.sh
│                   │       ├── split_ID0000001.sub
│                   │       ├── stage_in_remote_local_0_0.err.000
│                   │       ├── stage_in_remote_local_0_0.in
│                   │       ├── stage_in_remote_local_0_0.out.000
│                   │       ├── stage_in_remote_local_0_0.sub
│                   │       ├── stage_out_local_local_1_0.err.000
│                   │       ├── stage_out_local_local_1_0.in
│                   │       ├── stage_out_local_local_1_0.out.000
│                   │       ├── stage_out_local_local_1_0.sub
│                   │       ├── wc_ID0000002.err.000
│                   │       ├── wc_ID0000002.meta
│                   │       ├── wc_ID0000002.out.000
│                   │       ├── wc_ID0000002.sh
│                   │       ├── wc_ID0000002.sub
│                   │       ├── wc_ID0000003.err.000
│                   │       ├── wc_ID0000003.meta
│                   │       ├── wc_ID0000003.out.000
│                   │       ├── wc_ID0000003.sh
│                   │       ├── wc_ID0000003.sub
│                   │       ├── wc_ID0000004.err.000
│                   │       ├── wc_ID0000004.meta
│                   │       ├── wc_ID0000004.out.000
│                   │       ├── wc_ID0000004.sh
│                   │       ├── wc_ID0000004.sub
│                   │       ├── wc_ID0000005.err.000
│                   │       ├── wc_ID0000005.meta
│                   │       ├── wc_ID0000005.out.000
│                   │       ├── wc_ID0000005.sh
│                   │       └── wc_ID0000005.sub
│                   ├── braindump.txt
│                   ├── catalogs
│                   │   ├── rc.txt
│                   │   ├── sites.xml
│                   │   └── tc.txt
│                   ├── jobstate.log
│                   ├── monitord-notifications.log
│                   ├── monitord.done
│                   ├── monitord.info
│                   ├── monitord.log
│                   ├── monitord.started
│                   ├── monitord.subwf
│                   ├── pegasus-worker-4.8.0-x86_64_rhel_7.tar.gz
│                   ├── pegasus.7943275248767313075.properties
│                   ├── split-0.cache
│                   ├── split-0.dag
│                   ├── split-0.dag.condor.sub
│                   ├── split-0.dag.dagman.log
│                   ├── split-0.dag.dagman.out
│                   ├── split-0.dag.lib.err
│                   ├── split-0.dag.lib.out
│                   ├── split-0.dag.metrics
│                   ├── split-0.dag.metrics.out
│                   ├── split-0.dag.nodes.log
│                   ├── split-0.dot
│                   ├── split-0.exitcode.log
│                   ├── split-0.log
│                   ├── split-0.metadata
│                   ├── split-0.metrics
│                   ├── split-0.notify
│                   ├── split-0.stampede.db
│                   ├── split-0.static.bp
│                   └── split.dax
└── tc.txt

```

## Snakemake
### Summary

* Web: [Docs](http://snakemake.readthedocs.io/en/stable/), [BitBucket](https://bitbucket.org/snakemake/snakemake)
* License: [MIT License](https://tldrlegal.com/license/mit-license)

### Features
* Has native support for using in a **batch cluster HPC environment**. While each task can be configured with required nodes, cores, memory, etc. they all need to be executed in the batch system environment. 
* Has support for running jobs from **Docker and Singularity containers**.
* Has a Python API for starting and stopping jobs, which makes it possible to use with Jupyter notebooks.
* Limited commercial cloud support.

### Configuration and Ease of Use
* The **easiest to use**, and most user-friendly syntax. At the same time, Python code is supported in the `Snakefile` itself, so power users could write custom Python logic.
* The resulting `Snakefile` and configuration, is a **self-documenting experiment** that translates closely to how you would run scripts on the command line.

### Job Monitoring, Logging, and Summary
* The biggest disadvantage is there is **no job monitoring**.
* Very **limited visualization of resulting DAGs (but useful)**.

### Example Workflow
#### Definition
```yaml
configfile: "config.json"

rule all:
    input:
        '{data_dir}/share-counts/share-counts-by-url.tab'.format(**config),
        '{data_dir}/share-counts/share-counts-by-domain.tab'.format(**config),

rule create_news_sources_list:
    input:
        '{data_dir}/pageranks/part-r-00000'.format(**config),
        '{data_dir}/top500.tab'.format(**config)
    output:
        '{data_dir}/news-sources.tab'.format(**config)
    shell:
        'python {code_dir}/create_news_sources_list.py {{input}} {{output}}'.format(**config)

rule strip_tweets:
    input:
        '{data_dir}/tweets/raw'.format(**config),
        '{data_dir}/news-sources.tab'.format(**config),
    output:
        '{data_dir}/tweets/news'.format(**config)
    threads:
        config['default_num_threads']
    shell:
        'python {code_dir}/strip_tweets.py {{threads}} {{input}} {{output}}'.format(**config)

rule count_shares:
    input:
        '{data_dir}/tweets/news'.format(**config)
    output:
        '{data_dir}/share-counts/share-counts-by-url'.format(**config),
        '{data_dir}/share-counts/share-counts-by-domain'.format(**config)
    threads:
        config['default_num_threads']
    shell:
        'python {code_dir}/count_shares.py {{threads}} {{input}} {{output}}'.format(**config)

rule reduce_share_counts:
    input:
        '{data_dir}/share-counts/share-counts-by-{{agg}}'.format(**config)
    output:
        '{data_dir}/share-counts/share-counts-by-{{agg}}.tab'.format(**config)
    shell:
        'python {code_dir}/reduce.py --value_convert_fn=int --sort=reverse sum {{input}} {{output}}'.format(**config)
```

#### Execution
```bash
$ snakemake --cores 16
```

#### Directory Structure
```bash
$ tree
.
├── README.md
├── Snakefile
├── config.json
└── scripts
    ├── config.py
    ├── count_shares.py
    ├── create_news_sources_list.py
    ├── reduce.py
    ├── strip_tweets.py
    └── urls.py
```

## Airflow and Luigi

### Features
* Less friendly to HPC and batch system environments. However, with some limited coding, we can provide a solution. A SLURM one for luigi already exists, which would be very similar to a PBS/Torque solution. A solution for Airflow is on the list of planned features for 2017. luigi also has a sciluigi wrapper that makes working with scientific workflows easier.
* Major difference is Airflow has its own **scheduler**, while for luigi, you need to use cron jobs.
* DAGs are defined as Python code. Familiarity with Python and OOP is required.

### Job Monitoring, Logging, and Summary
* Both provide useful command-line logging.
* GUI for monitoring jobs is more extensive in Airflow.

## Airflow

* Web: [Docs](https://airflow.apache.org/), [Github](https://github.com/apache/incubator-airflow)
* License: [Apache 2](https://tldrlegal.com/license/apache-license-2.0-(apache-2.0))

### Example Workflow
#### Definition
```python
"""
Code that goes along with the Airflow located at:
http://airflow.readthedocs.org/en/latest/tutorial.html
"""
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta


default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2015, 6, 1),
    'email': ['airflow@airflow.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'queue': 'bash_queue',
    'pool': 'backfill',
    'priority_weight': 10,
    'end_date': datetime(2016, 1, 1),
}

dag = DAG(
    'tutorial', default_args=default_args, schedule_interval=timedelta(1))

# t1, t2 and t3 are examples of tasks created by instantiating operators
t1 = BashOperator(
    task_id='print_date',
    bash_command='date',
    dag=dag)

t2 = BashOperator(
    task_id='sleep',
    bash_command='sleep 5',
    retries=3,
    dag=dag)

templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id='templated',
    bash_command=templated_command,
    params={'my_param': 'Parameter I passed in'},
    dag=dag)

t2.set_upstream(t1)
t3.set_upstream(t1)
```

#### Execution
```shell
$ python workflow1.py
$ airflow list_dags
$ airflow list_tasks workflow1
$ airflow list_tasks workflow1 --tree
```

## luigi
### Summary

* Web: [Docs](http://luigi.readthedocs.io/en/stable/workflows.html), [Github](https://github.com/spotify/luigi)
* License: [Apache 2](https://tldrlegal.com/license/apache-license-2.0-(apache-2.0))

### Example Workflow
#### Definition
```python
class AggregateArtists(luigi.Task):
    date_interval = luigi.DateIntervalParameter()

    def output(self):
        return luigi.LocalTarget("data/artist_streams_%s.tsv" % self.date_interval)

    def requires(self):
        return [Streams(date) for date in self.date_interval]

    def run(self):
        artist_count = defaultdict(int)

        for input in self.input():
            with input.open('r') as in_file:
                for line in in_file:
                    timestamp, artist, track = line.strip().split()
                    artist_count[artist] += 1

        with self.output().open('w') as out_file:
            for artist, count in artist_count.iteritems():
                print >> out_file, artist, count

class Top10Artists(luigi.Task):
    date_interval = luigi.DateIntervalParameter()
    use_hadoop = luigi.BoolParameter()

    def requires(self):
        if self.use_hadoop:
            return AggregateArtistsHadoop(self.date_interval)
        else:
            return AggregateArtists(self.date_interval)

    def output(self):
        return luigi.LocalTarget("data/top_artists_%s.tsv" % self.date_interval)

    def run(self):
        top_10 = nlargest(10, self._input_iterator())
        with self.output().open('w') as out_file:
            for streams, artist in top_10:
                print >> out_file, self.date_interval.date_a, self.date_interval.date_b, artist, streams

    def _input_iterator(self):
        with self.input().open('r') as in_file:
            for line in in_file:
                artist, streams = line.strip().split()
                yield int(streams), int(artist)

class ArtistToplistToDatabase(luigi.contrib.postgres.CopyToTable):
    date_interval = luigi.DateIntervalParameter()
    use_hadoop = luigi.BoolParameter()

    host = "localhost"
    database = "toplists"
    user = "luigi"
    password = "abc123"  # ;)
    table = "top10"

    columns = [("date_from", "DATE"),
               ("date_to", "DATE"),
               ("artist", "TEXT"),
               ("streams", "INT")]

    def requires(self):
        return Top10Artists(self.date_interval, self.use_hadoop)
```

#### Execution
```shell
$ luigi --module top_artists AggregateArtists --local-scheduler --date-interval 2012-06
$ luigi --module examples.top_artists Top10Artists --local-scheduler --date-interval 2012-07

```
