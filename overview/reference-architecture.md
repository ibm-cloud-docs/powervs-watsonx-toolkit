---

copyright:
   years: 2025, 2025
lastupdated: "2025-04-30"

keywords: Power Virtual Server, watsonx, Artificial Intelligence, AI

subcollection: powervs-watsonx-toolkit

# use-case from 'code' column in
# https://github.ibm.com/digital/taxonomy/blob/main/topics/topics_flat_list.csv
use-case: BankingAndFinanceIndustry

# industry from 'code' column in
# https://github.ibm.com/digital/taxonomy/blob/main/industries/industries_flat_list.csv
industry: Banking, FinancialSector, Insurance

# compliance from 'code' column in
# https://github.ibm.com/digital/taxonomy/blob/main/compliance_entities/compliance_entities_flat_list.csv
compliance: AIAct, IBMCloudFFS

content-type: reference-architecture

---

{{site.data.keyword.attribute-definition-list}}

# IBM {{site.data.keyword.powerSys_notm}} and IBM AI Cloud Services Integration
{: #powervs-watsonx-ra}
{: toc-content-type="reference-architecture"}
{: toc-version="1.0"}



This document provides reference architectures and design decisions for applications and services hosted on IBM {{site.data.keyword.powerSys_notm}} to integrate with IBM managed watsonx SaaS services or client managed watsonx software in IBM Cloud.


## Overview
{: #overview}

Even though IBM Power is designed for AI and has built-in features to accelerate AI, as discussed in the [Getting Started page](/docs/powervs-watsonx-toolkit?topic=powervs-watsonx-toolkit-getting-started), and one can certainly train AI models anywhere and deploy them on {{site.data.keyword.powerSys_notm}}, this article is not focused on running AI models on {{site.data.keyword.powerSys_notm}}. Rather, this article discusses how to integrate {{site.data.keyword.powerSys_notm}} and watsonx, and take advantages from both of them.

The diagram below highlights the high-level communication between {{site.data.keyword.powerSys_notm}} and watsonx services:
1. To extend the mission critical workloads with watsonx, the applications hosted on {{site.data.keyword.powerSys_notm}} can invoke inference APIs on watsonx service endpoint;
2. To allow cloud native AI applications or watsonx services to make use of the data or applications hosted on {{site.data.keyword.powerSys_notm}}, watsonx needs to access the private services hosted on {{site.data.keyword.powerSys_notm}}. The watsonx.governance service can also use similar path to monitor AI models hosted on {{site.data.keyword.powerSys_notm}} if needed.

![{{site.data.keyword.powerSys_notm}} and watsonx high-level communication](../images/overview-high-level.svg){: caption="{{site.data.keyword.powerSys_notm}} and watsonx high-level communication" caption-side="bottom"}


## Architecture diagram
{: #architecture-diagram}

For watsonx, client could choose to use the SaaS offering and create instances of the services they are interested in. In this case, IBM managed the watsonx stack and client consumes the corresponding cloud services.

There are also cases that clients would like to train or fine tune the AI models, and host the models themselves. In this case, client needs to set up their own stack and manage watsonx themselves.

We will discuss the reference architectures for both watsonx as SaaS and watsonx as software.

### IBM {{site.data.keyword.powerSys_notm}} and watsonx SaaS integration
{: #architecture-diagram-saas}

First, let's discuss {{site.data.keyword.powerSys_notm}} and watsonx SaaS integration. Let's take a look at two flavors of the reference architecture.

In this case, client has some workloads on {{site.data.keyword.powerSys_notm}}, which could be databases, applications, or even AI models hosted on {{site.data.keyword.powerSys_notm}}. If the client also has other cloud workloads in VPC, we would recommend multiple VPCs for separation of concerns between edge networking controls, provider management, and consumer workloads. Refer to [VPC reference architecture](https://cloud.ibm.com/docs/framework-financial-services?topic=framework-financial-services-vpc-architecture-about) in IBM Cloud Framework for Financial Services for recommended architecture and best practices.

![{{site.data.keyword.powerSys_notm}} and watsonx SaaS integration (with workloads in VPC and on {{site.data.keyword.powerSys_notm}})](../images/powervs-ai-SaaS.svg "{{site.data.keyword.powerSys_notm}} and watsonx SaaS integration (with workloads in VPC and on {{site.data.keyword.powerSys_notm}})"){: caption="{{site.data.keyword.powerSys_notm}} and watsonx SaaS integration (with workloads in VPC and on {{site.data.keyword.powerSys_notm}})" caption-side="bottom"}

IBM {{site.data.keyword.powerSys_notm}} and watsonx SaaS connectivity:

* Applications and services on IBM {{site.data.keyword.powerSys_notm}}s may need to use watsonx SaaS, for example, application running on {{site.data.keyword.powerSys_notm}} needs to invoke LLM hosted in watsonx.ai
  - 1.1 Direct Connectivity (NAT): {{site.data.keyword.powerSys_notm}} provides a built-in NAT service for direct connectivity to IBM Cloud Services. This option offers a maximum speed of 10Gbps.
  - 1.2 Virtual Private Endpoint (VPE): For IBM Cloud Services that has VPE enabled (eg. COS), you can create a VPE pointing directly to the Cloud Service. This method typically provides the highest throuput.
* IBM Cloud Services (eg. watsonx.data) needs to connect to databases or services (eg. DB2 or Oracle) hosted on {{site.data.keyword.powerSys_notm}}
  - 2.1 Satellite Connector enables private connectivity to {{site.data.keyword.powerSys_notm}} workloads. Satellite Connector and agent need to be configured and deployed.
  - 2.2 IBM Cloud Services connect to target services on {{site.data.keyword.powerSys_notm}} via secure private channel.

This architecture includes {{site.data.keyword.powerSys_notm}} Workspace and three VPCs, which provide segmentation for edge traffic control, management functionality, and consumer workloads.

If the client only has workloads on {{site.data.keyword.powerSys_notm}} and no other workloads in VPC, we would recommend to set up edge VPC to control traffic flows and do not expose {{site.data.keyword.powerSys_notm}} directly. The diagram below is a simplied version of the reference architecture, where network flow and bastion host can be set up in the edge VPC.

![{{site.data.keyword.powerSys_notm}} and watsonx SaaS integration (with workloads on {{site.data.keyword.powerSys_notm}})](../images/powervs-ai-SaaS-edge.svg "{{site.data.keyword.powerSys_notm}} and watsonx SaaS integration (with workloads on {{site.data.keyword.powerSys_notm}})"){: caption="{{site.data.keyword.powerSys_notm}} and watsonx SaaS integration (with workloads on {{site.data.keyword.powerSys_notm}})" caption-side="bottom"}

IBM provides different flavors of deployable architectures to automatically provision {{site.data.keyword.powerSys_notm}} environment. Refer to [Power Systems Virtual Server with VPC landing zone](https://cloud.ibm.com/docs/{{site.data.keyword.powerSys_notm}}-vpc?topic={{site.data.keyword.powerSys_notm}}-vpc-automation-solution-overview) for details.

You can also refer to [Gen AI pattern for watsonx on IBM Cloud](https://cloud.ibm.com/docs/pattern-genai-rag?topic=pattern-genai-rag-genai-pattern), which discusses Gen AI patterns without {{site.data.keyword.powerSys_notm}}.


### IBM {{site.data.keyword.powerSys_notm}} and watsonx software integration
{: #architecture-diagram-software}

If client chooses to deploy their own stack and have a single tenant watsonx to train, tune, or host their own models, they can deploy watsonx on OpenShift cluster at location of their choice. The reference architecture below shows the watsonx stack deployed in workload VPC in IBM Cloud.

![{{site.data.keyword.powerSys_notm}} and watsonx software integration](../images/powervs-ai-software.svg "{{site.data.keyword.powerSys_notm}} and watsonx software integration"){: caption="{{site.data.keyword.powerSys_notm}} and watsonx software integration" caption-side="bottom"}

IBM {{site.data.keyword.powerSys_notm}} and watsonx software connectivity:
* Client deploys watsonx software in client owned VPC
* Power Workspace and VPC are connected via Transit Gateway



## Design concepts
{: #design-concepts}

Following the [Architecture Framework](https://cloud.ibm.com/docs/architecture-framework?topic=architecture-framework-intro), this document covers the following solution aspects and domains:

![Architecture design scope](../images/powervs-ai-heatmap.svg "Architecture design scope"){: caption="Architecture design scope" caption-side="bottom"}



## Requirements
{: #requirements}

The following table outlines the requirements that are addressed in this architecture.

| Aspect | Requirements |
| -------------- | -------------- |
| Compute            | Provide properly isolated compute resources with adequate compute capacity for the applications. |
| Storage            | Provide storage that meets the application and database performance requirements. |
| Networking         | Deploy workloads in isolated environment and enforce information flow policies. \n Provide secure, encrypted connectivity to the cloud’s private network for management purposes. \n Distribute incoming application requests across available compute resources. |
| Security           | Ensure all operator actions are executed securely through a bastion host. \n Protect the boundaries of the application against denial-of-service and application-layer attacks. \n Encrypt all application data in transit and at rest to protect from unauthorized disclosure. \n Encrypt all security data (operational and audit logs) to protect from unauthorized disclosure. \n Encrypt all data using customer managed keys to meet regulatory compliance requirements for additional security and customer control. \n Protect secrets through their entire lifecycle and secure them using access control measures. \n Firewalls must be restrictively configured to prevent all traffic, both inbound and outbound, except that which is required, documented, and approved. |
| DevOps            | Delivering software and services at the speed the market demands requires teams to iterate and experiment rapidly. They must deploy new versions frequently, driven by feedback and data. |
| Resiliency         | Support application availability targets and business continuity policies. \n Ensure availability of the application in the event of planned and unplanned outages. \n Backup application data to enable recovery in the event of unplanned outages. \n Provide highly available storage for security data (logs) and backup data. |
| Service Management | Monitor system and application health metrics and logs to detect issues that might impact the availability of the application. \n Generate alerts/notifications about issues that might impact the availability of applications to trigger appropriate responses to minimize down time. \n Monitor audit logs to track changes and detect potential security problems. \n Provide a mechanism to identify and send notifications about issues found in audit logs. |
{: caption="Requirements" caption-side="bottom"}


## Components
{: #components}

The following table outlines the products or services used in the architecture for each aspect.

| Aspects | Architecture components | How the component is used |
| -------------- | -------------- | -------------- |
| Data | [watsonx.ai](https://www.ibm.com/products/watsonx-ai) | Brings together new generative AI capabilities powered by foundation models and traditional machine learning (ML) into a powerful studio spanning the AI lifecycle |
|  | [watsonx.data](https://www.ibm.com/products/watsonx-data) | Enables you to scale analytics and AI with all your data, wherever it resides |
|  | [watsonx.governance](https://www.ibm.com/products/watsonx-governance) | Direct, manage and monitor the artificial intelligence activities |
|  | [Watsonx Assistant](https://www.ibm.com/products/Watsonx-assistant) | Conversational artificial intelligence platform |
|  | [Watson Discovery](https://www.ibm.com/products/watson-discovery) | Automates the discovery of information and insights with advanced Natural Language Processing and Understanding |
|  | [watsonx Orchestrate](https://www.ibm.com/docs/en/watsonx/watson-orchestrate/current?topic=getting-started-watsonx-orchestrate) | A digital assistant and platform that uses automation to help businesses streamline processes and save time |
| Compute | [IBM {{site.data.keyword.powerSys_notm}}](https://cloud.ibm.com/docs/power-iaas?topic=power-iaas-getting-started) | Virtual Server offering on IBM Power systems |
|  | [Virtual Servers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-about-advanced-virtual-servers&interface=ui) | Web, App, and database servers |
| | [Code Engine](https://cloud.ibm.com/docs/codeengine?topic=codeengine-about) | Run your application, batch job, or container on a managed serverless platform |
| | [Red Hat OpenShift Kubernetes Service (ROKS)](https://cloud.ibm.com/docs/openshift?topic=openshift-getting-started) | A managed offering to create your own cluster of compute hosts where you can deploy and manage containerized apps on IBM Cloud |
| Storage | [Cloud Object Storage](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-about-cloud-object-storage) | Provide flexible, cost-effective, and scalable cloud storage for unstructured data |
|  | [IBM Cloud Block Storage for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-creating-block-storage&interface=ui) | Persistent storage for use as boot and data storage for Virtual Servers in a VPC network |
| Networking | [VPC Virtual Private Network (VPN)](https://cloud.ibm.com/docs/iaas-vpn?topic=iaas-vpn-getting-started) | Remote access to manage resources in private network |
|  | [Virtual Private Endpoint (VPE)](https://cloud.ibm.com/docs/vpc?topic=vpc-about-vpe) | For private network access to Cloud Services, e.g., Key Protect, COS, etc. |
|  | [VPC Load Balancers](https://cloud.ibm.com/docs/vpc?topic=vpc-load-balancers) | Application Load Balancing for web servers, app servers, and database servers |
|  | [Direct Link 2.0](https://cloud.ibm.com/docs/dl?topic=dl-get-started-with-ibm-cloud-dl) | Seamlessly connect on-premises resources to cloud resources |
|  | [Transit Gateway (TGW)](https://cloud.ibm.com/docs/transit-gateway?topic=transit-gateway-getting-started) | Connects the Workload and Management VPCs within a region
|  | [Cloud Internet Services (CIS)](https://cloud.ibm.com/docs/cis?topic=cis-getting-started) | Global load balancing between regions |
|  | [Access Control List (ACL)](https://cloud.ibm.com/docs/vpc?topic=vpc-using-acls) | To control all incoming and outgoing traffic in Virtual Private Cloud |
| Security | [IAM](https://cloud.ibm.com/docs/account?topic=account-cloudaccess) | IBM Cloud Identity & Access Management |
|  | [Key Protect](https://cloud.ibm.com/docs/key-protect?topic=key-protect-about) | A full-service encryption solution that allows data to be secured and stored in IBM Cloud |
|  | [BYO Bastion Host on VPC VSI](https://cloud.ibm.com/docs/framework-financial-services?topic=framework-financial-services-vpc-architecture-connectivity-bastion-tutorial-teleport) | Remote access with Privileged Access Management |
|  | [App ID](https://cloud.ibm.com/docs/appid?topic=appid-getting-started) | Add authentication to web and mobile apps |
|  | [Secrets Manager](https://cloud.ibm.com/docs/secrets-manager?topic=secrets-manager-getting-started#getting-started) | Certificate and Secrets Management |
|  | [Security and Compliance Center (SCC)](https://cloud.ibm.com/docs/security-compliance?topic=security-compliance-getting-started) | Implement controls for secure data and workload deployments, and assess security and compliance posture |
|  | [Hyper Protect Crypto Services (HPCS)](https://cloud.ibm.com/docs/hs-crypto?topic=hs-crypto-get-started) | Hardware security module (HSM) and Key Management Service |
|  | [Virtual Network Function (VNF)](https://cloud.ibm.com/docs/vpc?topic=vpc-deploy-vnf) | Virtualized network services running on virtual machines. |
|  | [Event Notifications](https://cloud.ibm.com/docs/event-notifications?topic=event-notifications-getting-started) | Get notified about critical events that occur in your IBM Cloud account. |
| DevOps | [Continuous Integration (CI)](https://cloud.ibm.com/docs/containers?topic=containers-cicd) | 	A pipeline that tests, scans and builds the deployable artifacts from the application repositories |
|  | [Continuous Deployment (CD)](https://cloud.ibm.com/docs/ContinuousDelivery?topic=ContinuousDelivery-getting-started) | A pipeline that generates all of the evidence and change request summary content |
|  | [Continuous Compliance (CC)](https://cloud.ibm.com/docs/devsecops?topic=devsecops-tutorial-cc-toolchain) | A pipeline that continuously scans deployed artifacts and repositories |
|  | [Container Registry](https://cloud.ibm.com/apidocs/container-registry) | Highly available, and scalable private image registry |
| Resiliency | 	[VPC VSIs, VPC Block across multiple zones in two regions](https://cloud.ibm.com/docs/solution-tutorials?topic=solution-tutorials-vpc-multi-region) | Web, app, database high availability and disaster recovery |
| Service Management | [IBM Cloud Monitoring](https://cloud.ibm.com/docs/monitoring?topic=monitoring-about-monitor) | Apps and operational monitoring |
|  | [IBM Cloud Logs](https://cloud.ibm.com/docs/cloud-logs?topic=cloud-logs-getting-started) | Scalable logging service that persists logs and provides users with capabilities for querying, tailing, and visualizing logs |
{: caption="Components" caption-side="bottom"}
