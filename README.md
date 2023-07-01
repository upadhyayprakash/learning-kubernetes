# Learning Kubernetes

**Course Link**: https://www.linkedin.com/learning/learning-kubernetes-16086900

**Trainer**: Kim Schlesinger (LinkedIn: https://www.linkedin.com/in/kimschlesinger/)

**Date**: 28th June 2023

# Notes

### Glossary

- Container Image:
  > A file with executable code that runs withint a container.
- Container Registry:
  > Collection of Images that can be used by public to download the Image and spin the container. For eg. DockerHub, Google Container Registry.
- CNCF: Cloud Native Computing Foundation
- Cloud Native: Set of technologies that help deploy applications on Cloud and scale them as per need.
- Helm:
- Prometheus:
- GitOps:
- IaaC:
- Cluster:
- Kubectl(`k`): Also called as "Kube Control", is a CMD utility to interact with K8s API to manage the cluster control plane.
- Template:
- Replicas:

### History of Kubernetes

- Google open sourced **Borg**, an internal container orchestration system.
- Spotify adoption of Kubernetes over its homegrown Helios. Handling over 10 millions txn per second.

### Containers

- What is Container?
  > Application _`Code`_ along with _`Configs`_ to run the code is bundled as a Container.
- Advantages:
  - Portable: can run on any machine
  - Lightweight: compared to VMs. Hence can be quickly spun-up to scale up OR down as per need.
- Alternatives:
  - Podman
  - ContainerD
  - rkt
  - LXD

### Cloud Native

- Set of technologies that help deploy applications on Cloud and scale them as per need.
- Techs like Kubernetes, Service Mesh(Istio), Microservice etc are part of the ecosystem.

### Install Docker for MacOS

- Install Docker Desktop: In order to run K8s cluster in the Docker Runtime Engine.
- Install Minikube: To practise K8s cluster in localhost. Use `brew install minikube` to install.

### Start the minikube cluster

- All the k8s operations can be run on your machine using minikube.
- To start a minikube cluster,

  ```yaml
  minikube start
  ```

  ![Minikube Start Command](images/image_12%20start_minikube.png)

  > Learn more about minikube with the handbook (https://minikube.sigs.k8s.io/docs/handbook/)

### Kubernetes Objects:

- Namespace:
- Pod:
- Node:
- Service:
- Deployment:

### Yaml: YAML Ain't Markup Language

- It's a manifest document.
- It's a Data Serialization Language for communication between computer processes.
- Other are JSON & XML.
- File extensions: `.yaml` OR `.yml`
- Validate YAML Syntax online: `yamlchecker.com`

### Steps in Pod Creation

1. Creating a namespace

   ```yaml
   ---
   # creating a namespace 'development'
   apiVersion: v1
   kind: Namespace
   metadata:
   name: development
   ```

2. Create a Deployment: check the `deployment.yaml`

   ![Deployed Pods](images/image_3%20get_pods_namespace.png)

3. Run the manifest,

   ```yaml
   k apply -f deployment.yaml
   ```

   ![Run Manifest](images/image_6%20k_apply_file.png)

4. Inspect the created pods and events log

   ```yaml
   k describe pod <pod_name> -n <namespace>
   ```

5. Now that we've created the Pods, we need to check if it's really running our application inside it. We'd use BusyBox for this.

### BusyBox: Inspect the Pod

- It helps us check if the apps in the Pod are running as expected --> without having to expose to the internet!
- Create the busybox pod using `busybox.yaml` script, in `default` namespace.

  ```yaml
  k apply -f busybox.yaml
  ```

- To inspect our Application Pods, we need to get the Pod IP address and then use BusyBox to connect with the Pod. Run the following command,

  ```yaml
  k get pods -n <namespace> -o wide
  ```

- `-o wide` gives more details about the pod.

  ![Wide Details Pods](images/image_5%20get_pods_wide.png)

- Once the application pod's IP address is known, we can run the following script from **busybox** pod to open the _interactive_ terminal,

  ```yaml
  k exec -it <busy_box_pod_name> -- /bin/sh
  ```

  ![Busybox Exec](images/image_7%20busybox_exec_sh.png)

- Once logged-in, run the wget command (along with correct port number: `3000`) to get the service response,

  ```yaml
  wget <pod_ip_address>:<port_number>
  ```

- The response (i.e Pod Info) should've downloaded in `index.html`.

  ![Busybox Wget](images/image_8%20busybox_wget.png)

- You can also check the application log by,

  ```yaml
  k logs <pod_name> -n <namespace>
  ```

  ![Pod Log](images/image_9%20logs_pod.png)

### Kubernetes Service

- It's a load balancer that routes internet traffic to the pods within your cluster.
- It has a public & static IP address, so that even after your pods are restarted/terminated, your users can reach the single IP location to access your pod application.
- Check the yaml file `service.yaml` which is a **LoadBalancer** service to expose your pods to the internet.

### Resources

- You can set resource for your pods within a container.
- It helps you limit the resources utilisation within a node.
- If we don't specify, the pod could use all the available resources and may cuase outages. You can add the `resources` block after the `image` key in the container definition.

  ```yaml
  resources:
    requests:
      memory: '64Mi'
      cpu: '250m'
    limits:
      memory: '128Mi'
      cpu: '500m
  ```

### Delete k8s objects & tear down the Cluster

- You might want to delete your k8s resources when it's no longer needed.
- The delete command is,

  ```yaml
  k delete -f <yaml_file>
  ```

  ![Delete k8s Objects](images/image_10%20delete_k8s_objects.png)

- Deleting the resources can be followed in following sequence,

  ```bash
  Pods -> Service -> Namespace
  ```

- To quickly delete everything, you can delete the namespace.

- To delete the minikube cluster

  ```yaml
  minikube delete
  ```

  ![Delete Minikube](images/image_11%20delete_minikube.png)

  > You can see all the available k8s API resources using this command, `k api-resources` > ![k8s objects](images/image_2%20api_resources.png)

### Kubernetes Architecture

#### Control Plane

- They're like an airport's ATC tower that controls flights.
- Every cluster in k8s has a **Control Plane**.
- Components within Control Plane:

  ![System Pods](images/image_1%20get_system_pods.png)

  - **k8s API Server**:
    - It's an REST service running inside our cluster as a Pod which accepts users request. It can be interacted using `kubectl` or `kubeadm`.
    - The API server is running within `kube-system` namespace, and can be verified using command,
    ```yaml
    k get pods -n kube-system
    ```
  - **etcd**:
    - It's an open-source, highly available key-value store that stores all the data about state of the cluster.
    - Only API Server can directly communicate with this.
    - It runs as a pod. Check with `k get pods -n kube-system`
  - **Kube Scheduler**:

    - It assigns the newly created Pods to their Node.
    - It runs as a pod.

  - Controller Manager:

    - It continuously monitors the status of the cluster.
    - It checks if all the worker nodes are running and if a node is broken, it replaces it with a new node.

  - Cloud Controller Manager:

    - It helps our cluster communicate with the Cloud Providers APIs.

    > In Managed k8s solutions like AWS EKS or Google GKE, we can't access the Control Plane using `kubectl`, as it managed by the service provider themselves.

#### Worker Node

- They're like an airport terminals, where Airplanes are parked and passengers onboard.
- In order to be highly available, every cluster in k8s should have at least 3 `Worker Node`.
- Our k8s pods are schedules inside the Worker Nodes.
- It has 3 components:
  1. Kubelet: Communicates with Control Plane about new incoming pods. It also pulls the image used for creating Container.
  2. Container Runtime: `Kubelet` uses Contriner Runtime Interface (CRI) to create a Containers in the Container Engines such as Container-d, CRI-O, Kata Containers and AWS Firecracker.
     > - Dockershim is another Container Engine, but in version v1.24, the support for the same was removed. Hence Docker Engine can no longer be used as Container Runtime.
     > - But Docker Images can still be used, as it's not the same as Docker Engine.
  3. Kube-Proxy: Ensures that our pods/services can communicate with other pods/services. It communicates directly with kube-API Server.

#### How Control Plane and Worker Node work together

- Check the sequence diagram for `k apply -f deployment.yaml` command.

  ![Kubernetes Architecture](images/image_4%20kubernetes%20architecture.png)

- Also watch this video: https://www.youtube.com/watch?v=X40LJM0KuQ8

### Other ways to Deploy and Manage the Pods

- You can use k8s "Deployment" or "Daemonset"(one pod per node) or k8s "Job"(one-time-run) to deploy the pods.

### Handling Persistent Data

- You can either connect your k8s cluster to a managed DB like AWS RDS, or Azure SQL or Google Cloud SQL.
- Or you can use k8s Persistent Volumes. Persistent Volumes are a data storage within a cluster that are still accessible after the pod is destroyed.
- `statefulSet` object can be used to ensure that the newer pods can still connect to the same Persistent Volumne after the older one got deleted.

### Kubernetes Security

- Since our cluster is accessible to the internet, we must use best practices to secure our k8s cluster and avoid any security incidents.
- Hackers might try to compromise your k8s cluster in 3 ways:
  1. Steal the Data inside your cluster
  2. Take control of computation power to mine crypto currencies.
  3. Cause DDoS
- In order to avoid getting `root` access to the cluster, we can configure the `securityContext`
  - Set **runAsNonRoot** as `true` --> To ensure no processes are using root privilege
  - Set **readOnlyRootFilesystem** as `true` --> To ensure no one can write an executable file to the disk.
- Snyk:
  - You can scan your `.yaml` manifest files with tools like "snyk" for security risks.
  - Command: `snyk iac test deployment.yaml`
- Ensure you always update your k8s whenever a security patch is released.
- Also read **Kubernetes Hardening Guide** by US National Security Agency.

### What's Next step in Learning Kubernetes?

- Watch "KubeCon" conferences.
- Learn from other LinkedIn courses called, "Master Cloud-Native Infrastructure with Kubernetes"
-
