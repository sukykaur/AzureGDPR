# Azure Security and Compliance Blueprint: Data Warehouse for GDPR

## Overview
The General Data Protection Regulation (GDPR) is fundamentally about protecting and enabling the privacy rights of individuals.
The GDPR establishes strict global privacy requirements governing how you manage and protect personal data while respecting individual choice - no matter where data is sent, processed, or stored. Microsoft Azure services meet the stringent GDPR security requirements.

This Azure Security and Compliance Blueprint provides guidance for how to deliver a Microsoft Azure data warehouse architecture that helps ____. This solution provides guidance on the deployment and configuration of Azure resources for a common reference architecture, demonstrating ways in which customers can meet specific security and compliance requirements and serves as a foundation for customers to build and configure their own data warehouse solutions in Azure.

This reference architecture, associated control implementation guides, and threat models are intended to serve as a foundation for customers to adjust to their specific requirements and should not be used as-is in a production environment. Please note the following:
- The architecture provides a baseline to help customers deploy workloads to Azure in a GDPR-compliant manner.
- Customers are responsible for conducting appropriate security and compliance assessments of any solution built using this architecture, as requirements may vary based on the specifics of each customer's implementation.

## Architecture Diagram and Components

## Deployment Architecture

### Discover
**Identify which personal data exists and where it resides.**

Azure Active Directory enables administrators to search for user data, and then edit data associated with the user account.

### Manage
**Govern how personal data is used and accessed.**

Azure enables you to export your data at any time, without seeking approval from Microsoft. Azure Active Directory (AAD) enables you to export data associated with AAD accounts in a .csv file.

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

### Report
**Keep required documentation and manage data requests and breach notifications.**

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
