# Deploying a Reddit Clone on Kubernetes using Minikube with Ingress Enabled 

We're diving deeper into the Kubernetes universe in this hands-on project guide. I've divided this project into 2 parts for better understanding.

In the **first part**, we'll be setting up:
- Launching the EC2 instance with 't2.medium' type
- Minikube & Docker on Ubuntu 
- Build the Docker image
- Push it to docker hub
  
In the **2nd part**, 
- We'll deploy app using K8s
- We'll configure various manifest files like deployment.yml, service.yml & ingress.yml

But, before we begin this exciting Reddit clone deployment journey, let's make sure you meet the following prerequisites:
 
## Prerequisites

1. **Knowledge of Kubernetes (K8s):** Understanding the basics of Kubernetes, including Pods, Deployments, Services, and Ingress controllers, is essential. You should be familiar with how K8s manages containerized applications.

2. **Reddit-Clone Application Code:** You need the source code of your Reddit-clone application. Ensure it's well-structured and containerized, with necessary dependencies specified in a Dockerfile.

3. **Docker Knowledge:** A basic understanding of Docker is crucial, as containers are fundamental to Kubernetes deployments. You should be able to create Docker images from your application code.

4. **A Running Kubernetes Cluster:** You must have access to a Kubernetes cluster where you can deploy your application. This could be a local cluster (e.g., Minikube) or a cloud-based solution (e.g., Google Kubernetes Engine - GKE).

5. **Kubernetes CLI (kubectl):** Install and configure the kubectl command-line tool to interact with your Kubernetes cluster. It's essential for managing your deployments and resources.

6. **Ingress Controller Setup:** Before writing an Ingress controller, you need to ensure that an Ingress controller is set up and configured in your Kubernetes cluster. Common choices include Nginx Ingress Controller and Traefik.

7. **DockerHub Account:** This blog requires an active DockerHub account that will be used to upload and download the docker image we will be building. You can create an account by visiting [Docker Hub](https://hub.docker.com/).

8. **Git:** Git installed on your Ubuntu machine.

9. **Docker & Minikube Installed:** If you haven't already set up docker & Minikube cluster, please refer to the installation guide https://github.com/LondheShubham153/kubestarter/blob/main/minikube_installation.md to get started.

### With all the prerequisites satisfied, let’s dive into our project!

# PART 1

## Step 1: Launch the EC2 instance: 
Launch the ubuntu instance with the instance type as `t2.medium`

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/89a5db6d-91d2-456e-ba17-246c4181b8ad)


## Step 2: Install the Minikube & docker engine on Ubuntu
Refer to below guide for installation:
https://github.com/LondheShubham153/kubestarter/blob/main/minikube_installation.md

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/fe3b4460-879e-4e2d-bef3-60c6d297e670)


## Step 3: Clone the source code of the project
Run the given command in your Linux terminal to download the project files:

```bash
https://github.com/sakshirathoree/reddit-clone-k8s-ingress.git
```
## Step 4: Move to the project directory & Build the Dockerfile
The project folder includes a Dockerfile that will be used to build the image.

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/4a010a90-08a4-4225-8ad1-673eb18ef340)

- **Dockerfile**
```
FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone
RUN npm install 

EXPOSE 3000
CMD ["npm","run","dev"]
```
Run the below-given commands inside the root of the project folder to build the docker image, replace the placeholder <username> with your DockerHub username.
```
docker build --rm -t <username>/reddit-clone:latest .
```
The --rm flag ensures that any intermediate containers created during the build process will be automatically removed once the build is finished, helping to maintain a cleaner development environment.

Use the command `docker images` to view your newly built image.

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/6bfea09f-62ed-4f36-9e1d-7a43371d9176)

## Step 5: Push the docker image to Docker Hub
After building the image, we need to push it to docker hub, our Kubernetes deployment will later fetch the same image from docker hub to run the app.

- **Log in to the DockerHub from CLI using `docker login`**

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/d1cd1b9c-b043-421c-82e6-58cf708fb9f3)

- **Push the Docker image to DockerHub:**
  
  Use the command `docker push <username>/reddit-clone:latest` to push the image, replace the placeholder `<username>` with your username.
  After pushing the image, refresh your DockerHub website, and you will be able to see a new repository has been created containing your docker image.
  
![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/fa8fdb1b-bccc-4dac-b55b-f3e2bfa24372)

Congrats on reaching here! 🥳 You have now successfully built a docker image and uploaded it to Docker Hub, thus completing the basic steps of the continuous integration Process.
Now we will be focusing on deploying the docker image in a K8s cluster. For this, we will be using the Deployment object of K8s to create Pods that run our image.

# PART 2

 Before we begin with our deployment, Let's first create the `namespace` 

 ##  What is namespace?
 - namespace is a logical entity that allows you to isolate resources like pods, volumes, deployments etc. It's a way to logically partition or segregate resources within a cluster.
 - **Resource Isolation:** Namespaces help isolate resources, preventing naming conflicts between objects like Pods, Services, ConfigMaps, and more. This separation enhances security and resource management.
 - **Organization:** Namespaces enable logical organization and categorization of resources. You can group related objects and apply policies at the namespace level.

Let's create a `namespace` in a project folder:
```
kubectl create namespace reddit
```

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/275b42d0-4dd7-41bf-854f-ae3774a669e6)

 ##  What is Deployment?
 - Deployment in k8s is a controller which helps your applications reach the desired state, the desired state is defined inside the deployment.yml file
 - Deployments are used to manage and scale the replica sets of Pods(A Pod is the smallest deployable unit in Kubernetes, representing a single instance of a running process.
Pods can contain one or more containers, and  are used to deploy applications, and each Pod has its own unique IP address within the cluster)
 - They ensure a specified number of replicas are running and provide updates and rollbacks for application changes.

## Step 1: Creating a Deployment Manifest for K8s 
 -  **deployment.yml:**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  namespace: reddit
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: rajlaxmii/reddit-clone:latest
        ports:
        - containerPort: 3000
```
Make sure to replace the value of image key with that of your image name as shown in the YAML file.

Run `kubectl apply -f deployment.yml` to create the deployment object in K8s. You should see an output similar to this:

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/5310525d-8354-40cd-8a9d-b4f8047cf82d)

The deployment creates 2 pods running 1 container each with the reddit-clone image we pushed earlier. The key containerPort declares that the container accepts connections from port 3000.

Run `kubectl get deployment -n reddit` and `kubectl get pods -n reddit` to ensure that your deployment has been successfully created and pods are running. 

Your output should look like this:

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/44e5a924-7f43-494e-99cb-f7ad892d6edc)

**Don't forget to specify namespace using -n else it'll consider the default namespace & will not show any resources**

Our next step is to create a service object that will allow us to connect to the pods created by the deployment file.

##  What is Service?
- A Service in K8s serves as a link between distinct pods or microservices inside a cluster.
- It provide network access to a set of Pods, allowing them to be accessed by other Pods or external clients.
- They can be used for load balancing, service discovery, and routing network traffic.
- There are different types of Services, including ClusterIP, NodePort, and LoadBalancer.

## Step 2: Create a Service Manifest for K8s

- **service.yml:**
```
apiVersion: v1
kind: Service
metadata:
  
  name: reddit-clone-service
  namespace: reddit
spec:
  selector:
    # Selector for Pods
    app: reddit-clone
  type: NodePort
  # We are using NodePort service which is directly
  # accessible from outside the cluster, other types of services
  # are ClusterIP, LoadBalancer
  ports:
    # Port is the port service is gonna listen to
  - port: 3000
    # Target port is the port container is listening to
    targetPort: 3000
    nodePort: 30007
    protocol: TCP
```

    Run `kubectl apply -f service.yml` to create this service in K8s.
    
    Run `kubectl get svc -n reddit` to check if the service has been created, the output should look like this:
    
![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/00f18173-c51e-45b6-a757-ab9f32117dac)

## Step 3: To get the URL for your application
```
minikube service reddit-clone-service -n reddit created --url
```
  ![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/525ecfae-4912-4a11-bab1-0963391c152c)

**Make sure to open port 3000 & 30007 in the Deployment Server of the security group**

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/cd88eec9-51f9-4b0c-b975-8958b0582b1c)

Expose the App Deployment:
```
kubectl expose deployment reddit-clone-deployment -n reddit --type=NodePort
```
![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/f04a2731-95bc-4831-87c4-4c1f25b8b0fd)

Expose the App Service:
```
kubectl port-forward svc/reddit-clone-service -n reddit 3000:3000 --address 0.0.0.0
```

## Step 4: Access your Reddit app :

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/430422dc-d5eb-4c48-a2ae-79b7f4127c52)

The app has now been successfully deployed! Congratulations on reaching this far! 🥳
Now we will be configuring ingress.

## Minikube addons: Enable Ingress

### Ingress in Minikube Simplified:
- Ingress is a smart way to manage traffic in Kubernetes.
- While Kubernetes provides IP addresses for pods and services, Ingress simplifies external access.
- It serves as a single entry point for various services, ideal for managing multiple microservices with different URLs.
- Ingress dynamically routes traffic based on rules, eliminating the need for static node IPs.
- It's particularly useful when you need to access many microservices via distinct URLs or domains.
- An Ingress controller fulfills and routes these rules, ensuring traffic goes to the right place.
- It's a powerful tool for simplifying external access to your Minikube cluster.

## 1. Enable Ingress in Minikube
In Minikube, ingress comes as an addon and we need to enable it before configuring it. Run minikube addons enable ingress in the terminal to enable ingress. The output should be like this:

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/8c55c2ef-08f5-402c-8b91-26b9a07c18e9)

## 2.  Write the Ingress K8s Manifest
- **ingress.yml**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  ingressClassName: nginx
  rules:
  - host: redditclone.com
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: reddit-clone-service
            # Port Service is listening on
            port:
              number: 3000
  - host: "*.redditclone.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: reddit-clone-service
            # Port Service is listening on
            port:
              number: 3000
```

We have used the domain `**redditclone.com**`as our entry point to the ingress Create the ingress rule using the command kubectl apply -f ingress.yml

and then run kubectl get ingress, the output should be like this:

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/1818a2e7-2d10-45a8-890c-6ade947cd888)

We need to wait for the Address column to be populated, it will take some time before it is so keep checking kubectl get ingress.

After the address column is populated like this:

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/5332e272-855e-4104-8ad9-cdfc11524afb)

Note down the IP displayed in the address column.

## 3. Test the Ingress
We have now successfully deployed the ingress and now we can test it. To test the ingress, we need to send a request to redditclone.com , however, we don’t own this domain, and there’s no DNS record pointing redditclone.com to our cluster. So, we will be using curl command to forcefully resolve redditclone.com to the IP of ingress as noted above. The command is:
```
curl --resolve "redditclone.com:80:<IP of Ingress>" redditclone.com
```
Upon executing this command, we immediately get a rather big output of the HTML file which is of our Reddit clone.

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/cb01c352-8696-492d-a45c-cb40835b0852)

**NOTE:** If you are running minikube in a linux machine with GUI and want to access redditclone.com from your browser, you can edit /etc/hosts file using vi to add a line with the format like: <IP of Ingress> redditclone.com The file should then look something like this:

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/f65e8633-b981-4839-8fb7-e870a415c2b2)

After saving the file, you should be able to access the app by using URL redditclone.com in your browser.

![image](https://github.com/sakshirathoree/reddit-clone-k8s-ingress/assets/67737704/5bcd35bb-61d6-494f-94f2-070cbe95b23c)

## Well Done on Your Reddit Clone Deployment! 🔥
Congratulations on successfully deploying the Reddit clone app with Minikube! You've unlocked the door to the Kubernetes world, and this is just the beginning. Keep exploring, learning, and building with Kubernetes—it's a journey filled with endless possibilities.

If you've any queries, feel free to reach me on LinkedIn:
https://www.linkedin.com/in/rajlaxmi-rathore29/



