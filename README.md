![CI](https://github.com/elliottmurray/devops-test/workflows/CI/badge.svg)
![Deploy to GKE](https://github.com/elliottmurray/devops-test/workflows/Deploy%20to%20GKE/badge.svg)


# Assumptions and decisions I made
I tried to do this with Kubernetes and Github Actions. As I know very little about either I somewhat regretted this decision. However, in about 3-4 hours I have managed to get a basic CI and a CD pipeline working deploying to GKE. Instructions for GKE setup that I followed are attached although this was not thoroughly tested. I believe that by default kube-proxy follows a [round robin strategy](https://blog.getambassador.io/load-balancing-strategies-in-kubernetes-l4-round-robin-l7-round-robin-ring-hash-and-more-6a5b81595d6c) but I had to pick up a lot about ingress and load balancers in Kubernetes so I may be mistaken. I can see that the output from the node app changes but I'm not convinced it's completely round robin (though health checks may be affecting that conclusion)

Github actions are very lightweight and has a great market place however I found it hard to do more complex workflows in the time I had available. I turned off push to master and ensured there had to be a PR with review (admin can overwrite this) and added semantic PR and commits. Everything apart from master just runs a build job that runs tests. I had to duplicate this in another workflow but added the deploy step. This only runs on master (that you can only push to via a Merged PR) I am sure there is a more elegant way of doing this with GH actions but it was getting fiddly. The deploy creates a docker image that it pushes to gcr with a tag based on a git sha. That is used by Kustomize to do the deploy. I would probably add more sophisticated tagging as an improvement.

# Setup
I have assumed docker is installed. I ran it with node 12 locally. I managed this with direnv/nvm but would probably use .node_version ongoing. I have used github actions and you will need to set up some secrets/config so will have to re-fork this repo to get a deployment working. The secrets you will need to setup:

GKE_PROJECT
GKE_SA_KEY # a service account key from IAM & Admin -> Service Accounts


## Running this web application
 This is a NodeJS application:	This is a NodeJS application:

- `npm test` runs the application tests	- `npm test` runs the application tests
- `npm start` starts the http server


## Run in docker
Run the following to build and run locally:

```bash
docker build -t buildit .
docker run -p 3000:3000 -it buildit
```


## GKE setup
Assumed you have an account. Create a project called em-buildit and enable Kubernetes api's. Then run

```bash
gcloud config set project em-buildit # These are hardcoded in my deploy scripts so you will need to use these values
gcloud config set compute/zone europe-north1-a
gcloud container clusters create buildit-cluster

```

After you fork, push and merge a PR you should be able to run
```
curl `kubectl describe ing my-ingress | grep 'Address:' | awk '{ print $2 }'`
``` to see the output




