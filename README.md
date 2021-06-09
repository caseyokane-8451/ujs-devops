# ujs-devops


## Branch 2 - Deploy
For this section we're going to actually deploy our artifact so that our users (us in this case) can access use it.


### Process

#### Continuous Delivery (and Continuous Deployment)

Let's actually deploy our app somewhere. We have our artifact but it's useless unless we have a runtime environment to
host it. Heroku is an open-source alternative that we can utilize for this.

Let's get our app running somewhere. We'll use Heroku to get started fast. We have a couple options to get there

   
1. Go through the browser to create your app in Heroku (would suggest the CLI though for this exercise though)
    - [Create a new-app](https://dashboard.heroku.com/new-app) with your account
    - While creating the app, go ahead and add it to a new pipeline as the staging application (we will explain later)
    - Integrate your Heroku account with GitHub under "Deployment method"
        - Find your repo name and connect to it
    - Preform a **<ins>manual deploy</ins>** of the `2-deploy` branch
    
1. [Install the Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli#download-and-install) and verify it works
    - Find your API Key by navigating to `https://dashboard.heroku.com/account` -> Reveal API Key
    - Go ahead and create a GitHub Repository secret with our HEROKU_API_KEY while we have the value (docs [here](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository))
    - Export store your API key in your terminal window and list your apps
    ```
    export HEROKU_API_KEY=<your-api-key> 
    heroku apps
    ```
    - *If you see a Self-signed cert issue, please check out the Gotchas section of the README*

1. Navigate to our hello-world endpoint `https://<your-app-name>.herokuapp.com/hello-world`  and verify that it works

---

#### Build Once, Deploy Many

Now let's integrate this Heroku deployment as part of our GHA workflow.

Assuming we're using an artifact repository we want the build (CI) to happen as part of our workflow (from the last
branch). So after we build our artifact, lets try to deploy that.

Let's set up a new job for our deploys. In an ideal CI/CD workflow we'd have an artifact repository exist between our
deployments for [a variety of reasons](https://jfrog.com/knowledge-base/what-is-an-artifact-repository/). However, for
this quick POC we'll just use GitHub's ability to store our artifact

1. First, lets create a new Deploy Job that is dependent on the build job that we completed as part of the
previous branch. 
    - This is required since we need the artifact from the previous job (similar to what we'd do with an
    artifact repository).
    ```
      deploy:

        runs-on: ubuntu-latest
        needs: build
        
        steps:
    ```
1. Let's pull down our JAR that we published as part of the build job. 
    - This just involves to using the download-artifact (using the name of the artifact that we want to retrieve)
    action.
    ```
      - name: Retrieve Artifact
        uses: actions/download-artifact@v2
        with:
          name: JARtifact
    ```

1. Now set up Heroku configuration as part of the workflow
    - Let's start by installing the CLI and the corresponding Java plugin
    [as described here](https://devcenter.heroku.com/articles/deploying-executable-jar-files). This will require adding
    some more steps to our `deploy` job.
    ```
    - name: Install Heroku and java plugin
      run: |
        curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
        heroku plugins:install java
    ```

1. Let's deploy the jar with a GitHub action using this syntax from [the jar deploy](https://devcenter.heroku.com/articles/deploying-executable-jar-files#using-the-heroku-java-cli-plugin)
as well as include the HEROKU_API_KEY secret that we created earlier
    ```
    - name: Heroku deploy jar
      env:
        HEROKU_API_KEY: ${{ secrets.HEROKU_API_KEY }}
      run: heroku deploy:jar <jar-name>.jar --jdk 11 --app <app-name>
    ```
   
### Gotchas
- If you're seeing self-signed cert issues you probably need to set an additional variable [as discussed here](https://devcenter.heroku.com/articles/using-the-cli#using-an-http-proxy)
this will likely be found in your home directory in the `~/curl-ca-bundle.crt` file
- If you notice an issue with the buildpack failing initially, it's because you need to clear them first. You can see a 
more detailed explaination [here](https://stackoverflow.com/questions/53050441/error-your-buildpacks-do-not-contain-the-heroku-jvm-buildpackadd-heroku-jvm-to)
- Heroku only deploys code that you push to main/master. Pushing code to another branch of the heroku remote has no affect.
    - This shouldn't affect our case though, since we're simply using the JAR 
- Make sure you have a system.properties file present in your repo so that heroku is able to compile Java 11
[source](https://devcenter.heroku.com/changelog-items/1489)
- [Server port was required to deploy the jar](https://stackoverflow.com/questions/36751071/heroku-web-process-failed-to-bind-to-port-within-90-seconds-of-launch-tootall)


### References

- [Atlassian - Continuous Deployment vs. Continuous Delivery](https://www.atlassian.com/continuous-delivery/principles/continuous-integration-vs-delivery-vs-deployment)
- [Some comment on a Reddit thread - Build Once Deploy Many](https://www.reddit.com/r/devops/comments/d9ln04/build_once_deploy_many/f1iu60i?utm_source=share&utm_medium=web2x&context=3)
- [Deploying executable jars in Heroku](https://devcenter.heroku.com/articles/deploying-executable-jar-files)
- [Deploy to Heroku from GHA](https://dev.to/heroku/deploying-to-heroku-from-github-actions-29ej)
- [GitLab Example](https://lab.github.com/githubtraining/github-actions:-continuous-integration?overlay=register-box-overlay)
=======
## Introduction

**Howdy!** 🤠

This repository is meant to serve as a practical introduction into some of the basic concepts of DevOps and CI/CD using
an open-source (FREE) tech stack. We will try to cover broader strokes, however this will not be a
comprehensive view of everything that gets associated with the DevOps buzzword these days (that's a lot of stuff).

Regardless, hopefully this will be enough to pique your interest and point you in the direction of some good resources 
that you can look into as you see fit. 

## How it works 

We have 5 branches that contain different exercises that will cover different components of our stack:

- `0-code`: Writing our application that we want to deliver
- `1-build`: Building our desired artifact in a automated/repeatable manner 
- `2-deploy`: Deploying our code to a runtime environment so that others can access it
- `3-configure`: Configure our code so that we can deploy across multiple environments
- `4-monitor-maintain`: Monitor our code so that we can detect when issues arise and determine root cause, as well as how
to properly resolve the issue 
- `5-finale`: just a finalized branch that contains a base level solution using the lessons from the previous branches

The idea behind this exercise is that you: 
1. Start by forking this repository to a personal org
2. Checkout the branch `0-code` using `git checkout <branch-name>`  
3. Complete the exercises according to the instructions 
4. Move onto the next one (checking in changes if you'd like)


## Contributions/Feedback 

This is a new *implementation* of this exercise so there are bound to be some issues or areas of improvement. If you find
something unsavory/ineffective/cantankerous please feel free to create an issue [here](https://github.com/caseyokane-8451/ujs-devops/issues)
or reach out to me directly [here](mailto:casey.okane@8451.com).

**Dziękuję** (thanks)
