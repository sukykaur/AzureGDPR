# Azure Security and Compliance Blueprint: Data Warehouse for GDPR

## Overview
The General Data Protection Regulation (GDPR) is fundamentally about protecting and enabling the privacy rights of individuals.
The GDPR establishes strict global privacy requirements governing how you manage and protect personal data while respecting individual choice - no matter where data is sent, processed, or stored. Microsoft Azure services meet the stringent GDPR security requirements and Microsoft's contractual commitments guarantee that you can:
- Respond to requests to correct, amend or delete personal data.
- Detect and report personal data breaches.
- Demonstrate your compliance with the GDPR.

This Azure Security and Compliance Blueprint provides guidance for how to deliver a Microsoft Azure data warehouse architecture that helps ____. This solution provides guidance on the deployment and configuration of Azure resources for a common reference architecture, demonstrating ways in which customers can meet specific security and compliance requirements and serves as a foundation for customers to build and configure their own data warehouse solutions in Azure.

This reference architecture, associated control implementation guides, and threat models are intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment. Please note the following:
- The architecture provides a baseline to help customers deploy workloads to Azure in a GDPR-compliant manner.
- Customers are responsible for conducting appropriate security and compliance assessments of any solution built using this architecture, as requirements may vary based on the specifics of each customer's implementation.

## Architecture Diagram and Components

## Deployment Architecture

### Discover
**Identify which personal data exists and where it resides.**

A critical step to addressing GDPR requirements is to identify all personal data managed by the organization, so that they can adequately protect it and respond to data subject requests, such as erasure, rectification, and data portability.  

Azure Active Directory enables administrators to search for user data, and then edit data associated with the user account.

Using SQL queries, Microsoft customers can correct inaccurate or incomplete data hosted in Azure SQL Database.

### Manage
**Govern how personal data is used and accessed.**

Azure enables you to export your data at any time, without seeking approval from Microsoft. Azure Active Directory (AAD) enables you to export data associated with AAD accounts in a .csv file.

Using SQL queries, Microsoft customers can identify and then export personal data hosted in Azure SQL Database.

### Protect
**Establish security controls to prevent, detect, and respond to vulnerabilities and data breaches.**

The solution uses [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) for the management of keys and secrets. Azure Key Vault helps safeguard cryptographic keys and secrets used by cloud applications and services.
- Advanced access policies are configured on a need basis
- Key Vault access policies are defined with minimum required permissions to keys and secrets
- All keys and secrets in Key Vault have expiration dates
- All keys in Key Vault are protected by HSM [Key Type = HSM Protected 2048-bit RSA Key]
- All users/identities are granted minimum required permissions using Role Based Access Control (RBAC)
- Diagnostics logs for Key Vault are enabled with a retention period of at least 365 days.
- Permitted cryptographic operations for keys are restricted to the ones required

#### Data in Transit
Azure encrypts all communications to and from Azure datacenters by default. All transactions to Azure Storage through the Azure Portal occur via HTTPS.

#### Data at Rest

The architecture protects data at rest through encryption, database auditing, and other measures.

**Azure Storage**
To meet encrypted data at rest requirements, all [Azure Storage](https://azure.microsoft.com/services/storage/) uses [Storage Service Encryption](https://docs.microsoft.com/en-us/azure/storage/storage-service-encryption).

**Azure Disk Encryption**
[Azure Disk Encryption](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) leverages the BitLocker feature of Windows to provide volume encryption for OS and data disks. The solution integrates with Azure Key Vault to help control and manage the disk-encryption keys.

**Azure SQL Database**
The Azure SQL Database instance uses the following database security measures:
-	[AD Authentication and Authorization](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-aad-authentication) enables identity management of database users and other Microsoft services in one central location.
-	[SQL database auditing](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-auditing-get-started) tracks database events and writes them to an audit log in an Azure storage account.
-	SQL Database is configured to use [Transparent Data Encryption (TDE)](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql), which performs real-time encryption and decryption of data and log files to protect information at rest. TDE provides assurance that stored data has not been subject to unauthorized access.
-	[Firewall rules](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-firewall-configure) prevent all access to database servers until proper permissions are granted. The firewall grants access to databases based on the originating IP address of each request.
-	[SQL Threat Detection](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-threat-detection-get-started) enables the detection and response to potential threats as they occur by providing security alerts for suspicious database activities, potential vulnerabilities, SQL injection attacks, and anomalous database access patterns.
-	[Always Encrypted columns](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-always-encrypted-azure-key-vault) ensure that sensitive data never appears as plaintext inside the database system. After enabling data encryption, only client applications or app servers with access to the keys can access plaintext data.
-	[SQL Database Dynamic Data Masking (DDM)](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-dynamic-data-masking-get-started) limits sensitive data exposure by masking the data to non-privileged users or applications. DDM allows the database administrator to select a particular table-column that contains sensitive data, add a mask to it (there are a few available built-in masks that can be applied, as well as a customizable mask), and designate which database users are privileged and should have access to the real data. Once configured, any query on that table or column will contain masked results, except for queries run by privileged users. DDM can be done after the reference architecture deploys. **Note: Customers will need to adjust dynamic data masking settings to adhere to their database schema.**

### Report
**Keep required documentation and manage data requests and breach notifications.**

#### Logging and Auditing

[Operations Management Suite (OMS)](https://docs.microsoft.com/azure/security/azure-security-disk-encryption) provides extensive logging of system and user activity, as well as system health. The OMS [Log Analytics](https://azure.microsoft.com/services/log-analytics/) solution collects and analyzes data generated by resources in Azure and on-premises environments.
- **Activity Logs**: [Activity logs](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs) provide insight into operations performed on resources in a subscription.
- **Diagnostic Logs**: [Diagnostic logs](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs) include all logs emitted by every resource. These logs include Windows event system logs and Azure Blob storage, tables, and queue logs.
- **Firewall Logs**: The Application Gateway provides full diagnostic and access logs. Firewall logs are available for WAF-enabled Application Gateway resources.
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

## Guidance and Recommendations
### ExpressRoute and VPN
[ExpressRoute](https://docs.microsoft.com/en-us/azure/expressroute/expressroute-introduction) or a secure VPN tunnel needs to be configured to securely establish a connection to the resources deployed as a part of this data warehouse reference architecture. As ExpressRoute connections do not go over the Internet, these connections offer more reliability, faster speeds, lower latencies, and higher security than typical connections over the Internet. By appropriately setting up ExpressRoute or a VPN, customers can add a layer of protection for data in transit.

### Extract-Transform-Load (ETL) Process
[PolyBase](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/polybase-guide) can load data into Azure SQL Data Warehouse without the need for a separate ETL or import tool. PolyBase allows access to data through T-SQL queries. Microsoft's business intelligence and analysis stack, as well as third-party tools compatible with SQL Server, can be used with PolyBase.

### Azure Active Directory Setup
[Azure Active Directory](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-whatis) is essential to managing the deployment and provisioning access to personnel interacting with the environment. An existing Windows Server Active Directory can be integrated with AAD in [four clicks](https://docs.microsoft.com/en-us/azure/active-directory/connect/active-directory-aadconnect-get-started-express). Customers can also tie the deployed Active Directory infrastructure (domain controllers) to an existing AAD by making the deployed Active Directory infrastructure a subdomain of an AAD forest.
## Disclaimer

 - This document is for informational purposes only. MICROSOFT MAKES NO WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, AS TO THE INFORMATION IN THIS DOCUMENT. This document is provided "as-is." Information and views expressed in this document, including URL and other Internet website references, may change without notice. Customers reading this document bear the risk of using it.
 - This document does not provide customers with any legal rights to any intellectual property in any Microsoft product or solutions.
 - Customers may copy and use this document for internal reference purposes.
 - Certain recommendations in this document may result in increased data, network, or compute resource usage in Azure, and may increase a customer's Azure license or subscription costs.
 - This architecture is intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment.
 - This document is developed as a reference and should not be used to define all means by which a customer can meet specific compliance requirements and regulations. Customers should seek legal support from their organization on approved customer implementations.
