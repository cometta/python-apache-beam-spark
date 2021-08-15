## Python job with Apache Beam and Spark Cluster on Kubernetes

### Why 
 There is currently no step-by-step guide on how to configure Apache Beam with Spark cluster on Kubernetes for Python job. 

# Installation

- This guide only work for on real K8s cluster. If you are running a MacOS/Window, you will need to boot up Virtualbox with Linux iso, install Ubuntu and Kubernetes. 
- Make sure your K8s support ReadWriteMany storage. You can install NFS or Longhorn on K8s. Below are the optional steps to install Longhorn on the Kubernetes nodes
```
    sudo apt-get install open-iscsi nfs-common
    kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```



- Create a K8s namespace
    ```
    kubectl create namespace spark-beam
    ```
- Create storage, if you are using Virtualbox and self installed Kubernetes and Longhorn. Uncommented the storageClassName on the file before execute it
   ```
    kubectl apply -f pvc.yaml --namespace spark-beam
   ```
- If you are using a real K8s cluster and your team administrator already installed a network storage system
    ```
    kubectl apply -f pvc.yaml --namespace spark-beam
    ```

- Check storage is successfully created before preceed STATUS=Bound
    ```
    kubectl get pvc,pv --namespace spark-beam
    ```
- Deploy a Spark cluster with one master and one worker
    ```
    kubectl apply -f deployment.yaml --namespace spark-beam
    ```
- Wait until all pods are running, STATUS=Running
    ```
    kubectl get pods,services --namespace spark-beam
    ```

### Run Example
- Install Apache beam python library
    ```
    pip install apache-beam==2.30.0
    ```
- Get the IP and input it into example.py for --job_endpoint and --artifact_endpoint. No need to change to the port number 
    ```
    kubectl get nodes -o wide
    ```
- Run the example
    ```
    python example.py
    ```

### Build Custom Spark Image (Optional)
```    
git clone git@github.com:bitnami/bitnami-docker-spark.git

cd bitnami-docker-spark
git checkout 2.4.6-debian-10-r13
cd 2/debian-10
```
    

- Edit the Dockerfile by including below step to install docker-ce. The editted file should look like the one in this repo
    ```
    RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y libltdl7
    RUN apt-get update; \
    apt-get -y install apt-transport-https ca-certificates curl gnupg software-properties-common; \
    curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -; \
    apt-key fingerprint 0EBFCD88; \
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/debian \
        $(lsb_release -cs) \
        stable" ;\
    apt-get update; \
    apt-get -y install docker-ce
    ```

- finally build and push the image to Docker Hub
    ```
    docker build . -t docker.io/<account>/spark-custom-2.4.6
    docker push  docker.io/<account>/spark-custom-2.4.6
    ```

### Footnote
- Need to use Linux as the host for Kubernetes as we are running Docker in K8s
- Only work for Spark 2 and not version 3