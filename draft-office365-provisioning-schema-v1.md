# A schema and engine for provisioning of Office 365 Artefacts (Draft)
author:
  fullname: Wictor Wilén
  email: wictor@wictorwilen.se

license:
    Apache License v 2.0
    https://github.com/wictorwilen/OfficeProvisioning/blob/master/LICENSE

source: 
    https://github.com/wictorwilen/OfficeProvisioning


# Abstract

This document shows shows how a schema and engine for provisioning Office 365 
artefacts could be designed and implemented. The artefacts are defined in an 
XML file, as defined in the OfficeProvisioningSchema-1.0.xsd file, and the 
engine is any kind of software that can provision artefacts using an artefact
file. 

# Introduction

Currently there are no standard or de-facto standards for defining artefacts
that are to be provisioned in Office 365 (SharePoint Online, Exchange Online, 
Yammer, and more). The purpose of the specification is to define; a) a schema
that can be used to define artefacts that can be provisioned into an Office 365
service and b) an engine definition that uses an artefact file (defined by the
schema) and provisions the artefacts.

There are multiple initiatives, including initiatives from the Office 
PnP group, on how to provision artefacts into Office 365. However, most of 
these products/services/initiatives are focused on SharePoint and also inherits
the legacy of the SharePoint CAML language. Also most of these initiatives
are pure .NET based or implemented without a proper schema. This schema and 
engine allows for a standardized, and extensible, schema for defining what and
where to provision artefacts and a definition and requirements on an engine
using this provisioning schema. That is; anyone are free to use the schema and 
anyone are free to implement an engine.


# Provisioning Schema

The provisioning Schema is defined as an XML Schema file, with annotations for 
further documentation. 

## Schema Namespace
The schema namespace is: 
    https://github.com/wictorwilen/OfficeProvisioning/OfficeProvisioningSchema-1.0.xsd

## Provisioning Schema basics

The following are a high-level description of the schema

### Parameters

Each Artefact file has a set of Parameters. Parameters can be defined in the 
Artefact file or provided as arguments to the Provisioining Engine.
Parameters are used as replacements in the Artefact file. Specific attributes
in the schema has the type `ReplaceableString`. These attributes can have 
Parameters embedded, surrounded by curly braces ('{' and '}').

~~~
<p:SiteCollection 
  id="sc1" 
  template="STS#0" 
  title="Team sites" 
  url="https://{O365TenantName}.sharepoint.com/teams/team-sites"
  overwrite-options="allowed">
~~~

### Sequence

All artefacts that are to be provisioned belongs to a Sequence. The Sequences
must be provisioned in the order that they appear in the artefact file. Each 
sequence contains one or more Containers (see below) that is to be provisioned.
Each sequence can either be synchronous or asynchronous. For asynchronous 
sequences all containers can be provisioned asynchronous, if the engine 
supports it.

### Containers

A Container is the base of everything that can be provisioned.

The Schema currently support the following top level Containers
* Site Collection (SPO)
* TermStore (SPO)
* Mailbox (EXO)
* Yammer Group (Yammer)
* User (Azure AD)
* Group (Azure AD)

Other containers are
* Site (SPO)
* List, Library, Folder, File
* Site Group (SPO)
* Term Group, Term Set, Term (SPO)

### Extensions

The Extensions element in the schema can contain any arbitrary XML. This 
element is ment to be used for extensions to the Schema. It might be vendor
or engine specific extensions or fill gaps that the current schema does not 
currently support.

Extensions can be used in a Container or in a Sequence.

~~~
<p:Extensions>
  <my:CustomExtension  xmlns:my="http://tempuri"/>
  <my2:AnotherExtension  xmlns:my2="http://tempuri2"/>
</p:Extensions>
~~~

# Artefact file 

The artefact file is an XML file, that is defined by the Provisioning Schema.

# Provisioning Engine

A Provision Enginge is any software that uses an Artefact file, based on the 
provisioning schema, to provision artefacts into Office 365.

## Provisioning Engine requirements

A provisioning enginge MUST adhere to the following requirements:

* The Provisioning Engine MUST accept an Artefact file based on the 
Provisioning schema
* The provisioning Engine MUST validate the schema
* The Provisioning Engine MUST NOT support all top-level artefacts
* The Provisioning Engine MUST NOT provision any artefacts if the artefact
file contains any top-level container marked `provisioning-support="required"`
* The Provisioning Engine MUST support parameters
* The Provisioning Engine MUST NOT provision any artefacts if any required
Parameters are missing

## Provisioning Engine execution examples

A Windows Console based engine:
~~~
c:\> Provision.exe -file artefacts.xml -parameters "param1=value1, param2=value2"
~~~

A PowerShell based engine:
~~~
PS C:\> Provision.ps1 -file artefacts.xml -parameters @{"param1" = "value1", "param2" = "value2"}
~~~

# Disclaimer

This draft is provided "AS IS" with no warranties, and confers no rights. The 
content of this drat are my own personal opinions and do not represent my 
employer's view in anyway. In addition, my thoughts and opinions are open to 
change. 
