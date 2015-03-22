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
    http://github.com/wictorwilen/OfficeProvisioning/OfficeProvisioningSchema-1.0.xsd

## Provisioning Schema basics

The following are a high-level description of the schema

## Provisioning Schema vs ye olde SharePoint Schema

In order to provide as much backwards compatibility with CAML and the 
SharePoint schemas, this provisioning schema uses the same XML formatting
rules as the wss.xsd (Pascal Case elements, attributes and values).
Also some elements have a direct representation in the provisioning schema:
For instance this CAML Field definition can be copied to the provisioning 
schema and only a namespace definition has to be added.
From:
~~~
<Field ID="{7B2B1712-A73D-4ad7-A9D0-662F0291713D}" Name="HealthRuleCheckEnabled" 
  Type="Boolean" Group="_Hidden" AllowDeletion="FALSE" DisplayName="Aktiverad"  
  SourceID="http://schemas.microsoft.com/sharepoint/v3/fields" 
  StaticName="HealthRuleCheckEnabled"  />
~~~
to:
~~~
<p:Fields>
  <Field 
    xmlns="http://github.com/wictorwilen/OfficeProvisioning/OfficeProvisioningSchema-1.0.xsd"
    ID="{7B2B1712-A73D-4ad7-A9D0-662F0291713D}" Name="HealthRuleCheckEnabled" 
    Type="Boolean" Group="_Hidden" AllowDeletion="FALSE" DisplayName="Aktiverad"  
    SourceID="http://schemas.microsoft.com/sharepoint/v3/fields" 
    StaticName="HealthRuleCheckEnabled"
    ProvisioningAction="Provision" />
</p:Fields>
~~~

 

### Parameters

Each Artefact file has a set of Parameters. Parameters can be defined in the 
Artefact file or provided as arguments to the Provisioining Engine.
Parameters are used as replacements in the Artefact file. Specific attributes
in the schema has the type `ReplaceableString`. These attributes can have 
Parameters embedded, surrounded by curly braces ('{' and '}'). The 
`ReplaceableInt` can either be in integer or a integer parameter enclosed in 
curly brackets

~~~
<p:SiteCollection 
  ID="sc1" 
  Template="STS#0" 
  Title="Team sites" 
  Url="https://{O365TenantName}.sharepoint.com/teams/team-sites"
  PrimarySiteCollectionAdmin="garthf@contoso.com"
  OverwriteOptions="Allowed">
  <p:SiteTemplate Locale="{lcid}">STS#0</p:SiteTemplate>
...
</p:SiteCollection>
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

Future containers
* Mailbox (EXO)
* Yammer Group (Yammer)
* User (Azure AD)
* Group (Azure AD)

Other inner containers are
* Site (SPO)
* Lists, Files, Pages (SPO)
* Site Groups (SPO)
* Term Groups, Term Sets, Terms (SPO)

### To provision or not provision

The Schema has an attribute `ProvisioningAction` that is used to define how 
the Container or item should provisioned or not. The default value is 
`Provision` which indicates that the engine MUST provision the item. A value
of `Unprovision` indicates that the engine MUST unprovision or permanently
delete the item. A value of `Ignore` indicates that the engine MUST ignore/skip
provisioning/unprovisioning of the item.

### Extensions

The Extensions element in the schema can contain any arbitrary XML. This 
element is ment to be used for extensions to the Schema. It might be vendor
or engine specific extensions or fill gaps that the current schema does not 
currently support.

Extensions can be used in a `Container` or in a `Sequence`.

~~~
<p:Extensions>
  <pnp:TypeExtension xmlns:pnp="http://pnp" assembly="Abc.Def" type="Abc.Def.Ghi"/>
  <np:ThemeExtension xmlns:pnp="http://pnp"
    Name="Contoso"
    ColorFile="Resources/Themes/Contoso/contoso.spcolor"
    FontFile="Resources/Themes/Contoso/contoso.spfont"
    BackgroundFile="Resources/Themes/Contoso/contosobg.jpg"
    MasterPage="seattle.master"
    AlternateCSS="Resources/Themes/Contoso/Contoso.css"
    SiteLogo="Resources/Themes/Contoso/contosologo.png"
    Version="1" />
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
file contains any top-level container marked `ProvisioningSupport="Required"`
* The Provisioning Engine MUST support parameters
* The Provisioning Engine MUST NOT provision any artefacts if any required
Parameters are missing
* The Provisioning Engine SHOULD support passing in one ore more sequences to
be provisioned
* The Provisioning Engine MUST use the overwrite options specified in the
provisioning file.
* If no overwrite options are specified in the provisioning file, it MUST
be assumed that no artefacts should be overwritten.

## Provisioning context and permissions

The Engine MUST pass the correct credentials when provisioning artefacts. 
The engine might SHOULD ask the user for credentials or have an argument that 
could be used to pass credentials.

The Engine MUST handle access denied or incorrect permissions errors and log or
display those message to the end-user. If an access denied error is caught when
privisioning a Container that has the `ProvisioningSupport="Required"` set
a fatal error MUST be thrown and no further artefacts MUST be provisioned.

In this current draft it is assumed that the same credentials can be used for
all services.

## Provisioning Engine execution examples

A Windows Console based engine:
~~~
c:\> Provision.exe -file artefacts.xml -parameters "param1=value1, param2=value2"
~~~

A Windows Console based engine:
~~~
c:\> Provision.exe -file artefacts.xml -parameters "param1=value1, param2=value2"
     -sequences "sequence1, sequence3"
~~~

A PowerShell based engine:
~~~
PS C:\> Provision.ps1 -file artefacts.xml -parameters @{"param1" = "value1", "param2" = "value2"}
        -credentials $creds
~~~

# Disclaimer

This draft is provided "AS IS" with no warranties, and confers no rights. The 
content of this drat are my own personal opinions and do not represent my 
employer's view in anyway. In addition, my thoughts and opinions are open to 
change. 
