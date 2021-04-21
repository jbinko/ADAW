# Azure Data Analytical Workspace (ADAW) - Reference Architecture 2021 for Regulated Industries

## Project Overview

The goal of this project is to provide and document referential infrastructure architecture pattern,
guidance, explanations, and deployment resources/tools
to successfully deploy workspace for data analysis based on Azure services.

Focus is to provide the highest security level as this deployment pattern is
used in Highly Regulated Industries.

We are describing reusable pattern which can be further customized as each customer has different needs
and expectations. All resources are included inside this repository and we are open for feedback to further extend this pattern.

You can think about this guidance as Enterprise Ready plug-able
infrastructure building block for Data Analytics workloads compatible with Microsoft Best Practices for Landing Zones.

Data Analytical Workspace can be deployed multiple times in organization
for different teams, for different projects, for different environments (DEV, TEST, PROD, etc.).

Data Analytical Workspace can be deployed in automated way through provided scripts in cloud native way.
This provides consistent experience with focus on high quality security standards.
Approve once, deploy multiple times in the same secure way.

### Key Features

- Focus on Enterprise Grade Security standards
- Strong Support for Auditing, Monitoring and Diagnostics data
- Integrate and Keep network communication in perimeter where applicable
- Allow consumption of data sources inside perimeter, analyze data in cloud
- Benefit from Cloud Managed Services, reduce management and operations overhead
- Integrations with other cloud native tools - mainly Power Platform
- Protect and encrypt storage where potentially sensitive data are stored
- Protect keys and credentials in secure place

## Architecture Overview

Idea behind this pattern is based on following workflow.

Business Users needs to present, consume, slice and dice data in quick way on multiple devices from multiple places.
Ideally on data model which is optimized (transformed) for data domain they are aligned to.

To achieve this, you typically need to get data in scalable way from multiple data sources in raw format (typically sitting on-prem),
store them in scalable way with potentially huge volumes. Store them to cheap storage in multiple versions and with history, clean data,
combine data Together, pre-aggregate data and store them again in structured way with indexing capability to provide speed for access.
You probably also want to mask data and secure/hide data which should not be seen by users from other geo regions or departments.

For that, you need to understand (propagate security context) who is viewing the data from reporting tool to structured storage
and ensure you are filtering data for target user based on role. (Role Based Access Control, Row Level Security).
You do not want to do it programmatically (like in WHERE clause), database engine should do it for you.

### Architecture in Azure components

Let's have a look how architecture described above can be translated into specific Azure services.
![Architecture Overview](DocImages/Overview.png)

_**Business Users needs to present, consume, slice and dice data in quick way on multiple devices from multiple places**_

This can be achieved by easy to use PowerBI reporting tool, which can be used from anywhere and on multiple platforms and devices.

PowerBI runs in cloud as managed service. Service can be also integrated with perimeter for access from devices and can also access
data sources which are part of perimeter (on-premises or in cloud via private link). This PowerBI Premium VNET integration feature is in preview now.

You do not want to use PowerBI gateway for perimeter access which is typically multiplexing accessing users to one service identity and thus undermining security context against target database which is important to preserve.

PowerBI also supports integration with Azure Active Directory (cloud identity) and advanced security features. Identity and whole
security context can be thus propagated through PowerBI to the database engine and database engine can use native filtering capabilities
based on role accessing user belongs to.

Example can be: user from mobile device outside perimeter needs to access predefined and optimized report which is accessing and rendering sensitive data from data source hosted inside a perimeter. User might be required to establish VPN connectivity first, will be prompted to authenticate to PowerBI, verified with MFA and PowerBI rendering engine will pass user's identity of accessing user to the target database hosted in the perimeter. Database can verify accessing user and understand user's role/security group. With that database engine can do
query/data filtering and show only data user was allowed to see.

_**Need to get data in scalable way from multiple data sources in raw format (typically sitting on-prem), store them in scalable way with huge volumes to cheap storage in multiple versions and with history**_

PowerBI ideally needs to consume optimized data models for specific data domain which improves user experience,
reduces waiting time and improves data model maintenance. Before that, you typically need to run some ETL process
which will first get data from multiple data sources from many places, in raw format (snapshots) and store them for further processing.

In this architecture load is mainly role of Azure Data Factory - Easy to use ETL tool along with Azure Storage in Azure Data Lake (Hierarchical namespace) mode.

Azure Data Factory managed PaaS service allows you to connect many data sources through massive list of ever growing list of supported connectors.
Azure Data Factory has a workspace where ETL process can be designed and executed at scale. Workspace can be accessed from anywhere as access policy
allows and is specified by security administrators. Similar approach like accessing O365 workspace.

Azure Data Factory also understands and can store data in multiple file formats (standard or proprietary).
Azure Data Factory can access data and store them in scalable manner as snapshots to Azure Data Lake Storage.
Basically get data in scalable way from anywhere and store them in raw format to Azure Data Lake Storage in versioned way so more
sophisticated data models can be built based on such data.

To access data sources from cloud you need to have some kind of gateway which will not require to punch hole into the enterprise firewall. This
is what Azure Data Factory Self Host Integration Runtime provides. This component needs to be installed in
the on-premises environment (physical or virtual machine) and needs to be allowed to open communication to the cloud. Azure Data Factory
then can consume data from allowed data sources inside organization and process them in cloud by specific associated and only allowed Azure Data Factory instance.

Azure Data Lake is very cheap, practically unlimited storage space where data can be stored in raw formats, in multiple versions (e.g daily snapshots)
and storage is providing very secure (Role Based Access Control) way to store data. This storage is also optimized for support
of massive parallel processing of data. To provide even better security story, Azure Data Lake Storage is part of perimeter without exposing any public endpoint.

Storage will not be accessed directly by PowerBI (although technically possible and supported). Storage is kind of staging area in this case.
PowerBI should work with more structured and optimized domain driven data source built from raw data from storage.
