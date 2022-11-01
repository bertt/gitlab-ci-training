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

Jobs are smallest units which can be executed by Gitlab CI. Here are samples of common job configurations.

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

- How can we trigger the 'diagnostics' job?

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

You can run job when there are changes in some files. That's great for monorepos.

```yaml
# .gitlab-ci.yml

Build A:
  script: echo build A ...

Build B:
  script: echo build B ...
  only:
    changes:
      - b/**
```

Question: How to trigger job 'Build B'?

- Full Reference - <https://docs.gitlab.com/ce/ci/yaml/index.html#onlychanges--exceptchanges>

### 3.4 Variables

Gitlab CI offers lots of usable variables like:

- `CI`
- `CI_PROJECT_NAME`, `CI_PROJECT_PATH_SLUG`
- `CI_COMMIT_REF_NAME`, `CI_COMMIT_REF_SLUG`
- `CI_COMMIT_SHA`, `CI_COMMIT_TAG`
- `CI_PIPELINE_ID`, `CI_JOB_ID`
- `CI_REGISTRY`, `CI_REGISTRY_USER`, `CI_REGISTRY_PASSWORD`

See all variables: <https://docs.gitlab.com/ce/ci/variables/predefined_variables.html#variables-reference>

You can define own variables in:

- Group CI Settings
- Project CI Setting
- Globally in CI YAML
- In job in CI YAML

You can define variable in **Settings -> CI / CD -> Variables**. Same for project and group. You can define for example connection to your Kubernetes cluster, Docker credentials, ...

Variables can be defined as:

- **Masked** - Value is hidden from the CI output. You probably dont want to show any credential, even development one.
- **Protected** - Protected variable appears only in jobs on protected branches. If developers can't push to protected branches, there have no chance to get production deployment keys or deploy to production. After code has been merged to master (protected), then protected variables appears and you can deploy to production.

Example job:

```yaml
# .gitlab-ci.yml

variables:
  XXX_GLOBAL: global

job1:
  script: 
      - echo global variable $XXX_GLOBAL
      - echo Project name $CI_PROJECT_NAME
```

Exercise: Add a variable using Settings -> CI / CD -> Variables and echo value in job

### 3.5 Artifacts

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

When the job succeeds, you can browse and download artifact from GitLab.

More about artifacts: <https://docs.gitlab.com/ce/user/project/pipelines/job_artifacts.html>

Exercise: Run pipeline and download artifact from CI/CD -> Pipelines -> Download artifacts. Inspect the content of zip file.

Question: How can we clean up artifacts after 1 day?

### 3.6 Test Reports

[Docs](https://docs.gitlab.com/ee/ci/unit_test_reports.html)

Create a file 'test_example.py' containing:

```python
# test_example.py
def test_ok():
    assert True

def test_err():
    assert False
```

Create a .gitlab-ci.yml containing:

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

Question: Pipeline will fail, how to fix the build?

Exrcise: Download artifact and inspect test report (xml file)

### 3.7 Docker

- Fully supported
- Easiest way how to create build environment
- Easiest way how to run and distribute your software

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
