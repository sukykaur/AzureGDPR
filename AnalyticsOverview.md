# Azure Security and Compliance Blueprint: Analytics for GDPR

## Overview
The General Data Protection Regulation (GDPR) is fundamentally about protecting and enabling the privacy rights of individuals.
The GDPR establishes strict global privacy requirements governing how organizations manage and protect personal data while respecting individual choice - no matter where data is sent, processed, or stored.

Microsoft Azure services meet the stringent GDPR security requirements. Microsoft's [contractual terms](http://aka.ms/Online-Services-Terms) commit Microsoft to the requirements on processors in GDPR Article 28 and other Articles of GDPR. These commitments guarantee that organizations can:
- Respond to requests to correct, amend or delete personal data.
- Detect and report personal data breaches.
- Demonstrate compliance with the GDPR.

This Azure Security and Compliance Blueprint provides guidance for how to deliver a Microsoft Azure analytics architecture that helps organizations identify and catalog personal data in systems, build more secure environments, and simplify management of GDPR compliance. This solution provides guidance on the deployment and configuration of Azure resources for a common reference architecture, demonstrating ways in which customers can meet specific security and compliance requirements and serves as a foundation for customers to build and configure their own analytics solutions in Azure.

This reference architecture, associated implementation guide, and threat model are intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment. Please note the following:
- The architecture provides a baseline to help customers deploy workloads to Azure in a GDPR-compliant manner.
- Customers are responsible for conducting appropriate security and compliance assessments of any solution built using this architecture, as requirements may vary based on the specifics of each customer's implementation.

## Architecture Diagram and Components
This solution uses the following Azure services. Details of the deployment architecture are in the [Deployment Architecture](#deployment-architecture) section.

- Azure Functions
- Azure SQL Database
- Azure Machine Learning Services
- Azure Active Directory
- Azure Key Vault
- OMS
- Azure Monitor
- Azure Storage
- ExpressRoute/VPN Gateway
- Power BI Dashboard

## Deployment Architecture
Microsoft Azure services help customers in their preparation for meeting GDPR requirements. Microsoft has developed a four-step process that customers can follow on their journey to GDPR compliance:
1. Discover: Identify which personal data exists and where it resides.
2. Manage: Govern how personal data is used and accessed.
3. Protect: Establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.
4. Report: Keep required documentation and manage data requests and breach notifications.

The following section details this reference architecture's development and implementation elements as they relate to each step

### **Discover**
A critical step to addressing GDPR requirements is to identify all personal data managed by the organization and where it resides.

[Data Catalog](https://docs.microsoft.com/azure/data-catalog/data-catalog-what-is-data-catalog) makes data sources easily discoverable and understandable by the users who manage the data. Common data sources can be registered, tagged, and searched for personal data. The data remains in its existing location, but a copy of its metadata is added to Data Catalog, along with a reference to the data source location. The metadata is also indexed to make each data source easily discoverable via search and understandable to the users who discover it.

[Azure SQL Database](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-technical-overview) is a general-purpose relational database managed service in Microsoft Azure that supports structures such as relational data, JSON, spatial, and XML. SQL Database offers managed single SQL databases, managed SQL databases in an elastic pool, and SQL Managed Instances (in public preview). It delivers dynamically scalable performance and provides options such as columnstore indexes for extreme analytic analysis and reporting, and in-memory OLTP for extreme transactional processing. Microsoft handles all patching and updating of the SQL code base seamlessly and abstracts away all management of the underlying infrastructure.

Azure Active Directory (AAD) enables administrators to search for user data, and then edit data associated with a user account.
Using SQL queries, Microsoft customers can correct inaccurate or incomplete data hosted in Azure SQL Database.

### Manage
The goal of the second step is to govern how personal data is used and accessed within the organization.

Azure enables you to export your data at any time, without seeking approval from Microsoft. Azure Active Directory (AAD) enables you to export data associated with AAD accounts in a .csv file. Using SQL queries, Microsoft customers can identify and then export personal data hosted in Azure SQL Database. The [Extended Properties](https://docs.microsoft.com/sql/relational-databases/system-stored-procedures/sp-addextendedproperty-transact-sql) feature can be used to discontinue the processing of data subjects, as it allows users to add custom properties to database objects and tag data as "Discontinued" to support application logic to prevent the processing of associated personal data. [Row-Level Security](https://docs.microsoft.com/sql/relational-databases/security/row-level-security) enables users to define policies to restrict access to data to discontinue processing.

#### Identity Management
The following technologies provide capabilities to manage access to personal data in the Azure environment:
-	[Azure Active Directory (AAD)](https://azure.microsoft.com/services/active-directory/) is Microsoft's multi-tenant cloud-based directory and identity management service. All users for this solution are created in AAD, including users accessing the SQL Database.
-	Authentication to the application is performed using AAD. For more information, see [Integrating applications with Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications). Additionally, the database column encryption uses AAD to authenticate the application to Azure SQL Database. For more information, see how to [protect sensitive data in SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-always-encrypted-azure-key-vault).
-	[Azure Role-based Access Control (RBAC)](https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure) enables administrators to define fine-grained access permissions to grant only the amount of access that users need to perform their jobs. Instead of giving every user unrestricted permissions for Azure resources, administrators can allow only certain actions for accessing personal data. Subscription access is limited to the subscription administrator.
- [AAD Privileged Identity Management](https://docs.microsoft.com/azure/active-directory/active-directory-privileged-identity-management-getting-started) enables customers to minimize the number of users who have access to certain information such as personal data.  Administrators can use AAD Privileged Identity Management to discover, restrict, and monitor privileged identities and their access to resources. This functionality can also be used to enforce on-demand, just-in-time administrative access when needed.

### **Protect**
The goal of the third step is to establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.

-	[AAD Identity Protection](https://docs.microsoft.com/azure/active-directory/active-directory-identityprotection) detects potential vulnerabilities affecting an organization’s identities, configures automated responses to detected suspicious actions related to an organization’s identities, and investigates suspicious incidents to take appropriate action to resolve them.

#### **Virtual Network**
This reference architecture defines a private virtual network with an address space of 10.0.0.0/16.

**Network Security Groups**: [NSGs](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) contain Access Control Lists (ACLs) that allow or deny traffic within a VNet. NSGs can be used to secure traffic at a subnet or individual VM level. The following NSGs exist:
  -	An NSG for Active Directory
  - An NSG for the Workload

Each of the NSGs have specific ports and protocols open so that the solution can work securely and correctly. In addition, the following configurations are enabled for each NSG:
  -	[Diagnostic logs and events](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-nsg-manage-log) are enabled and stored in a storage account
  -	OMS Log Analytics is connected to the [NSG's diagnostics](https://github.com/krnese/AzureDeploy/blob/master/AzureMgmt/AzureMonitor/nsgWithDiagnostics.json)

**Subnets**: Each subnet is associated with its corresponding NSG.

#### Secrets Management
The solution uses [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) for the management of keys and secrets. Azure Key Vault helps safeguard cryptographic keys and secrets used by cloud applications and services. The following Azure Key Vault capabilities help customers protect personal data and access to such data:
- Advanced access policies are configured on a need basis.
- Key Vault access policies are defined with minimum required permissions to keys and secrets.
- All keys and secrets in Key Vault have expiration dates.
- All keys in Key Vault are protected by specialized hardware security modules (HSMs). The key type is an HSM Protected 2048-bit RSA Key.
- All users and identities are granted minimum required permissions using RBAC.
- Diagnostics logs for Key Vault are enabled with a retention period of at least 365 days.
- Permitted cryptographic operations for keys are restricted to the ones required.

#### **Data in Transit**
Azure encrypts all communications to and from Azure datacenters by default. All transactions to Azure Storage through the Azure Portal occur via HTTPS.

#### **Data at Rest**

The architecture protects data at rest through encryption, database auditing, and other measures.

**Azure Storage**
To meet encrypted data at rest requirements, all [Azure Storage](https://azure.microsoft.com/services/storage/) uses [Storage Service Encryption](https://docs.microsoft.com/en-us/azure/storage/storage-service-encryption).

**Azure Disk Encryption**
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
**Azure Security Center**: Azure Security Center enables customers to monitor traffic, collect logs, and analyze these data sources for threats. For incidents in which Microsoft holds some or all of the responsibility to respond, Microsoft has established a detailed [Security Incident Response Management process specific to Azure](https://gallery.technet.microsoft.com/Azure-Security-Response-in-dd18c678).

### **Report**
**Keep required documentation and manage data requests and breach notifications.**

**Azure Monitor**
[Azure Monitor](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/) helps you track performance, maintain security, and identify trends by enabling organizations to audit, create alerts, and archive data, including tracking API calls in customers' Azure resources.

**Application Insights**
[Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/) is an extensible Application Performance Management (APM) service for web developers building and managing apps on multiple platforms. Learn how to detect & diagnose issues and understand usage for your web apps and services using our quickstarts, tutorials, and reference documentation.

#### Logging and Auditing

[Operations Management Suite (OMS)](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) provides extensive logging of system and user activity, as well as system health. The OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) solution collects and analyzes data generated by resources in Azure and on-premises environments.
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

### Additional Services

**Azure Machine Learning**
[Azure Machine Learning](https://docs.microsoft.com/en-us/azure/machine-learning/preview/) services (preview) enable building, deploying, and managing machine learning and AI models using any Python tools and libraries. You can use a wide variety of data and compute services in Azure to store and process your data.

**Azure Functions**
[Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview) is a solution for easily running small pieces of code, or "functions," in the cloud. You can write just the code you need for the problem at hand, without worrying about a whole application or the infrastructure to run it. Functions can make development even more productive, and you can use your development language of choice, such as C#, F#, Node.js, Java, or PHP. Pay only for the time your code runs and trust Azure to scale as needed. Azure Functions lets you develop serverless applications on Microsoft Azure.

**Azure Event Grid**
[Azure Event Grid](https://docs.microsoft.com/en-us/azure/event-grid/overview) allows you to easily build applications with event-based architectures. You select the Azure resource you would like to subscribe to, and give the event handler or WebHook endpoint to send the event to. Event Grid has built-in support for events coming from Azure services, like storage blobs and resource groups. Event Grid also has custom support for application and third-party events, using custom topics and custom webhooks.

## Threat Model

The data flow diagram (DFD) for this reference architecture is available for [download](https://aka.ms/blueprintdwthreatmodel) or can be found below. This model can help customers understand the points of potential risk in the system infrastructure when making modifications.

## Compliance Documentation
The Azure Security and Compliance Blueprint – GDPR Customer Responsibility Matrix lists controller and processor responsibilities for all GDPR articles. Please note that for Azure services, a customer is usually the controller and Microsoft acts as the processor.

The Azure Security and Compliance Blueprint - GDPR Analytics Implementation Matrix provides information on which GDPR articles are addressed by the analytics architecture, including detailed descriptions of how the implementation meets the requirements of each covered article.


## Guidance and Recommendations
### ExpressRoute and VPN
[ExpressRoute](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction) or a secure VPN tunnel needs to be configured to securely establish a connection to the resources deployed as a part of this reference architecture. As ExpressRoute connections do not go over the Internet, these connections offer more reliability, faster speeds, lower latencies, and higher security than typical connections over the Internet. By appropriately setting up ExpressRoute or a VPN, customers can add a layer of protection for data in transit.

### Extract-Transform-Load (ETL) Process
[PolyBase](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/polybase-guide) can load data into Azure SQL Data Warehouse without the need for a separate ETL or import tool. PolyBase allows access to data through T-SQL queries. Microsoft's business intelligence and analysis stack, as well as third-party tools compatible with SQL Server, can be used with PolyBase.

### Azure Active Directory Setup
[Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-whatis) is essential to managing the deployment and provisioning access to personnel interacting with the environment. An existing Windows Server Active Directory can be integrated with AAD in [four clicks](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect-get-started-express). Customers can also tie the deployed Active Directory infrastructure (domain controllers) to an existing AAD by making the deployed Active Directory infrastructure a subdomain of an AAD forest.

### Azure AD Privileged Identity Management
With [Azure AD Privileged Identity Management](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-privileged-identity-management-configure?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json), you can manage, control, and monitor access within your organization. This includes access to resources in Azure AD, Azure Resources (Preview), and other Microsoft Online Services like Office 365 or Microsoft Intune.

## Disclaimer

 - This document is for informational purposes only. MICROSOFT MAKES NO WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, AS TO THE INFORMATION IN THIS DOCUMENT. This document is provided "as-is." Information and views expressed in this document, including URL and other Internet website references, may change without notice. Customers reading this document bear the risk of using it.
 - This document does not provide customers with any legal rights to any intellectual property in any Microsoft product or solutions.
 - Customers may copy and use this document for internal reference purposes.
 - Certain recommendations in this document may result in increased data, network, or compute resource usage in Azure, and may increase a customer's Azure license or subscription costs.
 - This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment.
 - This document is developed as a reference and should not be used to define all means by which a customer can meet specific compliance requirements and regulations. Customers should seek legal support from their organization on approved customer implementations.
