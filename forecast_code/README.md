# Forecast code

In the `run_forecast.R` script include the forecast workflow to generate a real-time forecast. The example model from the tutorial has be made into a function (`R/generate_example_forecast`) and is shown as an example. Once you have successfully modified the script to generate *your* forecast you can automate the submission steps (see below)!

Your model can either be a function (as has been shown here) or you can write the code directly in this script. 

# Workflow automation

## 1 Introduction

Workflow automation is key to making a forecast that is run every day, takes in new observations, updates parameters and initial conditions and produces a new forecast. And the automation means all of this is done without you needing to click a button every day.

## 2 The environment - Docker

To automate your forecast, the workflow needs to be fully reproducible. The environment/set up, packages, file paths need to be set up in a way that can be reproduced every day in the same way. As part of this reproducibility we will use a Docker container:

> A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. Containers isolate software from its environment and ensure that it works uniformly despite differences for instance between development and staging.

We will utilise a container from the `RockerProject` a Docker container that has R installed as well as some pre-installed packages. The VERA forecast Challenge has a container available which has the vera4castHelper package (plus tidyverse and other commonly used packages) already installed. We'll use the `rqthomas/vera-rocker:latest` container. 

## 3 The platform - Github Actions

There are a few ways that the running of a script can be automated but we will be using the Github Actions tools. Actions allow you to run a workflow based on a trigger, which in our case will be a time (but the trigger could also be something else, such as when you push or pull to a repository). Read more about [Github Actions](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions).

To start off with Github actions you need a workflow yaml file. Yaml files are computer readable 'instructions' that essentially say what the Action needs to do. We provide a ready-to-go yaml file in the `.github/workflows` directory.

Every time an action is triggered to start it will open a Linux machine environment and from this we can give it a series of instructions to get to run the script and submit our forecast. Below is an example of what your yaml file might look like to run an automated forecast.

A basic description of a Github action workflow:

> You can configure a GitHub Actions *workflow* to be triggered when an event occurs in your repository, such as a pull request being opened or a timer. Your workflow contains one or more *jobs* which can run in sequential order or in parallel. Each job will run inside its own virtual machine or container, and has one or more *steps* that either run a script that you define or run an action.

-   `on` tells you what triggers the workflow - here we use a `schedule` to determine the action based on a `cron` schedule (i.e. a timer) to run at 13 (UTC), everyday. You can update this to run on a different schedule based on timing codes found in <https://crontab.guru>.

-   `jobs` this is what you are telling the machine to do. You can see that within the job we have other instructions that tell the machine what `container` to use and the various `steps` in the job.

-   We use a container `image` that has the `vera4castHelpers` package plus others installed (`rqthomas/vera-rocker`).

-   The first step is to `Checkout repo` which uses a pre-made action `actions/checkout` to get a copy of the Github repo.

-   Next, within the container, we run R script `run_forecast.R` from the forecast_code directory - this is your forecast code that generates a forecast file and has code to submit the saved forecast to the VERA Challenge.

Note: the indentation matters, make sure the indentation is as formatted here!

Because this is a scheduled workflow this will run everyday, submitting your forecast to the Challenge. As long as your `run_forecast.R` has all the code to do this!

```         
on:
  workflow_dispatch:
  schedule:
  - cron: "0 13 * * *"

jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: rqthomas/vera-rocker:latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v2
        with:
          fetch-depth: 0

# Point to the right path, run the right Rscript command
      - name: Run automatic forecast submission
        run: Rscript forecast_code/run_forecast.R
```

## 4 Let's try and put this together

### 4.1 Writing your forecast code

Make sure that the `run_forecast` in the forecast_code directory contains all the code needed to generate your forecast and submit it to the Challenge (read targets, fit model, generate forecast, submit etc.).

Note: if you move this file or want to use a script that is stored elsewhere in the repository make sure you change the paths in the `submit_forecast.yaml`.

### 4.2 Enable Actions

In Github, go to your repository and navigate to the ACTIONS tab and make sure they are enabled. If you don't see an Actions tab (it should be between Pull Requests and Projects) go to Settings \> Actions \> General and check the Allow actions and reusable workflows.

### 4.3 Test the Action

-   Go back to Actions.
-   Click on the workflow in the left panel `.github/workflows/submit_forecast.yaml`
-   Test the workflow runs by using the Run workflow button \> Run workflow
-   Your job has been requested and will initiate soon
-   The progress of the job can be checked by clicking on it (Yellow running, Green completed, Red failed)
-   You can view the output of the job including what is produced in the console by clicking on the relevant sections of the job

You now have a fully automated forecast workflow!
