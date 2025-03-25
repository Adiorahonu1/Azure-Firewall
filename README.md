# 🔥 Azure Firewall (PJ-FW) Integration for Secure Network Traffic Filtering  

## **📌 Project Overview**  
This project demonstrates how **Azure Firewall (PJ-FW)** was integrated into a **mini IT infrastructure** for **centralized traffic filtering, network segmentation, and advanced security enforcement**.  

Azure Firewall acts as a **stateful, cloud-native security appliance**, allowing fine-grained control over inbound, outbound, and inter-subnet traffic.  

✅ **Key Security Features Implemented:**  
- **DNAT Rules for External Access** – SSH & Web access via the firewall.  
- **Network Traffic Filtering** – Controlled internal subnet communications.  
- **Threat Intelligence-Based Filtering** – Blocked malicious IPs & domains.  
- **Logging & Monitoring** – Integrated with **Azure Monitor & Log Analytics** for security insights.  

---

## **🔷 Azure Firewall Deployment Architecture**  

### **1️⃣ Virtual Network & Subnet Configuration**  
**To deploy Azure Firewall, a dedicated subnet is required (`AzureFirewall-Subnet`).**  

✅ **VNet Name:** `PJ-SEC-VNet`  
✅ **Address Space:** `10.0.0.0/16`  
✅ **Subnets:**  
- `Web-Subnet (10.0.1.0/24)` – Public-facing web server  
- `App-Subnet (10.0.2.0/24)` – Internal application server  
- `DB-Subnet (10.0.3.0/24)` – Secured database layer  
- `Jump-Subnet (10.0.4.0/24)` – Jump server for secure remote access  
- **`AzureFirewall-Subnet (10.0.5.0/24)` – Required for Azure Firewall deployment**  

📸 **Screenshot: Azure Firewall Subnet Configuration**  
<img src="assets/Screenshot 2025-03-24 at 08.09.59.png" alt="Azure Firewall Subnet Configuration" width="600">  

---

## **🔐 2️⃣ Azure Firewall (PJ-FW) Security Policies**  

### **🚦 DNAT Rules (Allow External Access to Specific Resources)**  

| **Rule Name** | **Source** | **Destination (Public IP)** | **Target (Internal IP)** | **Port/Protocol** | **Action** |
|--------------|-----------|----------------|----------------|----------------|---------|
| **allow-ssh** | Any | Azure Firewall Public IP | **Jump Server (10.0.4.10)** | TCP 22 (SSH) | **Allow** |
| **allow-web** | Any | Azure Firewall Public IP | **Web Server (10.0.1.10)** | TCP 80, 443 (HTTP, HTTPS) | **Allow** |

📸 **Screenshot: DNAT Rules Configuration allow-ssh**  
<img src="assets/Screenshot 2025-03-24 at 10.49.43.png" alt="Azure Firewall DNAT Rules" width="600">  

📸 **Screenshot: DNAT Rules Configuration allow-web**  
<img src="assets/Screenshot 2025-03-24 at 10.50.20.png" alt="Azure Firewall DNAT Rules" width="600">  

---


| **Rule Name**       | **Source**             | **Destination**          | **Port/Protocol**           | **Action**  |
|---------------------|----------------------|------------------------|--------------------------|-------------|
| **internaltraffic** | Web-Subnet, App-Subnet | App-Subnet, DB-Subnet  | TCP 5000, 8080, 3306, 5432 | **Allow**   |
| **icmp**      | Jump-Subnet, Web-Subnet, App-Subnet, DB-Subnet | Jump-Subnet, Web-Subnet, App-Subnet, DB-Subnet | ICMP (Ping) | **Allow**   |
| **dns**       | All Internal Subnets   | DNS Server (e.g., 10.0.0.10) | UDP 53 (DNS) | **Allow**   |

📸 **Screenshot: Network Rules Configuration**  
<img src="assets/Screenshot 2025-03-24 at 10.52.43.png" alt="Azure Firewall Network Rules" width="600">  

---

📸 **Screenshot: Resource Topology**  
<img src="assets/Screenshot 2025-03-25 at 08.04.51.png" alt="Resource Topology" width="600">  

## **📊 3️⃣ Azure Firewall Log Monitoring & Threat Detection**  

To enhance **threat visibility**, **Azure Firewall logs** were integrated with **Azure Monitor & Log Analytics**.  

### **🔹 Sample KQL Query to Detect Blocked Threats**  
```kql
AzureDiagnostics 
| where ResourceType == "AZUREFIREWALLS"
| where Action_s == "Deny"
| summarize BlockedAttempts=count() by SourceIP
| order by BlockedAttempts desc
