# Cloud-Native-Monitoring-Application
Cloud Native app on K8S to monitor system resources using python


### PRE-REQUISITES :

- [x]  Programmatic access and AWS configured with CLI.
- [x]  Python3 Installed.
- [x]  Docker and Kubectl installed.
- [x]  Code editor (Vscode)


### PROJECT OVERVIEW :

1. Creating a Monitoring Application in Python using Flask and psutil
2. Running the Python App locally.
   
3  Creating Dockerfile, Building DockerImage, Running Docker Container
4. Create ECR repository using Python Boto3 and pushing Docker Image to ECR
5. Usage of Kubernetes and Creating EKS cluster and Nodegroups
6. Create Kubernetes Deployments and Services using Python



### PROJECT WORKFLOW :

### Part 1: Deploying the Flask Application locally

### Step 1: Install dependencies

The application uses the **`psutil`** and **`Flask`, Plotly, boto3** libraries. Install them using pip:

```
pip3 install -r requirements.txt
```

### **Step 2: Run the Application**

To run the application, navigate to the root directory of the project and execute the following command:

```
python3 app.py
```


![WhatsApp Image 2024-06-20 at 11 22 01_fdb63694](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/da4f4ca0-c65a-448d-b3f6-fedc66feab4c)

![WhatsApp Image 2024-06-20 at 11 22 01_8f3eb487](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/f147bc8d-f61d-46ad-b458-334720c77f22)


![WhatsApp Image 2024-06-20 at 11 22 02_bb171743](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/0bdf17f9-d13e-4eb1-99a5-c5e80257270d)

![WhatsApp Image 2024-06-20 at 11 22 02_f4c45a55](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/85dd93b0-281f-4a98-b76d-89726d4b45a1)

![WhatsApp Image 2024-06-20 at 11 22 03_8a064b28](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/012db081-1966-4be4-8231-8dc855658459)







This will start the Flask server on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on your browser to access the application.

### Part 2: Dockerizing the Flask Application

### Step 1: Create a Dockerfile**

Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Step 2: Build the Docker image**

To build the Docker image, execute the following command:

```
docker build -t <image_name> .
```

### **Step 3: Run the Docker container**

To run the Docker container, execute the following command:

```
docker run -p 5000:5000 <image_name>
```

This will start the Flask server in a Docker container on **`localhost:5000`**. Navigate to [http://localhost:5000/](http://localhost:5000/) on the browser to access the application.

### Part 3: Pushing the Docker image to ECR

### **Step 1: Create an ECR repository**

Create an ECR repository using Python:

```
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```

### **Step 2: Push the Docker image to ECR**

Push the Docker image to ECR using the push commands on the console:

```
docker push <ecr_repo_uri>:<tag>
```

![WhatsApp Image 2024-06-20 at 11 22 03_1c94884d](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/66d86a78-adbc-4f85-9a89-1ad0207ffe77)


## **Part 4: Creating an EKS cluster and deploying the app using Python**

### **Step 1: Create an EKS cluster**

Create an EKS cluster and add node group

![WhatsApp Image 2024-06-20 at 11 22 03_418a7ecd](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/90c9d587-2eaf-4750-8222-e5b843e51738)



### **Step 2: Create a node group**

Create a node group in the EKS cluster.

### **Step 3: Create deployment and service**

```jsx
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="568373317874.dkr.ecr.us-east-1.amazonaws.com/my-cloud-native-repo:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

make sure to edit the name of the image on line 25 with your image Uri.

- Once you run this file by running “python3 eks.py” deployment and service will be created.
- Check by running following commands:

```jsx
kubectl get deployment -n default (check deployments)
kubectl get service -n default (check service)
kubectl get pods -n default (to check the pods)
```

```
kubectl get svc -n default
```

![WhatsApp Image 2024-06-20 at 11 30 11_c25e5c71](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/95f0d57f-42c6-4a15-acf1-42c0cb709f2b)


```
kubectl get pods -n default -w
```

![WhatsApp Image 2024-06-20 at 01 41 08_4b3d0463](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/1ee0e7c8-1ff7-4135-916a-e69ddfa3dc11)


```
kubectl get deployment -n default
```

![WhatsApp Image 2024-06-20 at 01 43 46_bae2df8f](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/b92e5a99-6435-4b12-8955-98e2385ca407)


```
kubectl edit deployment my-flask-app -n default
```

![WhatsApp Image 2024-06-20 at 01 46 18_3aa26da2](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/a612777c-9bd9-45f1-a952-080352a765fc)

```
kubectl get pods -n default -w
```
![WhatsApp Image 2024-06-20 at 01 46 48_23c62970](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/57279168-bfa6-4595-8a43-d736a51dc8f9)






Once your pod is up and running, run the port-forward to expose the service

```bash
kubectl port-forward service/<service_name> 5000:5000
```


```
kubectl port-forward svc/my-flask-service 5000:5000
```

![WhatsApp Image 2024-06-20 at 01 49 09_c5f9d561](https://github.com/vighas-ks-16/Cloud-Native-Monitoring-Application/assets/107311113/8c5394dc-968e-4247-900f-512fafabef3b)

