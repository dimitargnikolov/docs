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
* While a bit cumbersome, **the catalogs streamline accessing data on remote hosts**. You declare where your inputs, outputs, and scratch locations are in the catalogs, and Pegasus takes care of any file transfer that need to occur when you deploy the workflow on an HPC system.

### Job Monitoring, Logging, and Summary
* A **Web app** allows you to monitor the running of the job or how the job ran. There is **a lot of detail**, perhaps too much. It is difficult to get a picture at a glance of what a specific workflow is doing. For example, a simple `split`, follower by a few `ls`s generates multiple jobs (jobs are the subunits of the workflow), not only for the actual commands, but also for cleaning up, creating directories, registering the workflow. To see what each job does you have to drill down in it, view its tasks (tasks are the subunits of a job), and then you can see the actual command represented by the job. This structure is **a little cumbersome**, but you do get a detailed, transparent view of it.
* There are a number of **command-line tools** that allow you to monitor progress, debug, and view several summaries of the workflow from the command line. These are more streamlined than the Web interface, while still offering enough detail.

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
```
$ snakemake --cores 16
```

#### Directory Structure
```
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

#### PBS
```
asdf
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