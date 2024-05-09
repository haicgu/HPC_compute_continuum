
### README: Container Deployment Instructions

#### Overview
This guide provides instructions for building and deploying MQTT publisher and subscriber services using Podman and Kubernetes. Ensure you have the necessary permissions to execute these commands, as some may require `sudo` access.

#### Prerequisites
- Access to a Kubernetes cluster.(https://haicgu.github.io/access.html)
    ```
    <<Userid/password to be shared during the tutorial>>
    ssh guoehi-dev
    cd
    mkdir .kube
    cp ../kubernetes/config ~/.kube
   ```
- Check access to kubernetes cluster
  ```
  kubectl get nodes –A 
   ```    
- Login credentials for the GitHub Image Repository.
  ```
    <<to be shared during the tutorial>>
   ```
- Podman installed on your machine.
  
#### Setup Instructions

1. **Navigate to Home Directory**
   ```bash
   cd ~
   ```

2. **Clone the Repository**
   ```bash
   git clone https://github.com/rho770/oehi-cc-training.git ~/demo/
   ```

3. **Navigate to the Project Directory**
   ```bash
   cd ~/demo/workflow1
   ```

4. **Optional: Check/Edit Python Code**
   Modify the Python code within the repository if necessary.

5. **Login to GitHub Image Repository**
   ```bash
   sudo podman login ghcr.io -u haicgu -p [TOKEN]
   ```
   Replace `[TOKEN]` with your actual personal access token.

6. **Build the Publisher Docker Image**
   ```bash
   podman build -f publisher.dockerfile . -t ghcr.io/haicgu/"$USER"-publisher:latest
   ```

7. **Build the Subscriber Docker Image**
   ```bash
   podman build -f subscriber.dockerfile . -t ghcr.io/haicgu/"$USER"-subscriber:latest
   ```

8. **Push the Docker Images**
   - **Publisher Image**
     ```bash
     podman push ghcr.io/haicgu/"$USER"-publisher:latest
     ```
   - **Subscriber Image**
     ```bash
     podman push ghcr.io/haicgu/"$USER"-subscriber:latest
     ```

#### Deploying on Kubernetes

1. **Deploy the MQTT Publisher**
   ```bash
   envsubst < yaml/publisher.yaml | kubectl create -f -
   ```

2. **Deploy the MQTT Subscriber**
   ```bash
   envsubst < yaml/subscriber.yaml | kubectl create -f -
   ```

3. **Check the Logs**
   - **Find the Pods**
     ```bash
     kubectl get pods –n decice
     ```
   - **Logs for Publisher**
     ```bash
     kubectl logs –n decice –f [publisher_pod_name]
     ```
   - **Logs for Subscriber**
     ```bash
     kubectl logs –n decice –f [subscriber_pod_name]
     ```

#### Cleanup

- **Undeploy the MQTT Publisher**
  ```bash
  envsubst < yaml/publisher.yaml | kubectl delete -f -
  ```
  
- **Undeploy the MQTT Subscriber**
  ```bash
  envsubst < yaml/subscriber.yaml | kubectl delete -f -
  ```

#### Additional Information

Review the YAML files to understand the assignment of pods to Kubernetes nodes and locate the connection details for the broker.

### Note
- Replace placeholders such as `[TOKEN]` and `[publisher_pod_name]` with actual values.
- Ensure that `envsubst` is installed and available in your environment.

This README is intended to guide you through setting up your development environment for deployment. Adjust the instructions as needed to fit your specific project requirements.
