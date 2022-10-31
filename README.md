# Gitlab CI Training

## Agenda

- 1. [What is CI](#what-is-ci)
- 2. [Getting started](#wgetting-started)
- 3. [CI Jobs](#ci-jobs)
- 3. [from CI](#deployments-from-ci)
- [Example Project Pipeline](#example-project-pipeline)

## 1. What is CI

### 1.1 Git

Git is a version control system for tracking changes in computer files and coordinating work on those files among multiple people.

### 1.2 Gitlab

From project planning and source code management to CI/CD and monitoring, GitLab is a complete DevOps platform, delivered as a single application.

### 1.3 CI

In software engineering, continuous integration is the practice of merging all developer working copies to a shared mainline several times a day.

### 1.4 Usage of CI

Automatization of:

- build process
- testing - unit tests, integration tests, linting, formating
- deployment - dev, staging, production

### 1.5 Gitlab CI Features

- Integration into Gitlab (CE & EE)
- Versioned (YAML file in repository)
- Easy to scale
- Docker & Kubernetes support

### 1.6 Gitlab CI Architecture

![gitlab-ci-architecture](./gitlab-ci-architecture.png)

### 1.7 Gitlab Runner

Runs on every platform - Linux, Mac, Windows, Docker, Kubernetes

How to install & configure:

- https://docs.gitlab.com/runner/install/
- https://docs.gitlab.com/runner/register/

## 2 Getting started

Go to https://www.gitlab.com, create account and create a new project. Name for example: 'gitlabcitraining'

Go to Settings -> CI/CD -> Runners.

Question: How many 'shared' runners are available?

## 3. CI Jobs

### 3.1 Gitlab CI YAML

Configuration of everything aroud Gitlab CI is stored inside Git repository in file `.gitlab-ci.yml`

If you don't know YAML format, check out this simple YAML tutorial - <https://learnxinyminutes.com/docs/yaml/>

Here is a Gitlab CI YAML reference - <https://docs.gitlab.com/ce/ci/yaml/README.html>

### 3.2 First Job

Create file `.gitlab-ci.yml` with following content:

```yaml
# .gitlab-ci.yml

job:
  script: echo Hello World!
```

Push to Gitlab and go to CI/CD -> Pipelines. A first pipeline should be created, check the options and console messages.

Question: Which Docker image is used in the pipeline?

### 3.3 Job

Jobs are smalles units which can be executed by Gitlab CI. Here are samples of common job configurations.

Jobs are top level object in Gitlab CI YAML files instead of [few keywords](https://docs.gitlab.com/ce/ci/yaml/README.html#unavailable-names-for-jobs). Keywords are: `image`, `services`, `stages`, `types`, `before_script`, `after_script`, `variables`, `cache`

#### 3.3.1 Script

Every job require `script` - it's a shell script which will be executed by job. Script can be string or list of strings.

```yaml
# .gitlab-ci.yml

job1:
  script: echo Hello World!
job2:
  script:
    - echo Hello World!
    - echo Ahoj Svete!
```

Question: How can we change the yaml so job2 is executed before job1?

#### 3.3.2 Stages

You can define order of jobs by stages. You can define stages and their order. Jobs in same stage run in parallel and after CI finishes all job in stage, then start jobs from next stage.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test

build:
  stage: build
  script: echo Build!

test1:
  stage: test
  script: echo Test1!

test2:
  stage: test
  script: echo Test2!
```

#### 3.3.3 Before & After Script

You can define script which will be executed before and after job script. You can define those script globally or per job.

```yaml
# .gitlab-ci.yml

before_script:
  - echo Global before

after_script:
  - echo Global after

job1:
  script: echo Job1!

job2:
  script: echo Job2! && false

job3:
  before_script:
    - echo Local before
  after_script: []
  script: echo Job3!
```

Question: The pipeline fails, how to fix it? What is the output of Job1, Job2, Job 3?

#### 3.3.4 When

You can control when do you want run your jobs. By default, jobs are executed automatically when previous stage succeed. You can specify another condition, you can run jobs manually, always or on error.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test

build:
  stage: build
  script: echo Run build ...
  # script: echo Run build ... && false

test:
  stage: test
  script: echo Run test ...

deploy:
  stage: test
  script: echo Run deploy ...
  when: manual

diagnostics:
  stage: test
  script: echo Run diagnostics ...
  when: on_failure

reporting:
  stage: test
  script: echo Run CI reporting ...
  when: always
```

Questions: 

- How can we run job 'deploy'?

- How can trigger the 'diagnostics' job?

#### 3.3.5 Allow Failure

You can specify flag `allow_failure` to `true`, job can fail but pipeline will succeed.

```yaml
# .gitlab-ci.yml

test:
  script: echo test ... && false
  allow_failure: true
```

Manual jobs are allowed to fail by default, if you want to disallow failure, you have to set `allow_failure` to `false`.

```yaml
# .gitlab-ci.yml

test:
  when: manual
  script: echo test ... && false

test2:
  when: manual
  script: echo test ... && false
  allow_failure: false
```

#### 3.3.6 Only & Except

You can specify another condition when you can run jobs. You can specify branches and tags on which you want to run your jobs or not.

```yaml
# .gitlab-ci.yml

stages:
  - unit_test
  - integration_test
  - build
  - deploy

unit_test:
  stage: unit_test
  script: echo Unit Test ...
  only:
    - branches
    - tags
    - merge_requests

integration_test:
  stage: integration_test
  script: echo Integration Test ...
  only:
    - merge_requests

build:
  stage: build
  script: echo Build ...
  only:
    - main
    - tags

deploy:
  stage: deploy
  script: echo Deploy ...
  only:
    - tags
  when: manual
  allow_failure: false
```

Full reference here - <https://docs.gitlab.com/ce/ci/yaml/index.html#only--except>

Question: How can we run the 'integration_test' job?

#### 3.3.7 Only Changes

You can run job when are changes is some files. That's great for monorepos.

```yaml
# .gitlab-ci.yml

Build A:
  script: echo Cuild A ...
  only:
    changes:
      - a/**

Build B:
  script: echo Cuild B ...
  only:
    changes:
      - b/**
```

- Example monorepo with only changes - <https://github.com/ondrejsika/ondrejsikawebs>
- Full Reference - <https://docs.gitlab.com/ce/ci/yaml/index.html#onlychanges--exceptchanges>

### 3.4 Merge Request Pipelines

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script: echo build
  only:
    - branches
    - merge_request

test:
  stage: test
  script: echo test
  only:
    - merge_request

deploy:
  stage: deploy
  script: echo deploy
  only:
    - master
```

### 3.5 Variables

Gitlab CI offers you lots of usable variables like:

- `CI`
- `CI_PROJECT_NAME`, `CI_PROJECT_PATH_SLUG`
- `CI_COMMIT_REF_NAME`, `CI_COMMIT_REF_SLUG`
- `CI_COMMIT_SHA`, `CI_COMMIT_TAG`
- `CI_PIPELINE_ID`, `CI_JOB_ID`
- `CI_REGISTRY`, `CI_REGISTRY_USER`, `CI_REGISTRY_PASSWORD`

See all varibles: <https://docs.gitlab.com/ce/ci/variables/predefined_variables.html#variables-reference>

You can define own variables in:

- Group CI Settings
- Project CI Setting
- Globally in CI YAML
- In job in CI YAML

You can define varible in **Settings -> CI / CD -> Variables**. Same for project and group. You can define for example connection to your Kubernetes cluster, AWS credentials, ...

Variables can be defined as:

- **Masked** - Value is hidden from the CI output. You probably dont want to show any credential, even development one.
- **Protected** - Protected variable appears only in jobs on protected branches. If developers can't push to protected branches, there have no chance to get production deployment keys or deploy to production. After code has been merged to master (protected), then protected variables appears and you can deploy to production.

Example job:

```yaml
# .gitlab-ci.yml

variables:
  XXX_GLOBAL: global

job1:
  script: env | grep XXX

job2:
  variables:
    XXX_LOCAL: local
  script: env | grep XXX
```

You can also create variables from variables, but you can't use varible defined in same place. If you create global variabl, you can use only CI default variables and custom variables setted up in project or group CI settings.

```yaml
# .gitlab-ci.yml

variables:
  IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
```

### 3.6 Cache

Cache is used to specify a list of files and directories which should be cached between jobs. You can only use paths that are within the project workspace.

If cache is defined outside the scope of jobs, it means it is set globally and all jobs will use that definition.

You can specify cache key for better caching of different branches

```yaml
# .gitlab-ci.yml

job:
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - now
  script:
    - date >> now
    - cat now
```

More about caches: <https://docs.gitlab.com/ce/ci/caching/index.html>

#### Git Clean Flags

Instead of cache, you can use faster option how to keep some data in repository. If you are using cache, your repository is cleaned by `git clean -ffdx`, which means remove all ignored & untracked files and directories. Then your cache is downloaded and extracted into your working directory. This way is not fastest way if you have thousands files in cache (like node modules, for example).

Instead of cache, you can specify variable `GIT_CLEAN_FLAGS` and use own git clean arguments. If you want to ignore node modules from git clean, you can set `-ffdx -e .yarn-cache`.

```yaml
# .gitlab-ci.yml

variables:
  GIT_CLEAN_FLAGS: -ffdx -e .yarn-cache

job:
  script:
    - yarn install --frozen-lockfile --cache-folder .yarn-cache --prefer-offline
    - yarn build
```

### 3.7 Artifacts

Artifacts is used to specify a list of files and directories which should be attached to the job when it succeeds, fails, or always.

The artifacts will be sent to GitLab after the job finishes and will be available for download in the GitLab UI.

Artifact are also distributed to jobs in next stages.

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test

build:
  stage: build
  script:
    - mkdir -p out
    - echo '<h1>Hello World!</h1>' > out/index.html
  artifacts:
    paths:
      - out

test:
  stage: test
  script:
    - cat out/index.html
```

When the job succed, you can browse and download job from Gitlab.

More about artifacts: <https://docs.gitlab.com/ce/user/project/pipelines/job_artifacts.html>

#### Artifacts Dependencies

By default, all artifacts will be passed to jobs in following stages. If you want artifact only from specific jobs, you can use dependencies to choose which articact you want.

Fists usage is when artifact are on same path and there will be colision, see:

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test

build_A:
  stage: build
  script: mkdir -p out && echo '<h1>Hello from Project A!</h1>' > out/index.html
  artifacts:
    paths:
      - out

build_B:
  stage: build
  script: mkdir -p out && echo '<h1>Hello from Project B!</h1>' > out/index.html
  artifacts:
    paths:
      - out

test A:
  stage: test
  script: cat out/index.html
  dependencies:
    - build_A

test B:
  stage: test
  script: cat out/index.html
  dependencies:
    - build_B
```

Or you can use dependencies when you have lots of artifact and dont want to slow down your jobs by downloading unnecessary artifacts.

### 3.8 JUnit Test Reports

[Docs](https://docs.gitlab.com/ee/ci/unit_test_reports.html)

```python
# test_example.py
def test_ok():
    assert True

def test_err():
    assert False
```

```yml
# .gitlab-ci.yml
test:
  image: ondrejsika/pytest
  script:
    - pytest --junitxml=report.xml
  artifacts:
    reports:
      junit: report.xml
```

### 3.9 Docker

- Fully supported
- Easiest way how to create build environment
- Easiest way how to run and distribute your software

If you have a Docker or Kubernetes executor ([we have](https://github.com/ondrejsika/ondrejsika-gitlab-runner/blob/master/register-runner.sh#L12)) you can define image where you want to run your job.

You can specify image globally or in job:

```yaml
# .gitlab-ci.yml

image: ondrejsika/ci

build:
  image: node
  script:
    - yarn
    - yarn run build

deploy:
  script:
    - kubectl apply -f kubernetes.yml
```

You can also run docker commands from job because we have added Docker socket there. See [here](https://github.com/ondrejsika/ondrejsika-gitlab-runner/blob/master/create-runner.sh#L6)
and [here](https://github.com/ondrejsika/ondrejsika-gitlab-runner/blob/master/register-runner.sh#L14).

```yaml
# .gitlab-ci.yml

job:
  script:
    - docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD
    - docker build -t $CI_REGISTRY_IMAGE .
    - docker push $CI_REGISTRY_IMAGE
```

### 3.10 Docker Image `ondrejsika/ci`

That is my image which I use for most of CI jobs.

It contains lots of common tools like git, zip, curl, wget, Docker client, Docker Compose, Kubernetes client, ...

You can see the repository on Github - <https://github.com/ondrejsika/ondrejsika-ci-docker>

#### Pass Environment Variables to Docker

If you want to run Docker container and pass there all environment variables, you can use this:

```yaml
# .gitlab-ci.yml

job:
  script:
    - env > .env
    - docker run --rm --env-file .env debian env
```

### Environments

Environment is used to define that a job deploys to a specific environment.

If environment is specified and no environment under that name exists, a new one will be created automatically.

```yaml
# .gitlab-ci.yml

deploy:
  script: echo 'Deploy!'
  environment:
    name: $CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.dev.company.com
```

#### Stop Environmet

You can define stop job for the environment which you can stop manually from Gitlab. If stop job is defined, Gitlab automatically stop unnecessary environments, like envirments created from merged or deleted branches.

```yaml
# .gitlab-ci.yml

deploy:
  script: echo Deploy!
  environment:
    name: $CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.dev.company.com
    on_stop: stop

stop:
  script: echo Stop!
  when: manual
  environment:
    name: $CI_COMMIT_REF_SLUG
    action: stop
```

### Includes

[Docs](https://docs.gitlab.com/ee/ci/yaml/#include)

```yaml
# .gitlab-ci.yml

stages:
  - build
  - deploy

include:
  - .gitlab-ci-build.yml
  - .gitlab-ci-deploy.yml
```

```yaml
# .gitlab-ci-build.yml

build:
  stage: build
  script: echo build
```

```yaml
# .gitlab-ci-deploy.yml

deploy:
  stage: deploy
  script: echo deploy
```

### Child Pipelines

[Docs](https://docs.gitlab.com/ce/ci/parent_child_pipelines.html)

```yaml
# .gitlab-ci.yml
pipeline_a:
  trigger:
    include: a/.gitlab-ci.yml
    strategy: depend
  only:
    changes:
      - a/**

pipeline_b:
  trigger:
    include: b/.gitlab-ci.yml
    strategy: depend
  only:
    changes:
      - b/**
```

```yaml
# a/.gitlab-ci.yml

stages:
  - build
  - deploy

build:
  stage: build
  script: echo Build service A

deploy:
  stage: deploy
  script: echo Deploy service A
```

```yaml
# b/.gitlab-ci.yml

stages:
  - build
  - deploy

build:
  stage: build
  script: echo Build service B

deploy:
  stage: deploy
  script: echo Deploy service B
```

### Generated Pipelines

```yaml
# .gitlab-ci.yml
stages:
  - generate
  - pipeline

generate:
  stage: generate
  image: python:3.7-slim
  script: python generate-gitlab-ci.py
  artifacts:
    paths:
      - .gitlab-ci.generated.yml

pipeline:
  stage: pipeline
  trigger:
    include:
      - artifact: .gitlab-ci.generated.yml
        job: generate
    strategy: depend
```

```python
# generate-gitlab-ci.py
import json

SERVICES = (
    "foo",
    "bar",
    "baz",
)

def make_service(name):
    return {
        "build-%s" % name: {
            "stage": "build",
            "script":[
                "echo Build %s" % name,
            ]
        },
        "deploy-%s" % name: {
            "stage": "deploy",
            "script":[
                "echo Deploy %s" % name,
            ]
        }
    }

with open(".gitlab-ci.generated.yml", "w") as f:
    pipeline = {}
    pipeline.update({
        "stages": ["build", "deploy"]
    })
    for service in SERVICES:
        pipeline.update(make_service(service))
    f.write(json.dumps(pipeline))
```

### Multi Project Pipelines

[Docs](https://docs.gitlab.com/ce/ci/multi_project_pipelines.html)

Project **foo** depends on library **bar**. If you make any change in library **bar** you have to trigger pipeline in project **foo**. `ondrejsika/foo` is full path to project where you want to run pipeline.

```yaml
# .gitlab-ci.yml (library bar)
job:
  script: Do something

triger-pipelines:
  trigger: ondrejsika/foo
```

### Needs - Directed Acyclic Graph

- [Docs (DAG)](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/index.html)
- [Docs (Needs)](https://docs.gitlab.com/ee/ci/yaml/#needs)

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

linux build:
  stage: build
  script: sleep 10 && echo Done

mac build:
  stage: build
  script: sleep 20 && echo Done

lint:
  stage: test
  needs: []
  script: echo Done

linux unit tests:
  stage: test
  needs:
    - linux build
  script: echo Done

linux e2e tests:
  stage: test
  needs:
    - linux build
  script: sleep 10 && echo Done

mac unit tests:
  stage: test
  needs:
    - mac build
  script: sleep 5 && echo Done

mac e2e tests:
  stage: test
  needs:
    - mac build
  script: sleep 30 && echo Done

release linux:
  stage: deploy
  script: "echo Done"
  needs:
    - linux unit tests
    - linux e2e tests

release mac:
  stage: deploy
  script: "echo Done"
  needs:
    - mac unit tests
    - mac e2e tests
```

