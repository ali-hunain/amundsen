# Amundsen on K8s
Steps for building, publishing, and deploying services onto K8s
​
## Building Docker Image
​
- Setup Docker locally on your machine.
- inside amundsen repo execute.
```
docker -f <dockerfile> -t <image-name> .
```
- There are three dockerfiles for `<dockerfile>`, one for each service:
    - Dockerfile.frontend
    - Dockerfile.metadata
    - Dockerfile.search
- give you created image a unique name by setting the `<image-name>` part.
​
## Publish image to ECR
- Establish Connection with Amazon QA CLI.
- Execute:
```
aws ecr get-login-password --region eu-west-1 --profile <your-connection-profile> | docker login --username AWS --password-stdin 848569320300.dkr.ecr.eu-west-1.amazonaws.com
```
- Tag the image you created:
```
docker tag <image-name>:latest 848569320300.dkr.ecr.eu-west-1.amazonaws.com/<ecr-repo-name>:<tag>
```
- There are three repos for `<ecr-repo-name>`, one for each service:
    - amundsen-frontend
    - amundsen-metadata
    - amundsen-search
​
- Push the tagged image to the ecr repo:
```
docker push 848569320300.dkr.ecr.eu-west-1.amazonaws.com/<ecr-repo-name>:<tag>
```
​
## Initialize the Deployment 
- Initiate  the Image deployment over Kubernetes:
```
curl -sX POST \
    -F token=<token> \
    -F "ref=master" \
    -F "variables[IMAGE_TAG]=<tag>" \
    -F "variables[ACTION]=install" \
    -F "variables[DEV]=false" \
    -F "variables[STAGING]=false" \
    -F "variables[PROD]=false" \
    "https://gitlab.sre.red/api/v4/projects/<project-id>/trigger/pipeline"
```
​
- Token and project id for the services:
    - Frontend:
        - Token: 2a95b4a293accf0fcb31e587d629f5
        - project-id: 793
    - Metadata:
        - Token: 9f123e26c3b2361a90164ad4638bce
        - project-id: 792
    - Search:
        - Token: 028190c6487e97373623c6b9e6ebe3
        - project-id: 789
​
​
#### Note: don't forget  to enable (only) the env you want to deploy on (ex: DEV) by changing it to true while leaving the others as false.
#### Note: use the same version you used to tag the image and pushed to ecr-repo.
#### Note: The pipeline can be found under the service repo, under CI/CD, also the direct link can be found in the curl command response.


