# water
Introduction
Have you ever wanted a simple interface to manage your files on the cloud? Have you ever wanted to play with several cloud technologies? If you have answered YES to both then lets talk about water. water is a tool that leverages various cloud technologies on Google Cloud Platform to provide a simple, yet powerful experience.
Features:

API Driven - water provides simple REST APIs through which files can be managed.
Secure - We believe that HTTPS is the way to go and all of water services are secured with HTTPS.
No account creation required - Do you have a Google account? If yes, you can start using it today.
On Cloud - The entire application is deployed on Google Cloud. Google has years of exerience in running applications at scale and it can help water scale as well.
Self Healing - water uses the power of Kubernetes to be self managed. Kubernetes can bring up workloads that have died and ensures that the workloads are always alive with the minimum number of replicas.
High performant - water is built to scale. It uses concepts of CDN and in-memory caches to offer a very low latency user experience. It can be scaled automatically when traffic increase/decreases.
Responsive - The user experience is built on angular.js and provides a responsive experience out of the box. Hence water is also mobile friendly
Technologies Used:

Google Cloud Platform - GCP provides the entire infrastructure for powering water
Google Container Engine - Google's implementation of Kubernetes on which water is deployed.
Google AppEngine Flex - The newer version of AppEngine built on containers (we use this only to build our containers to work well with GCP offerings).
Google Container Registry - GCR is used to store all our built docker containers.
Google Cloud Storage - Cloud Storage is used to store all of the users files
Datastore - Datastore is used to store metadata associated with users and their files.
Pub/Sub - PubSub is used to write incoming files to a message bus which can be consumed.
Functions - Serverless technology used to read off of the message bus and write to Cloud Storage.
VPC Network - VPC network is used to allocate our global static IP addresses.
StackDriver - Stackdriver provides all our logging and monitoring capabilities.
Cloud DNS - Cloud DNS manages provides the nameservers on which A records are defined.
Cloud CDN - CDN provides edge caching of files being uploaded to water.
Google Domains - Domains is used to purchase required domain for the service.
Google Signin - Signin is used to provide the auth capabilities of Google accounts.
Helm - Helm is used to deploy standard application stacks on to Kubernetes.
Memcache - Memcache is used to provide in-memory caches of request responses.
Pre-requisites
Cloud set-up

Following are the pre-requisites for deploying water on your own:
Create a Google Cloud Platform account here.
Create a project name of choice
Enable all of the above GCP technologies on your GCP Console. (It is sufficient to just enable for now).
Create a Google API Console project and client ID using this reference.
In the Authorized Javascript origins section of your sign-in console provide all origins of choice. It is best to also provide http://localhost:8080 so that local development is easier.
On the API console enable APIs of all of the above mentioned cloud technologies.
Create a bucket on Cloud storage with a name of choice. Bucket creation can be done as described here. Create a multi-regional bucket for highest performance.
Click on "None" next on the lifecycle column and set the following rules:
Age = 75 -> Move to Nearline
Age = 365 -> Move to Coldline
Age = 730 -> Delete
Using the same bucket name, create a topic on PubSub as described here.
Follow the procedure in the functions README to have the function deployed.
Local development

Local development and testing is covered in the service README.
Cloud deployment

Following are the steps required to be followed to deploy on to Google Cloud Platform:
Required Tools
Install gcloud CLI and initialize it using:
curl https://sdk.cloud.google.com | bash
gcloud init


Install kubectl using:
brew install kubectl


Install helm using and get helm running using:
brew install kubernetes-helm
helm init


Cloud configuration and deployment
Create a Kubernetes cluster of Google Container Engine using the steps documented here.
The following attributes need to be set while configuring the Kubernetes cluster during creation:
Node Image - Container-Optimized
Machine Type - f1-micro (costs lesser)
Minimal Size - 3
Maximal Size -
Create a static global IP called water using the following command:
gcloud compute addresses create water --global


Build and push the docker container using:
cd $GOPATH/src/github.com/vjsamuel/water
gcloud container builds submit --tag gcr.io/<project-id>/water:1.0.12 .


Create the namespace using:
kubectl create -f specs/namespace.yml


Install memcache using:
helm install memcache stable/memcached --namespace=project


Deploy the required configurations:
kubectl create -f specs/configmap.yml


Deploy the application and service of water:
kubectl create -f specs/deployment.yml
kubectl create -f specs/service.yml


Enable autoscaling as per scale requirements using:
kubectl autoscale deployment water -n project --min=1 --max=2 --cpu-percent=90


To run the application in insecure mode(on port 80) the Ingress spec for the same can be deployed as:
kubectl create -f specs/ingress-insecure.yml


If water needs to be deployed as over SSL, then the required certificate needs to be procured from a suitable provide in .pem format as tls.crt and tls.key and uploaded to Kubernetes as:
kubectl create secret generic water-cert --from-file=tls.crt --from-file=tls.key -n project


(Hint: A free and easy way to get a certificate would be to use certbot)
(Command: sudo certbot certonly --manual -d <water subdomain>)
Once the secret is in place the ingress can be replaced with:
kubectl replace -f specs/ingress.yml


Goto Google CDN and click on "Add Origin" to add the newly provision load balancer so that all requests being served as cached based on the API that is being hit.
DNS Configuration
Now the entire application has been deployed and the DNS needs to be configured. Following are the steps required to create the DNS A records:
Goto Google Domains and purchase a domain of choice.
In Google Cloud DNS console create a zone and provide the name of the domain in the configuration.
Click on "Setup Registrar" and copy the name servers on the list and add them as custom nameservers in Google Domains.
Click on "Add record set" in the new Cloud DNS Zone and add the static IP that was created to a domain name of choice like water.<domain> as an A record. Save it and wait for the DNS to do the required replication.
This will help you be all set to start using water on your own cloud account!



