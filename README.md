# Observability: Monitoring & Logging
## Table of Contents

1. [Overview of Observability](#Overview-of-Observability)
2. [Architecture](#Architecture)
3. [Features](#Features)
4. [User Handbook](#User-Handbook)
   - [Pre-requisites](#Pre-requisites)
   - [Deploying Jenkins](#Deploying-Jenkins)
   - [Deploying Webapp via Jenkins](#Deploying-Webapp-via-Jenkins)
   - [Deploying Prometheus](#Deploying-Prometheus)
   - [Deploying Splunk](#Deploying-Splunk)
   - [Ingesting K8s Cluster Data to Splunk](#Ingesting-Data-to-Splunk)
6. [Full Documentation](#Full-Documentation)


---
## Overview of Observability

### Observability: Monitoring & Logging
Provides real-time insights into the health, performance, and status of workloads running within the Kubernetes environment. As the solution consists of structured logging, monitoring, and alerting, we enable efficient troubleshooting and rapid investigation of potential issues.

## Project Description

### Project Overview
The project aims to provide real-time insights into the health, performance, and status of workloads running within the Kubernetes environment. As the solution consists of structured logging, monitoring, and alerting, we enable efficient troubleshooting and rapid investigation of potential issues.
### Project Purpose
The purpose of this project is to provide real-time insights into the health, performance, and status of workloads running within the Kubernetes environment. As the solution consists of structured logging, monitoring, and alerting, we enable efficient troubleshooting and rapid investigation of potential issues.

## Objectives:

### General Objectives
To use prometheus, alertmanager, grafana, and splunk to monitor logs of webapp and k8s cluster for efficient troubleshooting and problem/incident investigation.

Specifically, the group aims to do the following per tools:

- Webapps
   - Deployed in a Kubernetes cluster
   - Properties that enable to simulate the following issues:
      - High HTTP 5xx errors
      - High HTTP 4xx errors
      - Exporters stopped working
      - Splunkforwarder service or sidecar stopped working
           
- Jenkins
   - Set up a CI/CD pipeline that builds and deploys the app whenever code changes.
       
- Prometheus
   - Metric gathering solution of webapp.
       
- Alerting using Alertmanager/Prometheus or Splunk
    - Create alerting rules based on the KPIs:
        - % availability
        - Resource Utilization
        - Out of Memory Errors
        - Error Rates
   - Establish threshold and implement annotations

- Grafana
   - Use Prometheus as data source
   - Create a set of dashboard to monitor the following:
      - Cluster overview
         - count of pods running
         - count of pods failed
         - count of pods ready
         - ccount of pods not ready
         - node count
         - etc
      - Chart view of containers
         - resource utilization
         - pod resources
         - node resources over time

- Splunk
   - ingest logs from both webapp and k8s cluster
   - create custom dashboard for different components of the webapp
      - user activity
      - error rates
   - configure user roles and permissions, allow new users to search through log with minimum privileges.

---
## Architecture
### Source Code Management(SCM)
<div align="center">
   <img src="assets/scm.png" alt="Example Image" width="80%" />
</div>

- Source Code Management (SCM): GitHub
- CI/CD Automation Server: Connects to GitHub
- Build Docker Images: CI/CD server pushes images to Dockerhub
- Deployment: Kubernetes cluster pulls images from Dockerhub
- Nodes:
   - Node 1: Contains CI/CD Automation Server and Static Website
   - Node 2: Contains Monitoring and Logging tools
- Monitoring and Logging Tools:
   - Prometheus
   - Grafana
   - Splunk
---
## Features
### Webapp Deployment
- Capable of generating log messages using standard logging patterns and levels.
- A static web application designed to simulate production issues, such as high 5XX and 4XX errors, disruptions in Prometheus exporters, and failures in the Splunk forwarder service or sidecar containers.

### Jenkins (CI/CD Pipeline)
- Automates the build and deployment of the web application whenever changes are made to the source code.

### Splunk
- Capable of ingesting logs from both the web application and the Kubernetes cluster.
- Parses and extracts relevant fields from log entries in a structured manner, facilitating easier log analysis while ensuring sensitive data, such as credentials, is filtered out.
- Dashboard Features:
   - Accurately tracks user activity and error rates.
   - Allows configuration of user roles and permissions, enabling new users to search through logs with restricted privileges.

### Prometheus
- Collects data metrics related to the web application or service, providing insights into performance and resource usage.

### Alerting
- Provides notifications based on key performance indicators (KPIs), such as availability percentage, resource utilization, out-of-memory errors, and error rates.
- Allows the establishment of thresholds and the implementation of annotations for better monitoring.
  
### Grafana
- Provides a dashboard integrated with Prometheus to monitor real-time metrics.
- Accurately counts the number of nodes or pods, categorized by their various statuses.
- Displays charts that visualize container resource utilization, pod resources, and node resources over time.

---
## User Handbook
## Pre-requisites
- Installing Helm
   - Follow the instructions in the [Helm Website](https://helm.sh/docs/intro/install/) using Script or visit the link below:
```bash
https://helm.sh/docs/intro/install/
```

- Installing Docker
   - To install docker, you may use the commands below:
     ```bash
      $ docker use context
      $ docker context use default 
      ```

- Fix if "$docker ps" does not work, use the following:
```bash
$ sudo chmod 666 /var/run/docker.sock 
```

- To install java for jenkins to work, use the command below:
```bash
$ $ brew install java-17-openjdk -y
```

- If git isn't installed, use this command:
```bash
$ brew install git -y 
```



## Deploying Jenkins

### 1. Installing Jenkins in a Kubernetes Cluster
- Create a namespace:
```bash
$ kubectl create namespace jenkins
```
- Create a Service Account:
```bash
$ vi jenkins-serviceaccount.yaml
```
- Copy and paste the following:
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: jenkins-admin
rules:
  - apiGroups: [""]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins-admin
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: jenkins-admin
subjects:
  - kind: ServiceAccount
    name: jenkins-admin
    namespace: jenkins
```
- Then apply the Service Account:
```bash
$ kubectl apply -f jenkins-serviceaccount.yaml
```
- Create PVC for Persistent Jenkins Data:
```bash
$ vi jenkins-volumes.yaml
```
- Copy and paste the following:
```bash
apiVersion: v1 
kind: PersistentVolumeClaim 
metadata: 
  name: jenkins-pvc 
  namespace: jenkins 
spec: 
  storageClassName: linode-block-storage-retain 
  accessModes: 
    - ReadWriteOnce 
  resources: 
    requests: 
      storage: 10Gi 
```
- Then apply the PVC:
```bash
$ kubectl apply -f jenkins-volumes.yaml
```
- Create Deployment:
```bash
$ vi jenkins-deployment.yaml
```
- Copy and paste the following:
```bash
apiVersion: apps/v1 
kind: Deployment 
metadata: 
  name: jenkins 
  namespace: jenkins 
spec: 
  replicas: 1 
  selector: 
    matchLabels: 
      app: jenkins-server 
  template: 
    metadata: 
      labels: 
        app: jenkins-server 
    spec: 
      securityContext: 
            fsGroup: 1000 
            runAsUser: 1000 
      serviceAccountName: jenkins-admin 
      containers: 
        - name: jenkins 
          image: jenkins/jenkins:lts 
          resources: 
            limits: 
              memory: "2Gi" 
              cpu: "1000m" 
            requests: 
              memory: "500Mi" 
              cpu: "500m" 
          ports: 
            - name: httpport 
              containerPort: 8080 
            - name: jnlpport 
              containerPort: 50000 
          livenessProbe: 
            httpGet: 
              path: "/login" 
              port: 8080 
            initialDelaySeconds: 90 
            periodSeconds: 10 
            timeoutSeconds: 5 
            failureThreshold: 5 
          readinessProbe: 
            httpGet: 
              path: "/login" 
              port: 8080 
            initialDelaySeconds: 60 
            periodSeconds: 10 
            timeoutSeconds: 5 
            failureThreshold: 3 
          volumeMounts: 
            - name: jenkins-data 
              mountPath: /var/jenkins_home 
      volumes: 
        - name: jenkins-data 
          persistentVolumeClaim: 
              claimName: jenkins-pvc
```
- Then apply the deployment:
```bash
$ kubectl apply -f jenkins-deployment.yaml
```
- Create a service to expose Jenkins:
```bash
$ vi jenkins-service.yaml
```
- Copy and paste the following:
```bash
apiVersion: v1 
kind: Service 
metadata: 
  name: jenkins-service 
  namespace: jenkins 
  annotations: 
      prometheus.io/scrape: 'true' 
      prometheus.io/path:   / 
      prometheus.io/port:   '8080' 
spec: 
  selector: 
    app: jenkins-server 
  type: NodePort 
  ports: 
    - port: 8080 
      targetPort: 8080 
      nodePort: 30000 
```
- Then apply the service:
```bash
$ kubectl apply -f jenkins-service.yaml
```
- To Access the Jenkins UI
   - To view the initial password:
```bash
$ kubectl get pods -n jenkins 
$ kubectl logs <pod-name> -n jenkins
```
   - To view the IP Address
```bash
$ kubectl get nodes -o wide
```
   - To access via browser
```bash
<node-IP-Address>:30000
```

### 2. Set up for Auto-Build per Commit in Jenkins
- Creating Webhook
   - Go to Repository **Settings > Webhooks**
      - &lt;JenkinsLink&gt;/github-webhook/



## Deploying Webapp via Jenkins
- Install the following plugins:
   - Docker
      - Docker API Plugin
      - Docker Common Plugin
      - Docker Pipeline Plugin
      - Docker Plugin
- Create credentials:
<table>
  <thead>
    <tr>
      <th>What for</th>
      <th>Kind</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Github</td>
      <td>Docker Hub</td>
      <td>KubeConfig</td>
    </tr>
    <tr>
      <td>Username with Password</td>
      <td>Username with Password</td>
      <td>Secret file</td>
    </tr>
    <tr>
      <td>github-credential</td>
      <td>dockerhub-credential</td>
      <td>kubeconfig-credential</td>
    </tr>
  </tbody>
</table>

- Add Nodes:
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Type</th>
      <th>Labels</th>
      <th>Root Dir</th>
      <th>Launch Method</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Macbook</td>
      <td>permanent</td>
      <td>node01</td>
      <td>/Users/academy</td>
      <td>Launch agent by connecting it to the controller</td>
    </tr>
  </tbody>
</table>

   - To connect the node:
      - Click node
      - Copy and paste in specified node the command given (run from agent command line, with the secret stored in a file).
         - Please do note that it will run on foreground.
- Creating the Pipeline
   - Create Pipeline, name it “----”
   - Under Build Trigger, tick “Github hook trigger for GITScm polling”
   - Under Pipeline, select “Pipeline script from SCM”, and fill in the details:
      - SCM: Git
      - Repository
      - Credentials
      - Branch
- Run Build
- The Webapp should be depoyed on the node that is specified.

## Deploying Prometheus
#### Pre-requisites:
- Setting up Prometheus using Helm
   - Create a "monitoring" namespace
```bash
$ kubectl create namespace monitoring
```
   - Add Helm Repository Info
```bash
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
$ helm repo update 
```
   - Install Prometheus Operator
```bash
$ helm install prometheus-operator prometheus-community/kube-prometheus-stack -n monitoring 
```
   - Change Services to NodePort
```bash
$ kubectl edit service -n monitoring <service-name>
```

   - Change and add the following to the service yml file, use the command above:
      - Service name: prometheus-operator-kube-p-prometheus
         - type: NodePort
         - nodePort: 32090
       - Service name: prometheus-operator-kube-p-alertmanager
         - type: NodePort
         - nodePort: 32091
       - Service name: prometheus-operator-grafana
         - type: NodePort
         - nodePort: 32092
        
#### Accessing UIs on the browser
- Prometheus:
   - &lt;Node-IP-Adress&gt;:32090
- Alert Manager:
   - &lt;Node-IP-Adress&gt;:32091
- Grafana:
   - &lt;Node-IP-Adress&gt;:32092
 
#### For Scraping the Webapp Metrics
- Use the custom CRD of Service Monitor by applying the following yaml file configuration.
- Create a Service Monitor:
```bash
$ vi service-monitor.yaml
```
- Copy and paste the following:
```bash
apiVersion: monitoring.coreos.com/v1 
kind: ServiceMonitor 
metadata: 
  labels: 
    release: prometheus-operator 
  name: webapp 
  namespace: monitoring 
spec: 
  endpoints: 
  - path: /metrics 
    port: web 
    targetPort: 3000 
  namespaceSelector: 
    matchNames: 
    - webapp 
  selector: 
    matchLabels: 
      app: webapp-service 
```
- Then apply the Service Monitor:
```bash
$ kubectl apply -f service-monitor.yaml
```
- It will scape metrics in /metrics endpoint of the webapp.
---
## Deploying Splunk
#### Deploying Splunk Enterprise with Splunk Operator using Helm:
- Installing Splunk Operator using the [Splunk Documentation](https://splunk.github.io/splunk-operator/#installing-the-splunk-operator)
   - To start the Splunk Operator, run the command below and it would be created on a specific namespace:
```bash
   $ kubectl apply -f https://github.com/splunk/splunk-operator/releases/download/2.7.0/splunk-operator-namespace.yaml --server-side  --force-conflicts
```
- Creating a Splunk Enterprise Deployment
   - Creating a Standalone Splunk:
      - Open the text editor using the command below:
        ```bash
        $ vi splunk-standalone.yaml
        ```
      - Copy and paste the following:
         ```bash
         apiVersion: enterprise.splunk.com/v4
         kind: Standalone
         metadata:
           name: s1
           namespace: splunk-operator
           finalizers:
             - enterprise.splunk.com/delete-pvc
         spec:
           etcVolumeStorageConfig:
             storageCapacity: 10Gi
           varVolumeStorageConfig:
             storageCapacity: 30Gi
           serviceTemplate:
             spec:
               type: NodePort
           startupProbe:
             initialDelaySeconds: 300
             periodSeconds: 10
             failureThreshold: 30
           livenessInitialDelaySeconds: 400
           readinessInitialDelaySeconds: 390
         ```
      - Then apply the Splunk Standalone:
        ```bash
        $ kubectl apply -f splunk-standalone.yaml -n splunk-operator
        ```
     
- Changing Services to NodePort
   - To edit an existing service configuration file, use this command:
      ```bash
      $ kubectl edit service -n splunk-operator <service-name>
      ```
     - In service named "splunk-s1-standalone-service":
        - http-web:
       ```bash
           - type: NodePort
           - nodePort: 32093
       ```
        - http-hec:
       ```bash
           - type: NodePort
           - nodePort: 32094
       ```
   
      - To access the Splunk Web UI:
         - &lt;Node-IP-Adress&gt;:32093

- Modifying the Password
   - Getting the Current Password
      ```bash
      $ kubectl get secret -n splunk-operator 
      $ kubectl get secret <splunk-s1-standalone-secret-v1> -n splunk-operator -o yaml
      ```
     - Copy the decoded password’s value and paste it together with the command below:
      ```bash
      $ echo “<Decode Password’s Value>” | base64 -d
      ```

   - Changing the Password
      ```bash
      $ echo “<new password>” | base64
      $ kubectl get secret -n splunk-operator
      $ kubectl edit secret -n splunk-operator <splunk-splunk-operator-secret> 
      ```
     - Then change the password’s value with the new encoded password.
      ```bash
      $ kubectl get secret -n splunk-operator
      ```
  #### Note:
  - Make sure that the labels of PV and PVC matched.
  - Make sure they have the same access modes.

---
## Ingesting Data to Splunk
- Using Splunk OpenTelemetry Collector via Helm
   - Add Helm Repo
     ```bash
      $ helm repo add splunk-otel-collector-chart https://signalfx.github.io/splunk-otel-collector-chart 
      ```
   - Generate values.yaml renamed as otel.yaml:
     ```bash
      $ helm show values splunk-otel-collector-chart/splunk-otel-collector > otel.yaml 
      ```
   - Change/Add the Following to the yaml file:
     ```bash
      clusterName: legalloggers-cluster 
      endpoint: https://139.162.49.20:32094/services/collector/event 
      token:  
      index: "k8s_events" 
      metricsIndex: "k8s_metrics" 
      insecureSkipVerify: true 
      metricsEnabled: true 
      logsEngine: otel 
      containerRuntime: "containerd" 
      excludeAgentLogs: false 
      ```
   - Deploy
     ```bash
      $ helm -n otel install legalloggers-cluster -f otel.yaml splunk-otel-collector-chart/splunk-otel-collector  
      ```
---
## Creating Webapp Alerting Rules
- Create PrometheusRule manifest file
```bash
$ vi prometheus-webalerts.yaml  
```
- Then copy and Paste the following configuration:
```bash
apiVersion: monitoring.coreos.com/v1 
kind: PrometheusRule 
metadata: 
  labels: 
    release: prometheus-operator 
  name: webapp-alerts 
  namespace: monitoring 
spec: 
  groups: 
  - name: webapp.rules 
    rules: 
    - alert: HighDowntime 
      annotations: 
        description: The web app is unreachable for more than 1 minute. 
        summary: Web App is down 
      expr: up{pod=~"webapp.*"} == 0 
      for: 1m 
      labels: 
        severity: critical 
    - alert: HighCPUUsage 
      annotations: 
        description: The CPU usage of {{ $labels.pod }} is above 80%. 
        summary: High CPU Usage 
      expr: sum(rate(container_cpu_usage_seconds_total{pod=~"webapp.*"}[1m])) > 0.8 
      for: 5m 
      labels: 
        severity: warning 
    - alert: High5xxErrorRate 
      annotations: 
        description: The error rate of HTTP 5xx requests is high. 
        summary: High HTTP 5xx Error Rate 
      expr: rate(http_requests_total{status=~"5.*"}[5m]) > 5 
      for: 5m 
      labels: 
        severity: critical 
    - alert: HighMemoryUsage 
      annotations: 
        description: The memory usage of {{ $labels.pod }} is above 80%. 
        summary: High Memory Usage 
      expr: sum(container_memory_working_set_bytes{pod=~"webapp.*"}) / sum(node_memory_MemTotal_bytes) 
        > 0.8 
      for: 5m 
      labels: 
        severity: warning 
    - alert: High4xxErrorRate 
      annotations: 
        description: The error rate of HTTP 4xx requests is high. 
        summary: High HTTP 4xx Error Rate 
      expr: rate(http_requests_total{status=~"4.."}[5m]) > 5 
      for: 5m 
      labels: 
        severity: warning 
    - alert: TestAlert 
      annotations: 
        description: This is a test alert to verify Alertmanager. 
        summary: Test alert 
      expr: vector(1) 
      for: 30s 
      labels: 
        severity: critical 
```
- Apply the yml configuration
```bash
$ kubectl apply -f prometheus-webalerts.yaml  
```
#### Note:
- Make sure the rule is applied.
- Visit alerts tab on prometheus and verify if the rule is applied.

---

## Full Documentation
You may access the full documentation of [Observability: Monitoring & Logging](https://legalloggers-project.vercel.app/documentation)  project using the link below:
```bash
https://legalloggers-project.vercel.app/documentation
```



