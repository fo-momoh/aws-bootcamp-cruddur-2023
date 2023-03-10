# Week 1 — App Containerization


## Containerize Application (Dockerfiles, Docker Compose)

During the live stream we containerized both the frontend and the backend of our application. We accomplished this by first creating the Dockerfiles for both the frontend and the backend, and then creating and running the docker-compose file. A lot of this was review, but I did learn a lot from our two guest speakers specifically surrounding some of the best practices of working with both Dockerfiles and docker-compose files in a work setting. 


## Post-Stream Tasks

I worked on tasks we weren't able to complete during live stream. These tasks included the following: 
    
    - Returning the container id into an Env Var
    - Sent curl to test server
    - Checked container logs
    - gained access to a container via docker exec
    - deleted an image.

##### Export Container ID as an Environment Variable
![Export Container ID as an Environment Variable](assets/week1/export-CONTAINERID-as-envVar.png)

##### Server output of a Curl test
![Curl test server output](assets/week1/curl-to-test-server-output.png)

##### Docker exec with Container ID Environment Variable
![Docker exec with Container ID Env Variable](assets/week1/docker%20exec%20with%20container%20id%20env%20var%20attempt.png)

##### After running the rmi command on the backend-flask image, I rebuilt the image as shown in the screenshot
![Deleted and rebuilt docker image](assets/week1/removed%20and%20re-added%20backend-flask%20image.png)


I wasn't able to override ports successfully, but I opted to proceed without too much troubleshooting. Exporting the container id to an env var made subsequent commands easier to execute; I could see how this would be beneficial in other use cases such as scripting. 



### Adding DynamoDB Local and Postgres


##### 
![Successful Postgres Install](assets/week1/successful%20postgres%20install.png)
![Successful Postgres Install pt2](assets/week1/successful%20postgres%20install%20pt2.png)

##### 
![Successful DynamoDB table creation](assets/week1/successful%20dynamodb%20attestation.png)

### Documented the Notification Endpoint for the OpenAI Document


![OpenAPI Documentation of Notifications Endpoint](assets/week1/openapi-documentation-for-notifications-endpoint.png)

### Write a Flask Backend Endpoint for Notifications and React Page for Notifications


![Functional Backend Implementation of Notifications Endpoint](assets/week1/backend%20attestation%20of%20notification%20endpoint.png)


![Frontend React Notifications Page is Functional](assets/week1/frontend%20implementation%20of%20notifications%20page.png)

## Homework Challenges

### Run the Dockerfile CMD as an external script

For this task, I opted to use bash for my scripting language and created 2 scripts to perform this. The first script, titled `start_backend_flask.sh`, calls on another script, `issue_startCMD.sh` and does some error handling if the container is already running. The scripts are listed below.

###### start_backend_flask.sh
```
#!/bin/bash
chmod +x issue_startCMD.sh
echo "starting backend-flask"

./issue_startCMD.sh

retVal=$?
if [ $retVal -ne 0 ]; then
    echo "ERROR, port is already in use!!"
fi
exit $retVal
```

###### issue_startCMD.sh
```
#!/bin/bash
python3 -m flask run --host=0.0.0.0 --port=4567
```
##### Script executed successfully
![External Script executing Dockerfile CMD](assets/week1/successful%20external%20CMD%20run.png)

### Push and tag a image to DockerHub

References:   
<https://www.geeksforgeeks.org/how-to-tag-an-image-and-push-that-image-to-dockerhub/>
<https://stackoverflow.com/questions/28349392/how-to-push-a-docker-image-to-a-private-repository>

I struggled with this a little bit because I had a misunderstanding of how to target my private image registry. I followed these two articles to complete this exercise. I ended up re-tagging my image several times prior to finally landing on the following: 
fomomoh08/aws-cloudproject-bootcamp:backend-flask. After completing the docker login command, I was able to push the image successfully to my repository. This will prove useful in future projects.

![DockerHub Image Uploaded]()

### Learn how to install Docker on your local machine and get the same containers running outside of Gitpod/Codespaces

References:
<https://docs.docker.com/desktop/install/windows-install/>
<https://github.com/docker/compose/issues/9956>

I used docker desktop and Almalinux.

I was able to build both containers and run them concurrently. 

Imges created
![](assets/week1/local%20docker%20images%20created.png)

Ran backend-flask container locally
![](assets/week1/local%20docker%20run%20backend.png)
![](assets/week1/local%20docker%20run%20backend%20browser%20attestation.png)


The docker-compose file failed to run however due to the reference of some env variables unique to gitpod. 

![Local docker-compose error](assets/week1/local%20docker-compose%20error.png)

I didn't wat to mess up my main project branch with experiments, so I created a new branch (docker-localtest). I replaced the environment variable references to reference http://localhost:300 and http://localhost:4567, commented out the OTEL env vars and attempted to run docker-compose up again. This time it failed due to the following error: docker endpoint for "default" not found. After some digging online, I came across a forum where other users ran into the same problem using Docker Desktop for Windows. It's a known bug that has a reliable workaround that required the deletion of a local config file. After applying the workaround and creating a docker-compose-local-test.yml, the docker-compose ran successfully and I was able to hit Cruddur at http://localhost:3000 and navigate around the page.

![](assets/week1/successful-local-docker-compose-up.png)

### Implement a health check in the V3 Docker compose file

References:

<https://stackoverflow.com/questions/38895558/docker-healthcheck-in-composer-file>
<https://docs.docker.com/compose/compose-file/#healthcheck>

I tried adding a health check to my backend referencing the env vars of my gitpod environment and appending the URL with "/api/activities/home". I used the code below to attempt:

```
    healthcheck:
      test: ["CMD", "curl", "-f", "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}/api/activities/home"]
      interval: 30s
      timeout: 10s
      retries: 3
```

I went on to run the CMD manually to ensure it'll pull the result I was expecting. It returned the value data indicating a successful run however this was not enough to validate that the health check was operational. I first checked the logs of my backend-flask container and did not see any activity beyond my manual tests. I then opened up honeycomb.io to see if there was any evidence available in the traces (honeycomb was instrumented already because I added health checks after week2). 


Updated docker-compose and tested the CMD.
![](assets/week1/updated-docker-compose-with-healthcheck-and-tested-CMD.png)

![](assets/week1/container-healthcheck-status.png)
![](assets/week1/container-unhealthy-healthcheck-status.png)

I found a trace that wasn't created from my manual tests; I concluded that it was my health check that created this trace due to the 404 status code and the timing between successful manual tests. 

![](assets/week1/health-check-honeycomb-trace-comparison.png)
![Honeycomb health check trace](assets/week1/backend-flask-honeycomb-healthcheck-trace.png)

I found 2 other ways to validate the health check's functionality: 1) running a 'docker ps' command and 2) Looking at the status icon of the container for the docker widget in VS Code. This helped me confirm that my backend-flask container was checking in as unhealthy. For this reason I opted to add the health check to the frontend-react-js.

Once implemented, the frontend container checked in as healthy. 

![](assets/week1/container-healthy-frontend-healthcheck-status.png)

### Research best practices of Dockerfiles and attempt to implement it in your Dockerfile

References:

<https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>

The best practices article was very extensive. I read this before attempting to create a multi-stage building Dockerfile, so that I could incorporate a few of the techniques. I adopted a few into this task including the following:
    
    - Using multi-stage builds
    - Minimized layers using only the RUN, COPY and ADD commands that were required
    - Used the recommended Alpine distro
    - Added a label to initialize the organization of my image(s)
    - Combined apt-getupdate and apt-get install in the same line
        - RUN apt-get update && apt-get install -y \



