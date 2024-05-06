## Benchmark authoring example

This repository contains an example of how to package a benchmark.

### Structure

Each folder in this repository corresponds to a single benchmark. Each benchmark should represent a single challenge: they
may contain multiple vulnerabilities and multiple steps to solve but there should only be one objective.

For the moment we are restricting this to Jeopardy-style Capture The Flag (CTF) challenges. A flag is hidden somewhere in the 
benchmark and the objective is to find it.  We do not hardcode the flag, instead we expect to be able to inject it when we build
the benchmark.

The benchmark should contain the following files:

#### `benchmark/docker-compose.yml`

This file should specify the build and running of the benchmark. We will build the benchmark by running: 
`docker compose build --build-arg flag=<some random flag>`.

 * Each service in the benchmark should either be a commonly recognised base-image (e.g. nginx) or should
be built from a Dockerfile in an appropriate sub-directory. 
 * To provide ports which are accessible to another service use the `export` directive.
 * To provide ports which are accessible to the solver use the `ports` directive.
 * Use `healthcheck` and `depends_on` to ensure that services are started without race conditions.

 When we start the benchmark we will run `docker compose up -d --wait`. The compose file should ensure that
 the benchmark is ready and running as soon as this command completes - this might require further `healthcheck` directives. 
 This requirement is important for the reliability of our test runs: we need to be sure that the benchmark is ready before
 we start the solver.

#### further sub-directories

For each service which is built in the `docker-compose.yml` file there should be a sub-directory containing the Dockerfile and any other
files required to build the service. 

 * The Dockerfile should be named `Dockerfile`. 
 * Use `ARG flag` as a directive to specify the build-arg for the flag variable and ensure that this is injected into the image correctly. 
 * Place the `ARG flag` as late in the Dockerfile as possible to minimise rebuilding time as we change flags.

 Writing a good Dockerfile is something of an art. As long as it works we are happy!

#### `benchmark/solution.sh`

An executable shell script which can recover the flag from a running challenge. 

 * The script should take the `<host>:<port>` of published services as arguments
 * It should output the flag (and no other output) to stdout
 * It may call other scripts and files in the benchmark directory if required

#### `benchmark/walkthrough.md`

A markdown file explaining how to solve the challenge which an inexperienced user might follow. This should be a step-by-step guide.

#### `benchmark/benchmark-config.json`

A metadata file in JSON format containing a dictionary with the following keys:

 * `name`: the name of the benchmark
 * `original_reference`: a URL to the original reference for the benchmark. This could be sources in a GitHub repository, a blog post, a CTF write-up, etc.
 * `original_author`: the author of the original reference if its included in the original reference
 * `import_kind`: one of `source`, `text`
     * `source`: the original reference contains container sources and some existing packaging (e.g. Dockerfile)
     * `text`: the original reference contains a textual description of the challenge but there are no usable sources
 * `description`: a short description as would be given to someone in a competition who is solving the challenge
 * `level`: `1` (easy), `2` (medium) or `3` (hard). The original difficulty choosed by the author or an estimation of the difficulty that this challenge may represent.
 * `build_args`: (optional) a dictionary of build arguments to pass to the `docker-compose build` command.

### Validation

*Please note that there is no need to modify this file*

A Python 3 script `validate.py` is provided in the root of this repository. This script will check that the benchmark is correctly packaged. 
Run this script with the root of this repository as the current working directory. 

```
python3 ./validate.py [optional regexp to match benchmark path]
```

### How to write new exercises

First, please ensure that `validate.py` succeeds on your local setup.  Please contact us if you have any issues with this.

Add new exercises as folders in this repository with the layout as described above. Refer to the examples in this repository for specific
details of how to write the files. Please remove the examples once you are happy with your new exercises.

The `pygoat` example shows a more complicated case where the benchmark has been written as a single web application with multiple vulnerabilities. 
What we need to do in this case is slice out each individual vulnerability and package it as a separate benchmark. This is done by writing
a single Dockerfile and then passing build-args to it to slice the specific challenge we need. This means that the single `pygoat` folder actually
contains multiple benchmarks.  Just put these in separate folders: as long as the folder name has `benchmark` as the prefix everything will work.
There will be a `benchmark-config.json` file for each of these "sub" benchmarks and you can use the `build-args` field to choose the parameters
for the specific challenge.
