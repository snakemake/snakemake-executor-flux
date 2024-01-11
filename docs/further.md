# Snakemake Executor Flux

This is an example implementation for an external snakemake plugin.
Since we already have one for Flux (and it can run in a container) the example
is for Flux. You can use this repository as a basis to design your own executor to work
with snakemake!

## Usage

### Tutorial

For this tutorial you will need Docker installer.

[Flux-framework](https://flux-framework.org/) is a flexible resource scheduler that can work on both high performance computing systems and cloud (e.g., Kubernetes).
Since it is more modern (e.g., has an official Python API) we define it under a cloud resource. For this example, we will show you how to set up a "single node" local
Flux container to interact with snakemake using the plugin here. You can use the [Dockerfile](examples/Dockerfile) that will provide a container with Flux and snakemake
Note that we install from source and bind to `/home/fluxuser/snakemake` with the intention of being able to develop (if desired).

First, build the container:

```bash
$ docker build -f example/Dockerfile -t flux-snake .
```

We will add the plugin here to `/home/fluxuser/plugin`, install it, and shell in as the fluxuser to optimally interact with flux.
After the container builds, shell in:

```bash
$ docker run -it flux-snake bash
```

And start a flux instance:

```bash
$ flux start --test-size=4
```

Go into the examples directory (where the Snakefile is) and run snakemake, targeting your executor plugin.

```bash
$ cd ./example

# This says "use the custom executor module named snakemake_executor_plugin_flux"
$ snakemake --jobs 1 --executor flux
```
```console
Building DAG of jobs...
Using shell: /bin/bash
Job stats:
job                         count    min threads    max threads
------------------------  -------  -------------  -------------
all                             1              1              1
multilingual_hello_world        2              1              1
total                           3              1              1

Select jobs to execute...

[Fri Jun 16 19:24:22 2023]
rule multilingual_hello_world:
    output: hola/world.txt
    jobid: 2
    reason: Missing output files: hola/world.txt
    wildcards: greeting=hola
    resources: tmpdir=/tmp

Job 2 has been submitted with flux jobid ƒcjn4t3R (log: .snakemake/flux_logs/multilingual_hello_world/greeting_hola.log).
[Fri Jun 16 19:24:32 2023]
Finished job 2.
1 of 3 steps (33%) done
Select jobs to execute...

[Fri Jun 16 19:24:32 2023]
rule multilingual_hello_world:
    output: hello/world.txt
    jobid: 1
    reason: Missing output files: hello/world.txt
    wildcards: greeting=hello
    resources: tmpdir=/tmp

Job 1 has been submitted with flux jobid ƒhAPLa79 (log: .snakemake/flux_logs/multilingual_hello_world/greeting_hello.log).
[Fri Jun 16 19:24:42 2023]
Finished job 1.
2 of 3 steps (67%) done
Select jobs to execute...

[Fri Jun 16 19:24:42 2023]
localrule all:
    input: hello/world.txt, hola/world.txt
    jobid: 0
    reason: Input files updated by another job: hello/world.txt, hola/world.txt
    resources: tmpdir=/tmp

[Fri Jun 16 19:24:42 2023]
Finished job 0.
3 of 3 steps (100%) done
Complete log: .snakemake/log/2023-06-16T192422.186675.snakemake.log
```

And that's it! Continue reading to learn more about plugin design, and how you can also design your own executor
plugin for use or development (that doesn't need to be added to upstream snakemake).

## Developer

To do the same run but bind the local plugin directory:

```bash
docker run -it -v $PWD/:/home/fluxuser/plugin flux-snake bash
```

The instructions for creating and scaffolding this plugin are [here](https://github.com/snakemake/poetry-snakemake-plugin#scaffolding-an-executor-plugin).
Instructions for writing your plugin with examples are provided via the [snakemake-executor-plugin-interface](https://github.com/snakemake/snakemake-executor-plugin-interface).
