# Lab 1: Infrastructure Foundations for NGINX for Azure

## Overview
Welcome to **App World 2026**. This lab is a core component of the session:
**"Mastering cloud-native app delivery: Unlocking advanced capabilities and use cases with F5’s ADCaaS."**

In this module, you will establish the core networking and identity foundations required to deploy NGINXaaS (NGINX for Azure). To ensure we maximize our time during the workshop, the base networking layer is treated as a pre-provisioned foundation, allowing us to focus on the advanced configuration and integration of F5's ADCaaS solutions.

## Objectives
* Understand the VNet and Subnet architecture required for NGINXaaS.
* Establish secure inbound traffic flow via Network Security Groups (NSGs).
* Configure Identity and Access Management (IAM) using Managed Identities.

---

## Part 1: Pre-provisioned Infrastructure
*The following components are deployed via automation prior to the workshop to provide a consistent starting environment.*

### 1.1 Azure Resource Group
All resources for this workshop are contained within a dedicated resource group.
* **Resource Group Name:** `AppWorld2026-ADCaaS-RG`
* **Region:** `[Workshop Region]`

### 1.2 Virtual Network (VNet) and Subnets
NGINX for Azure requires a delegated subnet to function as a native service.
* **VNet:** `adc-vnet` (10.1.0.0/16)
* **NGINX Subnet:** `nginx-delegated-subnet` (10.1.1.0/24) 
    * *Delegated to: `NGINX.NGINXPLUS/nginxproxies`*
* **Backend Subnet:** `workload-subnet` (10.1.2.0/24)

### 1.3 Network Security Group (NSG)
Security rules are applied to the subnets to ensure a "secure by default" posture:
* **Port 80/443:** Open for web traffic delivery.
* **Port 22:** Restricted for administrative access.

---

## Part 2: Managed Identity & Connectivity
*These steps ensure your ADCaaS instance can securely interact with Azure services (like Key Vault) and is accessible to the public.*

### 2.1 Create a Public IP Address
This static IP will serve as the entry point for your NGINX for Azure deployment.

```bash
az network public-ip create \
  --resource-group AppWorld2026-ADCaaS-RG \
  --name nginx-public-ip \
  --sku Standard \
  --allocation-method Static
