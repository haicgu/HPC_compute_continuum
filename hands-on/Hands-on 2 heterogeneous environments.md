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
     git clone https://github.com/haicgu/HPC_compute_continuum.git ~/demo/
     ```
   - Navigate to the demo folder and desired workflow:
     ```bash
     cd demo/hands-on/workflow-2
     ```
   - Login to GitHub Image Repository**
   ```bash
   podman login ghcr.io -u haicgu -p [TOKEN]
   ```
   Replace `[TOKEN]` with your actual personal access token.
   password TOKEN can be found in /mnt/dev-lscratch/tutorial/passwords

3. **Build and Push Docker Images**
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

4. **Deploy on Kubernetes**
   - Apply the Kubernetes YAML files to deploy the aggregator and collector:
     ```bash
     envsubst < yaml/aggregator_cloud.yaml | sed 's/-isc24_/-isc24-/g' | kubectl create -f -
     envsubst < yaml/collector_edge.yaml | sed 's/-isc24_/-isc24-/g' | kubectl create -f -
     envsubst < yaml/client.yaml | sed 's/-isc24_/-isc24-/g' | kubectl create -f -
     ```

5. **Monitoring and Logs**
   - Check the logs of the pods to monitor operations:
   - Find the pod name of your client
     ```bash
     kubectl get pods -n decice
     ```
     - Check the logs of the client
     ```bash
     kubectl logs -n decice -f [client_pod_name]
      ```
6. **Cleanup Resources**
   - To remove deployed resources: MQTT aggregator, collector and client
     ```bash
     envsubst < yaml/aggregator_cloud.yaml | sed 's/-isc24_/-isc24-/g' | kubectl delete -f -
     envsubst < yaml/collector_edge.yaml | sed 's/-isc24_/-isc24-/g' | kubectl delete -f -
     envsubst < yaml/client.yaml | sed 's/-isc24_/-isc24-/g' | kubectl delete -f -    
     ```

#### Additional Information
- Review the YAML files to understand how the aggregator and collector are scheduled on the Kubernetes edge node.
- Check the difference in the yaml files workflow-2/yaml/collector_edge.yaml workflow-1/yaml/publisher.yaml. What change is needed to run using KubeEdge on the Edge node?

# Advanced exercise with on-site hardware

## On-site devices

* RBG LED - can be controlled by sending an MQTT message to the topic `isc24/led1`
    * Possible formats:
        1. `R,G,B`, where R,G,B are integers in range `0..255`
        2. One of supported colors as a string - one of: ['red', 'green', 'blue', 'yellow', 'yelow', 'white', 'black']
* Buttons - on press will send messages withe a button code to `isc24/buttons`

## Possible task 1
1. Figure out, which code corresponds to which button
2. Deploy an application that will subscribe to the vents from buttons, convert codes to colors and light up LED with the color of a pressed button

   HINT: Modify the publisher of workflow-1 to make the LED glow the colors you wnat
   ```
    msg = f"red"
    result = client.publish("isc24/led1", msg)
    ```

## Possible task 2
1. Figure out, which code corresponds to which button
2. Deploy an application that will implement a game with color matching? The LED lights up with different random colors in a sequence, player is supposed to pres colored buttons in the same sequence


### Note
Replace placeholders such as `[pod_name]` with actual values from your Kubernetes cluster.

