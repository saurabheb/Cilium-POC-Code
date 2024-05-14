<!--
title: 'Networking And Security Solution for Kubernetes (Cilium) for Two Tier application'
description: 'This template demonstrates how to implement the K8s Cluster with Cilium'
layout: Doc
platform: AWS
-->

We can deploy the Cilium on Any K8s cluster as due to its wide support to identify the available cluster resources with version.In this repository you will get 2-3 way's to deploy the K8s Cluster as we have verfied the verified Cilium in various environment 


Please ref the below documentation to get the details of Networking And Security Solution for Kubernetes (Cilium) 

1. Introduction  

1.1. Basic introduction of Cilium  

 	Cilium is an open-source project to provide networking, security, and observability for cloud-native environments such as Kubernetes clusters and other container orchestration platforms. By leveraging the Power of eBPF(extended Berkeley Packet Filter) technology, Cilium offers unprecedented visibility, Security, and control over network traffic within Kubernetes clusters. 


1.2. Purpose of the Documentation. 

Documentation for Cilium is to provide guidance and information to the folks who are using or considering Cilium for networking and security in Kubernetes environments. 


1.3 Target Audience. 

            System Administrators. 
            Network Engineers. 
            DevOps/SRE Engineers. 
            Developers. 
            Technical Decision Makers. 

2. Deep Dive into the Cilium. 

2.1 Components of Cilium. 

              E-BPF based Datapath. 
              Cilium Agent. 
              Cilium CNI 
              Cilium Monitor (Hubble) 
              Cilium Identity Aware Proxy (CIIP) 

2.2. Key Features of Cilium: 

              Advanced Networking Policy Enforcement. 
              Network Visibility and Security. 
              Scalable and Performance. 
              Easy to integrate. 

 

2.3. Cilium Architecture. 

      Primary Component of Cilium - 

                    Cilium Agent 
                    Cilium Operator 
                    Cilium CLI 
                    CNI Plugin. 

 
2.4 Component Overview 

-- Cilium Agent: 

Cilium agent (cilium-agent) runs on each node in the cluster. At a high-level, the agent accepts configuration via Kubernetes or APIs that describes networking, service load-balancing, network policies, and visibility & monitoring requirements. 

The Cilium agent listens for events from orchestration systems such as Kubernetes to learn when containers or workloads are started and stopped. It manages the eBPF programs which the Linux kernel uses to control all network access in / out of those containers. 

-- Client (CLI)  

Cilium CLI client (cilium) is a command-line tool that is installed along with the Cilium agent. It interacts with the REST API of the Cilium agent running on the same node. The CLI allows inspecting the state and status of the local agent. It also provides tooling to directly access the eBPF maps to validate their state. 

--Cilium Operator 

Cilium Operator is responsible for managing duties in the cluster which should logically be handled once for the entire cluster, rather than once for each node in the cluster. The Cilium operator is not in the critical path for any forwarding or network policy decision. A cluster will generally continue to function if the operator is temporarily unavailable. However, depending on the configuration, failure in availability of the operator can lead to: 

Delays in IP Address Management (IPAM) and thus delay in scheduling of new workloads if the operator is required to allocate new IP addresses. 

 

--CNI Plugin 

CNI plugin (cilium-cni) is invoked by Kubernetes when a pod is scheduled or terminated on a node. It interacts with the Cilium API of the node to trigger the necessary datapath configuration to provide networking, load-balancing and network policies for the pod. 

 

--Hubble (Cilium Monitor) 

Graphical user interface (hubble-ui) utilizes relay-based visibility to provide a graphical service dependency and connectivity map. 

 
3. Network Policies. 

Cilium Network policy is a mechanism used to define and enforce rules that govern the flow of network traffic within a Kubernetes cluster. These policies are defined based on the labels assigned to Kubernetes objects such as pods, namespaces and services, allowing for granular control over network communication between these objects. 

Cilium network policies can specify rules to allow or deny traffic based on various criteria, including: 

Source and Destination Pods: Policies can define which pods are allowed to communicate with each other based on their labels. 
IP addresses and CIDR blocks: Policies can restrict traffic based on specific IP addresses or ranges. 
Ports and Protocols: Policies can specify which ports and protocols are allowed for communication. 
Application Layer Protocol Inspection: Cilium can perform deep packet inspection at the application layer to enforce policies based on the actual content of the traffic. 
Encryption and Security: Policies can enforce encryption requirement for specific types of traffic, ensuring that communication is secure. 

Cilium N/W policies are enforced using e-BPF(Extended-Berkeley Packet Filter) technology, which allows for efficient and low overhead enforcement of policies at the kernel level. This ensures that network policies are applied consistently and without impacting performance. 


4. Installation of Cilium. 

4.1. Cilium Requirement:  

https://docs.cilium.io/en/stable/operations/system_requirements/
 

4.2. Cilium Installation. 

Cilium will automatically detect and use the best configuration possible for the Kubernetes distribution we are using. All state is stored using Kubernetes custom resource definitions (CRDs).  

```
$cilium install 
```
Ref Document: https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/ 
following command to validate that your cluster has proper network connectivity: 

```
$cilium connectivity test 
```
 

5. Project for Testing. 

5.1 Default project. 

Cilium itself provides the one demo project for the testing of Cilium work/policies. 

In the demo project, we are not having UI available to show the communication as this application works on the HTTP POST and HTTP GET. 

We are creating the Policy in such way that only pod with tag org:empire will be able to communicate Deathstar application. 

```
apiVersion: "cilium.io/v2" 
kind: CiliumNetworkPolicy 
metadata: 
  name: "rule1" 
spec: 
  description: "L3-L4 policy to restrict deathstar access to empire ships only" 
  endpointSelector: 
    matchLabels: 
      org: empire 
      class: deathstar 
  ingress: 
  - fromEndpoints: 
    - matchLabels: 
        org: empire     #Only pods with tag org:empire will able to communicate 
    toPorts: 
    - ports: 
      - port: "80" 
        protocol: TCP 
```
 

Ref Link : https://docs.cilium.io/en/stable/gettingstarted/demo/ 


5.2. Two Tier Demo Project :  

We have created the one small project by scraping the web to showcase the demo of the Cilium policies. 
Ref GitHub Link : https://github.com/saurabheb/Demo-k8s-yaml 

In the Demo Project, we can add the tasks from the frontend UI, and it will get added into the backend MySQL database. 
We have deployed the 2 Frontend UI and one backend SQL database.  

Let’s consider that our deployment team mistakenly deployed the UAT changes by providing the Prod variable but still it's not going to impact anything over backend MYSQL as its only accepting the communication from Pod that having env:prod.  

```
Policy: 

apiVersion: "cilium.io/v2" 
kind: CiliumNetworkPolicy 
metadata: 
  name: "rule1" 
spec: 
  description: "L3-L4 policy to restrict deathstar access to empire ships only" 
  endpointSelector: 
    matchLabels: 
      app: mysql 
  ingress: 
  - fromEndpoints: 
    - matchLabels: 
        app: two-tier-app-prod 
    toPorts: 
    - ports: 
      - port: "3306" 
        protocol: TCP 

```
 

6.0 Challenges faced during setup 

Lack of Documentation available over internet: 

We have reviewed the available resources over internet on Cilium and started collecting the details and implemented it by failing multiple times. 
During the project tenure, we have received the good support from the Cilium Community on documentation and resolution of errors. 
