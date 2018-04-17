# Azure Security and Compliance Blueprint - Web Application for GDPR

## Overview

## Architecture Diagram and Components

This solution deploys a reference architecture for an GDPR web application with a database backend. The architecture includes a web tier, data tier, Active Directory infrastructure, application gateway, and load balancer. Virtual machines deployed to the web and data tiers are configured in an availability set, and SQL Server instances are configured in an AlwaysOn availability group for high availability. Virtual machines are domain-joined, and Active Directory group policies are used to enforce security and compliance configurations at the operating system level. A management jumpbox (bastion host) provides a secure connection for administrators to access deployed resources.

![alt text](images/fedramp-architectural-diagram.png?raw=true "Azure Security and Compliance Blueprint - FedRAMP Web Applications Automation")

This solution uses the following Azure services. Details of the deployment architecture are located in the [deployment architecture](#deployment-architecture) section.

- Azure Virtual Machines
	- (1) Management/bastion (Windows Server 2016 Datacenter)
	- (2) Active Directory domain controller (Windows Server 2016 Datacenter)
	- (2) SQL Server cluster node (SQL Server 2016 on Windows Server 2012 R2)
	- (1) SQL Server witness (Windows Server 2016 Datacenter)
	- (2) Web/IIS (Windows Server 2016 Datacenter)
- Availability Sets
	- (1) Active Directory domain controllers
	- (1) SQL cluster nodes and witness
	- (1) Web/IIS
- Azure Virtual Network
	- (1) /16 virtual networks
	- (5) /24 subnets
	- DNS settings are set to both domain controllers
- Azure SQL Load Balancer
- Azure Application Gateway
	- (1) WAF Application Gateway enabled
		- Firewall Mode: Prevention
		- Rule set: OWASP 3.0
		- Listener: Port 443
- Azure Storage
	- (7) Geo-redundant storage accounts
- Recovery Services vault
- Azure Key Vault
- Azure Active Directory (AAD)
- Azure Resource Manager
- Operations Management Suite (OMS)

## Deployment Architecture
Microsoft Azure services help customers in their preparation for meeting GDPR requirements. Microsoft has developed a four-step process that customers can follow on their journey to GDPR compliance:
1. Discover: Identify which personal data exists and where it resides.
2. Manage: Govern how personal data is used and accessed.
3. Protect: Establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.
4. Report: Keep required documentation and manage data requests and breach notifications.

The following section details this reference architecture's development and implementation elements as they relate to each step.

### Discover
A critical step to addressing GDPR requirements is to identify all personal data managed by the organization and where it resides.

### Manage
The goal of the second step is to govern how personal data is used and accessed within the organization.

#### Identity Management
The following technologies provide capabilities to manage access to personal data in the Azure environment:
- [Azure Active Directory (Azure AD)](https://azure.microsoft.com/services/active-directory/) is Microsoft's multi-tenant cloud-based directory and identity management service.
- Authentication to a customer-deployed web application can be performed using Azure AD. For more information, see [Integrating applications with Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications).  
- [Azure Role-Based Access Control (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal) enables precisely focused access management for Azure. Subscription access is limited to the subscription administrator, and access to resources can be limited based on user role.
- A deployed IaaS Active Directory instance provides identity management at the OS-level for deployed IaaS virtual machines.

### Protect
The goal of the third step is to establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.

#### Application Gateway

The architecture reduces the risk of security vulnerabilities using an Application Gateway with web application firewall (WAF), and the OWASP ruleset enabled. Additional capabilities include:

-[End-to-End-SSL](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- Enable [SSL Offload](https://docs.microsoft.com/azure/application-gateway/application-gateway-ssl-portal)
- Disable [TLS v1.0 and v1.1](https://docs.microsoft.com/azure/application-gateway/application-gateway-end-to-end-ssl-powershell)
- [Web application firewall](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-overview) (WAF mode)
- [Prevention mode](https://docs.microsoft.com/azure/application-gateway/application-gateway-web-application-firewall-portal) with OWASP 3.0 ruleset

#### Virtual Network
The architecture defines a private virtual network with an address space of 10.200.0.0/16.

**Network Security Groups**: This solution deploys resources in an architecture with a separate web subnet, database subnet, Active Directory subnet, and management subnet inside of a virtual network. Subnets are logically separated by network security group rules applied to the individual subnets to restrict traffic between subnets to only that necessary for system and management functionality.

See the configuration for [Network Security Groups](https://github.com/Azure/fedramp-iaas-webapp/blob/master/nestedtemplates/virtualNetworkNSG.json) deployed with this solution. Organizations can configure Network Security groups by editing the file above using [this documentation](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) as a guide.

Each of the subnets has a dedicated network security group (NSG):
- 1 NSG for Application Gateway (LBNSG)
- 1 NSG for Jumpbox (MGTNSG)
- 1 NSG for Primary and Backup Domain Controllers (ADNSG)
- 1 NSG for SQL Servers and File Share Witness (SQLNSG)
- 1 NSG for Web Tier (WEBNSG)

**Bastion Host**: A management jumpbox (bastion host) provides a secure connection for administrators to access deployed resources. The NSG associated with the management subnet where the jumpbox virtual machine is located allows connections only on TCP port 3389 for RDP.

#### Secrets Management
The solution uses [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) to manage keys and secrets.

-  helps safeguard cryptographic keys and secrets used by cloud applications and services.
- The solution is integrated with Azure Key Vault to manage IaaS virtual machine disk-encryption keys and secrets.

#### Data in Transit
Azure encrypts all communications to and from Azure datacenters by default. Additionally, all transactions to Azure Storage through the Azure Portal occur via HTTPS.

#### Data at Rest
The architecture protects data at rest by using several encryption measures.

**Azure Storage**: To meet data-at-rest encryption requirements, all storage accounts use [Storage Service Encryption](https://docs.microsoft.com/azure/storage/common/storage-service-encryption).

**SQL Database**: SQL Database is configured to use [Transparent Data Encryption (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption), which performs real-time encryption and decryption of data and log files to protect information at rest. TDE provides assurance that stored data has not been subject to unauthorized access.

**Azure Disk Encryption**: Azure Disk Encryption is used to encrypted Windows IaaS virtual machine disks. [Azure Disk Encryption](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) leverages the BitLocker feature of Windows to provide volume encryption for OS and data disks. The solution is integrated with Azure Key Vault to help control and manage the disk-encryption keys.

#### Security

**Malware Protection**: [Microsoft Antimalware](https://docs.microsoft.com/azure/security/azure-security-antimalware) for Virtual Machines provides real-time protection capability that helps identify and remove viruses, spyware, and other malicious software, with configurable alerts when known malicious or unwanted software attempts to install or run on protected virtual machines.

**Patch Management**: Windows virtual machines deployed by this Azure Security and Compliance Blueprint Automation are configured by default to receive automatic updates from Windows Update Service. This solution also deploys the Azure Automation solution through which Update Deployments can be created to deploy patches to Windows servers when needed.

#### Business Continuity

**Web Tier**: The solution deploys web tier virtual machines in an [Availability Set](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets). Availability sets ensure that the virtual machines are distributed across multiple isolated hardware clusters to improve availability.

**Database Tier**: The solution deploys database tier virtual machines in an Availability Set as an [AlwaysOn availability group](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-availability-group-overview). The Always On availability group feature provides for high-availability and disaster-recovery capabilities.

**Active Directory**: All virtual machines deployed by the solution are domain-joined, and Active Directory group policies are used to enforce security and compliance configurations at the operating system level. Active Directory virtual machines are deployed in an Availability Set.

### Report
The goal of the fourth and final step is to retain the required documentation and to manage data subject requests and breach notifications.

A key topic of the GDPR is data transfers in and out of the European Union (EU). This reference architecture can be deployed to a specific region or a national cloud to specify where data will be stored and to reduce the need for transfer of personal data outside of the EU. These choices include multiple regional choices within Europe as well as the German sovereign data storage region.

#### Logging and Auditing

OMS provides extensive logging of system and user activity, as well as system health. The OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) solution collects and analyzes data generated by resources in Azure and on-premises environments.
- **Activity Logs**: [Activity logs](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) provide insight into operations performed on resources in a subscription. Activity logs can help determine who initiated an operation, when it occurred, and what the status of the operation was.
- **Diagnostic Logs**: [Diagnostic logs](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) include all logs emitted by every resource. These logs include Windows event system logs, Azure storage logs, Key Vault audit logs, and Application Gateway access and firewall logs.
- **Log Archiving**: All diagnostic logs write to a centralized and encrypted Azure storage account for archival. The retention is user-configurable, up to 730 days, to meet organization-specific retention requirements. These logs connect to Azure Log Analytics for processing, storing, and dashboard reporting.

Additionally, the following OMS solutions are included as a part of this architecture:
-	[AD Assessment](https://docs.microsoft.com/azure/log-analytics/log-analytics-ad-assessment): The Active Directory Health Check solution assesses the risk and health of server environments on a regular interval and provides a prioritized list of recommendations specific to the deployed server infrastructure.
-	[Antimalware Assessment](https://docs.microsoft.com/azure/log-analytics/log-analytics-malware): The Antimalware solution reports on malware, threats, and protection status.
-	[Azure Automation](https://docs.microsoft.com/azure/automation/automation-hybrid-runbook-worker): The Azure Automation solution stores, runs, and manages runbooks.
-	[Security and Audit](https://docs.microsoft.com/azure/operations-management-suite/oms-security-getting-started): The Security and Audit dashboard provides a high-level insight into the security state of resources by providing metrics on security domains, notable issues, detections, threat intelligence, and common security queries.
-	[SQL Assessment](https://docs.microsoft.com/azure/log-analytics/log-analytics-sql-assessment): The SQL Health Check solution assesses the risk and health of server environments on a regular interval and provides customers with a prioritized list of recommendations specific to the deployed server infrastructure.
-	[Update Management](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-update-management): The Update Management solution allows customer management of operating system security updates, including a status of available updates and the process of installing required updates.
-	[Agent Health](https://docs.microsoft.com/azure/operations-management-suite/oms-solution-agenthealth): The Agent Health solution reports how many agents are deployed and their geographic distribution, as well as how many agents which are unresponsive and the number of agents which are submitting operational data.
-	[Azure Activity Logs](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity): The Activity Log Analytics solution assists with analysis of the Azure activity logs across all Azure subscriptions for a customer.
-	[Change Tracking](https://docs.microsoft.com/azure/log-analytics/log-analytics-activity): The Change Tracking solution allows customers to easily identify changes in the environment.

## Compliance Documentation

The Azure Security and Compliance Blueprint - GDPR Customer Responsibility Matrix lists controller and processor responsibilities for all GDPR articles. Please note that for Azure services, a customer is usually the controller and Microsoft acts as the processor.

The Azure Security and Compliance Blueprint - GDPR IaaS WebApp Implementation Matrix provides information on which GDPR articles are addressed by the IaaS WebApp architecture, including detailed descriptions of how the implementation meets the requirements of each covered article.

## Disclaimer

- This document is for informational purposes only. MICROSOFT MAKES NO WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, AS TO THE INFORMATION IN THIS DOCUMENT. This document is provided "as-is." Information and views expressed in this document, including URL and other Internet website references, may change without notice. Customers reading this document bear the risk of using it.  
- This document does not provide customers with any legal rights to any intellectual property in any Microsoft product or solutions.  
- Customers may copy and use this document for internal reference purposes.  
- Certain recommendations in this document may result in increased data, network, or compute resource usage in Azure, and may increase a customer's Azure license or subscription costs.  
- This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment.
- This document is developed as a reference and should not be used to define all means by which a customer can meet specific compliance requirements and regulations. Customers should seek legal support from their organization on approved customer implementations.
