### MQTT Data Aggregation and Collection at the Edge

#### Overview
This guide details the steps for setting up and deploying MQTT data aggregators and collectors in a Kubernetes environment, specifically tailored for edge computing scenarios.

#### Prerequisites
- Access to a Kubernetes cluster.(https://haicgu.github.io/access.html)
    ```
    <<Userid/password to be shared during the tutorial>>
    ssh guoehi-dev
    cd
    mkdir .kube
    cp /mnt/dev-lscratch/tutorial/config ~/.kube
   ```
- Check access to kubernetes cluster
  ```
  kubectl get nodes -A
   ```
- Login credentials for the GitHub Image Repository.
  ```
    password TOKEN can be found in /mnt/dev-lscratch/tutorial/passwords
   ```
- Podman installed on your machine.
   - Podman rootless is already preinstalled on the `dev` node
   - Running unprivileged Podman won't support storage on NFS. If you get warnings (or errors) about running on NFS, configure Podman to use local storage instead. E.g. on the `dev` node: `export XDG_DATA_HOME="/mnt/dev-lscratch/$USER/local_share"; mkdir -p "$XDG_DATA_HOME"`

#### Setup Instructions

1. **Prepare Your Environment**
   - Make sure you are in your home directory:
     ```bash
     cd ~
     ```
   - Clone the repository:
     ```bash
     git clone https://github.com/rho770/oehi-cc-training.git ~/demo/
     ```
   - Navigate to the demo folder and desired workflow:
     ```bash
     cd ~/demo/workflow2
     ```

2. **Build and Push Docker Images**
   - **Aggregator Image**
     - Build the Docker image:
       ```bash
       podman build -f aggregator.dockerfile . -t ghcr.io/haicgu/"$USER"-aggregator:latest
       ```
     - Push to the registry:
       ```bash
       podman push ghcr.io/haicgu/"$USER"-aggregator:latest
       ```
   - **Collector Image**
     - Build the Docker image:
       ```bash
       podman build -f collector.dockerfile . -t ghcr.io/haicgu/"$USER"-collector:latest
       ```
     - Push to the registry:
       ```bash
       podman push ghcr.io/haicgu/"$USER"-collector:latest
       ```
    - **Client Image**
     - Build the Docker image:
       ```bash
       podman build -f client.dockerfile . -t ghcr.io/haicgu/"$USER"-client-wf:latest
       ```
     - Push to the registry:
       ```bash
       podman push ghcr.io/haicgu/"$USER"-client-wf:latest
       ```

3. **Deploy on Kubernetes**
   - Apply the Kubernetes YAML files to deploy the aggregator and collector:
     ```bash
     envsubst < yaml/aggregator_cloud.yaml | kubectl create -f -
     envsubst < yaml/collector_edge.yaml | kubectl create -f -
     envsubst < yaml/client.yaml | kubectl create -f -
     ```

4. **Monitoring and Logs**
   - Check the logs of the pods to monitor operations:
     ```bash
     kubectl get pods -n decice
     kubectl logs –n decice –f client_pod_name
     ```

5. **Cleanup Resources**
   - To remove deployed resources: MQTT aggregator, collector and client
     ```bash
     envsubst < yaml/aggregator_cloud.yaml | kubectl delete -f -
     envsubst < yaml/collector_edge.yaml | kubectl delete -f -
     envsubst < yaml/client.yaml | kubectl delete -f -    
     ```

#### Additional Information
- Review the YAML files to understand how the aggregator and collector are scheduled on the Kubernetes edge node.

### Note
Replace placeholders such as `[pod_name]` with actual values from your Kubernetes cluster.

