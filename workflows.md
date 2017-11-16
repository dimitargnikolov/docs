# Workflow Management Tools

## Pegasus
### Summary
asdf

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

Pegasus has direct support for execution on a batch HPC system powered by PBS/Torque, thourgh the Site Catalog.

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
asdf

* Web: [Docs](http://snakemake.readthedocs.io/en/stable/), [BitBucket](https://bitbucket.org/snakemake/snakemake)
* License: [MIT License](https://tldrlegal.com/license/mit-license)

### Advantages
* asdf 1
* asdf 2

### Disadvantages
* asdf 1
* asdf 2

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

## Airflow
### Summary
asdf

* Web: [Docs](https://airflow.apache.org/), [Github](https://github.com/apache/incubator-airflow)
* License: [Apache 2](https://tldrlegal.com/license/apache-license-2.0-(apache-2.0))

### Advantages
* 
* asdf 2

### Disadvantages
* asdf 1
* asdf 2

### Example Workflow
#### Definition
```
asdf
```

#### Execution
```
asdf
```

#### Directory Structure
```
asdf
```

#### PBS
```
asdf
```

## luigi
### Summary
asdf

* Web: [Docs](http://luigi.readthedocs.io/en/stable/workflows.html), [Github](https://github.com/spotify/luigi)
* License: [Apache 2](https://tldrlegal.com/license/apache-license-2.0-(apache-2.0))

### Advantages
* asdf 1
* asdf 2

### Disadvantages
* asdf 1
* asdf 2

### Example Workflow
#### Definition
```
asdf
```

#### Execution
```
asdf
```

#### Directory Structure
```
asdf
```

#### PBS
```
asdf
```

## Conclusion
asdf