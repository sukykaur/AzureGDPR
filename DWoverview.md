# Azure Security and Compliance Blueprint: Data Warehouse for GDPR

## Overview
The General Data Protection Regulation (GDPR) is fundamentally about protecting and enabling the privacy rights of individuals.
The GDPR establishes strict global privacy requirements governing how organizations manage and protect personal data while respecting individual choice - no matter where data is sent, processed, or stored. Microsoft Azure services meet the stringent GDPR security requirements and Microsoft's contractual commitments guarantee that organizations can:
- Respond to requests to correct, amend or delete personal data.
- Detect and report personal data breaches.
- Demonstrate compliance with the GDPR.

This Azure Security and Compliance Blueprint provides guidance for how to deliver a Microsoft Azure data warehouse architecture that helps organizations identify and catalog personal data in systems, build more secure environments, and simplify management of GDPR compliance. This solution provides guidance on the deployment and configuration of Azure resources for a common reference architecture, demonstrating ways in which customers can meet specific security and compliance requirements and serves as a foundation for customers to build and configure their own data warehouse solutions in Azure.

This reference architecture, associated implementation guide, and threat model are intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment. Please note the following:
- The architecture provides a baseline to help customers deploy workloads to Azure in a GDPR-compliant manner.
- Customers are responsible for conducting appropriate security and compliance assessments of any solution built using this architecture, as requirements may vary based on the specifics of each customer's implementation.

## Architecture Diagram and Components
This solution provides a reference architecture which implements a high-performance and secure cloud data warehouse. There are two separate data tiers in this architecture: one where data is imported, stored, and staged within a clustered SQL environment, and another for the Azure SQL Data Warehouse where the data is loaded using an ETL tool (e.g. [PolyBase](https://docs.microsoft.com/azure/sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase) T-SQL queries) for processing. Once data is stored in Azure SQL Data Warehouse, analytics can run at a massive scale.

Microsoft Azure offers a variety of reporting and analytics services for the customer. This solution includes SQL Server Reporting Services (SSRS) for quick creation of reports from the Azure SQL Data Warehouse. All SQL traffic is encrypted with SSL through the inclusion of self-signed certificates. As a best practice, Azure recommends the use of a trusted certificate authority for enhanced security.

Data in the Azure SQL Data Warehouse is stored in relational tables with columnar storage, a format that significantly reduces the data storage costs while improving query performance.  Depending on usage requirements, Azure SQL Data Warehouse compute resources can be scaled up or down or shut off completely if there are no active processes requiring compute resources.

A SQL load balancer manages SQL traffic, ensuring high performance. All virtual machines in this reference architecture deploy in an availability set with SQL Server instances configured in an AlwaysOn availability group for high-availability and disaster-recovery capabilities.

This data warehouse reference architecture also includes an Active Directory (AD) tier for management of resources within the architecture. The Active Directory subnet enables easy adoption under a larger AD forest structure, allowing for continuous operation of the environment even when access to the larger forest is unavailable. All virtual machines are domain-joined to the Active Directory tier and use Active Directory group policies to enforce security and compliance configurations at the operating system level.

A virtual machine serves as a management bastion host, providing a secure connection for administrators to access deployed resources. The data loads into the staging area through this management bastion host. **Azure recommends configuring a VPN or Azure ExpressRoute connection for management and data import into the reference architecture subnet.**

This solution uses the following Azure services. Details of the deployment architecture are in the [Deployment Architecture](#deployment-architecture) section.

-	Azure Virtual Machines
    - (1) Bastion Host
    -	(2) Active Directory domain controller
    -	(2) SQL Server Cluster Node
    -	(1) SQL Server Witness
-	Availability Sets
    - Active Directory Domain Controllers
    - SQL Cluster Nodes and Witness
-	Virtual Network
    - (4) Subnets
    -	(4) Network Security Groups
- SQL Data Warehouse
- SQL Server Reporting Services
- Azure SQL Load Balancer
- Azure Active Directory
- Recovery Services Vault
- Azure Key Vault
- Operations Management Suite (OMS)
- Azure Data Catalog
- Azure Security Center

## Deployment Architecture
Microsoft Azure services help customers in their preparation for meeting GDPR requirements. Microsoft has developed a four-step process that customers can follow on their journey to GDPR compliance:
1. Discover: Identify which personal data exists and where it resides.
2. Manage: Govern how personal data is used and accessed.
3. Protect: Establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.
4. Report: Keep required documentation and manage data requests and breach notifications.

The following section details this reference architecture's development and implementation elements as they relate to each step.

### Discover

A critical step to addressing GDPR requirements is to identify all personal data managed by the organization and where it resides.

[Data Catalog](https://docs.microsoft.com/azure/data-catalog/data-catalog-what-is-data-catalog) makes data sources easily discoverable and understandable by the users who manage the data. Common data sources can be registered, tagged, and searched for personal data. The data remains in its existing location, but a copy of its metadata is added to Data Catalog, along with a reference to the data source location. The metadata is also indexed to make each data source easily discoverable via search and understandable to the users who discover it.

This solution deploys a [SQL Data Warehouse](https://docs.microsoft.com/azure/sql-data-warehouse/sql-data-warehouse-overview-what-is), which is an Enterprise Data Warehouse (EDW) that leverages Massively Parallel Processing (MPP) to quickly run complex queries across petabytes of data, allowing users to efficiently identify personal data. Users can use simple PolyBase T-SQL queries to import big data into the SQL Data Warehouse and utilize the power of MPP to run high-performance analytics. [SQL Server Reporting Services](https://docs.microsoft.com/en-us/sql/reporting-services/report-data/sql-azure-connection-type-ssrs) provides quick creation of reports with tables, charts, maps, gauges, matrixes, and more for Azure SQL Data Warehouse.

Azure Active Directory (AAD) enables administrators to search for user data, and then edit data associated with a user account.
Furthermore, Microsoft customers can correct inaccurate or incomplete data hosted in Azure SQL Database by using SQL queries.

### Manage
The goal of the second step is to govern how personal data is used and accessed within the organization.

Azure enables customers to export their data at any time, without seeking approval from Microsoft. For example, customers can export a virtual machine (VM) in a virtual hard disk (VHD) and AAD enables customers to export data associated with AAD accounts in a .csv file. Using SQL queries, Microsoft customers can identify and then export personal data hosted in Azure SQL Database. The [Extended Properties](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addextendedproperty-transact-sql) feature can be used to discontinue the processing of data subjects, as it allows users to add custom properties to database objects and tag data as "Discontinued" to support application logic to prevent the processing of associated personal data. [Row-Level Security](https://docs.microsoft.com/sql/relational-databases/security/row-level-security) enables users to define policies to restrict access to data to discontinue processing.

#### Identity Management
The following technologies provide capabilities to manage access to personal data in the Azure environment:
-	[Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) is Microsoft's multi-tenant cloud-based directory and identity management service. All users for this solution are created in AAD, including users accessing the SQL Database.
-	Authentication to the application is performed using AAD. For more information, see [Integrating applications with Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications). Additionally, the database column encryption uses AAD to authenticate the application to Azure SQL Database. For more information, see how to [protect sensitive data in SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault).
-	[Azure Role-based Access Control (RBAC)](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) enables administrators to define fine-grained access permissions to grant only the amount of access that users need to perform their jobs. Instead of giving every user unrestricted permissions for Azure resources, administrators can allow only certain actions for accessing personal data. Subscription access is limited to the subscription administrator.
- [AAD Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) enables customers to minimize the number of users who have access to certain information such as personal data.  Administrators can use AAD Privileged Identity Management to discover, restrict, and monitor privileged identities and their access to resources. This functionality can also be used to enforce on-demand, just-in-time administrative access when needed.

### Protect
The goal of the third step is to establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.

[AAD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) detects potential vulnerabilities affecting an organization’s identities, configures automated responses to detected suspicious actions related to an organization’s identities, and investigates suspicious incidents to take appropriate action to resolve them.

#### Virtual Network
This reference architecture defines a private virtual network with an address space of 10.0.0.0/16. Network Security Groups (NSGs), subnets, and a bastion host is used to limit the risk of unauthorized access, use, or disclosure of data.

**Network Security Groups**: [NSGs](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) contain Access Control Lists (ACLs) that allow or deny traffic within a VNet. NSGs can be used to secure traffic at a subnet or individual VM level. The following NSGs exist:
  -	An NSG for the Data Tier (SQL Server Clusters, SQL Server Witness, and SQL Load Balancer)
  -	An NSG for the management bastion host
  -	An NSG for Active Directory
  - An NSG for SQL Server Reporting Services

Each of the NSGs have specific ports and protocols open so that the solution can work securely and correctly. In addition, the following configurations are enabled for each NSG:
  -	[Diagnostic logs and events](https://docs.microsoft.com/azure/virtual-network/virtual-network-nsg-manage-log) are enabled and stored in a storage account
  -	OMS Log Analytics is connected to the [NSG's diagnostics](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)

**Subnets**: Each subnet is associated with its corresponding NSG.

**Bastion Host**: The bastion host is the single point of entry that allows users to access the deployed resources in this environment. The bastion host provides a secure connection to deployed resources by only allowing remote traffic from public IP addresses on a safe list. To permit remote desktop (RDP) traffic, the source of the traffic needs to be defined in the Network Security Group (NSG).

This solution creates a virtual machine as a domain-joined bastion host with the following configurations:
-	[Antimalware extension](https://docs.microsoft.com/azure/security/azure-security-antimalware)
-	[OMS extension](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-oms)
-	[Azure Diagnostics extension](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-extensions-diagnostics-template)
-	[Azure Disk Encryption](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) using Azure Key Vault
-	An [auto-shutdown policy](https://azure.microsoft.com/blog/announcing-auto-shutdown-for-vms-using-azure-resource-manager/) to reduce consumption of virtual machine resources when not in use
-	[Windows Defender Credential Guard](https://docs.microsoft.com/windows/access-protection/credential-guard/credential-guard) enabled so that credentials and other secrets run in a protected environment that is isolated from the running operating system

#### Secrets Management
The solution uses [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) for the management of keys and secrets. Azure Key Vault helps safeguard cryptographic keys and secrets used by cloud applications and services. The following Azure Key Vault capabilities help customers protect personal data and access to such data:
- Advanced access policies are configured on a need basis.
- Key Vault access policies are defined with minimum required permissions to keys and secrets.
- All keys and secrets in Key Vault have expiration dates.
- All keys in Key Vault are protected by specialized hardware security modules (HSMs). The key type is an HSM Protected 2048-bit RSA Key.
- All users and identities are granted minimum required permissions using RBAC.
- Diagnostics logs for Key Vault are enabled with a retention period of at least 365 days.
- Permitted cryptographic operations for keys are restricted to the ones required.

#### Data in Transit
Azure encrypts all communications to and from Azure datacenters by default. All SQL traffic in this reference architecture is encrypted with SSL through the inclusion of self-signed certificates. As a best practice, Azure recommends the use of a trusted certificate authority for enhanced security. Additionally, all transactions to Azure Storage through the Azure Portal occur via HTTPS.

For the confidentiality, integrity, and availability of information that is exchanged, including personal data, customers will need to configure ExpressRoute or secure VPN tunnel to securely establish a connection to the resources deployed as a part of this data warehouse reference architecture. Further details on both options are in the [Guidance and Recommendations](#-guidance-and-recommendations) section below.

#### Data at Rest

The architecture protects data at rest through multiple measures, including encryption and database auditing.

**Azure Storage**:
To meet encrypted data at rest requirements, all [Azure Storage](https://azure.microsoft.com/services/storage/) uses [Storage Service Encryption](https://docs.microsoft.com/azure/storage/storage-service-encryption). This helps protect and safeguard personal data in support of organizational security commitments and compliance requirements defined by the GDPR.

**Azure Disk Encryption**:
[Azure Disk Encryption](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) leverages the BitLocker feature of Windows to provide volume encryption for OS and data disks. The solution integrates with Azure Key Vault to help control and manage the disk-encryption keys.

**Azure SQL Database**:
The Azure SQL Database instance uses the following database security measures:
-	[AD Authentication and Authorization](https://docs.microsoft.com/azure/sql-database/sql-database-aad-authentication) enables identity management of database users and other Microsoft services in one central location.
-	[SQL database auditing](https://docs.microsoft.com/azure/sql-database/sql-database-auditing-get-started) tracks database events and writes them to an audit log in an Azure storage account.
-	SQL Database is configured to use [Transparent Data Encryption (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql), which performs real-time encryption and decryption of the database, associated backups, and transaction log files to protect information at rest. TDE provides assurance that stored personal data has not been subject to unauthorized access.
-	[Firewall rules](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure) prevent all access to database servers until proper permissions are granted. The firewall grants access to databases based on the originating IP address of each request.
-	[SQL Threat Detection](https://docs.microsoft.com/azure/sql-database/sql-database-threat-detection-get-started) enables the detection and response to potential threats as they occur by providing security alerts for suspicious database activities, potential vulnerabilities, SQL injection attacks, and anomalous database access patterns.
-	[Always Encrypted columns](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault) ensure that sensitive personal data never appears as plaintext inside the database system. After enabling data encryption, only client applications or app servers with access to the keys can access plaintext data.

After the reference architecture deploys, customers can use [SQL Database Dynamic Data Masking (DDM)](https://docs.microsoft.com/azure/sql-database/sql-database-dynamic-data-masking-get-started) to limit sensitive personal data exposure by masking the data to non-privileged users or applications. DDM allows the database administrator to select a particular table-column that contains sensitive personal data, add a mask to it (there are a few available built-in masks that can be applied, as well as a customizable mask), and designate which database users are privileged and should have access to the real data. Once configured, any query on that table or column will contain masked results, except for queries run by privileged users.

For users of Azure SQL Database, DDM can automatically discover potentially sensitive data and suggest the appropriate masks to be applied. This can help with the identification of personal data qualifying for GDPR protection, and for reducing access such that it does not exit the database via unauthorized access. **Note: Customers will need to adjust DDM settings to adhere to their database schema.**

#### Security

**Azure Security Center**: [Azure Security Center](https://docs.microsoft.com/en-us/azure/security-center/security-center-intro) enables customers to monitor traffic, collect logs, and analyze these data sources for threats. Additionally, Azure Security Center accesses existing configuration of Azure services to provide configuration and service recommendations to help improve security posture and protect personal data. For incidents in which Microsoft holds some or all of the responsibility to respond, Microsoft has established a detailed [Security Incident Response Management process specific to Azure](https://gallery.technet.microsoft.com/Azure-Security-Response-in-dd18c678).

**Malware Protection**: [Microsoft Antimalware](https://docs.microsoft.com/azure/security/azure-security-antimalware) for Virtual Machines provides real-time protection capability that helps identify and remove viruses, spyware, and other malicious software, with configurable alerts when known malicious or unwanted software attempts to install or run on protected virtual machines.

**Patch Management**: Windows virtual machines deployed as part of this reference architecture are configured by default to receive automatic updates from Windows Update Service. This solution also includes the OMS [Azure Automation](https://docs.microsoft.com/azure/automation/automation-intro) service through which updated deployments can be created to patch virtual machines when needed.

#### Business Continuity
**High Availability**: Server workloads are grouped in an [Availability Set](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-manage-availability?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) to help ensure high availability of virtual machines in Azure. At least one virtual machine is available during a planned or unplanned maintenance event, meeting the 99.95% Azure SLA.

**Recovery Services Vault**: The [Recovery Services Vault](https://docs.microsoft.com/azure/backup/backup-azure-recovery-services-vault-overview) houses backup data and protects all configurations of Azure Virtual Machines in this architecture. With a Recovery Services Vault, customers can restore files and folders from an IaaS VM without restoring the entire VM, enabling faster restore times.

### Report
The goal of the fourth and final step is to retain the required documentation and to manage data subject requests and breach notifications.

A key topic of the GDPR is data transfers in and out of the European Union (EU). This reference architecture can be deployed to a specific region or a national cloud to specify where data will be stored and to reduce the need for transfer of personal data outside of the EU. These choices include multiple regional choices within Europe as well as the German sovereign data storage region.

#### Logging and Auditing

Azure Security Center provides tools to enable customers to collect and review security logs from across Azure applications and services. Data collected by Azure Security Center can be aggregated in [Operations Management Suite (OMS)](https://docs.microsoft.com/en-us/azure/operations-management-suite/operations-management-suite-overview).

OMS provides extensive logging of system and user activity, as well as system health. The OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) solution collects and analyzes data generated by resources in Azure and on-premises environments.
- **Activity Logs**: [Activity logs](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) provide insight into operations performed on resources in a subscription. Activity logs can help determine who initiated an operation, when it occurred, and what the status of the operation was.
- **Diagnostic Logs**: [Diagnostic logs](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) include all logs emitted by every resource. These logs include Windows event system logs and Azure Blob storage, tables, and queue logs.
- **Log Archiving**: All diagnostic logs write to a centralized and encrypted Azure storage account for archival with a defined retention period of 2 days. These logs connect to Azure Log Analytics for processing, storing, and dashboard reporting.

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

## Threat Model

The data flow diagram (DFD) for this reference architecture is available for [download](https://aka.ms/blueprintdwthreatmodel) or can be found below. This model can help customers understand the points of potential risk in the system infrastructure when making modifications.

## Compliance Documentation
The Azure Security and Compliance Blueprint – GDPR Customer Responsibility Matrix lists controller and processor responsibilities for all GDPR articles. Please note that for Azure services, a customer is usually the controller and Microsoft acts as the processor.

The Azure Security and Compliance Blueprint - GDPR Data Warehouse Implementation Matrix provides information on which GDPR articles are addressed by the data warehouse architecture, including detailed descriptions of how the implementation meets the requirements of each covered article.

## Guidance and Recommendations
### VPN and ExpressRoute
A secure VPN tunnel or [ExpressRoute](https://docs.microsoft.com/azure/expressroute/expressroute-introduction) needs to be configured to securely establish a connection to the resources deployed as a part of this data warehouse reference architecture. By appropriately setting up a VPN or ExpressRoute, customers can add a layer of protection for data in transit.

By implementing a secure VPN tunnel with Azure, a virtual private connection between an on-premises network and an Azure Virtual Network can be created. This connection takes place over the Internet and allows customers to securely “tunnel” information inside an encrypted link between the customer's network and Azure. Site-to-Site VPN is a secure, mature technology that has been deployed by enterprises of all sizes for decades. The [IPsec tunnel mode](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc786385(v=ws.10)) is used in this option as an encryption mechanism.

Because traffic within the VPN tunnel does traverse the Internet with a site-to-site VPN, Microsoft offers another, even more secure connection option. Azure ExpressRoute is a dedicated WAN link between Azure and an on-premises location or an Exchange hosting provider. As ExpressRoute connections do not go over the Internet, these connections offer more reliability, faster speeds, lower latencies, and higher security than typical connections over the Internet. Furthermore, because this is a direct connection of customer's telecommunication provider, the data does not travel over the Internet and therefore is not exposed to it.

Best practices for implementing a secure hybrid network that extends an on-premises network to Azure are [available](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid).

### Extract-Transform-Load (ETL) Process
[PolyBase](https://docs.microsoft.com/sql/relational-databases/polybase/polybase-guide) can load data into Azure SQL Data Warehouse without the need for a separate ETL or import tool. PolyBase allows access to data through T-SQL queries. Microsoft's business intelligence and analysis stack, as well as third-party tools compatible with SQL Server, can be used with PolyBase.

### Azure Active Directory Setup
[Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-whatis) is essential to managing the deployment and provisioning access to personnel interacting with the environment. An existing Windows Server Active Directory can be integrated with AAD in [four clicks](https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-get-started-express). Customers can also tie the deployed Active Directory infrastructure (domain controllers) to an existing AAD by making the deployed Active Directory infrastructure a subdomain of an AAD forest.

### Additional Services
Azure Commercial offers a wide variety of services that handle formatted and unformatted data storage and staging to be used in data warehousing, including:
-	[Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction) is a managed cloud service that is built for complex hybrid extract-transform-load (ETL), extract-load-transform (ELT), and data integration projects. Azure Data Factory has capabilities to help trace and locate personal data, including visualization and monitoring tools to identify when data arrived and where it came from. Using Azure Data Factory, customers can create and schedule data-driven workflows called pipelines that ingest data from disparate data stores. Customers can then process and transform the data for output into data stores such as Azure SQL Data Warehouse.
-	[Azure Data Lake Store](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-overview) enables the capture of data of any size, type, and ingestion speed in a single place for operational and exploratory analytics. Azure Data Lake has capabilities that enable the extraction and conversion of data. Azure Data Lake Store is compatible with most open source components in the Hadoop ecosystem and integrates nicely with other Azure services such as Azure SQL Data Warehouse.

## Disclaimer

 - This document is for informational purposes only. MICROSOFT MAKES NO WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, AS TO THE INFORMATION IN THIS DOCUMENT. This document is provided "as-is." Information and views expressed in this document, including URL and other Internet website references, may change without notice. Customers reading this document bear the risk of using it.
 - This document does not provide customers with any legal rights to any intellectual property in any Microsoft product or solutions.
 - Customers may copy and use this document for internal reference purposes.
 - Certain recommendations in this document may result in increased data, network, or compute resource usage in Azure, and may increase a customer's Azure license or subscription costs.
 - This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment.
 - This document is developed as a reference and should not be used to define all means by which a customer can meet specific compliance requirements and regulations. Customers should seek legal support from their organization on approved customer implementations.
