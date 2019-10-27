---
date: 2019-10-27
title: HL7 FHIR on Azure - Device Framework
tags:
- Serverless
- IoT
- FHIR

---
\---

title: HL7 FHIR on Azure - Device Framework

abstract: Recently we were brought in to design and develop an IoT solution for an Medical Device manufacturer. Their Point-of-Care devices are used.  They had been using WCF services on-premise, along with a SQL Database Server. They were looking to move to Azure, but still wanted to utilize their SQL Database.

categories: HL7 FHIR, Devices, IoT

weblogName: DocDB

postDate: 2018-04-28T19:15:42.6271741-05:00

customFields:

  Version:

    key: Version

    value: 4.2.1

    draft: true

    ---

HL7 FHIR on Azure - Device Framework

==================================

!\[\](![](https://i.imgur.com/mJNnNuT.png))

<div style="page-break-after: always;"></div>

| Table of Contents                        |

|------------------------------------------|

| \[FOREWARD\](#forward)                      |

| \[ABOUT THE AUTHOR\](#about-the-author)  |

| \[CHAPTER 1 - USE CASE\](#chapter-1---use-case) |

| \[CHAPTER 2 - CONFIGURING OUR DATA REPOSITORY\](#chapter-2---configuring-our-data-repository) |

| \[CHAPTER 3 - AZURE COSMOS DB DOCUMENT ATTACHMENT STORAGE COMPARISON\](#chapter-3---azure-cosmos-db-document-attachment-storage-comparison) |

| \[CHAPTER 4 - POSTING DEVICE SOFTWARE UPDATE TO HL7 FHIR SERVER (AZURE COSMOS DB)\](#chapter-4---posting-device-software-update-to-hl7-fhir-server-azure-cosmos-db) |

| \[CHAPTER 5 - USING AZURE DURABLE FUNCTIONS FOR UPDATING HEALTHCARE DEVICES\](#chapter-5---using-azure-durable-functions-for-updating-healthcare-devices) |

| \[CHAPTER 6 - SAVING MEDICAL DEVICE TO CLOUD MESSAGES AS HL7 FHIR OBSERVATION RESOURCES\](#chapter-6---saving-medical-device-to-cloud-messages-as-hl7-fhir-observation-resources) |

| \[CHAPTER 7 -  GET NEW OR MODIFIED HL7 FHIR OBSERVATION RESOURCES FROM AZURE COSMOS DB\](#chapter-7----get-new-or-modified-hl7-fhir-observation-resources-from-azure-cosmos-db) |

| \[CHAPTER 8 - PATIENT CARE DEVICE DATA INTEGRATION\](#chapter-8---patient-care-device-data-integration) |

| \[CHAPTER 9 - PROVISIONING\](#chapter-9---provisioning) |

| \[CHAPTER 10 - CONSIDERING SOFTWARE AS AN IOT DEVICE\](#chapter-10---considering-software-as-an-iot-device) |

| \[CHAPTER 11 - IOT EDGE\](#chapter-11---iot-edge) |

| \[CHAPTER 12 - MONITORING AND NOTIFICATIONS\](#chapter-12---monitoring-and-notifications) |

| \[CHAPTER 13 - COPY DATA FROM AZURE TO ON PREMISE SQL\](#chapter-13---copy-data-from-azure-to-on-premise-sql-database) |

| \[CHAPTER 14 - DICOM\](#chapter-14---dicom) |

| \[CHAPTER 15 - RECEIVING DEVICE MESSAGES\](#chapter-15---receiving-device-messages) |

| \[CHAPTER 16 - TIME SERIES INSIGHTS\](#chapter-16---time-series-insights) |

| \[CHAPTER 17 - PROCESSING QUEUE MESSAGES\](#chapter-17---processing-queue-messages) |

| \[CHAPTER 18 - IMPLEMENTING A CUSTOM APPLICATION GATEWAY\](#chapter-18---implementing-a-custom-application-gateway) |

| \[CHAPTER 19 - MICROSERVICES\](#chapter-19-microservices) |

| \[CHAPTER 20 - MAPPING OUR DEVICE DATA TO FHIR RESOURCES\](#chapter-20--mapping-our-device-data-to-fhir-resources) |

| \[CHAPTER 21 - HL7 FHIR DEVICE DATA INTEGRATION\](#chapter-21--hl7-fhir-device-data-integration) |

| \[APPENDECIES\](#appendecies)              |

| \[APPENDIX I - MEDICAL DEVICE AND IMPLANTABLES TRACKING USING UDI \](#appendix-i-medical-device-and-implantables-tracking-using-udi) |

	

<!-- End Document Outline -->

<div style="page-break-after: always;"></div>

\# Foreward

| |

|----------|

| !\[\](![](https://i.imgur.com/h887LNQ.jpg)) |

<div style="page-break-after: always;"></div>

\# About the author

\### Howard S. Edidin 

Howard has close to thirty years of experience in delivering Enterprise Integration Solutions. For the last 18 years, Howard has been specializing in Healthcare and Life Sciences. He is a <a href="[https://mvp.microsoft.com/en-us/PublicProfile/5001819?fullName=Howard](https://mvp.microsoft.com/en-us/PublicProfile/5001819?fullName=Howard "https://mvp.microsoft.com/en-us/PublicProfile/5001819?fullName=Howard")" target="_blank">Microsoft MVP for the Data Platform</a> and a P-TSP for Microsoft Healthcare and Life Sciences (Cosmos DB and Azure). Howard is also a DocumentDB Wizard and TechNet Ninja. He  co-authored  <a href="[https://www.apress.com/us/book/9781430267645](https://www.apress.com/us/book/9781430267645 "https://www.apress.com/us/book/9781430267645")" target="_blank">HL7 for BizTalk</a>, and has been a technical reviewer for several books.

Howard is very heavily involved with development and implementation of a new HL7 Standard, FHIR. He is considered a “go to” person by Microsoft, for HL7 integration on Azure. He is the only non-employeee member of the **Microsoft HL7 FHIR and Interoperability Virtual Team**. He authored an eBook, **<a href="[https://info.microsoft.com/HL7FHIRonAzure-Registration.html](https://info.microsoft.com/HL7FHIRonAzure-Registration.html "https://info.microsoft.com/HL7FHIRonAzure-Registration.html")" target="_blank">HL7 FHIR on Azure</a>** for Microsoft. 

Howard is employed by Cognizant as a Associate Director, Azure Architect and SME. He specializes in IoT - Medical Devices and HL7 FHIR Integration.   

<div style="page-break-after: always;"></div>

\# CHAPTER 1 - USE CASE

Recently we were brought in to design and develop an IoT solution for an Medical Device manufacturer. Their Point-of-Care devices are used throughout the world.

They had been using WCF services on-premise, along with a SQL Database Server. They were looking to move to Azure, but still wanted to utilize their SQL Database.

\## Business Requirements

The **Software Update Service** is to be one of several **Microservices** that will comprise the complete IoT System. These services include **Provisioning**, **Management**, **Logging**,

\**Analysis**, **Security** and **Monitoring and Alerts**.

> !\[\](![](https://i.imgur.com/eSPsKWl.png)) **INFO:** Our Microservices Architecture does not require Azure Service Fabric or Azure Container Service. They are **Hybrid Services** managed by **Azure Api Management**.

These services will use a common Data Repository.

\## Functional Requirements

\-   The service would support an Multi-Tenancy Architecture.

\-   Ability to have the software updates as close as possible to their  customers Medical Device System (MDS), Virtual Medical Device (VMD) or Channels.

\-   A Form-Post from a Web Application will be used to send the software

    update file, along with metadata to the Software Update Service.

\-   All Device Data is to be stored as HL7 FHIR Resources.

\-   All data must be encrypted at Rest and Intransit.

\-   Ability to receive data from a Device.

\-   Ability to preform analysis on Device data.

>  !\[\](![](https://i.imgur.com/DiPYycK.png)) **INFORMATION**\\

> There are four HL7 FHIR Resources that we will be using: Device,

> DeviceMetric, DeviceComponent and Observation

>

> In FHIR, the "Device" is the "administrative" resource for the device

> (it does not change much and has manufacturer information etc.),

> whereas the DeviceComponent and DeviceMetric (which is really a kind

> of DeviceComponent) model the physical part, including operation

> status and is much more volatile.

>

> The physical composition of a Device is done by the DeviceComponents

> pointing to their "parent" component using DeviceComponent.parent. All

> components point to the "logical" Device they belong to, using

> DeviceComponent.source.

>

> The **Observation Resource** is used to record the data from the

> Device.

>

> HL7 FHIR has a REST API. The API is hosted in an Azure Web

> Application. It is used by Healthcare EHR systems to query the Device,

> DeviceComponent, DeviceMetric, and Observation Resources.

<div style="page-break-after: always;"></div>

\## The patient device data is here

!\[\](![](https://i.imgur.com/YBtkVIo.jpg))

<div style="page-break-after: always;"></div>

\## Also here

!\[\](![](https://i.imgur.com/P9ZjYKK.png))

<div style="page-break-after: always;"></div>

\## And here

!\[\](![](https://i.imgur.com/HsvW5mj.png))

<div style="page-break-after: always;"></div>

\## HL7 FHIR Resource Document Templates

The following are the HL7 FHIR Resource Templates that are required.

> !\[\](![](https://i.imgur.com/dMrFdY8.png)) **NOTE**:

>

> When a Device is provisioned, the Device, DeviceComponent,

> DeviceMetric, and Observation Resource Documents are created and

> stored in the repository.\\

> The Observation Resource Document is updated from the data received

> from a Device.

\### Device Document Template

 \`\`\`json

{

   "resourceType" : "Device",

   // from Resource: id, meta, implicitRules, and language

   // from DomainResource: text, contained, extension, and modifierExtension

   "identifier" : \[{ Identifier }\], // Instance identifier

   "udi" : { // Unique Device Identifier (UDI) Barcode string

     "deviceIdentifier" : "<string>", // Mandatory fixed portion of UDI

     "name" : "<string>", // Device Name as appears on UDI label

     "jurisdiction" : "<uri>", // Regional UDI authority

     "carrierHRF" : "<string>", // UDI Human Readable Barcode String

     "carrierAIDC" : "<base64Binary>", // UDI Machine Readable Barcode String

     "issuer" : "<uri>", // UDI Issuing Organization

     "entryType" : "<code>" // barcode | rfid | manual +

   },

   "status" : "<code>", // active | inactive | entered-in-error | unknown

   "type" : { CodeableConcept }, // What kind of device this is

   "lotNumber" : "<string>", // Lot number of manufacture

   "manufacturer" : "<string>", // Name of device manufacturer

   "manufactureDate" : "<dateTime>", // Date when the device was made

   "expirationDate" : "<dateTime>", // Date and time of expiry of this device (if applicable)

   "model" : "<string>", // Model id assigned by the manufacturer

   "version" : "<string>", // Version number (i.e. software)

   "patient" : { Reference(Patient) }, // Patient to whom Device is affixed

   "owner" : { Reference(Organization) }, // Organization responsible for device

   "contact" : \[{ ContactPoint }\], // Details for human/organization for support

   "location" : { Reference(Location) }, // Where the resource is found

   "url" : "<uri>", // Network address to contact device

   "note" : \[{ Annotation }\], // Device notes and comments

   "safety" : \[{ CodeableConcept }\] // Safety Characteristics of Device

 }

\`\`\`

<div style="page-break-after: always;"></div>

\### DeviceComponent Document Template

This is referenced by the DeviceMetric Resource

 \`\`\`json

    {

      "resourceType" : "DeviceComponent",

      // from Resource: id, meta, implicitRules, and language

      // from DomainResource: text, contained, extension, and modifierExtension

      "identifier" : \[{ Identifier }\], // Instance identifier

      "type" : { CodeableConcept }, // R!  What kind of component it is

      "lastSystemChange" : "<instant>", // Recent system change timestamp

      "source" : { Reference(Device) }, // Top-level device resource link

      "parent" : { Reference(DeviceComponent) }, // Parent resource link

      "operationalStatus" : \[{ CodeableConcept }\], // Current operational status of the component, for example On, Off or Standby

      "parameterGroup" : { CodeableConcept }, // Current supported parameter group

      "measurementPrinciple" : "<code>", // other | chemical | electrical | impedance | nuclear | optical | thermal | biological | mechanical | acoustical | manual+

      "productionSpecification" : \[{ // Specification details such as Component Revisions, or Serial Numbers

        "specType" : { CodeableConcept }, // Type or kind of production specification, for example serial number or software revision

        "componentId" : { Identifier }, // Internal component unique identification

        "productionSpec" : "<string>" // A printable string defining the component

      }\],

      "languageCode" : { CodeableConcept }, // Language code for the human-readable text strings produced by the device

      "property" : \[{ // Other Attributes

        "type" : { CodeableConcept }, // R!  Code that specifies the property

        "valueQuantity" : \[{ Quantity }\], // Property value as a quantity

        "valueCode" : \[{ CodeableConcept }\] // Property value as a code

      }\]

    }

\`\`\`

<div style="page-break-after: always;"></div>

\#### Structure of a DeviceComponent Resource

A Context Scanner object of a medical device that implements or derives

from ISO/IEEE 11073 standard is responsible for observing device

configuration changes. After instantiation, the Context Scanner object

is responsible for announcing the object instances in the device's MDIB,

a hierarchical containment (MDS-\\>VMD-\\>Channel-\\>Metric). The

DeviceComponent resource can be used to describe the characteristics,

operational status and capabilities of a medical-related component of a

medical device. It can be a physical component that is integrated inside

the device, a removable physical component, or a non-physical component

that allows physiological measurement data and its derived data to be

grouped in a hierarchical information organization. Devices are

conceptualized using the following main structure:\\

1\. **MedicalDeviceSystem** - An actual device that external system

communicate with. In 11073, this is known as a MDS. 2.

\**VirtualMedicalDevice** - A medical-related subsystem of a medical

device. It can either be a physical hardware piece or a pure software

plugin component of a medical device. In 11073, this is known as a VMD.

3\. **Channel** - A non-physical component that allows physiological

measurement data and its derived data to be grouped in a hierarchical

information organization.

\### DeviceMetric Document Template

This is referenced by the Observation Resource 

 \`\`\`json

    {

      "resourceType" : "DeviceMetric",

      // from Resource: id, meta, implicitRules, and language

      // from DomainResource: text, contained, extension, and modifierExtension

      "identifier" : \[{ Identifier }\], // Instance identifier

      "type" : { CodeableConcept }, // R!  Identity of metric, for example Heart Rate or PEEP Setting

      "unit" : { CodeableConcept }, // Unit of Measure for the Metric

      "source" : { Reference(Device) }, // Describes the link to the source Device

      "parent" : { Reference(DeviceComponent) }, // Describes the link to the parent DeviceComponent

      "operationalStatus" : "<code>", // on | off | standby | entered-in-error

      "color" : "<code>", // black | red | green | yellow | blue | magenta | cyan | white

      "category" : "<code>", // R!  measurement | setting | calculation | unspecified

      "measurementPeriod" : { Timing }, // Describes the measurement repetition time

      "calibration" : \[{ // Describes the calibrations that have been performed or that are required to be performed

        "type" : "<code>", // unspecified | offset | gain | two-point

        "state" : "<code>", // not-calibrated | calibration-required | calibrated | unspecified

        "time" : "<instant>" // Describes the time last calibration has been performed

      }\]

    }

\`\`\`

<div style="page-break-after: always;"></div>

\### Observation Document Template

 \`\`\`json

    {

      "resourceType" : "Observation",

      // from Resource: id, meta, implicitRules, and language

      // from DomainResource: text, contained, extension, and modifierExtension

      "identifier" : \[{ Identifier }\], // Business Identifier for observation

      "basedOn" : \[{ Reference(CarePlan|DeviceRequest|ImmunizationRecommendation|

       MedicationRequest|NutritionOrder|ServiceRequest) }\], // Fulfills plan, proposal or order

      "partOf" : \[{ Reference(MedicationAdministration|MedicationDispense|

       MedicationStatement|Procedure|Immunization|ImagingStudy) }\], // Part of referenced event

      "status" : "<code>", // R!  registered | preliminary | final | amended +

      "category" : \[{ CodeableConcept }\], // Classification of  type of observation

      "code" : { CodeableConcept }, // R!  Type of observation (code / type)

      "subject" : { Reference(Patient|Group|Device|Location) }, // Who and/or what this is about

      "focus" : { Reference(Any) }, // The "focal point" of the observation

      "context" : { Reference(Encounter|EpisodeOfCare) }, // Healthcare event during which this observation is made

      // effective\[x\]: Clinically relevant time/time-period for observation. One of these 3:

      "effectiveDateTime" : "<dateTime>",

      "effectivePeriod" : { Period },

      "effectiveTiming" : { Timing },

      "issued" : "<instant>", // Date/Time this version was made available

      "performer" : \[{ Reference(Practitioner|PractitionerRole|Organization|

       CareTeam|Patient|RelatedPerson) }\], // Who is responsible for the observation

      // value\[x\]: Actual result. One of these 11:

      "valueQuantity" : { Quantity },

      "valueCodeableConcept" : { CodeableConcept },

      "valueString" : "<string>",

      "valueBoolean" : <boolean>,

      "valueInteger" : <integer>,

      "valueRange" : { Range },

      "valueRatio" : { Ratio },

      "valueSampledData" : { SampledData },

      "valueTime" : "<time>",

      "valueDateTime" : "<dateTime>",

      "valuePeriod" : { Period },

      "dataAbsentReason" : { CodeableConcept }, // C? Why the result is missing

      "interpretation" : { CodeableConcept }, // High, low, normal, etc.

      "comment" : "<string>", // Comments about result

      "bodySite" : { CodeableConcept }, // Observed body part

      "method" : { CodeableConcept }, // How it was done

      "specimen" : { Reference(Specimen) }, // Specimen used for this observation

      "device" : { Reference(Device|DeviceComponent|DeviceMetric) }, // (Measurement) Device

      "referenceRange" : \[{ // Provides guide for interpretation

        "low" : { Quantity(SimpleQuantity) }, // C? Low Range, if relevant

        "high" : { Quantity(SimpleQuantity) }, // C? High Range, if relevant

        "type" : { CodeableConcept }, // Reference range qualifier

        "appliesTo" : \[{ CodeableConcept }\], // Reference range population

        "age" : { Range }, // Applicable age range, if relevant

        "text" : "<string>" // Text based reference range in an observation

      }\],

      "hasMember" : \[{ Reference(Observation|QuestionnaireResponse|Sequence) }\], // Related resource that belongs to the Observation group

      "derivedFrom" : \[{ Reference(DocumentReference|ImagingStudy|Media|

       QuestionnaireResponse|Observation|Sequence) }\], // Related measurements the observation is made from

      "component" : \[{ // Component results

        "code" : { CodeableConcept }, // R!  Type of component observation (code / type)

        // value\[x\]: Actual component result. One of these 11:

        "valueQuantity" : { Quantity },

        "valueCodeableConcept" : { CodeableConcept },

        "valueString" : "<string>",

        "valueBoolean" : <boolean>,

        "valueInteger" : <integer>,

        "valueRange" : { Range },

        "valueRatio" : { Ratio },

        "valueSampledData" : { SampledData },

        "valueTime" : "<time>",

        "valueDateTime" : "<dateTime>",

        "valuePeriod" : { Period },

        "dataAbsentReason" : { CodeableConcept }, // C? Why the component result is missing

        "interpretation" : { CodeableConcept }, // High, low, normal, etc.

        "referenceRange" : \[{ Content as for Observation.referenceRange }\] // Provides guide for interpretation of component result

      }\]

    }

\`\`\`

<div style="page-break-after: always;"></div>

\### Location Document Template

\`\`\`json

{

  "resourceType" : "Location",

  // from Resource: id, meta, implicitRules, and language

  // from DomainResource: text, contained, extension, and modifierExtension

  "identifier" : \[{ Identifier }\], // Unique code or number identifying the location to its users

  "status" : "<code>", // active | suspended | inactive

  "operationalStatus" : { Coding }, // The Operational status of the location (typically only for a bed/room)

  "name" : "<string>", // Name of the location as used by humans

  "alias" : \["<string>"\], // A list of alternate names that the location is known as or was known as in the past

  "description" : "<string>", // Additional details about the location that could be displayed as further information to identify the location beyond its name

  "mode" : "<code>", // instance | kind

  "type" : \[{ CodeableConcept }\], // Type of function performed

  "telecom" : \[{ ContactPoint }\], // Contact details of the location

  "address" : { Address }, // Physical location

  "physicalType" : { CodeableConcept }, // Physical form of the location

  "position" : { // The absolute geographic location

    "longitude" : <decimal>, // R!  Longitude with WGS84 datum

    "latitude" : <decimal>, // R!  Latitude with WGS84 datum

    "altitude" : <decimal> // Altitude with WGS84 datum

  },

  "managingOrganization" : { Reference(Organization) }, // Organization responsible for provisioning and upkeep

  "partOf" : { Reference(Location) }, // Another Location this one is physically part of

  "hoursOfOperation" : \[{ // What days/times during a week is this location usually open

    "daysOfWeek" : \["<code>"\], // mon | tue | wed | thu | fri | sat | sun

    "allDay" : <boolean>, // The Location is open all day

    "openingTime" : "<time>", // Time that the Location opens

    "closingTime" : "<time>" // Time that the Location closes

  }\],

  "availabilityExceptions" : "<string>", // Description of availability exceptions

  "endpoint" : \[{ Reference(Endpoint) }\] // Technical endpoints providing access to services operated for the location

}

\`\`\`

\#### Location Example  

\`\`\`json

{

  "resourceType": "Location",

  "id": "2",

  "text": {

    "status": "generated",

    "div": "<div xmlns=\\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml "http://www.w3.org/1999/xhtml")\\">Burgers UMC, South Wing, second floor, Neuro Radiology Operation Room 1</div>"

  },

  "identifier": \[

    {

      "value": "B1-S.F2.1.00"

    }

  \],

  "status": "suspended",

  "operationalStatus": {

    "system": "[http://hl7.org/fhir/v2/0116](http://hl7.org/fhir/v2/0116 "http://hl7.org/fhir/v2/0116")",

    "code": "H",

    "display": "Housekeeping"

  },

  "name": "South Wing Neuro OR 1",

  "alias": \[

    "South Wing OR 5",

    "Main Wing OR 2"

  \],

  "description": "Old South Wing, Neuro Radiology Operation Room 1 on second floor",

  "mode": "instance",

  "type": \[

    {

      "coding": \[

        {

          "system": "[http://hl7.org/fhir/v3/RoleCode](http://hl7.org/fhir/v3/RoleCode "http://hl7.org/fhir/v3/RoleCode")",

          "code": "RNEU",

          "display": "Neuroradiology unit"

        }

      \]

    }

  \],

  "telecom": \[

    {

      "system": "phone",

      "value": "2329"

    }

  \],

  "physicalType": {

    "coding": \[

      {

        "system": "[http://hl7.org/fhir/location-physical-type](http://hl7.org/fhir/location-physical-type "http://hl7.org/fhir/location-physical-type")",

        "code": "ro",

        "display": "Room"

      }

    \]

  },

  "managingOrganization": {

    "reference": "Organization/f001"

  },

  "partOf": {

    "reference": "Location/1"

  }

}

\`\`\`

<div style="page-break-after: always;"></div>

Profiles

\--------

\### Point-of-Care Device

A point-of-care device model is made of multiple resource instances and the relationships between them. The following diagram shows the FHIR resource profiles (dark) and references to other profiles or FHIR core resources (light).

!\[Point of Care Profile\](![](https://i.imgur.com/ClULmQl.png))

Point of Care Profile

\### Observation Profiles

There following table shows the Observation Profile Content

| StructureDefinition                  | Description                              | Example                                  |

|--------------------------------------|------------------------------------------|------------------------------------------|

| Numeric Observation Profile          | Actual value of a numerical measurement, calculation, or setting | \[Heart Rate Numeric Observation\](#Heart-Rate-Numeric-Observation-Example) |

| Compound Numeric Observation Profile | Actual value of a numerical measurement, calculation, or setting with multiple components | \[NBP Compound Numeric Observation\](#NBP-Compound-Numeric-Observation-Example) |

| Enumeration Observation Profile      | Actual value of status or annotation information as codes or text | \[Rhythm Status Enumeration Observation\](#Rhythm-Status-Enumeration-Observation-Example) |

| Array Observation Profile            | Actual value (wave samples) of a real-time waveform or wave snippet |                                          |

| Extension                     | Extension                                | Example                                  |

|-------------------------------|------------------------------------------|------------------------------------------|

| Measurement Status Extension  | Status information about the observed value   | \[SpO2 Numeric Observation\](#SpO2-Numeric-Observation-Example) |

The following are the examples listed in the table above.

\##### Heart Rate Numeric Observation Example

 \`\`\`json

    {

        "resourceType": "Observation",

        "id": "622",

        "meta":

        {

            "profile":

            \[

                "[http://devices.fhir.org/StructureDefinition/NumericObservation](http://devices.fhir.org/StructureDefinition/NumericObservation "http://devices.fhir.org/StructureDefinition/NumericObservation")"

            \]

        },

        "status": "final",

        "category":

        \[

            {

                "coding":

                \[

                    {

                        "system": "[http://hl7.org/fhir/observation-category](http://hl7.org/fhir/observation-category "http://hl7.org/fhir/observation-category")",

                        "code": "vital-signs",

                        "display": "Vital Signs"

                    }

                \]

            }

        \],

        "code":

        {

            "coding":

            \[

                {

                    "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

                    "code": "8867-4",

                    "display": "Heart rate"

                },

                {

                    "system": "urn:iso:std:iso:11073:10101",

                    "code": "147842",

                    "display": "MDC_ECG_CARD_BEAT_RATE"

                }

            \]

        },

        "subject":

        {

            "reference": "Patient/371"

        },

        "effectiveDateTime": "2017-06-02T11:04:46+02:00",

        "valueQuantity":

        {

            "value": "60",

            "unit": "bpm",

            "system": "[http://unitsofmeasure.org](http://unitsofmeasure.org "http://unitsofmeasure.org")",

            "code": "/min"

        },

        "device":

        {

            "reference": "DeviceMetric/215"

        }

    }

\`\`\`

<div style="page-break-after: always;"></div>

\##### NBP Compound Numeric Observation Example

 \`\`\`json

    {

        "resourceType": "Observation",

        "id": "623",

        "meta":

        {

            "profile":

            \[

                "[http://devices.fhir.org/StructureDefinition/CompoundNumericObservation](http://devices.fhir.org/StructureDefinition/CompoundNumericObservation "http://devices.fhir.org/StructureDefinition/CompoundNumericObservation")"

            \]

        },

        "status": "final",

        "category":

        \[

            {

                "coding":

                \[

                    {

                        "system": "[http://hl7.org/fhir/observation-category](http://hl7.org/fhir/observation-category "http://hl7.org/fhir/observation-category")",

                        "code": "vital-signs",

                        "display": "Vital Signs"

                    }

                \]

            }

        \],

        "code":

        {

            "coding":

            \[

                {

                    "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

                    "code": "85354-9",

                    "display": "Blood pressure panel with all children optional"

                },

                {

                    "system": "urn:iso:std:iso:11073:10101",

                    "code": "150020",

                    "display": "MDC_PRESS_BLD_NONINV"

                }

            \]

        },

        "subject":

        {

            "reference": "Patient/371"

        },

        "effectiveDateTime": "2017-06-02T11:00:59+02:00",

        "device":

        {

            "reference": "DeviceMetric/216"

        },

        "component":

        \[

            {

                "code":

                {

                    "coding":

                    \[

                        {

                            "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

                            "code": "8480-6",

                            "display": "Systolic blood pressure"

                        },

                        {

                            "system": "urn:iso:std:iso:11073:10101",

                            "code": "150021",

                            "display": "MDC_PRESS_BLD_NONINV_SYS"

                        }

                    \]

                },

                "valueQuantity":

                {

                    "value": "120",

                    "unit": "mmHg",

                    "system": "[http://unitsofmeasure.org](http://unitsofmeasure.org "http://unitsofmeasure.org")",

                    "code": "mm\[Hg\]"

                }

            },

            {

                "code":

                {

                    "coding":

                    \[

                        {

                            "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

                            "code": "8462-4",

                            "display": "Diastolic blood pressure"

                        },

                        {

                            "system": "urn:iso:std:iso:11073:10101",

                            "code": "150022",

                            "display": "MDC_PRESS_BLD_NONINV_DIA"

                        }

                    \]

                },

                "valueQuantity":

                {

                    "value": "80",

                    "unit": "mmHg",

                    "system": "[http://unitsofmeasure.org](http://unitsofmeasure.org "http://unitsofmeasure.org")",

                    "code": "mm\[Hg\]"

                }

            },

            {

                "code":

                {

                    "coding":

                    \[

                        {

                            "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

                            "code": "8478-0",

                            "display": "Mean blood pressure"

                        },

                        {

                            "system": "urn:iso:std:iso:11073:10101",

                            "code": "150023",

                            "display": "MDC_PRESS_BLD_NONINV_MEAN"

                        }

                    \]

                },

                "valueQuantity":

                {

                    "value": "90",

                    "unit": "mmHg",

                    "system": "[http://unitsofmeasure.org](http://unitsofmeasure.org "http://unitsofmeasure.org")",

                    "code": "mm\[Hg\]"

                }

            }

        \]

    }

\`\`\`

<div style="page-break-after: always;"></div>

\##### Rhythm Status Enumeration Observation Example

 \`\`\`json

    {

        "resourceType": "Observation",

        "id": "631",

        "meta":

        {

            "profile":

            \[

                "[http://devices.fhir.org/StructureDefinition/EnumerationObservation](http://devices.fhir.org/StructureDefinition/EnumerationObservation "http://devices.fhir.org/StructureDefinition/EnumerationObservation")"

            \]

        },

        "status": "final",

        "code":

        {

            "coding":

            \[

                {

                    "system": "urn:iso:std:iso:11073:10101",

                    "code": "184327",

                    "display": "MDC_ECG_STAT_RHY"

                }

            \]

        },

        "subject":

        {

            "reference": "Patient/371"

        },

        "effectiveDateTime": "2017-06-02T11:04:44+02:00",

        "valueCodeableConcept":

        {

            "coding":

            \[

                {

                    "system": "urn:iso:std:iso:11073:10101",

                    "code": "147474",

                    "display": "MDC_ECG_SINUS_RHY"

                }

            \],

            "text": "Sinus rhythm"

        },

        "device":

        {

            "reference": "DeviceMetric/221"

        }

    }

\`\`\`

<div style="page-break-after: always;"></div>

\##### SpO2 Numeric Observation Example

 \`\`\`json

    {

        "resourceType": "Observation",

        "id": "632",

        "meta":

        {

            "profile":

            \[

                "[http://devices.fhir.org/StructureDefinition/NumericObservation](http://devices.fhir.org/StructureDefinition/NumericObservation "http://devices.fhir.org/StructureDefinition/NumericObservation")",

                "[http://hl7.org/fhir/StructureDefinition/oxygensat](http://hl7.org/fhir/StructureDefinition/oxygensat "http://hl7.org/fhir/StructureDefinition/oxygensat")"

            \]

        },

        "extension":

        \[

            {

                "url": "[http://devices.fhir.org/StructureDefinition/observation-measurement-status](http://devices.fhir.org/StructureDefinition/observation-measurement-status "http://devices.fhir.org/StructureDefinition/observation-measurement-status")",

                "valueCode": "questionable"

            },

            {

                "url": "[http://devices.fhir.org/StructureDefinition/observation-measurement-status](http://devices.fhir.org/StructureDefinition/observation-measurement-status "http://devices.fhir.org/StructureDefinition/observation-measurement-status")",

                "valueCode": "msmt-state-in-alarm"

            }

        \],

        "status": "final",

        "category":

        \[

            {

                "coding":

                \[

                    {

                        "system": "[http://hl7.org/fhir/observation-category](http://hl7.org/fhir/observation-category "http://hl7.org/fhir/observation-category")",

                        "code": "vital-signs",

                        "display": "Vital Signs"

                    }

                \]

            }

        \],

        "code":

        {

            "coding":

            \[

                {

                    "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

                    "code": "59408-5",

                    "display": "Oxygen saturation in Arterial blood by Pulse oximetry"

                },

                {

                    "system": "urn:iso:std:iso:11073:10101",

                    "code": "150456",

                    "display": "MDC_PULS_OXIM_SAT_O2"

                }

            \]

        },

        "subject":

        {

            "reference": "Patient/371"

        },

        "effectiveDateTime": "2017-06-02T11:04:48+02:00",

        "valueQuantity":

        {

            "value": "88",

            "unit": "%",

            "system": "[http://unitsofmeasure.org](http://unitsofmeasure.org "http://unitsofmeasure.org")",

            "code": "%"

        },

        "device":

        {

            "reference": "DeviceMetric/222"

        }

    }

\`\`\`

Summary

\-------

\-   You have gained knowledge about the HL7 FHIR Resources for Devices.

\-   You have a basic understanding of the Observation Resource Profile.

\-   We have shown you the HL7 FHIR Resource document templates.

\-   You have been shown examples for the different types of Observation

    Profiles.

Next Steps

\----------

Now let's see how to configure our data repository

<div style="page-break-after: always;"></div>

\# CHAPTER 2 - CONFIGURING OUR DATA REPOSITORY

We have already created an Azure Cosmos DB account and added a database, which we named **PointofCare**. We are going to use the <a href="[https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-introduction](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-introduction "https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-introduction")" target="_blank">SQL API Model</a>.  

\## Now we need to create two Document Collections. 

\* One will be used for Device, DeviceComponent, and DeviceMetric and named **Devices**.

\* The other for Observations and named **Diagnostics**

\### Why two different Collections?  

We are going to <a href="[https://docs.microsoft.com/en-us/azure/cosmos-db/partition-data](https://docs.microsoft.com/en-us/azure/cosmos-db/partition-data "https://docs.microsoft.com/en-us/azure/cosmos-db/partition-data")" target="_blank">**Partition**</a> our collections. The HL7 FHIR Resources in both Collections do not have an unique node that we can use for a partition key. 

Let's look again at the the **Point-of-Care device model diagram** 

\---------

!\[Point of Care Profile\](![](https://i.imgur.com/ClULmQl.png))  

<u>FHIR resource profiles (dark) and references to other profiles or FHIR core resources (light).</u>

 

 

 

\#### Partition Keys

Looking at the diagram above, there are two references, **Location** and **Patient**. Location is specifically referenced by the Device Resource, where the Patient Resource is referenced by both the Device and Observation Resources. 

\* Our **Devices Collection** partition key will be <a href="[https://www.hl7.org/fhir/location.html](https://www.hl7.org/fhir/location.html "https://www.hl7.org/fhir/location.html")" target="_blank">Location</a>. The HL7 FHIR Device Resource has a \`Device.Location\` node. 

\* The **Diagnostics Collection** will use the \`Observation.subject\` node. The subject can be a <a href="[https://hl7.org/fhir/2018May/patient.html](https://hl7.org/fhir/2018May/patient.html "https://hl7.org/fhir/2018May/patient.html")" target="_blank">Patient</a>, <a href="[https://hl7.org/fhir/2018May/group.html](https://hl7.org/fhir/2018May/group.html "https://hl7.org/fhir/2018May/group.html")" target="_blank">Group</a>, <a href="[https://www.hl7.org/fhir/device.html](https://www.hl7.org/fhir/device.html "https://www.hl7.org/fhir/device.html")" target="_blank">Device</a>, or <a href="[https://hl7.org/fhir/2018May/location.html](https://hl7.org/fhir/2018May/location.html "https://hl7.org/fhir/2018May/location.html")" target="_blank">Location</a>.  **We will be using the Patient Resource**. 

> !\[\](![](https://i.imgur.com/eSPsKWl.png)) INFO:

>

>Almost every element in HL7 FHIR Resources is optional. 

>Both Location and Patient have a cardinality of \`0.1\`.  Because Partition Key values are required, we are going to change these to \`1.1\`.  

 

\#### Reference Resource

Resources contain two types of references:

\* Internal "contained" references - references to other resources packaged inside the source resource.

\* External references - references to resources found elsewhere.  

  

References are always defined and represented in one particular direction - from one resource (source) to another (target). References are either provided as a literal URL, which may either be absolute or relative, or as a logical identifier.  

In a resource, references are represented with a reference (literal reference), an identifier (logical reference), and a display (text description of target).  See Listing 1. below.

Listing 1. Reference Resource Template

 

\`\`\`json

{

  // from Element: extension

  "reference" : "<string>", // C? Literal reference, Relative, internal or absolute URL

  "identifier" : { Identifier }, // Logical reference, when literal reference is not known

  "display" : "<string>" // Text alternative for the resource

}

\`\`\`

\-----------

\## Additional FHIR Resources

In addition to the above FHIR Resources, there are two additional resources that are used within a <a href="https:///hl7.org/fhir/2018May/workflow.html" target="_blank">Workflow</a>.  These Resources are stored in the  **Diagnostics Collection**.

\#### The following table describes these Resources

| Resource Name         | Type     | Description                              |

|-----------------------|----------|------------------------------------------|

| <a href="https:///hl7.org/fhir/2018May/devicerequest.html" target="_blank">Device Request</a>        | <a href="https:///hl7.org/fhir/2018May/request.html" target="_blank">Requests</a> | This resource describes the request for the use of a device by a patient. The device may be any pertinent device specified in the Device resource. Examples of devices that may be requested include wheelchair, hearing aids, or an insulin pump. The request may lead to the dispensing of the device to the patient or for use by the patient. |

| <a href="https:///hl7.org/fhir/2018May/deviceusestatement.html" target="_blank">Device Use Statement</a>  | <a href="https:///hl7.org/fhir/2018May/event.html" target="_blank">Event</a>    | This resource records the use of a healthcare-related device by a patient. The record is the result of a report of use by the patient, another provider or a related person. The resource can be used to note the use of an assistive device such as a wheelchair or hearing aid, a contraceptive such an intra-uterine device, or other implanted devices such as a pacemaker. |

> !\[NOTE\](![](https://i.imgur.com/DiPYycK.png))   NOTE: We will see how these Resources are used later.

\## Summary

\* You have learned how to configure the Azure Cosmos DB Database.

\* You have learned about the Collections required.

\* You have learned about Partition Keys and which ones will be used.

\* We have explained how we selected the Partition Keys for the Collections.

\## Next Steps 

Comparison of Cosmos DB Document Attachment Storage methods 

<div style="page-break-after: always;"></div>

\# CHAPTER 3 - AZURE COSMOS DB DOCUMENT ATTACHMENT STORAGE COMPARISON

Recently I was working on an IoT solution for an international Safety Products manufacturer. One of the micro-service we were designing was to handle updating of device software.  

One of the requirements was to be able to have the software updates as close as possible to the device charging station. There software updates were archived into Zip files. 

\### There are two methods for creating a Azure Cosmos DB Document Attachment. 

\#### Store the file as an attachment to a Document 

The raw attachment is included as the body of the POST.   

Two headers must be set: 

\* **Slug** - The name of the attachment.

\* **contentType** - Set to the MIME type of the attachment.

\#### Store the URL for the file in an attachment to a Document

The body for the POST include the following.

 

\* **id** - It is the unique name that identifies the attachment, i.e. no two attachments will share the same id. The id must not exceed 255 characters.

\* **Media** - This is the URL link or file path where the attachment resides.

> **INFO**: You can read more about creating an attachment <a href="[https://docs.microsoft.com/en-us/rest/api/documentdb/create-an-attachment#remarks](https://docs.microsoft.com/en-us/rest/api/documentdb/create-an-attachment#remarks "https://docs.microsoft.com/en-us/rest/api/documentdb/create-an-attachment#remarks")" target="_blank">here</a>

The following is an example 

\`\`\`json

{  

    "id": "device\\A234",  

    "contentType": "application/x-zip-compressed",  

    "media": "www.bing.com/A234.zip"  

}

\`\`\`

\### Which method works best?

Let's see how they compare.

\## Storage Comparison

| Service                        | Global Distribution             | Storage                                  | Limitations                          |

|-------------------------------:|---------------------------------|------------------------------------------|--------------------------------------|

| **<a href="[https://azure.microsoft.com/en-us/services/storage/blobs/](https://azure.microsoft.com/en-us/services/storage/blobs/ "https://azure.microsoft.com/en-us/services/storage/blobs/")" target="_blank">Blob Storage</a>**                   | Only 1 region                   | None                                     | Unlimited                            |

| **<a href="[https://docs.microsoft.com/en-us/rest/api/documentdb/attachments](https://docs.microsoft.com/en-us/rest/api/documentdb/attachments "https://docs.microsoft.com/en-us/rest/api/documentdb/attachments")" target="_blank">Document Attachment</a>**            | Available for all Azure Regions | Supported                                | Limited to 2GB per Cosmos DB Account |

| **<a href="[https://azure.microsoft.com/en-us/services/cdn/](https://azure.microsoft.com/en-us/services/cdn/ "https://azure.microsoft.com/en-us/services/cdn/")" target="_blank">Content Delivery Network (CDN)</a>** | Available for all Azure Regions | You can enable Azure Content Delivery Network (CDN) to cache content from Azure storage. Azure CDN offers developers a global solution for delivering high-bandwidth content. It can cache blobs and static content of compute instances at physical nodes in the United States, Europe, Asia, Australia, and South America. | Unlimited                            |

\------------

\## Conclusion

\* Since the total size of all the attachment files is an unknown, attaching the file to the document would not work. 

\* Blob Storage is out since there is only one region available.

\* Because it supports caching and multi-region instances, Azure Content Delivery Network (CDN) is the only storage solutions that meets my client's requirements

\## Next Steps

<div style="page-break-after: always;"></div>

\# CHAPTER 4 - POSTING DEVICE SOFTWARE UPDATE TO HL7 FHIR SERVER (AZURE COSMOS DB)

\## Functional Requirements 

\* The service would support an Multi-Tenancy Architecture.

\* Ability to have the software updates as close as possible to their customers Medical Device System (MDS), Virtual Medical Device (VMD) or a Channel.

\* A Form-Post from a web application will be used to send the software update file, along with metadata to the Software Update Service.

\* All Device Data was to be stored as <a href="[https://www.hl7.org/fhir/](https://www.hl7.org/fhir/ "https://www.hl7.org/fhir/")" target="_blank">HL7 FHIR Resources</a>. 

\* All data must be encrypted at Rest and Intransit.

\## Architecture

The following diagram shows the data flow 

!\[Form-Data\](![](https://i.imgur.com/iKtPzMb.png))

\----------------

\## Steps

1\. Web App does a Form-Data POST to Function App.

2\. Function App Inserts Software Update file in Azure Blob Storage.

3\. Function App updates HL7 FHIR DeviceComponent Resource (document).

4\. Function App returns Response to Web App. 

\## FormData Function App  

There are two examples. One stores our Cosmos DB Account Key in the configuration and the other stores it in a Azure KeyVault.

\#### FormData Function App Source Code

\`\`\`csharp

\#region

using System;

using System.Configuration;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using HttpMultipartParser;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.WindowsAzure.Storage;

using Attachment = Microsoft.Azure.Documents.Attachment;

\#endregion

namespace CosmosDBFormData

{

	public static class FormData

	{

		private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

		private static readonly string AuthKey = ConfigurationManager.AppSettings\["authKey"\];

		private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

		private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

	

		private static readonly string ConnectionString = ConfigurationManager.AppSettings\["storageConnectionString"\];

		private static readonly string BlobContainer = ConfigurationManager.AppSettings\["blobContainer"\];

		private static readonly string CdnUrl = ConfigurationManager.AppSettings\["cdnUrl"\];

		private static readonly DocumentClient Client = new DocumentClient(new Uri(Endpoint), AuthKey,

			new ConnectionPolicy

			{

				ConnectionMode = ConnectionMode.Direct,

				ConnectionProtocol = Protocol.Tcp

			}

		);

		

		\[FunctionName("FormData")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "post", Route = null)\]

			HttpRequestMessage req, TraceWriter log)

		{

		

			var storageAccount = CloudStorageAccount.Parse(ConnectionString);

			var cloudBlobClient = storageAccount.CreateCloudBlobClient();

			var cloudBlobContainer = cloudBlobClient.GetContainerReference(BlobContainer);

			var data = req.Content.ReadAsStreamAsync();

			var parser = new MultipartFormDataParser(data.Result);

			var deviceId = parser.GetParameterValue("DeviceId");

			var file = parser.Files.FirstOrDefault();

			

			if (file == null)

			{

			

				return req.CreateErrorResponse(HttpStatusCode.BadRequest, "The Attachment File is missing");

			}

			var fileName = file.FileName;

			var attachment = file.Data;

			var feedOptions = new FeedOptions {MaxItemCount = 1};

			try

			{

				// Get the DeviceComponent document

				var doc = Client.CreateDocumentQuery(UriFactory.CreateDocumentCollectionUri(Database, Collection),

					$"SELECT * FROM d WHERE d.resourceType = \\'DeviceComponent\\' AND d.source.reference = \\' Device/\\'{deviceId}", feedOptions);

				Document d = doc.FirstOrDefault();

				try

				{

					var cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(fileName);

					try

					{

						await cloudBlockBlob.UploadFromStreamAsync(attachment);

					}

					catch (StorageException ex)

					{

						return req.CreateErrorResponse(HttpStatusCode.NotFound, ex.Message);

					}

					try

					{

						await Client.CreateAttachmentAsync(d.Id, new Attachment

						{

							Id = fileName,

							ContentType = file.ContentType,

							MediaLink = $"{CdnUrl}/{fileName}"

						});

						try

						{

							var update = await Client.UpsertDocumentAsync(UriFactory.CreateDocumentCollectionUri(Database, Collection), d);

							return req.CreateResponse(HttpStatusCode.OK,update.Resource.Id);

						}

						catch (DocumentClientException ex)

						{

							var statusCode = ex.StatusCode;

							return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

						}

					}

					catch (DocumentClientException ex)

					{

						var statusCode = ex.StatusCode;

						return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

					}

				}

				catch (DocumentClientException ex)

				{

					var statusCode = ex.StatusCode;

					return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

				}

			}

			catch (DocumentClientException ex)

			{

				var statusCode = ex.StatusCode;

				return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

			}

		}

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\### The following example shows how we can leverage Azure KeyVault to store our Cosmos DB Key.

\#### FormDataSecure Function App

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Configuration;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using HttpMultipartParser;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.KeyVault;

using Microsoft.Azure.Services.AppAuthentication;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.WindowsAzure.Storage;

\#endregion

namespace CosmosDBFormData

{

	public static class FormDataSecure

	{

		private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

		private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

		private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

		private static readonly string ConnectionString = ConfigurationManager.AppSettings\["storageConnectionString"\];

		private static readonly string BlobContainer = ConfigurationManager.AppSettings\["blobContainer"\];

		private static readonly string CdnUrl = ConfigurationManager.AppSettings\["cdnUrl"\];

		private static readonly HttpClient HttpClient = new HttpClient();

		public static DocumentClient Client;

		\[FunctionName("FormDataSecure")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "post", Route = null)\]

			HttpRequestMessage req, TraceWriter log)

		{

			if (Client == null) Client = await GetSecureDocumentClient();

			var storageAccount = CloudStorageAccount.Parse(ConnectionString);

			var cloudBlobClient = storageAccount.CreateCloudBlobClient();

			var cloudBlobContainer = cloudBlobClient.GetContainerReference(BlobContainer);

			var data = req.Content.ReadAsStreamAsync();

			var parser = new MultipartFormDataParser(data.Result);

			// HL7 FHIR Device resource document

			var deviceId = parser.GetParameterValue("DeviceId");

			var file = parser.Files.FirstOrDefault();

			if (file == null) return req.CreateErrorResponse(HttpStatusCode.BadRequest, "The Attachment File is missing");

			var fileName = file.FileName;

			var attachment = file.Data;

			var feedOptions = new FeedOptions {MaxItemCount = 1};

			try

			{

				// Get the DeviceComponent document

				var doc = Client.CreateDocumentQuery(UriFactory.CreateDocumentCollectionUri(Database, Collection),

					$"SELECT * FROM d WHERE d.resourceType = \\'DeviceComponent\\' AND d.source.reference = \\' Device/\\'{deviceId}",

					feedOptions);

				if (doc == null){

					return req.CreateResponse(HttpStatusCode.NotFound);

				}

				Document document = doc.FirstOrDefault();

				try

				{

					var cloudBlockBlob = cloudBlobContainer.GetBlockBlobReference(fileName);

					try

					{

						await cloudBlockBlob.UploadFromStreamAsync(attachment);

					}

					catch (StorageException ex)

					{

						return req.CreateErrorResponse(HttpStatusCode.NotFound, ex.Message);

					}

					try

					{

						await Client.CreateAttachmentAsync(document.Id, new Attachment

						{

							Id = fileName,

							ContentType = file.ContentType,

							MediaLink = $"{CdnUrl}/{fileName}"

						});

						try

						{

							var update = await Client.UpsertDocumentAsync(UriFactory.CreateDocumentCollectionUri(Database, Collection), document);

							return req.CreateResponse(HttpStatusCode.OK, update.Resource.Id);

						}

						catch (DocumentClientException ex)

						{

							var statusCode = ex.StatusCode;

							return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

						}

					}

					catch (DocumentClientException ex)

					{

						var statusCode = ex.StatusCode;

						return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

					}

				}

				catch (DocumentClientException ex)

				{

					var statusCode = ex.StatusCode;

					return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

				}

			}

			catch (DocumentClientException ex)

			{

				var statusCode = ex.StatusCode;

				return req.CreateErrorResponse((HttpStatusCode) statusCode, ex.Message);

			}

		}

		private static async Task<DocumentClient> GetSecureDocumentClient()

		{

			var azureServiceTokenProvider = new AzureServiceTokenProvider();

			var endpointUrl = Endpoint;

			var keyVaultClient =

				new KeyVaultClient(new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback),

					HttpClient);

			var key = await keyVaultClient.GetSecretAsync(ConfigurationManager.AppSettings\["keyVaultAccessUri"\]);

			return new DocumentClient(new Uri(endpointUrl), key.Value);

		}

	}

}

\`\`\`

\## Summary

\* You have seen how easy it is to use an Azure Function App to receive a Form-Post.

\* You have learned how to save the Software Update file to CDN as an Cosmos DB Document Attachment.

\* You have also seen how easy it is to use Azure KeyVault to store our Cosmos DB Account Key.

\## Next Steps

<div style="page-break-after: always;"></div>

\# CHAPTER 5 - USING AZURE DURABLE FUNCTIONS FOR UPDATING HEALTHCARE DEVICES 

In the previous chapter we have learned how to save the Software Update file to CDN as an Cosmos DB Document Attachment. Now we are going to use Azure Durable Functions to do the following:

1\. Receive a HTTP POST from the Web Application or Service.

2\. Query the HL7 FHIR Server (Azure Cosmos DB) for the DeviceComponent resource (Document).

3\. Get the Software Update file URL from the Document Attachment.

4\. Invoke the  \`firmwareUpdate\` operation.

5\. Query the Device Twin for the last updated date and time reported.

6\. Update the DeviceComponent resource (Document) 

> !\[\](![](https://i.imgur.com/dMrFdY8.png)) NOTE: 

>

> Azure Durable Functions is an extension of <a href="[https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview "https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview")" target="_blank">Azure Functions</a> and <a href="[https://docs.microsoft.com/en-us/azure/app-service/web-sites-create-web-jobs](https://docs.microsoft.com/en-us/azure/app-service/web-sites-create-web-jobs "https://docs.microsoft.com/en-us/azure/app-service/web-sites-create-web-jobs")" target="_blank">Azure Webjobs</a>. that lets you write stateful functions in a serverless environment. The extension manages state, checkpoints, and restarts for you.

>

> Azure Durable Functions is built on the <a href="[https://github.com/Azure/durabletask](https://github.com/Azure/durabletask "https://github.com/Azure/durabletask")" target="_blank">Azure Durable Task Framework</a>.

>

>Primary advantages of using Azure Durable Functions 

>* They define workflows in code. No JSON schemas or designers are needed.

>* They can call other functions synchronously and asynchronously. Output from called functions can be saved to local variables.

>* They automatically checkpoint their progress whenever the function awaits. Local state is never lost if the process recycles

>

>!\[\](![](https://i.imgur.com/DiPYycK.png)) To learn more about **Azure Durable Functions**, you can read the complete <a href="[https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview](https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview "https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview")" target="_blank">documentation</a>.  It will help you understand the source code for this service.

 

\## Architecture

!\[Software Update Architecture\](![](https://i.imgur.com/dReOKTe.png))

\--------------  

\## Requirements

The following parameters are required in the Post Body.  

| Name     | Description                  |

|----------|-------------------------------|

| deviceId | The Id for the Device         |

| version  | The version id for the update |

| type     | The <a href="[https://www.hl7.org/fhir/valueset-specification-type.html](https://www.hl7.org/fhir/valueset-specification-type.html "https://www.hl7.org/fhir/valueset-specification-type.html")" target="_blank">operation</a> type:  hardware-revison, software-revison, firmware-revision, protocol-revision, unspecified, and <a href="[https://www.gmdnagency.org/About/Database](https://www.gmdnagency.org/About/Database "https://www.gmdnagency.org/About/Database")" target="_blank">gmdn</a>        |

\------------

<div style="page-break-after: always;"></div>

\## Examples 

\#### The following is a example of a <a href="[https://www.hl7.org/fhir/devicecomponent.html](https://www.hl7.org/fhir/devicecomponent.html "https://www.hl7.org/fhir/devicecomponent.html")" target="_blank">DeviceComponent</a> Document

\`\`\`json

{

  "resourceType": "DeviceComponent",

  "id": "example-prodspec",

  "text": {

    "status": "generated",

    "div": "<div xmlns=\\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml "http://www.w3.org/1999/xhtml")\\"><p><b>Generated Narrative with Details</b></p><p><b>id</b>: example-prodspec</p><p><b>identifier</b>: 789123</p><p><b>type</b>: MDC_DEV_ANALY_SAT_O2_MDS <span>(Details : {urn:iso:std:iso:11073:10101 code '2000' = '2000', given as 'MDC_DEV_ANALY_SAT_O2_MDS'})</span></p><p><b>lastSystemChange</b>: 07/10/2014 2:45:00 PM</p><p><b>source</b>: <a>Device/d1</a></p><p><b>operationalStatus</b>: Off <span>(Details : {\[not stated\] code 'off' = 'off', given as 'Off'})</span></p><blockquote><p><b>productionSpecification</b></p><p><b>specType</b>: Serial number <span>(Details : {\[not stated\] code 'serial-number' = 'serial-number', given as 'Serial number'})</span></p><p><b>productionSpec</b>: xa-12324-b</p></blockquote><blockquote><p><b>productionSpecification</b></p><p><b>specType</b>: Hardware Revision <span>(Details : {\[not stated\] code 'hardware-revision' = 'hardware-revision', given as 'Hardware Revision'})</span></p><p><b>productionSpec</b>: 1.1</p></blockquote><blockquote><p><b>productionSpecification</b></p><p><b>specType</b>: Software Revision <span>(Details : {\[not stated\] code 'software-revision' = 'software-revision', given as 'Software Revision'})</span></p><p><b>productionSpec</b>: 1.12</p></blockquote><blockquote><p><b>productionSpecification</b></p><p><b>specType</b>: Firmware Revision <span>(Details : {\[not stated\] code 'firmware-revision' = 'firmware-revision', given as 'Firmware Revision'})</span></p><p><b>productionSpec</b>: 1.0.23</p></blockquote><p><b>languageCode</b>: en-US <span>(Details : {[http://tools.ietf.org/html/bcp47](http://tools.ietf.org/html/bcp47 "http://tools.ietf.org/html/bcp47") code 'en-US' = 'en-US)</span></p></div>"

  },

  "identifier": {

    "value": "789123"

  },

  "type": {

    "coding": \[

      {

        "system": "urn:iso:std:iso:11073:10101",

        "code": "2000",

        "display": "MDC_DEV_ANALY_SAT_O2_MDS"

      }

    \]

  },

  "lastSystemChange": "2014-10-07T14:45:00Z",

  "source": {

    "reference": "Device/d1"

  },

  "operationalStatus": \[

    {

      "coding": \[

        {

          "code": "off",

          "display": "Off"

        }

      \]

    }

  \],

  "productionSpecification": \[

    {

      "specType": {

        "coding": \[

          {

            "code": "serial-number",

            "display": "Serial number"

          }

        \]

      },

      "productionSpec": "xa-12324-b"

    },

    {

      "specType": {

        "coding": \[

          {

            "code": "hardware-revision",

            "display": "Hardware Revision"

          }

        \]

      },

      "productionSpec": "1.1"

    },

    {

      "specType": {

        "coding": \[

          {

            "code": "software-revision",

            "display": "Software Revision"

          }

        \]

      },

      "productionSpec": "1.12"

    },

    {

      "specType": {

        "coding": \[

          {

            "code": "firmware-revision",

            "display": "Firmware Revision"

          }

        \]

      },

      "productionSpec": "1.0.23"

    }

  \],

  "languageCode": {

    "coding": \[

      {

        "system": "[http://tools.ietf.org/html/bcp47](http://tools.ietf.org/html/bcp47 "http://tools.ietf.org/html/bcp47")",

        "code": "en-US"

      }

    \]

  }

}

\`\`\`

\--------------

<div style="page-break-after: always;"></div>

\## Source Code

\#### Software Update Source Code

The Visual Studio 2017 Application Target Framework is NET STANDARD 2.0

\`\`\`csharp

\#region

using System;

using System.Configuration;

using System.Globalization;

using System.IO;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using Hl7.Fhir.Model;

using Microsoft.Azure.Devices;

using Microsoft.Azure.Devices.Common.Extensions;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.WebJobs;

using Newtonsoft.Json;

using Task = System.Threading.Tasks.Task;

\#endregion

namespace SoftwareUpdate

{

	public static class Update

	{

		private static RegistryManager _registryManager;

		private static readonly string ConnString = ConfigurationManager.AppSettings\["connString"\];

		private static ServiceClient _client;

		private static string\[\] _targetDeviceId;

		private static string _documentId;

		private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

		private static readonly string AuthKey = ConfigurationManager.AppSettings\["authKey"\];

		private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

		private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

		private static readonly DocumentClient Client = new DocumentClient(new Uri(Endpoint), AuthKey,

			new ConnectionPolicy

			{

				ConnectionMode = ConnectionMode.Direct,

				ConnectionProtocol = Protocol.Tcp

			}

		);

		\[FunctionName("Update")\]

		public static async Task<string> RunOrchestrator(

			\[OrchestrationTrigger\] DurableOrchestrationContext context)

		{

			_targetDeviceId = context.GetInput<string\[\]>();

			var deviceId = _targetDeviceId\[0\];

			var updateVersionId = _targetDeviceId\[1\];

			var specType = _targetDeviceId\[2\];

			_registryManager = RegistryManager.CreateFromConnectionString(ConnString);

			// Get the DeviceComponent Document

			var doc = await context.CallActivityAsync<dynamic>("GetDoc", deviceId);

			_documentId = doc.Id;

			var spec = new DeviceComponent.ProductionSpecificationComponent

			{

				ComponentId = {Value = Guid.NewGuid().ToString()},

				ProductionSpec = updateVersionId

			};

			var coding = new Coding(null, specType, specType.ToUpper());

			spec.SpecType.Coding.Add(coding);

			var ps = new DeviceComponent().ProductionSpecification;

			ps.Add(spec);

            

			var binder = new Binder();

			binder.BindingData.Add("documentId", _documentId);

			binder.BindingData.Add("attachmentId", updateVersionId);

			// Get the Attachment

			string attachment = await context.CallActivityAsync<dynamic>("GetAttachment", binder);

            // Start the update

			var result = await context.CallActivityAsync<CloudToDeviceMethodResult>("StartUpdate", attachment);

			if (result.Status != 0)

				return result.Status.ToString();

            // Query the Device Twin

			var status = await context.CallActivityAsync<string>("QueryTwin", DateTime.Now);

			var instant = new DateTimeOffset(DateTime.Parse(status));

			// Update the LastSystemChange value

			var deviceComponentUpdate = new DeviceComponent

			{

				Id = _documentId,

				Type = doc.Type,

				Text = doc.Text,

				Source = doc.Source,

				Parent = doc.Parent,

				FhirComments = doc.FhirComments,

				ImplicitRules = doc.ImplicitRules,

				Identifier = doc.Identifier,

				OperationalStatus = doc.OperationalStatus,

				Contained = doc.Contained,

				LastSystemChange = instant,

				MeasurementPrinciple = doc.MeasurementPrinciple,

				ParameterGroup = doc.ParameterGroup,

				MeasurementPrincipleElement = doc.MeasurementPrincipleElement,

				LanguageCode = doc.LanguageCode,

				ProductionSpecification = ps,

				Language = doc.Language

			};

			var json = JsonConvert.SerializeObject(deviceComponentUpdate);

			// Update the Device Docuument

			var upDate = await context.CallActivityAsync<HttpResponseMessage>("UpdateDoc", json);

			return upDate.StatusCode.ToString();

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("QueryTwin")\]

		public static async Task<string> QueryTwinFwUpdateReported(\[ActivityTrigger\] DateTime startTime)

		{

			var lastUpdated = startTime;

			var deviceId = _targetDeviceId\[0\];

			while (true)

			{

				var twin = await _registryManager.GetTwinAsync(deviceId);

				if (twin.Properties.Reported.GetLastUpdated().ToUniversalTime() > lastUpdated.ToUniversalTime())

				{

					lastUpdated = twin.Properties.Reported.GetLastUpdated().ToUniversalTime();

					var status = twin.Properties.Reported\["iothubDM"\]\["firmwareUpdate"\]\["status"\].Value;

					if (status == "downloadFailed" || status == "applyFailed" || status == "applyComplete") return status;

					return lastUpdated.ToString(CultureInfo.InvariantCulture);

				}

				await Task.Delay(500);

			}

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("StartUpdate")\]

		public static async Task<CloudToDeviceMethodResult> StartFirmwareUpdate(\[ActivityTrigger\] string payLoad)

		{

			var deviceId = _targetDeviceId\[0\];

			using (var client = new WebClient())

			{

				var content = client.DownloadData(payLoad);

				using (var stream = new MemoryStream(content))

				{

					var file = JsonConvert.SerializeObject(stream);

					_client = ServiceClient.CreateFromConnectionString(ConnString);

					var method = new CloudToDeviceMethod("firmwareUpdate") {ResponseTimeout = TimeSpan.FromSeconds(30)};

					method.SetPayloadJson(file);

					var result = await _client.InvokeDeviceMethodAsync(deviceId, method);

					return result;

				}

			}

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("GetDoc")\]

		public static async Task<dynamic> GetDoc(\[ActivityTrigger\] string id)

		{

			try

			{

				var feedOptions = new FeedOptions {MaxItemCount = 1};

				var doc = Client.CreateDocumentQuery(UriFactory.CreateDocumentCollectionUri(Database, Collection),

					$"SELECT * FROM d WHERE d.resourceType = \\'DeviceComponent\\' AND d.source.reference = \\' Device/\\'{id}", feedOptions);

				var document = doc.FirstOrDefault();

				return document;

			}

			catch (DocumentClientException)

			{

				return null;

			}

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("UpdateDoc")\]

		public static async Task<HttpResponseMessage> UpdateDoc(\[ActivityTrigger\] dynamic document)

		{

			string id = document.id;

			object doc = document.doc;

			try

			{

				await Client.UpsertDocumentAsync(UriFactory.CreateDocumentUri(Database, Collection, id), doc);

				return new HttpResponseMessage(HttpStatusCode.OK);

			}

			catch (DocumentClientException e)

			{

				var resp = new HttpResponseMessage

				{

					StatusCode = HttpStatusCode.BadRequest,

					Content = new StringContent(e.Message)

				};

				return resp;

			}

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("GetAttachment")\]

		public static async Task<dynamic> GetAttachment(\[ActivityTrigger\] Binder binder)

		{

			try

			{

				var documentId = binder.BindingData.GetValueOrDefault("documentId").ToString();

				var attachmentId = binder.BindingData.GetValueOrDefault("attachmentId").ToString();

				var result =

					await Client.ReadAttachmentAsync(UriFactory.CreateAttachmentUri(Database, Collection, documentId, attachmentId));

				return result.Resource.MediaLink;

			}

			catch (DocumentClientException e)

			{

				var resp = new HttpResponseMessage

				{

					StatusCode = HttpStatusCode.BadRequest,

					Content = new StringContent(e.Message)

				};

				return resp;

			}

		}

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\-------------------

\## Summary

\* We have learned how to use Azure Durable Functions to orchestrate the Software Update process.

\* You learned how to update a HL7 FHIR Device Component Resource stored in Azure Cosmos DB.

> !\[\](![](https://i.imgur.com/DiPYycK.png))  **INFO**:  We will be using the Azure Durable Functons Monitoring Design Pattern in our Logging Microservice.

\## Next Steps

Next we will see how easy it is to capture the data from a Device and store it in a HL7 <a href="[https://www.hl7.org/fhir/observation.html](https://www.hl7.org/fhir/observation.html "https://www.hl7.org/fhir/observation.html")" target="_blank">Observation Resource</a> in Azure Cosmos DB.

<div style="page-break-after: always;"></div>

\# CHAPTER 6 - SAVING MEDICAL DEVICE TO CLOUD MESSAGES AS HL7 FHIR OBSERVATION RESOURCES

We are going to learn how to get our Medical Device to Cloud data. 

The following are the Requirements.  

\## Requirements

\* Ability to capture our Medical Device output data.

\* Ability to filter the data.

\* Ability to update the Observation Resource document with the output data.

\* Ability to modify the \`EndPoint\` Response after updating the Observation Resource document.

\* Ability to parse the Observation Resource document and \`post\` it to any \`EndPoint\`.

\## Solution

\#### We are going to create a Function App, with a **EventHubTrigger** to do the following:

1\. Loop through the **EventHub** messages.

2\. Create a **EventGrid** message.

3\. Post the **EventGrid** message.

\#### Next we will create a Function App, \`DeviceGrid\`  that handles the updating of the Observation Resource document.

1\. Subscribes to our EventGrid messages

2\. Upserts the Observation Resource document with the data from the EventGrid messages

\#### Leverage Azure API Management Policies. 

1\. Publish \`DeviceGrid\` Function App to Azure API Management.

2\. Create a policy that subscribes to the EventGrid topic.

3\. Extract specific device data and post it to any EndPoint. 

\## Function Apps

\### DeviceToCloud - Function App

We are using an \`EventHubTrigger\` to loop through the **EventHub** Messages.  

Then we create a **EventGrid** message and publish it.

\##### Source Code

\`\`\`csharp

\#region

using System;

using System.Configuration;

using System.Globalization;

using System.Net.Http;

using System.Net.Http.Headers;

using System.Security.Cryptography;

using System.Text;

using System.Threading.Tasks;

using System.Web;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.Azure.WebJobs.ServiceBus;

using Newtonsoft.Json;

\#endregion

namespace DeviceMetrics

{

	public static class DeviceToCloud

	{

		private static readonly string TopicEndpoint = ConfigurationManager.AppSettings\["topicEndpoint"\];

		private static readonly string TopicKey = ConfigurationManager.AppSettings\["topicKey"\];

		\[FunctionName("DeviceToCloud")\]

		public static void Run(\[EventHubTrigger("DeviceToCloud", Connection = " EventHubConnectionString")\]

			string\[\] myEventHubMessages, TraceWriter log)

		{

			log.Info($"C# Event Hub trigger function processed a message: {myEventHubMessages.Length}");

			foreach (var msg in myEventHubMessages)

			{

				dynamic data = JsonConvert.DeserializeObject(msg);

				var eventGridMessage = new GridEvent

				{

					Id = Guid.NewGuid().ToString(),

					EventTime = data.EventTime,

					EventType = data.EventType, 

					Data = data.Data,

					Subject = data.Subject,

					Topic = data.Topic,

					DeviceId = data.DeviceId

				};

				try

				{

					SendEvent(TopicEndpoint, TopicKey, eventGridMessage).ConfigureAwait(false);

				}

				catch (HttpRequestException e)

				{

					log.Error(e.Message, e);

				}

			}

		}

\`\`\`

\`\`\`csharp

		public static async Task SendEvent(string topicEndpoint, string topicKey, object data)

		{

			// Create a SAS token for the call to the event grid. We can do this with 

			// the SAS key as well.

			var sas = CreateEventGridSasToken(topicEndpoint, DateTime.Now.AddDays(1), topicKey);

			// Instantiate an instance of the HTTP client with the 

			// event grid topic endpoint.

			var client = new HttpClient {BaseAddress = new Uri(topicEndpoint)};

			// Configure the request headers with the content type

			// and SAS token needed to make the request.

			client.DefaultRequestHeaders.Accept.Clear();

			client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

			client.DefaultRequestHeaders.Add("aeg-sas-token", sas);

			// Serialize the data

			var json = JsonConvert.SerializeObject(data);

			var stringContent = new StringContent(json, Encoding.UTF8, "application/json");

			// Publish grid event

			await client.PostAsync(string.Empty, stringContent);

		}

\`\`\`

\`\`\`csharp

		private static string CreateEventGridSasToken(string resourcePath, DateTime expirationUtc, string topicKey)

		{

			const char resource = 'r';

			const char expiration = 'e';

			const char signature = 's';

			// Encode the topic resource path and expiration parameters

			var encodedResource = HttpUtility.UrlEncode(resourcePath);

			var encodedExpirationUtc = HttpUtility.UrlEncode(expirationUtc.ToString(CultureInfo.InvariantCulture));

			// Format the unsigned SAS token

			var unsignedSas = $"{resource}={encodedResource}&{expiration}={encodedExpirationUtc}";

			// Create an HMCASHA256 policy with the topic key

			using (var hmac = new HMACSHA256(Convert.FromBase64String(topicKey)))

			{

				// Encode the signature and create the fully signed URL with the

				// appropriate parameters.

				var bytes = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(unsignedSas)));

				var encodedSignature = HttpUtility.UrlEncode(bytes);

				var signedSas = $"{unsignedSas}&{signature}={encodedSignature}";

				return signedSas;

			}

		}

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\### **DeviceEventGrid** - Function App 

\#### Source Code

\`\`\`csharp

\#region

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

\#endregion

namespace DeviceMetrics

{

	public static class DeviceEventGrid

	{

		private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

		private static readonly string AuthKey = ConfigurationManager.AppSettings\["authKey"\];

		private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

		private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

		private static readonly DocumentClient Client = new DocumentClient(new Uri(Endpoint), AuthKey,

			new ConnectionPolicy

			{

				ConnectionMode = ConnectionMode.Direct,

				ConnectionProtocol = Protocol.Tcp

			}

		);

		\[FunctionName("DeviceEventGrid")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "post")\]

			HttpRequestMessage req,

			TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			var jsonContent = await req.Content.ReadAsStringAsync();

			var gridEvent = JsonConvert.DeserializeObject<List<GridEvent<Dictionary<string, string>>>>(jsonContent)

				?.SingleOrDefault();

			if (gridEvent == null) return req.CreateErrorResponse(HttpStatusCode.BadRequest, $@"Missing event details");

			var gridEventType = req.Headers.GetValues("Aeg-Event-Type").FirstOrDefault();

			if (gridEventType == "SubscriptionValidation")

			{

				// Retrieve the validation code and echo back.

				var validationCode = gridEvent.Data\["validationCode"\];

				var validationResponse = JsonConvert.SerializeObject(new

				{

					validationResponse = validationCode

				});

				return new HttpResponseMessage

				{

					StatusCode = HttpStatusCode.OK,

					Content = new StringContent(validationResponse)

				};

			}

			if (gridEventType == "Notification")

			{

				// Get the Observation Resource Document

				dynamic doc = await Client.ReadDocumentAsync(UriFactory.CreateDocumentUri(Database, Collection,

					gridEvent.Data\["deviceId"\]));

				var valueQuanity = new ValueQuantity

				{

					Code = doc.Component.ValueQuantity.Code,

					System = doc.Component.ValueQuantity.System,

					Unit = gridEvent.Data\["unit"\],

					Value = gridEvent.Data\["value"\]

				};

				var component = new Component

				{

					Code = doc.Component.Code,

					ValueQuantity = valueQuanity

				};

				// partial values 

				var observation = new CompoundNumericObservation

				{

					Id = doc.Id,

					Component = {\[0\] = component},

					EffectiveDateTime = new DateTimeOffset(DateTime.Parse(gridEvent.Data\["timestamp"\]))

				};

				try

				{

					var result =

						await Client.UpsertDocumentAsync(UriFactory.CreateDocumentUri(Database, Collection, gridEvent.Data\["deviceId"\]),

							observation);

					return req.CreateResponse(result.StatusCode);

				}

				catch (DocumentClientException e)

				{

					var resp = new HttpResponseMessage

					{

						StatusCode = (HttpStatusCode) e.StatusCode,

						Content = new StringContent(e.Message)

					};

					return resp;

				}

			}

			return req.CreateErrorResponse(HttpStatusCode.BadRequest,

				$@"Unknown request type");

		}

\`\`\`

\`\`\`csharp

		public class GridEvent<T> where T : class

		{

			public string Id { get; set; }

			public string EventType { get; set; }

			public string Subject { get; set; }

			public DateTime EventTime { get; set; }

			public T Data { get; set; }

			public string Topic { get; set; }

		}

	}

}

\`\`\`

\-------------------------

\## Azure API Management 

\### Subscribing to \`EventGrid\` Events Using Azure API Management 

After we pubish our Function App to Api Management, we will create a new Policy for our Function App. This allows us to extract specific device data post it to any EndPoint. 

The following is an example policy.

\`\`\`xml

<policies>

  <inbound>

    <base />

    <set-variable value="@(context.Request.Headers\["Aeg-Event-Type"\].Contains("SubscriptionValidation"))" name="isEventGridSubscriptionValidation" />

    <set-variable value="@(context.Request.Headers\["Aeg-Event-Type"\].Contains("Notification"))" name="isEventGridNotification" />

    <choose>

      <when condition="@(context.Variables.GetValueOrDefault<bool>("isEventGridSubscriptionValidation"))">

        <return-response>

          <set-status code="200" reason="OK" />

          <set-body>@{

            var events = context.Request.Body.As<string>();

            JArray a = JArray.Parse(events);

            var eventGridData = a.First\["data"\];

            var validationCode = eventGridData\["validationCode"\];

            var jOutput =

              new JObject(

                new JProperty("validationResponse", validationCode)

                );

            return jOutput.ToString();

          }</set-body>

        </return-response>

      </when>

      <when condition="@(context.Variables.GetValueOrDefault<bool>("isEventGridNotification"))">

        <send-one-way-request mode="new">

          <set-url>https://XXXXXXXXXXXXXXXXXXXXXX}</set-url>

          <set-method>POST</set-method>

          <set-body>@{

            var events = context.Request.Body.As<string>();

            JArray a = JArray.Parse(events);

            var eventGridData = a.First\["data"\];

            var deviceId = eventGridData\["deviceId"\];

            var deviceData = eventGridData\["deviceData"\];

            var patientd = eventGridData\["patientd"\];

            return new JObject(

                new JProperty("deviceId", deviceId),

                new JProperty("data", deviceData),

                new JProperty("patientd", patientd)                ;

          }</set-body>

        </send-one-way-request>

      </when>

    </choose>

  </inbound>

  <backend>

    <base />

  </backend>

  <outbound>

    <base />

  </outbound>

  <on-error>

    <base />

  </on-error>

</policies>

\`\`\`

\## Summary

\* We are able to leverage the use an Azure EventGrid to create an EventGrid message.

\* We are using a Function App to loop through the EventHub messages.

\* We are able to create and publish EventGrid messages within the Function App.

\* We created another Function App that subscribes to our EventGrid Topic.

\* This Function App Upserts the H7 FHIR Observation Resource with the \`ValueQuantity\` data.

\* We are using a Azure API management \`Policy\` to parse the \`ValueQuantity\` data and \`post\` it to any **EndPoint**. 

\## Next Steps

<div style="page-break-after: always;"></div>

\# CHAPTER 7 -  GET NEW OR MODIFIED HL7 FHIR OBSERVATION RESOURCES FROM AZURE COSMOS DB

\## Requirements

\* The output from the application is to be a array of HL7 FHIR **Observation** Resources (Cosmos DB Documents).

\* The output will be filtered by the FHIR \`ResourceType\` and \`DeviceId\`. 

\* Allow for additional filter conditions. 

\* The application will loop through all the \`partitions\` for our Diagnostics Collections.

\* The output will be a \`List\` of documents

\### The Function App is expecting a \`post\` operation.  

The \`body\` must contain the following properties.

\`\`\`json

{

    "startfromBeginning": true,

    "maximumItemCount": 100,

    "resourceType" : "Observation",

    "deviceId': "test123"

}

\`\`\`

\#### The following is a description of the above properties

| Property             | Description                              | Default value          |

|----------------------|------------------------------------------|------------------------|

| \`startfromBeginning\` | Boolean value - Start from the Current document  | \`false\`                  |

| \`maximumItemCount\`   | Integer value - Maximum number of documents to return   | Default \`-1\` returns all |

| \`resourceType\`       | String value - The HL7 FHIR Resource Type name | required               |

| \`deviceId\`           | String value - The HL7 FHIR Device Id Type value | required               |

<div style="page-break-after: always;"></div>

\## Function App

\### Azure Function App GetNewOrModifed source code

\`\`\`csharp

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using System.Web.Http;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

namespace CosmosDbChangeFeedQuery

{

    public static class GetNewOrModifed

    {

        private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

        private static readonly string AuthKey = ConfigurationManager.AppSettings\["authKey"\];

        private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

        private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

        private static readonly DocumentClient Client = new DocumentClient(new Uri(Endpoint), AuthKey,

            new ConnectionPolicy

            {

                ConnectionMode = ConnectionMode.Direct,

                ConnectionProtocol = Protocol.Tcp

            }

        );

        \[FunctionName("GetNewOrModifed")\]

        public static async Task<dynamic> Run(\[HttpTrigger(AuthorizationLevel.Function,  "post", Route = null)\]HttpRequestMessage req, TraceWriter log)

        {

            log.Info("C# HTTP trigger function processed a request.");

            

            var docs = new List<dynamic>();

            // Get request body

            dynamic data = await req.Content.ReadAsAsync<object>();

            bool startfromBeginning = data.startfromBeginning;

            int maximumItemCount = data.maximumItemCount;

            var resourceType = data.resourceType;

            var deviceId = data.deviceId;

            var collectionLink = UriFactory.CreateDocumentCollectionUri(Database, Collection);

            var partitionKeyRanges = new List<PartitionKeyRange>();

            FeedResponse<PartitionKeyRange> pkRangesResponse;

            do

            {

                pkRangesResponse = await Client.ReadPartitionKeyRangeFeedAsync(collectionLink);

                partitionKeyRanges.AddRange(pkRangesResponse);

            } while (pkRangesResponse.ResponseContinuation != null);

            foreach (var pkRange in partitionKeyRanges)

            {

                var changeFeedOptions = new ChangeFeedOptions

                {

                    StartFromBeginning = startfromBeginning,

                    RequestContinuation = null,

                    MaxItemCount = maximumItemCount,

                    PartitionKeyRangeId = pkRange.Id

                };

                using (var query = Client.CreateDocumentChangeFeedQuery(collectionLink, changeFeedOptions))

                {

                    do

                    {

                        if (query != null)

                        {

                            var results = await query.ExecuteNextAsync<dynamic>().ConfigureAwait(false);

                            if (results.Count <= 0) continue;

                            // we can add more filter conditions

                            docs.AddRange(results.Where(doc => doc.resourceType == resourceType));

                            docs.AddRange(results.Where(doc => doc.deviceId == deviceId));

                        }

                        else

                        {

                            throw new HttpResponseException(new HttpResponseMessage(HttpStatusCode.NotFound));

                        }

                    } while (query.HasMoreResults);

                }

            }

            if (docs.Count > 0)

            {

                return docs;

            }

            var msg = new StringContent($"No documents found for " + resourceType + " Resource and DeviceId" + deviceId);

            var response = new HttpResponseMessage

            {

                StatusCode = HttpStatusCode.NotFound,

                Content = msg

            };

            return response;

         

        }

    }

}

\`\`\`

\#### In our Function App we can add or modify filter conditions. 

\`\`\`csharp

// filter conditions

docs.AddRange(results.Where(doc => doc.resourceType == resourceType));

docs.AddRange(results.Where(doc => doc.deviceId == deviceId));

\`\`\`

\## Summary

\* We can use a <a href="[https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook "https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-http-webhook")" target="_blank">HttpTrigger</a> instead of a <a href="[https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb "https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb")" target="_blank">CosmosDBTrigger</a>

\* We can leverage the \`CreateDocumentChangeFeedQuery\` operation to filter for **Observation** Resources.

\* We do not need to use a **<a href="[https://docs.microsoft.com/en-us/azure/cosmos-db/change-feed](https://docs.microsoft.com/en-us/azure/cosmos-db/change-feed "https://docs.microsoft.com/en-us/azure/cosmos-db/change-feed")" target="_blank">Lease Collection</a>** 

\------------

<div style="page-break-after: always;"></div>

\# CHAPTER 8 - PATIENT CARE DEVICE DATA INTEGRATION

The IHE Patient Care Device (PCD) Domain is concerned with use cases in which at least one actor is a regulated patient-centric point-of-care medical device that communicates with at least one other actor such as a medical device or information system.

\## Integrating the Healthcare Enterprise (IHE) 

IHE is an initiative by healthcare professionals and industry to improve the way computer systems in healthcare share information. IHE promotes the coordinated use of established standards such as DICOM and HL7 to address specific clinical needs in support of optimal patient care. Systems developed in accordance with IHE communicate with one another better, are easier to implement, and enable care providers to use information more effectively.

\## Patient Care Device Technical Framework

!\[\](![](https://i.imgur.com/jryp2XF.png))

\### IHE PCD Profiles

The following are the IHE PCD Profiles used in this Use Case

\* **Point-of-care Device Manager** provides for management and configuration. <a href="[http://www.ihe.net/uploadedFiles/Documents/PCD/IHE_Suppl_PCD_MEM-DMC.pdf](http://www.ihe.net/uploadedFiles/Documents/PCD/IHE_Suppl_PCD_MEM-DMC.pdf "http://www.ihe.net/uploadedFiles/Documents/PCD/IHE_Suppl_PCD_MEM-DMC.pdf")" target="_blank">Medical Equipment Management Device Management Communication (MEMDMC) Profile</a>

\* **Device Observation Consumer** supports publication of information acquired from point-of-care medical devices to applications such as clinical information systems and electronic health record systems, using a consistent messaging format and device semantic content.

\* **Device Observation Reporter**  actor receives data from PCDs, including those based on proprietary formats, and maps the received data to transactions providing consistent syntax and semantics.

\* **Device Information Server** is the repository for the HL7 FHIR Resources. 

Each of these profiles is defined in full detail in the <a href="[http://www.ihe.net/uploadedFiles/Documents/PCD/IHE_PCD_TF_Vol1.pdf](http://www.ihe.net/uploadedFiles/Documents/PCD/IHE_PCD_TF_Vol1.pdf "http://www.ihe.net/uploadedFiles/Documents/PCD/IHE_PCD_TF_Vol1.pdf")" target="_blank">IHE PCD Technical Framework</a>

\## Patient Care Device Integration Message Flow 

!\[\](![](https://i.imgur.com/sGs7Ddg.png))

\### Use Case Diagram

!\[Device Data Integration Use Case\](![](https://i.imgur.com/KGnshPj.png))

\----------------------

<div style="page-break-after: always;"></div>

\### Message Flow Steps

> !\[Note\](![](https://i.imgur.com/DiPYycK.png))  NOTE: Azure Logic App are used for the Point-of-care Device Manager, Device Observation Consumer, and Device Observation Reporter steps

\#### Point-of-care Device Manager    

1\. Receives data from device

2\. Transforms data to Telemetry document 

3\. Sends Telemetry document to Device Observation Consumer

\#### Device Observation Consumer

1\. Receives Telemetry document

2\. Creates Event Grid message 

3\. Sends Event Grid message to Device Observation Reporter

\#### Device Observation Reporter

1\. Receives Event Grid message

2\. Saves Event Grid data to Device Data Message.

3\. Receives device data from IoT Hub.

4\. Transforms Device Data Message to HL7 FHIR Observation Resource.

5\. Sends FHIR Observation Resource to Device Information Server.

\#### Device Information Server

1\. Receives Observation Resource

2\. Upserts Observation Resource document in Cosmos DB Collection.

\#### User (Device Observation Consumer)

1\. Requests and receives Device Related FHIR Resources from Cosmos DB Collections

\## Summary

\* You have seen how data collected from home and hospital medical devices is processed and stored in the FHIR Repository as Observation Resources. 

<div style="page-break-after: always;"></div>

\# CHAPTER 9 - PROVISIONING

For our Provisioning Microservice we are going to use a Azure Durable Function

\## Design 

Our Durable Function supports both X509 Certificates and TPM Endorsment Key registrations.  

\## Parameters

Our **Provision Function**, which uses an \`OrchestrationTrigger\` expects the following input parameters. 

| Parameter         | Description             |

|-------------------|-------------------------|

| Identity          | choice: <br>tpm<br>cert |

| DeviceId          | For TPM                 |

| EnrollmentGroupId | For Cert                |

| RegistrationId    | For TPM                 |

| TpmEndorsementKey | For TPM                 |

| X509RootCertPath  | For Cert                |

We will need to add the \`ProvisioningConnectionString\` in our config file.  This is the connection string for the Azure Provisioning Service.

\## Steps

1\. **Provision Function** validates the Identity.

2\. If the Identity is equal to \`tpm\` then call the **Provision_TPM** \`ActivityTrigger\` Function.

3\. Else call the **Provision_X509** \`ActivityTrigger\` Function.

4\. Both will return a result to the **Provision Function**.

5\. The **Provision Function** will return a response to the consumer.

<div style="page-break-after: always;"></div>

\## Durable Function Source Code

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System.Configuration;

using System.Security.Cryptography.X509Certificates;

using System.Threading.Tasks;

using Microsoft.Azure.Devices.Provisioning.Service;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Host;

\#endregion

namespace DeviceFramework

{

	public static class Provision

	{

		private static string _deviceId;

		private static string _enrollmentGroupId;

		private static string _registrationId;

		private static string _tpmEndorsementKey;

		private static string _x509RootCertPath;

		public static readonly string ProvisioningConnectionString =

			ConfigurationManager.AppSettings\["ProvisioningConnectionString"\];

		\[FunctionName("Provision")\]

		public static async Task<dynamic> RunOrchestrator(

			\[OrchestrationTrigger\] DurableOrchestrationContext context)

		{

			var data = context.GetInput<dynamic>();

			_deviceId = data.deviceId;

			if (data.idenity == Identity.Cert)

			{

				_x509RootCertPath = data.x509RootCertPath;

				_enrollmentGroupId = data.enrollmentGroupId;

				var result = await context.CallActivityAsync<Task>("Provision_X509", _x509RootCertPath);

				return result;

			}

			else

			{

				_tpmEndorsementKey = data.tpmEndorsementKey;

				_registrationId = data.registrationId;

				var result = await context.CallActivityAsync<Task>("Provision_TPM", _tpmEndorsementKey);

				return result;

			}

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("Provision_TPM")\]

		public static async Task StartTpm(\[ActivityTrigger\] string tpmEndorsementKey, TraceWriter log)

		{

			Attestation attestation = new TpmAttestation(tpmEndorsementKey);

			var individualEnrollment =

				new IndividualEnrollment(_registrationId, attestation)

				{

					DeviceId = _deviceId,

					ProvisioningStatus = ProvisioningStatus.Enabled

				};

			var serviceClient = ProvisioningServiceClient.CreateFromConnectionString(ProvisioningConnectionString);

			await serviceClient.CreateOrUpdateIndividualEnrollmentAsync(individualEnrollment).ConfigureAwait(false);

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("Provision_X509")\]

		public static async Task StartX509(\[ActivityTrigger\] string x509RootCertPath, TraceWriter log)

		{

			using (var provisioningServiceClient =

				ProvisioningServiceClient.CreateFromConnectionString(ProvisioningConnectionString))

			{

				var certificate = new X509Certificate2(x509RootCertPath);

				Attestation attestation = X509Attestation.CreateFromRootCertificates(certificate);

				var enrollmentGroup =

					new EnrollmentGroup(

						_enrollmentGroupId,

						attestation)

					{

						ProvisioningStatus = ProvisioningStatus.Enabled

					};

				await provisioningServiceClient.CreateOrUpdateEnrollmentGroupAsync(enrollmentGroup).ConfigureAwait(false);

			}

		}

	}

\`\`\`

\`\`\`csharp

	public enum Identity

	{

		Tpm,

		Cert

	}

}

\`\`\`

\## Summary

\* We have created a Durable Function App that can handle both TPM and X509 Certificate device registrations.

\* Using minimal code we have a created Iot Device Provisioning Microservice.

<div style="page-break-after: always;"></div>

\# CHAPTER 10 - CONSIDERING SOFTWARE AS AN MEDICAL DEVICE

\## Recently we had a discussion on what is considered a Medical Device.  A few questions came to mind. 

One of these questions was:

\### Can software be considered a Medical Device?   

I started to look for an answer.

I found one publication from The International Medical Device Regulators Forum provides Key Definitions in a <a href="[http://www.imdrf.org/docs/imdrf/final/technical/imdrf-tech-131209-samd-key-definitions-140901.pdf](http://www.imdrf.org/docs/imdrf/final/technical/imdrf-tech-131209-samd-key-definitions-140901.pdf "http://www.imdrf.org/docs/imdrf/final/technical/imdrf-tech-131209-samd-key-definitions-140901.pdf")" target="_blank">document</a> published in December 2013. 

\###  I sent an email to the FDA Device Determination department.

> From: Howard Edidin \[mailto: <xxx@XXXX.com>\]  

> Sent: Tuesday, March 20, 2018 5:42 PM  

> To: Device Determina tion <DeviceDetermination@fda.hhs.gov>  

> Subject: Medical Device Clarification  

> 

> Does a medical device need to be a physical machine?

> 

> **The reason I am asking is that we are developing an Artificial intelligence Bot Service.** 

> **The service uses Cognitive Services to record a patients mental state when answering questions.** 

> 

> The Bot Service will be provisioned and managed by an IoT hub in a Healthcare facility.  

> 

> Could the Bot Service be classified as a medical device?

\### I received the following response:

>  

> From: Device Determination <DeviceDetermination@fda.hhs.gov>   

> Sent: Monday, March 26, 2018 10:21 AM  

> To: Howard Edidin <xxx@XXXX.com>; Device Determination  

> DeviceDetermination@fda.hhs.gov>  

> Subject: RE: Medical Device Clarification  

> 

> Hello Howard:

> 

> A device used to record a patient’s mental state could be considered a medical device if it is intended to treat, mitigate and/or diagnose a patient. 

> This response represents my best judgment on how the product would be regulated. This response is not a classification decision and does not constitute FDA clearance or approval for commercial distribution of your product.

> 

> Thank you,

> 

> FDA Device Determination

The FDA published a document, <a href="[https://www.fda.gov/downloads/MedicalDevices/DeviceRegulationandGuidance/GuidanceDocuments/UCM524904.pdf](https://www.fda.gov/downloads/MedicalDevices/DeviceRegulationandGuidance/GuidanceDocuments/UCM524904.pdf "https://www.fda.gov/downloads/MedicalDevices/DeviceRegulationandGuidance/GuidanceDocuments/UCM524904.pdf")" target="_blank">Software as a Medical Device (SAMD): Clinical Evaluation</a> in December 2017.

<div style="page-break-after: always;"></div>

\## An Azure IoT Hub can store just about any type of data from a Device.

There is support for:  

\* Sending Device to Cloud messages.

\* Invoking direct methods on a device

\* Uploading files from a device

\* Managing Device Identities

\* Scheduling Jobs on single for multiple devices

\## The following is the List of of built-in endpoints

!\[\](![](https://i.imgur.com/qoU3XhN.png))

\### Custom Endpoints can also be created. 

IoT Hub currently supports the following Azure services as additional endpoints:

\* Azure Storage containers

\* Event Hubs

\* Service Bus Queues

\* Service Bus Topics

\## Architecture 

If we look through the documentation on the <a href="[https://docs.microsoft.com/en-us/azure/architecture/](https://docs.microsoft.com/en-us/azure/architecture/ "https://docs.microsoft.com/en-us/azure/architecture/")" target="_blank">Azure Architecture Center</a>, we can see a list of Architectural Styles.

If we were to design an IoT Solution, we would want to follow <u>Best Practices</u>. We can do this by using the Azure Architectural Style of <a href="[https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven](https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven "https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven")" target="_blank">Event Driven Architecture</a>.  Event-driven architectures are central to <a href="[https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven#iot-architecture](https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven#iot-architecture "https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/event-driven#iot-architecture")" target="_blank">IoT solutions</a>. 

Merging Event Driven Architecture with Microservices</a> can be used to separate the IoT Business Services.  

These services include: 

\* Provisioning

\* Management 

\* Software Updating

\* Security

\* Logging and Notifications

\* Analytics 

> !\[\](![](https://i.imgur.com/eSPsKWl.png)) INFO: Our Microservices Architecture does not require Azure Service Fabric or Azure Container Service. It is an **Hybrid Service** managed by **Azure Api Management**.

\## Creating our services 

To create these services, we start by selecting our <a href="[https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-overview](https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-overview "https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/compute-overview")" target="_blank">Compute Options</a>.

\### App Services

The use of Azure Functions is becoming commonplace.  They are an excellent replacement for API Applications.  And they can be published to Azure Api Management. 

We are able to create a <a href="[https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-serverless-api](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-serverless-api "https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-serverless-api")" target="_blank">**Serverless API**</a>, or use <a href="[https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview](https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview "https://docs.microsoft.com/en-us/azure/azure-functions/durable-functions-overview")" target="_blank">**Durable Functions**</a> that allow us to create workflows and maintain state in a serverless environment.  

<a href="[https://docs.microsoft.com/en-us/azure/logic-apps/](https://docs.microsoft.com/en-us/azure/logic-apps/ "https://docs.microsoft.com/en-us/azure/logic-apps/")" target="_blank">**Logic Apps**</a> provide us with the capability of building automated scalable workflows.

 

\### Data Store

Having a single data store is usually not the best approach. Instead, it's often better to store different types of data in <a href="[https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/data-store-overview](https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/data-store-overview "https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/data-store-overview")" target="_blank">different data stores</a>, each focused towards a specific workload or usage pattern. These stores include Key/value stores, Document databases, Graph databases, Column-family databases, Data Analytics, Search Engine databases, Time Series databases, Object storage, and Shared files.

This may hold true for other Architectural Styles. In our Event-driven Architecture, it is ideal to store all data related to IoT Devices in the IoT Hub. This data includes results from all  events within the Logic Apps, Function Apps, and Durable Functions. 

\-------

<div style="page-break-after: always;"></div>

Which brings us back to our topic... **Considering Software as an IoT Device**

Since **Azure IoT** supports the \`TransportType.Http1\` protocol, we can use the \`Microsoft.Azure.Devices.Client\`Library to **send Event data to our IoT Hub** from any type of software.  We also have the capability of **receiving configuration data** from the IoT Hub.

\**The following is the source code for our **SendEvent** Function App.**

\### <u>SendEvent</u> Function App

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c) 2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Data.Services.Client;

using System.Net;

using System.Net.Http;

using System.Text;

using System.Threading.Tasks;

using Microsoft.Azure.Devices.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

using TransportType = Microsoft.Azure.Devices.Client.TransportType;

\#endregion

namespace IoTHubClient

{

	public static class SendEvent

	{

		private static readonly string IotHubUri = ConfigurationManager.AppSettings\["hubEndpoint"\];

		\[FunctionName("SendEventToHub")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "post", Route = "device/{id}/{key:guid}")\]

			HttpRequestMessage req, string id, Guid key, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			// Get request body

			dynamic data = await req.Content.ReadAsAsync<object>();

			var deviceId = id;

			var deviceKey = key.ToString();

			if (string.IsNullOrEmpty(deviceKey) || string.IsNullOrEmpty(deviceId))

				return req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a deviceid and deviceKey in the Url");

			var telemetry = new Dictionary<Guid, object>();

			foreach (var item in data.telemetryData)

			{

				var telemetryData = new TelemetryData

				{

					MetricId = item.metricId,

					MetricValue = item.metricValue,

					MericDateTime = item.metricDateTime,

					MetricValueType = item.metricValueType

				};

				telemetry.Add(Guid.NewGuid(), telemetryData);

			}

			var deviceData = new DeviceData

			{

				DeviceId = deviceId,

				DeviceName = data.deviceName,

				DeviceVersion = data.deviceVersion,

				DeviceOperation = data.deviceOperation,

				DeviceType = data.deviceType,

				DeviceStatus = data.deviceStatus,

				DeviceLocation = data.deviceLocation,

				SubscriptionId = data.subcriptionId,

				ResourceGroup = data.resourceGroup,

				EffectiveDateTime = new DateTimeOffset(DateTime.Now),

				TelemetryData = telemetry

			};

			var json = JsonConvert.SerializeObject(deviceData);

			var message = new Message(Encoding.ASCII.GetBytes(json));

			try

			{

				var client = DeviceClient.Create(IotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey(deviceId, deviceKey),

					TransportType.Http1);

				await client.SendEventAsync(message);

				return req.CreateResponse(HttpStatusCode.OK);

			}

			catch (DataServiceClientException e)

			{

				var resp = new HttpResponseMessage

				{

					StatusCode = (HttpStatusCode) e.StatusCode,

					Content = new StringContent(e.Message)

				};

				return resp;

			}

		}

	}

\`\`\`

\`\`\`csharp

	public class DeviceData

	{

		public string DeviceId { get; set; }

		public string DeviceName { get; set; }

		public string DeviceVersion { get; set; }

		public string DeviceType { get; set; }

		public string DeviceOperation { get; set; }

		public string DeviceStatus { get; set; }

		public DeviceLocation DeviceLocation { get; set; }

		public string AzureRegion { get; set; }

		public string ResourceGroup { get; set; }

		public string SubscriptionId { get; set; }

		public DateTimeOffset EffectiveDateTime { get; set; }

		public Dictionary<Guid, object> TelemetryData { get; set; }

	}

\`\`\`

\`\`\`csharp

	public class TelemetryData

	{

		public string MetricId { get; set; }

		public string MetricValueType { get; set; }

		public string MetricValue { get; set; }

		public DateTime MericDateTime { get; set; }

	}

\`\`\`

\`\`\`csharp

	public enum DeviceLocation

	{

		Cloud,

		Container,

		OnPremise

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Software Device Properties 

\##### The following values are required in the Url Path

\`Route = "device/{id}/{key:guid}")\`

| Name | Description        |

|------|--------------------|

| id   | Device Id (String) |

| key  | Device Key (Guid)  |

\-------------------

\##### The following are the properties to be sent in the Post Body 

| Name                          | Description                      |

|-------------------------------|----------------------------------|

| deviceName                    | Device Name                      |

| deviceVersion                 | Device version number            |

| deviceType                    | Type of Device                   |

| deviceOperation               | Operation name or type           |

| deviceStatus                  | Default: Active                  |

| deviceLocation                | Cloud<br>Container<br>OnPremise  |

| subscriptionId                | Azure Subscription Id            |

| resourceGroup                 | Azure Resource group             |

| azureRegion                   | Azure Region                     |

| telemetryData                 | Array                |

|-------------------------------|----------------------|

| telemetryData.metricId        | Array item id        |

| telemetryData.metricValueType | Array item valueType |

| telemetryData.metricValue     | Array item value     |

| telemetryData.metricTimeStamp | Array item TimeStamp |

\----------------

\## Summary

\* We can easily add the capability of sending messages and events to our Function and Logic Apps. 

\* Optionally, we can send the data to an Event Grid.

\* We have a single data store for all our IoT events.

\* We can identify performance issues within our services.

\* Having a single data store makes it easier to perform Analytics.

\* We can use a Azure Function App to **Send Device to Cloud Messages**. In this case our Function App will be also be taking the role of a Device.

<div style="page-break-after: always;"></div>

\-----------------------

\# CHAPTER 11 - IOT EDGE

Azure Iot Edge allows us to deploy complex event processing, machine learning, image recognition and other high value AI without writing it in house. Azure services like Azure Functions, Azure Stream Analytics, and Azure Machine Learning can all be run on premises via Azure IoT Edge.

\## IoT Edge is composed of:  

\* **Modules** - are units of execution; Complex Event Processing, Image and Video Recognition, etc..

\* **Runtime** -  enables custom and cloud logic 

\* **Cloud Interfaces** - create, configure, and send workloads along with Monitoring workloads.

\## The following are two deployment and one storing daa examples.

\### Deploying an Azure Function Example

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System.Text;

using System.Threading.Tasks;

using Microsoft.Azure.Devices.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

\#endregion

namespace Edge

{

	public static class FilterMessages

	{

		private const int TemperatureThreshold = 25;

		// Filter messages based on the temperature value in the body of the message and the temperature threshold value.

		\[FunctionName("FilterMessages")\]

		public static async Task Run(Message messageReceived, IAsyncCollector<Message> output, TraceWriter log)

		{

			var messageBytes = messageReceived.GetBytes();

			var messageString = Encoding.UTF8.GetString(messageBytes);

			if (!string.IsNullOrEmpty(messageString))

			{

				var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

				if (messageBody != null && messageBody.Machine.Temperature > TemperatureThreshold)

				{

					// Send the message to the output as the temperature value is greater than the threashold

					var filteredMessage = new Message(messageBytes);

					// Copy the properties of the original message into the new Message object

					foreach (var prop in messageReceived.Properties) filteredMessage.Properties.Add(prop.Key, prop.Value);

					

					// Add a new property to the message to indicate it is an alert

					filteredMessage.Properties.Add("MessageType", "Alert");

					// Send the message        

					await output.AddAsync(filteredMessage);

					log.Info("Received and transferred a message with temperature above the threshold");

				}

			}

		}

	}

\`\`\`

\`\`\`csharp

	//Define the expected schema for the body of incoming messages

	public class MessageBody

	{

		public Machine Machine { get; set; }

		public Ambient Ambient { get; set; }

		public string TimeCreated { get; set; }

	}

	public class Machine

	{

		public double Temperature { get; set; }

		public double Pressure { get; set; }

	}

	public class Ambient

	{

		public double Temperature { get; set; }

		public int Humidity { get; set; }

	}

}

\`\`\`

\-------------------

<div style="page-break-after: always;"></div>

\### Deploying Code in a Module Sample

\------------------

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Configuration;

using System.IO;

using System.Runtime.InteropServices;

using System.Security.Cryptography.X509Certificates;

using System.Threading.Tasks;

using Microsoft.AspNetCore.Http;

using Microsoft.AspNetCore.Mvc;

using Microsoft.Azure.Devices.Client;

using Microsoft.Azure.Devices.Client.Transport.Mqtt;

using Microsoft.Azure.Devices.Shared;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

using Microsoft.Azure.Devices.Client.Exceptions;

\#endregion

namespace Edge

{

	public static class EdgeHubTrigger

	{

		public enum MessageResponse

		{

			None = 0,

			Completed = 1,

			Abandoned = 2

		}

		public static readonly string ConnectionString = ConfigurationManager.AppSettings\["EdgeHubConnectionString"\];

		public static bool Success = true;

		public static string Results;

		public static int TemperatureThreshold { get; set; } = 25;

		\[FunctionName("EdgeHubTrigger")\]

		public static IActionResult Run(\[HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)\]

			HttpRequest req, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			string name = req.Query\["id"\];

		

			var bypassCertVerification = RuntimeInformation.IsOSPlatform(OSPlatform.Windows);

			if (!bypassCertVerification) InstallCert();

			Init(ConnectionString, bypassCertVerification).Wait();

            

			return Success != false

				? (ActionResult) new OkObjectResult($"Success, {name}")

				: new BadRequestObjectResult(Results);

		}

\`\`\`

\`\`\`csharp

		private static async Task Init(string connectionString, bool bypassCertVerification = false)

		{

			TraceWriter log = null;

			log.Info("Connection String {0}", connectionString);

			var mqttSetting = new MqttTransportSettings(TransportType.Mqtt_Tcp_Only);

			// During dev you might want to bypass the cert verification. It is highly recommended to verify certs systematically in production

			if (bypassCertVerification)

				mqttSetting.RemoteCertificateValidationCallback = (sender, certificate, chain, sslPolicyErrors) => true;

			ITransportSettings\[\] settings = {mqttSetting};

			// Open a connection to the Edge runtime

			var ioTHubModuleClient = DeviceClient.CreateFromConnectionString(connectionString, settings);

			await ioTHubModuleClient.OpenAsync();

			log.Info("IoT Hub module client initialized.");

			// Read TemperatureThreshold from Module Twin Desired Properties

			var moduleTwin = await ioTHubModuleClient.GetTwinAsync();

			var moduleTwinCollection = moduleTwin.Properties.Desired;

			if (moduleTwinCollection\["TemperatureThreshold"\] != null)

				TemperatureThreshold = moduleTwinCollection\["TemperatureThreshold"\];

			// Attach callback for Twin desired properties updates

			try

			{

				await ioTHubModuleClient.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertiesUpdate, null);

				Success = true;

				Results = "Success";

			}

			catch (IotHubException e)

			{

				log.Error(e.Message, e);

				Results = e.Message;

				Success = false;

			}

		}

\`\`\`

\`\`\`csharp

		private static void InstallCert()

		{

			TraceWriter log = null;

			var certPath = Environment.GetEnvironmentVariable("EdgeModuleCACertificateFile");

			if (string.IsNullOrWhiteSpace(certPath))

			{

				Success = false;

				// We cannot proceed further without a proper cert file

				log.Info($"Missing path to certificate collection file: {certPath}");

				Results = "Missing path to certificate file.";

				throw new InvalidOperationException("Missing path to certificate file.");

			}

			if (!File.Exists(certPath))

			{

				Success = false;

				Results = "Missing certificate file.";

				throw new InvalidOperationException("Missing certificate file.");

			}

			var store = new X509Store(StoreName.Root, StoreLocation.CurrentUser);

			store.Open(OpenFlags.ReadWrite);

			store.Add(new X509Certificate2(X509Certificate.CreateFromCertFile(certPath)));

			log.Info("Added Cert: " + certPath);

			

			store.Close();

		}

\`\`\`

\`\`\`csharp

		private static Task OnDesiredPropertiesUpdate(TwinCollection desiredProperties, object userContext)

		{

			TraceWriter log = null;

			try

			{

				if (desiredProperties\["TemperatureThreshold"\] != null)

					TemperatureThreshold = desiredProperties\["TemperatureThreshold"\];

			}

			catch (AggregateException ex)

			{

				foreach (var exception in ex.InnerExceptions) log.Error("Error when receiving desired property: {0}", exception);

			}

			catch (Exception ex)

			{

				log.Error("Error when receiving desired property: {0}", ex);

			}

			return Task.CompletedTask;

		}

\`\`\`

\`\`\`csharp

		public class MessageBody

		{

			public Ambient Ambient { get; set; }

			public Machine Machine { get; set; }

			public string TimeCreated { get; set; }

		}

		public class Machine

		{

			public double Pressure { get; set; }

			public double Temperature { get; set; }

		}

		public class Ambient

		{

			public int Humidity { get; set; }

			public double Temperature { get; set; }

		}

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Store Data at the Edge Sample

Using Azure Cosmos DB Collection

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Text;

using System.Threading.Tasks;

using Microsoft.Azure.Devices.Client;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

\#endregion

namespace Edge

{

	public static class StoreData

	{

		private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

		private static readonly string AuthKey = ConfigurationManager.AppSettings\["authKey"\];

		private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

		private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

		private static readonly DocumentClient Client = new DocumentClient(new Uri(Endpoint), AuthKey,

			new ConnectionPolicy

			{

				ConnectionMode = ConnectionMode.Direct,

				ConnectionProtocol = Protocol.Tcp

			}

		);

	

		\[FunctionName("StoreData")\]

		public static async Task Run(Message messageReceived, IAsyncCollector<Message> output, TraceWriter log)

		{

			const int temperatureThreshold = 25;

			var messageBytes = messageReceived.GetBytes();

			var messageString = Encoding.UTF8.GetString(messageBytes);

			if (!string.IsNullOrEmpty(messageString))

			{

				// Get the body of the message and deserialize it

				var messageBody = JsonConvert.DeserializeObject<MessageBody>(messageString);

				var device = new DeviceData

				{

					DeviceId = messageBody.DeviceId,

					Id = Guid.NewGuid().ToString(),

					TimeStamp = DateTime.Parse(messageBody.TimeCreated)

				};

				device.Readings.Add("Machine.Temperature", messageBody.Machine.Temperature);

				device.Readings.Add("Machine.Pressure", messageBody.Machine.Pressure);

				device.Readings.Add("Ambient.Temperature", messageBody.Ambient.Temperature);

				device.Readings.Add("Ambient.Humidity", messageBody.Ambient.Humidity);

				try

				{

					await Client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri(Database, Collection), device);

				}

				catch (DocumentClientException e)

				{

					log.Error(e.Message,e);

				}

				if (messageBody.Machine.Temperature > temperatureThreshold)

				{

					// Send the message to the output as the temperature value is greater than the threshold

					var filteredMessage = new Message(messageBytes);

					// Copy the properties of the original message into the new Message object

					foreach (var prop in messageReceived.Properties)

					{

						filteredMessage.Properties.Add(prop.Key, prop.Value);

					}

					// Add a new property to the message to indicate it is an alert

					filteredMessage.Properties.Add("MessageType", "Alert");

					// Send the message        

					await output.AddAsync(filteredMessage);

					log.Info("Received and transferred a message with temperature above the threshold");

				}

			}

		}

\`\`\`

\`\`\`csharp

		public class MessageBody

		{

			public Ambient Ambient { get; set; }

			public Machine Machine { get; set; }

			public string TimeCreated { get; set; }

			public string DeviceId { get; set; }

		}

		public class Machine

		{

			public double Pressure { get; set; }

			public double Temperature { get; set; }

		}

		public class Ambient

		{

			public int Humidity { get; set; }

			public double Temperature { get; set; }

		}

	}

\`\`\`

\`\`\`csharp

	public class DeviceData

	{

		public string Id { get; set; }

		public string Value { get; set; }

		public string DeviceId { get; set; }

		public DateTime TimeStamp { get; set; }

		public Dictionary<string, double> Readings { get; set; }

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\# CHAPTER 12 - MONITORING AND NOTIFICATIONS

\## Using an Azure Durable Function to Monitor our IoT Hub Diagnostics Logs

We will be using <a href="[https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/index](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/index "https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/index")" target="_blank">Azure Monitor </a> along with <a href="[https://docs.microsoft.com/en-us/azure/service-health/resource-health-overview](https://docs.microsoft.com/en-us/azure/service-health/resource-health-overview "https://docs.microsoft.com/en-us/azure/service-health/resource-health-overview")" target="_blank">Azure Resource Health</a> as a Service within our **Monitoring and Notifications Microservice.**

\### Steps

1\. Our \`Monitor\` OrchestrationTrigger function receives \`DurableOrchestrationContext\` **monitorContext**

2\. The **monitorContext** is validated.

3\. While \`CurrentUtcDateTime <  CurrentUtcDateTime.AddHours(6)\`  

    *  Calls \`Monitor_GetIsClear\` function which loops through the partitionIds.

    * \`Monitor_GetIsClear\` function calls \`ReceiveMessagesFromDeviceAsync\` method, which gets the Diagnostic Log data.

4\. The function \`Monitor_SendDiagnosticAlert\` creates a **Twilio** SMS message and sends the Diagnostic Log data using the <a href="[https://www.twilio.com/sms](https://www.twilio.com/sms "https://www.twilio.com/sms")" target="_blank">**Twilio** Rest API.</a>

\-------------

\### Azure Durable Function Source Code Example

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Linq;

using System.Text;

using System.Threading;

using System.Threading.Tasks;

using System.Web.Script.Serialization;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.ServiceBus.Messaging;

using Twilio.Rest.Api.V2010.Account;

using Twilio.Types;

\#endregion

namespace Monitoring

{

	public static class Monitor

	{

		private static readonly string MonitoringConnectionString =

			ConfigurationManager.AppSettings\["MonitoringConnectionString"\];

		private static readonly string MonitoringEndpointName = ConfigurationManager.AppSettings\["MonitoringEndpointName"\];

		private static readonly string TwilioPhoneNumber = ConfigurationManager.AppSettings\["TwilioPhoneNumber"\];

		public static EventHubClient EventHubClient;

		private static AzureMonitorDiagnosticLog _messsage;

		\[FunctionName("Monitor")\]

		public static async Task Run(\[OrchestrationTrigger\] DurableOrchestrationContext monitorContext, TraceWriter log)

		{

			var input = monitorContext.GetInput<MonitorRequest>();

			if (!monitorContext.IsReplaying)

				log.Info(

					$"Received monitor request. Location: {input?.AzureMonitorDiagnosticLog.Location}. Operation: {input?.AzureMonitorDiagnosticLog.OperationName}.");

			VerifyRequest(input);

			var endTime = monitorContext.CurrentUtcDateTime.AddHours(6);

			if (!monitorContext.IsReplaying)

				log.Info($"Instantiating monitor for {input.AzureMonitorDiagnosticLog.Location}. Expires: {endTime}.");

			while (monitorContext.CurrentUtcDateTime < endTime)

			{

				if (!monitorContext.IsReplaying)

					log.Info(

						$"Checking for {input.AzureMonitorDiagnosticLog.ResourceId} at {monitorContext.CurrentUtcDateTime}.");

				var isClear = await monitorContext.CallActivityAsync<dynamic>("Monitor_GetIsClear", input.DeviceId);

				if (isClear != null)

				{

					if (!monitorContext.IsReplaying)

						await monitorContext.CallActivityAsync("Monitor_SendDiagnosticAlert", TwilioPhoneNumber);

					break;

				}

				// Wait for the next checkpoint

				var nextCheckpoint = monitorContext.CurrentUtcDateTime.AddMinutes(30);

				if (!monitorContext.IsReplaying)

					log.Info($"Next check for {input.AzureMonitorDiagnosticLog.ResourceId} at {nextCheckpoint}.");

				await monitorContext.CreateTimer(nextCheckpoint, CancellationToken.None);

			}

			log.Info($"Monitor expiring.");

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("Monitor_GetIsClear")\]

		public static async Task<dynamic> GetIsClear(\[ActivityTrigger\] string deviceId)

		{

			EventHubClient = EventHubClient.CreateFromConnectionString(MonitoringConnectionString, MonitoringEndpointName);

			var d2CPartitions = EventHubClient.GetRuntimeInformation().PartitionIds;

			var cts = new CancellationTokenSource();

			var tasks = d2CPartitions.Select(partition => ReceiveMessagesFromDeviceAsync(partition, cts.Token)).Cast<Task>()

				.ToList();

			cts.Cancel();

			if (tasks.Count > 0) Task.WaitAll(tasks.ToArray());

			var result = await Task.FromResult(tasks.ToArray());

			return result;

		}

\`\`\`

\`\`\`csharp

		\[FunctionName("Monitor_SendDiagnosticAlert")\]

		public static void SendDiagnosticAlert(

			\[ActivityTrigger\] string phoneNumber,

			TraceWriter log,

			\[TwilioSms(AccountSidSetting = "TwilioAccountSid", AuthTokenSetting = "TwilioAuthToken",

				From = "%TwilioPhoneNumber%")\]

			out CreateMessageOptions message)

		{

			message = new CreateMessageOptions(new PhoneNumber(phoneNumber))

			{

				Body = $"Results {CreateResponse(_messsage)} "

			};

		}

\`\`\`

\`\`\`csharp

		private static void VerifyRequest(MonitorRequest request)

		{

			if (request == null) throw new ArgumentNullException(nameof(request), "An input object is required.");

			if (string.IsNullOrEmpty(request.DeviceId))

				throw new ArgumentNullException(nameof(request.DeviceId), "A Deviceid is required.");

		}

\`\`\`

\`\`\`csharp

		private static string CreateResponse(AzureMonitorDiagnosticLog log)

		{

			var builder = new StringBuilder();

			builder.Append("Diagnostic Log");

			builder.AppendLine();

			builder.Append($"Category: {log.Category}");

			builder.Append($"Location: {log.Location}");

			builder.Append($"OperationName: {log.OperationName}");

			builder.Append($"ResultType: {log.ResultType}");

			builder.Append($"CallerIpAddress: {log.CallerIpAddress}");

			builder.Append($"DurationMs: {log.DurationMs}");

			builder.Append($"Identity: {log.Identity}");

			builder.Append($"Level: {log.Level}");

			builder.Append($"ResourceId: {log.ResourceId}");

			builder.Append($"Time: {log.Time}");

			builder.AppendLine();

			builder.Append("Properties");

			foreach (var item in log.Properties) builder.Append(item);

			return builder.ToString();

		}

\`\`\`

\`\`\`csharp

		internal static async Task<AzureMonitorDiagnosticLog> ReceiveMessagesFromDeviceAsync(string partition,

			CancellationToken ct)

		{

			var eventHubReceiver = EventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);

			while (true)

			{

				if (ct.IsCancellationRequested)

				{

					await eventHubReceiver.CloseAsync();

					break;

				}

				var eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0, 0, 10));

				if (eventData != null)

				{

					var data = Encoding.UTF8.GetString(eventData.GetBytes());

					//deserialize json data to azure monitor object 

					_messsage = new JavaScriptSerializer().Deserialize<AzureMonitorDiagnosticLog>(data);

				}

			}

			return _messsage;

		}

	}

\`\`\`

\`\`\`csharp

	public class MonitorRequest

	{

		public string DeviceId { get; set; }

		public AzureMonitorDiagnosticLog AzureMonitorDiagnosticLog { get; set; }

	}

	public class AzureMonitorDiagnosticLog

	{

		public string Time { get; set; }

		public string ResourceId { get; set; }

		public string OperationName { get; set; }

		public string Category { get; set; }

		public string Level { get; set; }

		public string ResultType { get; set; }

		public string ResultDescription { get; set; }

		public string DurationMs { get; set; }

		public string CallerIpAddress { get; set; }

		public string CorrelationId { get; set; }

		public string Identity { get; set; }

		public string Location { get; set; }

		public Dictionary<string, string> Properties { get; set; }

	}

}

\`\`\`

\## Summary

\* We have seen how we can use an Azure Durable Function to orchestrate our Monitoring and Notification Service.

<div style="page-break-after: always;"></div>

\## Sending Data to Log Analytics 

In addition to our Monitoring example above, we can use an Azure Function App to send data from any service or appcation to Log Analytics.

In this case we are going to use the Azure Log Analytics <a href="[https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-data-collector-api#create-a-request](https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-data-collector-api#create-a-request "https://docs.microsoft.com/en-us/azure/log-analytics/log-analytics-data-collector-api#create-a-request")" target="_blank">HTTP Data Collector API </a>.

\### Steps

1\. Create a **Collector** object:	\`var collector = new Collector(WorkspaceId, SharedKey);\`.

2\. Get the **name** of the event type that is being submitted to Log Analytics.

3\. Pass in the JSON payload from the request body

4\.  \`var result = collector.Collect(logType, data).ConfigureAwait(false);\`

5\.  Return the response to the consuming service.

		

\#### Source Code example

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System.Configuration;

using System.IO;

using HTTPDataCollectorAPI;

using Microsoft.AspNetCore.Http;

using Microsoft.AspNetCore.Mvc;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

\#endregion

namespace Monitoring

{

	public static class DataCollector

	{

		/// <param>

		///     Primary or Secondary Key obtained from your Microsoft Operations Management Suite account, Settings > Connected

		///     Sources.

		///     <name>SharedKey</name>

		/// </param>

		private static readonly string SharedKey = ConfigurationManager.AppSettings\["sharedKey"\];

		/// <param>

		///     Workspace Id obtained from your Microsoft Operations Management Suite account, Settings > Connected Sources.

		///     <name>WorkspaceId</name>

		/// </param>

		private static readonly string WorkspaceId = ConfigurationManager.AppSettings\["workSpaceId"\];

		\[FunctionName("DataCollector")\]

		public static IActionResult Run(\[HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)\]

			HttpRequest req, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			var collector = new Collector(WorkspaceId, SharedKey);

			// name of the event type that is being submitted to Log Analytics

			string logType = req.Query\["logType"\];

			var requestBody = new StreamReader(req.Body).ReadToEnd();

			dynamic data = JsonConvert.DeserializeObject(requestBody);

			var result = collector.Collect(logType, data).ConfigureAwait(false);

			return result != null

				? (ActionResult) new OkObjectResult(result)

				: new BadRequestObjectResult("Please pass a Logtype on the query string and data in the request body");

		}

	}

}

\`\`\`

\## Summary

\* We are able to send data from other services to our Log Analytics.

<div style="page-break-after: always;"></div>

\# CHAPTER 13 - COPY DATA FROM AZURE TO ON PREMISE SQL DATABASE

Looking back at \[Chapter 1 \](#chapter-1---use-case), one of the requirements is to be able to 

copy data from cloud to a On-Premise SQL Database.   

We are going to use the <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-connection")" target="_blank">Azure On-Premise data gateway</a> within a Azure Logic App.

> !\[\](![](https://i.imgur.com/DiPYycK.png))  NOTE: You can install the gateway following these <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-gateway-install")" target="_blank">steps</a>

Lets' assume that we have installed and configured the Azure On-Premise data gateway on our On-Premise SQL Server

\## Steps

The following are the steps for creating our Logic App

1\. Create a new Logic App.

2\. Create the On-premises data gateway  

!\[\](![](https://i.imgur.com/37iAH4j.png))  

3\. Configure connector

 

!\[\](![](https://i.imgur.com/asyWrna.png))  

<div style="page-break-after: always;"></div>

\# CHAPTER 14 - DICOM 

HL7 FHIR supports Diagnostic Medicine.  There are several FHIR Resources.  

The following table describes these FHIR Resources.

| Resource                                 | Description                              |

|------------------------------------------|------------------------------------------|

| <a href="[https://www.hl7.org/fhir/imagingstudy.html](https://www.hl7.org/fhir/imagingstudy.html "https://www.hl7.org/fhir/imagingstudy.html")" target="_blank">ImagingStudy</a> | Representation of the content produced in a DICOM imaging study. A study comprises a set of series, each of which includes a set of Service-Object Pair Instances (SOP Instances - images or other data) acquired or produced in a common context. A series is of only one modality (e.g. X-ray, CT, MR, ultrasound), but a study may have multiple series of different modalities. |

| <a href="[http://hl7.org/fhir/2018May/media.html](http://hl7.org/fhir/2018May/media.html "http://hl7.org/fhir/2018May/media.html")" target="_blank">Media</a>                                    | A photo, video, or audio recording acquired or used in healthcare. The actual content may be inline or provided by direct reference. |

| <a href="[https://www.hl7.org/fhir/sequence.html](https://www.hl7.org/fhir/sequence.html "https://www.hl7.org/fhir/sequence.html")" target="_blank">Sequence</a> | This resource is designed to describe sequence variations with clinical significance |

| <a href="[https://www.hl7.org/fhir/observation.html](https://www.hl7.org/fhir/observation.html "https://www.hl7.org/fhir/observation.html")" target="_blank">Observation</a> | Observations are a central element in healthcare, used to support diagnosis, monitor progress, determine baselines and patterns and even capture demographic characteristics. Most observations are simple name/value pair assertions with some metadata, but some observations group other observations together logically, or even are multi-component observations. Note that the DiagnosticReport resource provides a clinical or workflow context for a set of observations and the Observation resource is referenced by DiagnosticReport to represent lab, imaging, and other clinical and diagnostic data to form a complete report. |

\## HL7 FHIR Resource Document Templates

\### ImagingStudy Document Template

\`\`\`json

{

  "resourceType" : "ImagingStudy",

  // from Resource: id, meta, implicitRules, and language

  // from DomainResource: text, contained, extension, and modifierExtension

  "uid" : "<oid>", // R!  Formal DICOM identifier for the study

  "accession" : { Identifier }, // Related workflow identifier ("Accession Number")

  "identifier" : \[{ Identifier }\], // Other identifiers for the study

  "availability" : "<code>", // ONLINE | OFFLINE | NEARLINE | UNAVAILABLE

  "modalityList" : \[{ Coding }\], // All series modality if actual acquisition modalities

  "patient" : { Reference(Patient) }, // R!  Who the images are of

  "context" : { Reference(Encounter|EpisodeOfCare) }, // Originating context

  "started" : "<dateTime>", // When the study was started

  "basedOn" : \[{ Reference(ReferralRequest|CarePlan|ProcedureRequest) }\], // Request fulfilled

  "referrer" : { Reference(Practitioner) }, // Referring physician

  "interpreter" : \[{ Reference(Practitioner) }\], // Who interpreted images

  "endpoint" : \[{ Reference(Endpoint) }\], // Study access endpoint

  "numberOfSeries" : "<unsignedInt>", // Number of Study Related Series

  "numberOfInstances" : "<unsignedInt>", // Number of Study Related Instances

  "procedureReference" : \[{ Reference(Procedure) }\], // The performed Procedure reference

  "procedureCode" : \[{ CodeableConcept }\], // The performed procedure code

  "reason" : { CodeableConcept }, // Why the study was requested

  "description" : "<string>", // Institution-generated description

  "series" : \[{ // Each study has one or more series of instances

    "uid" : "<oid>", // R!  Formal DICOM identifier for this series

    "number" : "<unsignedInt>", // Numeric identifier of this series

    "modality" : { Coding }, // R!  The modality of the instances in the series

    "description" : "<string>", // A short human readable summary of the series

    "numberOfInstances" : "<unsignedInt>", // Number of Series Related Instances

    "availability" : "<code>", // ONLINE | OFFLINE | NEARLINE | UNAVAILABLE

    "endpoint" : \[{ Reference(Endpoint) }\], // Series access endpoint

    "bodySite" : { Coding }, // Body part examined

    "laterality" : { Coding }, // Body part laterality

    "started" : "<dateTime>", // When the series started

    "performer" : \[{ Reference(Practitioner) }\], // Who performed the series

    "instance" : \[{ // A single SOP instance from the series

      "uid" : "<oid>", // R!  Formal DICOM identifier for this instance

      "number" : "<unsignedInt>", // The number of this instance in the series

      "sopClass" : "<oid>", // R!  DICOM class type

      "title" : "<string>" // Description of instance

    }\]

  }\]

}

\`\`\`

<div style="page-break-after: always;"></div>

\#### ImagingStudy Example

\`\`\`json

{

  "resourceType": "ImagingStudy",

  "id": "example-xr",

  "text": {

    "status": "generated",

    "div": "<div xmlns=\\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml "http://www.w3.org/1999/xhtml")\\">XR Wrist 3+ Views. John Smith (MRN: 09236). Accession: W12342398. Performed: 2017-01-01. 1 series, 2 images.</div>"

  },

  "uid": "urn:oid:2.16.124.113543.6003.1154777499.30246.19789.3503430046",

  "accession": {

    "use": "usual",

    "type": {

      "coding": \[

        {

          "system": "[http://hl7.org/fhir/v2/0203](http://hl7.org/fhir/v2/0203 "http://hl7.org/fhir/v2/0203")",

          "code": "ACSN"

        }

      \]

    },

    "value": "W12342398",

    "assigner": {

      "reference": "Organization/dicom-organization"

    }

  },

  "identifier": \[

    {

      "use": "secondary",

      "value": "55551234",

      "assigner": {

        "reference": "Organization/dicom-organization"

      }

    }

  \],

  "availability": "ONLINE",

  "modalityList": \[

    {

      "system": "[http://dicom.nema.org/resources/ontology/DCM](http://dicom.nema.org/resources/ontology/DCM "http://dicom.nema.org/resources/ontology/DCM")",

      "code": "DX"

    }

  \],

  "patient": {

    "reference": "Patient/dicom"

  },

  "context": {

    "reference": "Encounter/example"

  },

  "started": "2017-01-01T11:01:20+03:00",

  "basedOn": \[

    {

      "reference": "ProcedureRequest/example"

    }

  \],

  "referrer": {

    "reference": "Practitioner/example"

  },

  "interpreter": \[

    {

      "reference": "Practitioner/example"

    }

  \],

  "endpoint": \[

    {

      "reference": "Endpoint/example-wadors"

    }

  \],

  "numberOfSeries": 1,

  "numberOfInstances": 2,

  "procedureReference": \[

    {

      "reference": "Procedure/example"

    }

  \],

  "procedureCode": \[

    {

      "coding": \[

        {

          "system": "[http://www.radlex.org](http://www.radlex.org "http://www.radlex.org")",

          "code": "RPID2589",

          "display": "XR Wrist 3+ Views"

        }

      \],

      "text": "XR Wrist 3+ Views"

    }

  \],

  "reason": {

    "coding": \[

      {

        "system": "[http://snomed.info/sct](http://snomed.info/sct "http://snomed.info/sct")",

        "code": "357009",

        "display": "Closed fracture of trapezoidal bone of wrist"

      }

    \]

  },

  "description": "XR Wrist 3+ Views",

  "series": \[

    {

      "uid": "urn:oid:2.16.124.113543.6003.1154777499.30246.19789.3503430045.1",

      "number": 3,

      "modality": {

        "system": "[http://dicom.nema.org/resources/ontology/DCM](http://dicom.nema.org/resources/ontology/DCM "http://dicom.nema.org/resources/ontology/DCM")",

        "code": "DX"

      },

      "description": "XR Wrist 3+ Views",

      "numberOfInstances": 2,

      "availability": "ONLINE",

      "endpoint": \[

        {

          "reference": "Endpoint/example-wadors"

        }

      \],

      "bodySite": {

        "system": "[http://snomed.info/sct](http://snomed.info/sct "http://snomed.info/sct")",

        "code": "T-15460",

        "display": "Wrist Joint"

      },

      "laterality": {

        "system": "[http://snomed.info/sct](http://snomed.info/sct "http://snomed.info/sct")",

        "code": "419161000",

        "display": "Unilateral left"

      },

      "started": "2011-01-01T11:01:20+03:00",

      "performer": \[

        {

          "reference": "Practitioner/example"

        }

      \],

      "instance": \[

        {

          "uid": "urn:oid:2.16.124.113543.6003.1154777499.30246.19789.3503430045.1.1",

          "number": 1,

          "sopClass": "urn:oid:1.2.840.10008.5.1.4.1.1.2",

          "title": "PA VIEW"

        },

        {

          "uid": "urn:oid:2.16.124.113543.6003.1154777499.30246.19789.3503430045.1.2",

          "number": 2,

          "sopClass": "urn:oid:1.2.840.10008.5.1.4.1.1.2",

          "title": "LL VIEW"

        }

      \]

    }

  \]

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Sequence Document Template

\`\`\`json

{

  "resourceType" : "Sequence",

  // from Resource: id, meta, implicitRules, and language

  // from DomainResource: text, contained, extension, and modifierExtension

  "identifier" : \[{ Identifier }\], // Unique ID for this particular sequence. This is a FHIR-defined id

  "type" : "<code>", // aa | dna | rna

  "coordinateSystem" : <integer>, // R!  Base number of coordinate system (0 for 0-based numbering or coordinates, inclusive start, exclusive end, 1 for 1-based numbering, inclusive start, inclusive end)

  "patient" : { Reference(Patient) }, // Who and/or what this is about

  "specimen" : { Reference(Specimen) }, // Specimen used for sequencing

  "device" : { Reference(Device) }, // The method for sequencing

  "performer" : { Reference(Organization) }, // Who should be responsible for test result

  "quantity" : { Quantity }, // The number of copies of the seqeunce of interest.  (RNASeq)

  "referenceSeq" : { // A sequence used as reference

    "chromosome" : { CodeableConcept }, // Chromosome containing genetic finding

    "genomeBuild" : "<string>", // The Genome Build used for reference, following GRCh build versions e.g. 'GRCh 37'

    "referenceSeqId" : { CodeableConcept }, // Reference identifier

    "referenceSeqPointer" : { Reference(Sequence) }, // A Pointer to another Sequence entity as reference sequence

    "referenceSeqString" : "<string>", // A string to represent reference sequence

    "strand" : <integer>, // Directionality of DNA ( +1/-1)

    "windowStart" : <integer>, // R!  Start position of the window on the  reference sequence

    "windowEnd" : <integer> // R!  End position of the window on the reference sequence

  },

  "variant" : \[{ // Variant in sequence

    "start" : <integer>, // Start position of the variant on the  reference sequence

    "end" : <integer>, // End position of the variant on the reference sequence

    "observedAllele" : "<string>", // Allele that was observed

    "referenceAllele" : "<string>", // Allele in the reference sequence

    "cigar" : "<string>", // Extended CIGAR string for aligning the sequence with reference bases

    "variantPointer" : { Reference(Observation) } // Pointer to observed variant information

  }\],

  "observedSeq" : "<string>", // Sequence that was observed

  "quality" : \[{ // An set of value as quality of sequence

    "type" : "<code>", // R!  indel | snp | unknown

    "standardSequence" : { CodeableConcept }, // Standard sequence for comparison

    "start" : <integer>, // Start position of the sequence

    "end" : <integer>, // End position of the sequence

    "score" : { Quantity }, // Quality score for the comparison

    "method" : { CodeableConcept }, // Method to get quality

    "truthTP" : <decimal>, // True positives from the perspective of the truth data

    "queryTP" : <decimal>, // True positives from the perspective of the query data

    "truthFN" : <decimal>, // False negatives

    "queryFP" : <decimal>, // False positives

    "gtFP" : <decimal>, // False positives where the non-REF alleles in the Truth and Query Call Sets match

    "precision" : <decimal>, // Precision of comparison

    "recall" : <decimal>, // Recall of comparison

    "fScore" : <decimal> // F-score

  }\],

  "readCoverage" : <integer>, // Average number of reads representing a given nucleotide in the reconstructed sequence

  "repository" : \[{ // External repository which contains detailed report related with observedSeq in this resource

    "type" : "<code>", // R!  directlink | openapi | login | oauth | other

    "url" : "<uri>", // URI of the repository

    "name" : "<string>", // Repository's name

    "datasetId" : "<string>", // Id of the dataset that used to call for dataset in repository

    "variantsetId" : "<string>", // Id of the variantset that used to call for variantset in repository

    "readsetId" : "<string>" // Id of the read

  }\],

  "pointer" : \[{ Reference(Sequence) }\] // Pointer to next atomic sequence

}

\`\`\`

<div style="page-break-after: always;"></div>

\#### Sequence Example

\`\`\`json

{

  "resourceType": "Sequence",

  "id": "example-pgx-2",

  "text": {

    "status": "generated",

    "div": "<div xmlns=\\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml "http://www.w3.org/1999/xhtml")\\"><p><b>Generated Narrative with Details</b></p><p><b>id</b>: example-pgx-2</p><p><b>type</b>: dna</p><p><b>coordinateSystem</b>: 0</p><p><b>patient</b>: <a>Patient/example</a></p><h3>ReferenceSeqs</h3><table><tr><td>-</td><td><b>ReferenceSeqId</b></td><td><b>Strand</b></td><td><b>WindowStart</b></td><td><b>WindowEnd</b></td></tr><tr><td>*</td><td>NG_007726.3 <span>(Details : {[http://www.ncbi.nlm.nih.gov/nuccore](http://www.ncbi.nlm.nih.gov/nuccore "http://www.ncbi.nlm.nih.gov/nuccore") code 'NG_007726.3' = 'NG_007726.3)</span></td><td>1</td><td>55227970</td><td>55227980</td></tr></table><h3>Variants</h3><table><tr><td>-</td><td><b>Start</b></td><td><b>End</b></td><td><b>ObservedAllele</b></td><td><b>ReferenceAllele</b></td><td><b>VariantPointer</b></td></tr><tr><td>*</td><td>55227978</td><td>55227979</td><td>G</td><td>T</td><td><a>Target Haplotype Observation</a></td></tr></table></div>"

  },

  "type": "dna",

  "coordinateSystem": 0,

  "patient": {

    "reference": "Patient/example"

  },

  "referenceSeq": {

    "referenceSeqId": {

      "coding": \[

        {

          "system": "[http://www.ncbi.nlm.nih.gov/nuccore](http://www.ncbi.nlm.nih.gov/nuccore "http://www.ncbi.nlm.nih.gov/nuccore")",

          "code": "NG_007726.3"

        }

      \]

    },

    "strand": 1,

    "windowStart": 55227970,

    "windowEnd": 55227980

  },

  "variant": \[

    {

      "start": 55227978,

      "end": 55227979,

      "observedAllele": "G",

      "referenceAllele": "T",

      "variantPointer": {

        "reference": "Observation/example-haplotype2",

        "display": "Target Haplotype Observation"

      }

    }

  \]

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Observation Document Template

\`\`\`json

{

  "resourceType" : "Observation",

  // from Resource: id, meta, implicitRules, and language

  // from DomainResource: text, contained, extension, and modifierExtension

  "identifier" : \[{ Identifier }\], // Business Identifier for observation

  "basedOn" : \[{ Reference(CarePlan|DeviceRequest|ImmunizationRecommendation|

   MedicationRequest|NutritionOrder|ProcedureRequest|ReferralRequest) }\], // Fulfills plan, proposal or order

  "status" : "<code>", // R!  registered | preliminary | final | amended +

  "category" : \[{ CodeableConcept }\], // Classification of  type of observation

  "code" : { CodeableConcept }, // R!  Type of observation (code / type)

  "subject" : { Reference(Patient|Group|Device|Location) }, // Who and/or what this is about

  "context" : { Reference(Encounter|EpisodeOfCare) }, // Healthcare event during which this observation is made

  // effective\[x\]: Clinically relevant time/time-period for observation. One of these 2:

  "effectiveDateTime" : "<dateTime>",

  "effectivePeriod" : { Period },

  "issued" : "<instant>", // Date/Time this was made available

  "performer" : \[{ Reference(Practitioner|Organization|Patient|RelatedPerson) }\], // Who is responsible for the observation

  // value\[x\]: Actual result. One of these 11:

  "valueQuantity" : { Quantity },

  "valueCodeableConcept" : { CodeableConcept },

  "valueString" : "<string>",

  "valueBoolean" : <boolean>,

  "valueRange" : { Range },

  "valueRatio" : { Ratio },

  "valueSampledData" : { SampledData },

  "valueAttachment" : { Attachment },

  "valueTime" : "<time>",

  "valueDateTime" : "<dateTime>",

  "valuePeriod" : { Period },

  "dataAbsentReason" : { CodeableConcept }, // C? Why the result is missing

  "interpretation" : { CodeableConcept }, // High, low, normal, etc.

  "comment" : "<string>", // Comments about result

  "bodySite" : { CodeableConcept }, // Observed body part

  "method" : { CodeableConcept }, // How it was done

  "specimen" : { Reference(Specimen) }, // Specimen used for this observation

  "device" : { Reference(Device|DeviceMetric) }, // (Measurement) Device

  "referenceRange" : \[{ // Provides guide for interpretation

    "low" : { Quantity(SimpleQuantity) }, // C? Low Range, if relevant

    "high" : { Quantity(SimpleQuantity) }, // C? High Range, if relevant

    "type" : { CodeableConcept }, // Reference range qualifier

    "appliesTo" : \[{ CodeableConcept }\], // Reference range population

    "age" : { Range }, // Applicable age range, if relevant

    "text" : "<string>" // Text based reference range in an observation

  }\],

  "related" : \[{ // Resource related to this observation

    "type" : "<code>", // has-member | derived-from | sequel-to | replaces | qualified-by | interfered-by

    "target" : { Reference(Observation|QuestionnaireResponse|Sequence) } // R!  Resource that is related to this one

  }\],

  "component" : \[{ // Component results

    "code" : { CodeableConcept }, // R!  Type of component observation (code / type)

    // value\[x\]: Actual component result. One of these 10:

    "valueQuantity" : { Quantity },

    "valueCodeableConcept" : { CodeableConcept },

    "valueString" : "<string>",

    "valueRange" : { Range },

    "valueRatio" : { Ratio },

    "valueSampledData" : { SampledData },

    "valueAttachment" : { Attachment },

    "valueTime" : "<time>",

    "valueDateTime" : "<dateTime>",

    "valuePeriod" : { Period },

    "dataAbsentReason" : { CodeableConcept }, // C? Why the component result is missing

    "interpretation" : { CodeableConcept }, // High, low, normal, etc.

    "referenceRange" : \[{ Content as for Observation.referenceRange }\] // Provides guide for interpretation of component result

  }\]

}

\`\`\`

<div style="page-break-after: always;"></div>

\#### Observation Example

\`\`\`json

{

  "resourceType": "Observation",

  "id": "example-phenotype",

  "text": {

    "status": "generated",

    "div": "<div xmlns=\\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml "http://www.w3.org/1999/xhtml")\\"><p><b>Generated Narrative with Details</b></p><p><b>id</b>: example-phenotype</p><p><b>status</b>: final</p><p><b>code</b>: CYP2C9 gene product metabolic activity interpretation <span>(Details : {LOINC code '79716-7' = 'CYP2C9 gene product metabolic activity interpretation in Blood or Tissue Qualitative by CPIC', given as 'CYP2C9 gene product metabolic activity interpretation'})</span></p><p><b>subject</b>: <a>J*********** C***********</a></p><p><b>issued</b>: 03/04/2013 3:30:10 PM</p><p><b>value</b>: Normal metabolizer <span>(Details : {LOINC code 'LA25391-6' = 'Normal metabolizer', given as 'Normal metabolizer'})</span></p><p><b>specimen</b>: <a>Molecular Specimen ID: MLD45-Z4-1234</a></p><h3>Relateds</h3><table><tr><td>-</td><td><b>Type</b></td><td><b>Target</b></td></tr><tr><td>*</td><td>derived-from</td><td><a>Observation/example-diplotype1</a></td></tr></table></div>"

  },

  "extension": \[

    {

      "url": "[http://hl7.org/fhir/StructureDefinition/observation-geneticsGene](http://hl7.org/fhir/StructureDefinition/observation-geneticsGene "http://hl7.org/fhir/StructureDefinition/observation-geneticsGene")",

      "valueCodeableConcept": {

        "coding": \[

          {

            "system": "[http://www.genenames.org](http://www.genenames.org "http://www.genenames.org")",

            "code": "2623",

            "display": "CYP2C9"

          }

        \]

      }

    }

  \],

  "status": "final",

  "code": {

    "coding": \[

      {

        "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

        "code": "79716-7",

        "display": "CYP2C9 gene product metabolic activity interpretation"

      }

    \]

  },

  "subject": {

    "reference": "Patient/727127",

    "display": "J*********** C***********"

  },

  "issued": "2013-04-03T15:30:10+01:00",

  "valueCodeableConcept": {

    "coding": \[

      {

        "system": "[http://loinc.org](http://loinc.org "http://loinc.org")",

        "code": "LA25391-6",

        "display": "Normal metabolizer"

      }

    \]

  },

  "specimen": {

    "reference": "Specimen/genetics-example1-somatic",

    "display": "Molecular Specimen ID: MLD45-Z4-1234"

  },

  "related": \[

    {

      "type": "derived-from",

      "target": {

        "reference": "Observation/example-diplotype1"

      }

    }

  \]

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Media Document Template

\`\`\`json

{

  "resourceType" : "Media",

  // from Resource: id, meta, implicitRules, and language

  // from DomainResource: text, contained, extension, and modifierExtension

  "identifier" : \[{ Identifier }\], // Identifier(s) for the image

  "basedOn" : \[{ Reference(ServiceRequest|CarePlan) }\], // Procedure that caused this media to be created

  "partOf" : \[{ Reference(Any) }\], // Part of referenced event

  "status" : "<code>", // R!  preparation | in-progress | not-done | suspended | aborted | completed | entered-in-error | unknown

  "type" : { CodeableConcept }, // Classification of media as image, video, or audio

  "modality" : { CodeableConcept }, // The type of acquisition equipment/process

  "view" : { CodeableConcept }, // Imaging view, e.g. Lateral or Antero-posterior

  "subject" : { Reference(Patient|Practitioner|Group|Device|Specimen|Location) }, // Who/What this Media is a record of

  "context" : { Reference(Encounter|EpisodeOfCare) }, // Encounter / Episode associated with media

  // created\[x\]: When Media was collected. One of these 2:

  "createdDateTime" : "<dateTime>",

  "createdPeriod" : { Period },

  "issued" : "<instant>", // Date/Time this version was made available

  "operator" : { Reference(Practitioner|PractitionerRole|Organization|

   CareTeam|Patient|Device|RelatedPerson) }, // The person who generated the image

  "reasonCode" : \[{ CodeableConcept }\], // Why was event performed?

  "bodySite" : { CodeableConcept }, // Observed body part

  "deviceName" : "<string>", // Name of the device/manufacturer

  "device" : { Reference(Device|DeviceMetric|DeviceComponent) }, // Observing Device

  "height" : "<positiveInt>", // Height of the image in pixels (photo/video)

  "width" : "<positiveInt>", // Width of the image in pixels (photo/video)

  "frames" : "<positiveInt>", // Number of frames if > 1 (photo)

  "duration" : <decimal>, // Length in seconds (audio / video)

  "content" : { Attachment }, // R!  Actual Media - reference or data

  "note" : \[{ Annotation }\] // Comments made about the media

}

\`\`\`

<div style="page-break-after: always;"></div>

\#### Media Document Example

\`\`\`json

{

  "resourceType": "Media",

  "id": "xray",

  "text": {

    "status": "generated",

    "div": "<div xmlns=\\"[http://www.w3.org/1999/xhtml](http://www.w3.org/1999/xhtml "http://www.w3.org/1999/xhtml")\\">Xray of left hand for Patient Henry Levin (MRN 12345) 2016-03-15</div>"

  },

  "basedOn": \[

    {

      "identifier": {

        "system": "[http://someclinic.org/fhir/NamingSystem/imaging-orders](http://someclinic.org/fhir/NamingSystem/imaging-orders "http://someclinic.org/fhir/NamingSystem/imaging-orders")",

        "value": "111222",

        "assigner": {

          "display": "XYZ Medical Clinic"

        }

      }

    }

  \],

  "status": "completed",

  "modality": {

    "coding": \[

      {

        "system": "[http://snomed.info/sct](http://snomed.info/sct "http://snomed.info/sct")",

        "code": "39714003",

        "display": "Skeletal X-ray of wrist and hand"

      }

    \]

  },

  "subject": {

    "reference": "Patient/example"

  },

  "context": {

    "reference": "Encounter/example"

  },

  "createdDateTime": "2016-03-15",

  "bodySite": {

    "coding": \[

      {

        "system": "[http://snomed.info.sct](http://snomed.info.sct "http://snomed.info.sct")",

        "code": "85151006",

        "display": "Structure of left hand (body structure)"

      }

    \]

  },

  "height": 432,

  "width": 640,

  "content": {

    "id": "a1",

    "contentType": "image/jpeg",

    "url": "[http://someimagingcenter.org/fhir/Binary/A12345](http://someimagingcenter.org/fhir/Binary/A12345 "http://someimagingcenter.org/fhir/Binary/A12345")",

    "creation": "2016-03-15"

  }

}

\`\`\`

<div style="page-break-after: always;"></div>

\## Using Azure Cosmos DB as a FHIR Repository for Dicom 

A referenced <a href="[https://www.dicomlibrary.com/dicom/sop/](https://www.dicomlibrary.com/dicom/sop/ "https://www.dicomlibrary.com/dicom/sop/")" target="_blank">DICOM SOP instance</a> could be:

\* A single- or multi-frame, still or video image captured by a variety of imaging modalities, such as X-ray, MR, and ultrasound.

\* A set of various presentation parameters, including annotation and markup.

\* A set of measurements or a report, including radiation dose report and CAD analysis.

\* An encapsulated PDF or CDA document.

\* A list of instances, such as key “of interest” images, or instances to be “deleted”; or

Other DICOM content.  

The **<a href="[http://hl7.org/fhir/2018May/media.html](http://hl7.org/fhir/2018May/media.html "http://hl7.org/fhir/2018May/media.html")" target="_blank">Media</a>** resource captures a specific type of Observation - an Observation whose value is audio, video or image data. This resource is the preferred representation of such forms of information as it exposes the metadata relevant for interpreting the information. However, in some legacy environments, media information may occasionally appear in Observation instead. Systems should be aware of this possibility.

The Media resource is able to contain medical images in a DICOM format. These images may also be made accessible through an ImagingStudy resource, which provides a direct reference to the image to a WADO-RS server.

\`Media.content\` field stores the actual content of the media - inline or by direct reference to the media source file. This is a required field and is the **Attachment** Data Type. This type is for containing or referencing attachments - additional data content defined in other formats. The most common use of this type is to include images or reports in some report format such as PDF. However it can be used for any data that has a MIME type.

<div style="page-break-after: always;"></div>

\## Anatomy of a Dicom file

As we already know a DICOM file storing one image contain the image data and data belonging to the patient and data (name, age, etc.) belonging to the examination (date of acquisition, manufacturer, etc.) and identifiers: the study UID, the series’ UID’s, and the image UID’s.  

he software that interprets the image will have to be able to find, first of all, the part of the DICOM file containing the image; also all of the identifiers and the other data contained in the DICOM file. The DICOM standard has a special pair of characters, the parentheses and the comma: ’(’ and ’)’ and ’,’. Now, numbers of 2x4 hexadecimal digits enclosed by the these parentheses and separated by the comma uniquely identify a specific DICOM field or data. For instance this tag:

\`(0010,0010)\` is the identifier of the patient’s name - „ten-ten is the patient name” as DICOM experts would say. The last thing that we have to learn is that the data, in this case the patient name is enclosed by a pair of the tag shown above:

Here is a decoded segment of the DICOM information found in a DICOM file:

\### Dicom-File-Format

| Dicom-Meta-Information-Header            |

|------------------------------------------|

| Used TransferSyntax: LittleEndianExplicit |

| (0002,0000) UL 182 # 4, 1 MetaElementGroupLength |

| (0002,0001) OB 00\\01 # 2, 1 FileMetaInformationVersion |

| (0002,0002) UI =CTImageStorage # 26, 1 MediaStorageSOPClassUID  |

| (0002,0003) UI 1.3.12.2.1107.5.1.1.20377.20031125114113176.4 # 46, 1 MediaStorageSOPInstanceUID  |

| (0002,0010) UI =LittleEndianImplicit # 18, 1 TransferSyntaxUID  |

| (0002,0012) UI 1.2.826.0.1.3680043.2.93.0.99 # 30, 1 ImplementationClassUID  |

| (0002,0013) SH ERAD_60 # 8, 1 ImplementationVersionName |

| Dicom-Data-Set                           |

|------------------------------------------|

| Used TransferSyntax: LittleEndianImplicit |

| (0008,0005) CS ISO_IR 100 # 10, 1 SpecificCharacterSet  |

| (0008,0008) CS ORIGINAL\\PRIMARY\\LOCALIZER\\CT_SOM4 TOP # 38, 4 ImageType  |

| (0008,0016) UI =CTImageStorage # 26, 1 SOPClassUID  |

| (0008,0018) UI 1.3.12.2.1107.5.1.1.20377.20031125114113176.4 # 46, 1 SOPInstanceUID  |

| (0008,0020) DA 20031125 # 8, 1 StudyDate  |

| (0008,0021) DA 20031125 # 8, 1 SeriesDate  |

| (0008,0022) DA 20031125 # 8, 1 AcquisitionDate  |

| (0008,0023) DA 20031125 # 8, 1 ContentDate  |

| (0008,0030) TM 113945.000000 # 14, 1 StudyTime  |

| (0008,0031) TM 114003.384000 # 14, 1 SeriesTime  |

| (0008,0032) TM 114109.299000 # 14, 1 AcquisitionTime  |

| (0008,0033) TM 114109.299000 # 14, 1 ContentTime  |

| (0008,0040) US 0 # 2, 1 ACR_NEMA_OldDataSetType  |

| (0008,0041) LO IMA TOPO # 8, 1 ACR_NEMA_DataSetSubtype  |

| (0008,0060) CS CT # 2, 1 Modality        |

| (0008,1090) LO SOMATOM PLUS 4 # 14, 1 ManufacturersModelName  |

\#### Key to Dicom-File-Format

\* The first column contains the DICOM tags.

\* The second column contains the actual content (prefixed by data type).

\* The third column contains the actual number of characters belonging.

\* The fourth column tells if the actual field has more than one data contained (these are back slash separated like in the second row) and the official DICOM name of the field.

The <u>DICOM standard has specific names</u> for the different DICOM tags or data or DICOM field identified by a certain pair of tags. For instance (0008,0060) is the tag identifying the DICOM field called Modality. The content of this field above is CT. Another example is the StudyDate being the official name of the field containing the actual date of the study: the tag is (0008,0020) and the content above is 20031006, that is November 6, 2003.

> !\[\](![](https://i.imgur.com/eSPsKWl.png)) INFO:  All told there are more than 750 DICOM Tags 

Sample Dicom Image

!\[\](![](https://i.imgur.com/B8YCt9A.png))

\### The following is the Tag list from the above sample 

\`\`\`

Tag	        Description	                Value

(0002,0002)	Media Storage SOP Class UID	 1.2.840.10008.5.1.4.1.1.4

(0002,0003)	Media Storage SOP Instance UID	 1.3.12.2.1107.5.8.1.12345.200510141312350402721

(0002,0010)	Transfer Syntax UID	 1.2.840.10008.1.2.1

(0002,0012)	Implementation Class UID	 2.16.840.1.113669.2.931128

(0002,0013)	Implementation Version Name	 VB33D 

(0008,0005)	Specific Character Set	 ISO_IR 100

(0008,0008)	Image Type	 ORIGINAL\\PRIMARY\\OTHER

(0008,0016)	SOP Class UID	 1.2.840.10008.5.1.4.1.1.4

(0008,0018)	SOP Instance UID 	1.3.12.2.1107.5.8.1.12345.200510141312350402721

(0008,0020)	Study Date	20051014

(0008,0021)	Series Date	20051014

(0008,0022)	Acquisition Date 20051014	

(0008,0023)	Content Date	20051014

(0008,0030)	Study Time	125256.915

(0008,0031)	Series Time	130031.772

(0008,0032)	Acquisition Time 	130033.21

(0008,0033)	Content Time	130444.789

(0008,0050)	Accession Number 	

(0008,0060)	Modality	 MR

(0008,0070)	Manufacturer	 SIEMENS 

(0008,0080)	Institution Name	 MDL RUSEV MRI SOFIA 

(0008,0090)	Referring Physician's Name	 

(0008,1010)	Station Name	 mrc 

(0008,103E)	Series Description	 MN-166-CL-001/T1W-MN

(0008,1090)	Manufacturer's Model Name 	MAGNETOM EXPERT 

(0009,0010)	Private Tag	 SPI RELEASE 1 

(0009,0012)	Private Tag	 SIEMENS CM VA0  CMS 

(0009,0013)	Private Tag	 SIEMENS CM VA0  LAB 

(0009,1010)	Private Tag	 SPI VERSION 01.00 

(0009,1015)	Private Tag	 000S00MR002005101414123454

(0009,1040)	Private Tag	0

(0009,1041)	Private Tag	 MRUPNONE

(0009,1210)	Private Tag	 STANDARD

(0009,1226)	Private Tag	20051014

(0009,1227)	Private Tag	125343

(0009,1316)	Private Tag	 723a7812

(0009,1320)	Private Tag	 VC

(0010,0010)	Patient's Name	 BU001015/MN-166-CL-001/V01

(0010,0020)	Patient ID	 BU001015

(0010,0030)	Patient's Birth Date	19511010

(0010,0040)	Patient's Sex	 F 

(0010,1010)	Patient's Age	 054Y

(0010,1030)	Patient's Weight 	60

(0011,0010)	Private Tag	 SPI RELEASE 1 

(0011,0011)	Private Tag	 SIEMENS CM VA0  CMS 

(0011,1110)	Private Tag	20051014

(0011,1111)	Private Tag	125256.915

(0011,1123)	Private Tag	60

(0018,0015)	Body Part Examined	 

(0018,0020)	Scanning Sequence	RM

(0018,0021)	Sequence Variant 	OSP 

(0018,0022)	Scan Options	 

(0018,0023)	MR Acquisition Type 	2D

(0018,0024)	Sequence Name	 se1 

(0018,0025)	Angio Flag	 N 

(0018,0050)	Slice Thickness	3.00E+00

(0018,0080)	Repetition Time	5.50E+02

(0018,0081)	Echo Time	1.20E+01

(0018,0083)	Number of Averages2	

(0018,0084)	Imaging Frequency 	4.05E+01

(0018,0085)	Imaged Nucleus	 1H

(0018,0086)	Echo Numbers(s)	1

(0018,0087)	Magnetic Field Strength	0.9500702

(0018,0088)	Spacing Between Slices	3

(0018,0091)	Echo Train Length	1

(0018,0093)	Percent Sampling	100

(0018,0094)	Percent Phase Field of View	87.5

(0018,1000)	Device Serial Number	9076

(0018,1020)	Software Versions(s)	 VB33G 

(0018,1200)	Date of Last Calibration 	20040203

(0018,1201)	Time of Last Calibration 	131253

(0018,1250)	Receive Coil Name	 CP Head 

(0018,1310)	Acquisition Matrix	 0 512 224 0 

(0018,1312)	In-plane Phase Encoding Direction	 ROW 

(0018,1314)	Flip Angle	90

(0018,1316)	SAR	0

(0018,5100)	Patient Position	 HFS 

(0019,0010)	Private Tag	 SIEMENS CM VA0  CMS 

(0019,0012)	Private Tag	 SIEMENS MR VA0  GEN 

(0019,0014)	Private Tag	 SIEMENS MR VA0  COAD

(0019,0015)	Private Tag	 SIEMENS CM VA0  ACQU

(0019,1010)	Private Tag	50

(0019,1020)	Private Tag	 EXAMNONE

(0019,1030)	Private Tag	 A   IRS 

(0019,1050)	Private Tag	0

(0019,1060)	Private Tag	131072

(0019,1210)	Private Tag	2.50E+02

(0019,1211)	Private Tag	2.50E+02

(0019,1212)	Private Tag	1.35E-43

(0019,1213)	Private Tag	1.50E+01

(0019,1214)	Private Tag	1

(0019,1220)	Private Tag	224

(0019,1221)	Private Tag	224

(0019,1226)	Private Tag	111

(0019,1228)	Private Tag	-112

(0019,1230)	Private Tag	512

(0019,1231)	Private Tag	512

(0019,1240)	Private Tag	0

(0019,1245)	Private Tag	4

(0019,1250)	Private Tag	2

(0019,1260)	Private Tag	9.00E+01

(0019,1270)	Private Tag	4

(0019,1281)	Private Tag	 NONE    

(0019,1283)	Private Tag	 NONE    

(0019,1285)	Private Tag	 NONE    

(0019,1287)	Private Tag	 NONE  

(0019,1290)	Private Tag	0

(0019,1294)	Private Tag	1.57E+00

(0019,1298)	Private Tag	 000.000000E+00\\00.000000E+00\\00.000000E+00

(0019,1412)	Private Tag	9.50E-01

(0019,1414)	Private Tag	1.00E+01

(0019,1416)	Private Tag	 008.962891E+00\\01.460547E+01

(0019,1420)	Private Tag	1.66E+02

(0019,1421)	Private Tag	2

(0019,1422)	Private Tag	2.50E+01

(0019,1424)	Private Tag	4.65E+01

(0019,1426)	Private Tag	5.49E+01

(0019,1450)	Private Tag	9.38E+01

(0019,1451)	Private Tag	5.50E+01

(0019,1452)	Private Tag	5.54E+01

(0019,1454)	Private Tag	-6.00E-01

(0019,1455)	Private Tag	9.91E+01

(0019,1456)	Private Tag	33500

(0019,1460)	Private Tag	2.15E-01

(0019,1462)	Private Tag	6.90E-01

(0019,1470)	Private Tag	3.13E+00

(0019,1471)	Private Tag	3.13E+00

(0019,1472)	Private Tag	8.00E+00

(0019,1480)	Private Tag	 001.430000E+02\\01.560000E+02\\01.470000E+02

(0019,1482)	Private Tag	2.00E+02

(0019,14A0)	Private Tag	1

(0019,14A2)	Private Tag	1.15E+00

(0019,14A5)	Private Tag	 001.500000E+00\\06.341736E-02\\00.000000E+00

(0019,14A6)	Private Tag	 001.000000E+02\\06.042819E+00\\06.360782E+00

(0019,14D1)	Private Tag	0.00E+00

(0019,14D2)	Private Tag	 NONE

(0019,14D3)	Private Tag	8.75E-01

(0019,14D4)	Private Tag	256

(0019,14D5)	Private Tag	0

(0019,14D6)	Private Tag	0

(0019,14D7)	Private Tag	129

(0019,14D8)	Private Tag	258

(0019,14D9)	Private Tag	1

(0019,14DA)	Private Tag	 PLUSX 

(0019,1510)	Private Tag	 /usr/appl/proto/014/c/head/MN-166-CL-001/T1W-MN.prg 

(0019,1511)	Private Tag	 /usr/appl/sequence/se_12b130.wfc

(0019,1512)	Private Tag	 SIEMENS 

(0019,1513)	Private Tag	 se1 

(0020,000D)	Study Instance UID	 1.3.12.2.1107.5.2.2.9076.20051014125256000

(0020,000E)	Series Instance UID	 1.3.12.2.1107.5.2.2.9076.20051014130031000003

(0020,0010)	Study ID	1

(0020,0011)	Series Number	3

(0020,0012)	Acquisition Number 	1

(0020,0013)	Instance Number	12

(0020,0030)	Image Position	 -01.216560E+02\\-8.717901E+01\\09.253483E+01

(0020,0032)	Image Position (Patient)	 -122.14433\\-87.650655\\92.661205 

(0020,0035)	Private Tag	 009.999999E-01\\-0.000000E+00\\-0.000000E+00\\00.000000E+00\\09.659258E-01\\-2.588190E-01

(0020,0037)	Image Orientation (Patient)	 0.9999999\\-0\\-0\\0\\0.9659258\\-0.258819 

(0020,0050)	Location	-6.68E+01

(0020,0052)	Frame of Reference UID	1.3.12.2.1107.5.2.2.9076.20051014125343000

(0020,0070)	Image Geometry Type	 PLANAR

(0020,0080)	Masking Image	 

(0020,1001)	Private Tag	1

(0020,1040)	Position Reference Indicator	 

(0020,1041)	Slice Location	-6.68E+01

(0020,4000)	Image Comments	 Magnevist 0.1mmol/kg,TPK                            

(0021,0010)	Private Tag	 SIEMENS MED 

(0021,0011)	Private Tag	 SIEMENS CM VA0  CMS 

(0021,0013)	Private Tag	 SIEMENS MR VA0  GEN 

(0021,0023)	Private Tag	 SIEMENS MR VA0  RAW 

(0021,1020)	Private Tag	0

(0021,1120)	Private Tag	 002.500000E+02\\02.500000E+02

(0021,1122)	Private Tag	1.00E+00

(0021,1124)	Private Tag	 000.000000E+00\\00.000000E+00

(0021,1126)	Private Tag	0

(0021,1130)	Private Tag	 HEAD

(0021,1132)	Private Tag	 HEAD

(0021,1160)	Private Tag	 002.855656E+00\\-3.309007E+01\\-6.030883E+01

(0021,1161)	Private Tag	 -00.000000E+00\\02.588190E-01\\09.659257E-01

(0021,1163)	Private Tag	-6.68E+01

(0021,1165)	Private Tag	5

(0021,116A)	Private Tag	 009.999999E-01\\00.000000E+00\\00.000000E+00

(0021,116B)	Private Tag	 000.000000E+00\\-9.659258E-01\\02.588190E-01

(0021,1170)	Private Tag	 R \\AH\\HP

(0021,1171)	Private Tag	 L \\PF\\FA

(0021,1182)	Private Tag	 MEA 

(0021,1322)	Private Tag	0

(0021,1324)	Private Tag	0

(0021,1330)	Private Tag	0

(0021,1331)	Private Tag	0

(0021,1334)	Private Tag	0

(0021,1339)	Private Tag	3.00E+00

(0021,1340)	Private Tag	23

(0021,1341)	Private Tag	23

(0021,1342)	Private Tag	14

(0021,1343)	Private Tag	1

(0021,1344)	Private Tag	1.00E+00

(0021,134F)	Private Tag	 INTERLEAVED 

(0021,1356)	Private Tag	5.50E+02

(0021,1370)	Private Tag	1

(0021,2300)	Private Tag	 IMAG

(0021,2301)	Private Tag	512

(0021,2302)	Private Tag	512

(0021,2303)	Private Tag	3.33E+04

(0021,2304)	Private Tag	 000.000000E+00\\00.000000E+00\\00.000000E+00

(0021,2305)	Private Tag	 000.000000E+00\\00.000000E+00\\00.000000E+00

(0021,2306)	Private Tag	    256\\  256\\    0

(0021,2307)	Private Tag	    256\\  256\\    0

(0021,2308)	Private Tag	 000.000000E+00\\00.000000E+00\\00.000000E+00

(0021,2309)	Private Tag	0.00E+00

(0021,2310)	Private Tag	4.10E+03

(0021,2331)	Private Tag	005.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01\\05.544000E+01

(0028,0002)	Samples per Pixel	1

(0028,0004)	Photometric Interpretation	 MONOCHROME2

(0028,0010)	Rows	256

(0028,0011)	Columns	256

(0028,0030)	Pixel Spacing	 009.765625E-01\\09.765625E-01

(0028,0040)	Private Tag	 RECT

(0028,0100)	Bits Allocated	16

(0028,0101)	Bits Stored	12

(0028,0102)	High Bit	11

(0028,0103)	Pixel Representation	0

(0028,1050)	Window Center	752

(0028,1051)	Window Width	1348

(0029,0011)	Private Tag	 SIEMENS CM VA0  CMS 

(0029,1110)	Private Tag	 STD 1       

(0029,1120)	Private Tag	 NONE      \\NONE       \\NONE         

(0029,1152)	Private Tag	1

(0051,0010)	Private Tag	 SIEMENS CM VA0  CMS 

(0051,1010)	Private Tag	 BU001015\\F 54Y\\H-SP-CR\\IMAGE 12\\\\14-OCT-2005\\13:00\\TA    04:10\\AC        2\\Magnevist 0.1mmol/kg,TPK\\\\MDL RUSEV MRI SOFIA\\VB33G\\ 224 *256os\\se1      90\\SCAN 1\\TR    550.0\\TE   12.0/1\\\\?\\?\\SL      3.0\\SP    -66.8\\Tra>Cor -15\\\\FoV 219*250\\?\\?\\TP        0\\?\\?\\?\\?\\SER 1-3\\\\10-OCT-1951\\*\\\\STU/IMA 3/12\\\\MAGNETOM EXPERT\\BU001015/MN-166-CL-001/V01\\13:00:33 

(7FE0,0010)	Pixel Data	5876

\`\`\`

<div style="page-break-after: always;"></div>

\### The following is the JSON Template for the Attachment 

\`\`\`json

{

  // from Element: extension

  "contentType" : "<code>", // Mime type of the content, with charset etc. 

  "language" : "<code>", // Human language of the content (BCP-47)

  "data" : "<base64Binary>", // Data inline, base64ed

  "url" : "<url>", // Uri where the data can be found

  "size" : "<unsignedInt>", // Number of bytes of content (if url provided)

  "hash" : "<base64Binary>", // Hash of the data (sha-1, base64ed)

  "title" : "<string>", // Label to display in place of the data

  "creation" : "<dateTime>" // Date attachment was first created

}

\`\`\`

\### Cosmos DB Collection

The **Media Resource**, **ImagingStudy**, **Observation**  and **Sequence**  FHIR Resources can be stored in the **Diagnostics Collection** since they contain the \`patient\` field. This field is the Partition Key.

\## Steps  

1\.  Receive Dicom Images.

2\.  Save Dicom Images to CDN storage.

2\.  Read Metadata.

3\.  Covert Metadata to Json.

3\.  Create new **Media** FHIR Resource Document.

4\.  Input Json into **Media** Document fields.

5\.  Input Metadata into \`Attachment\` fields.

6\.  Set \`Media.content.attachment.url\` with CDN storage Uri. 

<div style="page-break-after: always;"></div>

\## Dicom Function App 

\#### The following example Decodes the Tags and saves them as a Cosmos DB Document

\`\`\`csharp

\#region License

// MIT License

// 

// Copyright (c)  2018  Howard Edidin

// 

// Permission is hereby granted, free of charge, to any person obtaining a copy

// of this software and associated documentation files (the "Software"), to deal

// in the Software without restriction, including without limitation the rights

// to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

// copies of the Software, and to permit persons to whom the Software is

// furnished to do so, subject to the following conditions:

// 

// The above copyright notice and this permission notice shall be included in all

// copies or substantial portions of the Software.

// 

// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

// SOFTWARE.

// 

//  

\#endregion

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Newtonsoft.Json;

namespace DicomDecoder

{

	public static class Decode

	{

		private static readonly string AuthKey = ConfigurationManager.AppSettings\["authKey"\];

		private static readonly string Collection = ConfigurationManager.AppSettings\["collection"\];

		private static readonly string Database = ConfigurationManager.AppSettings\["database"\];

		private static readonly string Endpoint = ConfigurationManager.AppSettings\["endpoint"\];

		private static readonly DocumentClient Client = new DocumentClient(new Uri(Endpoint), AuthKey,

			new ConnectionPolicy

			{

				ConnectionMode = ConnectionMode.Direct,

				ConnectionProtocol = Protocol.Tcp

			}

		);

		private static readonly DicomDecoder Decoder = new DicomDecoder();

		

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp		

		\[FunctionName("Decode")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)\] HttpRequestMessage req, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			// Get request body

			dynamic data = await req.Content.ReadAsAsync<object>();

			ResourceResponse<Document> tags = await ParseTags(data);

			return req.CreateResponse(HttpStatusCode.OK, tags);

		}

		

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp	

		private static async Task<ResourceResponse<Document>> ParseTags(dynamic data)

		{

			var dicomInput = data.GetInput<dynamic>();

			string fileName = dicomInput.fileName;

			//string fileUrl = dicomInput.fileUrl;

			List<Tuple<string, string, string, string>> tags = null;

			Decoder.DicomFileName = fileName;

			if (!Decoder.ReadFileInfo()) return null;

			var str = Decoder.dicomInfo;

			foreach (var t in str)

			{

				var s1 = t;

				ExtractStrings(s1, out var s4, out var s5, out var s11, out var s12);

				var tagList = new TagList

				{

					EventTag = s11,

					GroupTag = s12,

					TagDescription = s4,

					TagValue = s5

				};

				new Dictionary<string, string>().Add(tagList.GroupTag + "-" + tagList.EventTag, tagList.TagValue);

				tags.Add(Tuple.Create(tagList.EventTag, tagList.GroupTag, tagList.TagDescription, tagList.TagValue));

			}

			var result = await CreateDocument(tags);

			return result;

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static async Task<ResourceResponse<Document>> CreateDocument(

			List<Tuple<string, string, string, string>> dicomData)

		{

			var id = Guid.NewGuid().ToString();

			//var media = new Media();

			var doc = new Doc {Id = id};

			var tags = new TagList();

			var array = dicomData.ToArray();

			foreach (var item in array)

			{

				tags = new TagList

				{

					EventTag = item.Item1,

					GroupTag = item.Item2,

					TagDescription = item.Item3,

					TagValue = item.Item4

				};

				doc.TagLists.Add(tags);

			}

			doc.TagDictionary.Add("EventTag", tags.EventTag);

			doc.TagDictionary.Add("GroupTag", tags.GroupTag);

			doc.TagDictionary.Add("TagDescription", tags.TagDescription);

			doc.TagDictionary.Add("TagValue", tags.TagValue);

			var dicom = JsonConvert.SerializeObject(doc);

			try

			{

				return await Client.CreateDocumentAsync(UriFactory.CreateDocumentCollectionUri(Database, Collection), dicom);

			}

			catch (DocumentClientException)

			{

				return null;

			}

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp		

		private static void ExtractStrings(string s1, out string s4, out string s5, out string s11, out string s12)

		{

			var ind = s1.IndexOf("//", StringComparison.Ordinal);

			//var s2 = s1.Substring(0, ind);

			s11 = s1.Substring(0, 4);

			s12 = s1.Substring(4, 4);

			var s3 = s1.Substring(ind + 2);

			ind = s3.IndexOf(":", StringComparison.Ordinal);

			s4 = s3.Substring(0, ind);

			s5 = s3.Substring(ind + 1);

		}

	}

	

}

\`\`\`

<div style="page-break-after: always;"></div>

\#### The following example Decodes the Tags and returns them as an Event Grid message.

\`\`\`csharp

\#region License

// MIT License

// 

// Copyright (c)  2018  Howard Edidin

// 

// Permission is hereby granted, free of charge, to any person obtaining a copy

// of this software and associated documentation files (the "Software"), to deal

// in the Software without restriction, including without limitation the rights

// to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

// copies of the Software, and to permit persons to whom the Software is

// furnished to do so, subject to the following conditions:

// 

// The above copyright notice and this permission notice shall be included in all

// copies or substantial portions of the Software.

// 

// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

// SOFTWARE.

// 

//  

\#endregion

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Net;

using System.Net.Http;

using System.Threading.Tasks;

using Microsoft.Azure.Documents;

using Microsoft.Azure.Documents.Client;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

namespace DicomDecoder

{

	public static class DecodeToEventGrid

	{

		private static readonly string Topic = ConfigurationManager.AppSettings\["eventGridTopicEndpoint"\];

		private static readonly string Key = ConfigurationManager.AppSettings\["eventGridTopicKey"\];

		private static readonly string EventType = ConfigurationManager.AppSettings\["eventType"\];

		private static readonly string Subject = ConfigurationManager.AppSettings\["subject"\];

		private static readonly HttpClient httpClient;

		private static readonly DicomDecoder Decoder = new DicomDecoder();

		\[FunctionName("DecodeToEventGrid")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)\] HttpRequestMessage req, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			// Get request body

			dynamic data = await req.Content.ReadAsAsync<object>();

			ResourceResponse<Document> tags = await ParseTags(data);

			return req.CreateResponse(HttpStatusCode.OK, tags);

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static async Task<bool> ParseTags(dynamic data)

		{

			var dicomInput = data.GetInput<dynamic>();

			string fileName = dicomInput.fileName;

			string fileUrl = dicomInput.fileUrl;

			List<Tuple<string, string, string, string>> tags = null;

			Decoder.DicomFileName = fileName;

			Decoder.fileUrl = fileUrl;

			if (!Decoder.ReadFileInfo()) return false;

			var str = Decoder.dicomInfo;

			foreach (var t in str)

			{

				var s1 = t;

				ExtractStrings(s1, out var s4, out var s5, out var s11, out var s12);

				var tagList = new TagList

				{

					EventTag = s11,

					GroupTag = s12,

					TagDescription = s4,

					TagValue = s5

				};

				new Dictionary<string, string>().Add(tagList.GroupTag + "-" + tagList.EventTag, tagList.TagValue);

				tags.Add(Tuple.Create(tagList.EventTag, tagList.GroupTag, tagList.TagDescription, tagList.TagValue));

			}

			var result = await CreateEvent(tags);

			return result;

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static async Task<bool> CreateEvent(

			List<Tuple<string, string, string, string>> dicomData)

		{

			var id = Guid.NewGuid().ToString();

			//var media = new Media();

			var doc = new Doc {Id = id};

			var tags = new TagList();

			var array = dicomData.ToArray();

			foreach (var item in array)

			{

				tags = new TagList

				{

					EventTag = item.Item1,

					GroupTag = item.Item2,

					TagDescription = item.Item3,

					TagValue = item.Item4

				};

				doc.TagLists.Add(tags);

			}

			doc.TagDictionary.Add("EventTag", tags.EventTag);

			doc.TagDictionary.Add("GroupTag", tags.GroupTag);

			doc.TagDictionary.Add("TagDescription", tags.TagDescription);

			doc.TagDictionary.Add("TagValue", tags.TagValue);

			var events = new List<Event<Doc>>

			{

				new Event<Doc>

				{

					Data = doc,

					EventTime = DateTime.UtcNow,

					EventType = EventType,

					Id = Guid.NewGuid().ToString(),

					Subject = Subject

				}

			};

			httpClient.DefaultRequestHeaders.Clear();

			httpClient.DefaultRequestHeaders.Add("aeg-sas-key", Key);

			await httpClient.PostAsJsonAsync(Topic, events);

			return true;

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static void ExtractStrings(string s1, out string s4, out string s5, out string s11, out string s12)

		{

			var ind = s1.IndexOf("//", StringComparison.Ordinal);

			//var s2 = s1.Substring(0, ind);

			s11 = s1.Substring(0, 4);

			s12 = s1.Substring(4, 4);

			var s3 = s1.Substring(ind + 2);

			ind = s3.IndexOf(":", StringComparison.Ordinal);

			s4 = s3.Substring(0, ind);

			s5 = s3.Substring(ind + 1);

		}

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

ImagingStudy is used for DICOM imaging and associated information. Use Media to track non-DICOM images, video, or audio. Binary can be used to store arbitrary content. DocumentReference allow indexing and retrieval of clinical “documents” with relevant metadata.

\## Mapping Tags to FHIR ImagingStudy

\`\`\`txt

10\.4.7.5 Mappings for DICOM Tag Mapping ([http://nema.org/dicom](http://nema.org/dicom "http://nema.org/dicom")) 

ImagingStudy	Reference IHE radiology TF vol 2 table 4.14-1

    identifier	(0020,0010)+(0020,000D)+(0008,0050)+(0008,0050)

    status	

    modality	(0008,0061)

    subject	(0010/*)

    context	

    started	(0008,0020)+(0008,0030)

    basedOn	(0032,1064)

    referrer	(0008,0090)+(0008,0096)

    interpreter	(0008,1060)

    endpoint	

    numberOfSeries	(0020,1206)

    numberOfInstances	(0020,1208)

    procedureReference	(0008,1032)

    procedureCode	(0008,0032)

    location	(0008,1040)+(0040,0243)

    reasonCode	

    reasonReference	

    note	(0008,1030)+(0040,0280)

    series	

        identifier	(0020,000E)

        number	(0020,0011)

        modality	(0008,0060)

        description	(0008,103E)

        numberOfInstances	(0020,1209)

        endpoint	

        bodySite	(0018,0015)

        laterality	(0020,0060)

        specimen	(0040,0551) + (0040,0562)

        started	(0008,0021) + (0008,0031)

        performer	(0008,1050) | (0008,1072)

            function	

            actor	

        instance	

            identifier	(0008,0018)

            number	(0020,0013)

            sopClass	(0008,0016)

            title	(0070,0080) | (0040,A043) > (0008,0104) | (0042,0010) | (0008,0008)

\`\`\`

<div style="page-break-after: always;"></div>

\# CHAPTER 15 - RECEIVING DEVICE MESSAGES

We need to be able to receive messages from our Devices. We will be using an Azure Function App

\## Steps

1\. Receive an \`DeviceId\` from a \`Get\` Request.

2\. Register the device.

    1. If not exists, then create primary \`Key\`

    2. If exists, then get primary \`Key\`

3\. Receive messages from Device.

4\. Return the message.

\## ReceiveDeviceMessages Function App

\### Source Code

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Configuration;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Text;

using System.Threading;

using System.Threading.Tasks;

using Microsoft.Azure.Devices;

using Microsoft.Azure.Devices.Common.Exceptions;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.ServiceBus.Messaging;

\#endregion

namespace AzureIoTHub

{

	public static class ReceiveDeviceMessages

	{

		private static readonly string IotHubConnectionString = ConfigurationManager.AppSettings\["iotHubConnectionString"\];

		private static readonly string TimeSpanSeconds = ConfigurationManager.AppSettings\["iotHubConnectionString"\];

		private static readonly string IotHubD2CEndpoint = ConfigurationManager.AppSettings\["iotHubD2CEndpoint"\];

		\[FunctionName("ReceiveDeviceMessages")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)\]

			HttpRequestMessage req, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			if (string.IsNullOrEmpty(IotHubConnectionString) || string.IsNullOrEmpty(TimeSpanSeconds) ||

			    string.IsNullOrEmpty(IotHubD2CEndpoint))

				return req.CreateResponse(HttpStatusCode.BadRequest, "Configuration Settings are missing or invalid");

			var deviceId = req.GetQueryNameValuePairs()

				.FirstOrDefault(q => string.Compare(q.Key, "deviceId", StringComparison.OrdinalIgnoreCase) == 0)

				.Value;

			if (string.IsNullOrEmpty(deviceId))

				return req.CreateResponse(HttpStatusCode.BadRequest, "Please pass the deviceId on the query string");

			var key = await CreateDeviceIdentityAsync(deviceId);

			log.Info("Device Key: ", key);

			var results = await ReceiveMessagesFromDeviceAsync(CancellationToken.None);

		

			return req.CreateResponse(HttpStatusCode.OK, results);

		}

		public static async Task<string> CreateDeviceIdentityAsync(string deviceName)

		{

			var registryManager = RegistryManager.CreateFromConnectionString(IotHubConnectionString);

			Device device;

			try

			{

				device = await registryManager.AddDeviceAsync(new Device(deviceName));

			}

			catch (DeviceAlreadyExistsException)

			{

				device = await registryManager.GetDeviceAsync(deviceName);

			}

			return device.Authentication.SymmetricKey.PrimaryKey;

		}

		public static async Task<string\[\]> ReceiveMessagesFromDeviceAsync(CancellationToken cancelToken)

		{

			var eventHubClient = EventHubClient.CreateFromConnectionString(IotHubConnectionString, IotHubD2CEndpoint);

			var d2CPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;

			var result = await Task.WhenAll(d2CPartitions.Select(partition =>

				ReceiveMessagesFromDeviceAsync(eventHubClient, partition, cancelToken)));

			return result;

		}

		private static async Task<string> ReceiveMessagesFromDeviceAsync(EventHubClient eventHubClient, string partition,

			CancellationToken ct)

		{

			var data = string.Empty;

			var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);

			while (true)

			{

				if (ct.IsCancellationRequested)

					break;

				var eventData = await eventHubReceiver.ReceiveAsync(TimeSpan.FromSeconds(double.Parse(TimeSpanSeconds)));

				if (eventData == null) continue;

				data = Encoding.UTF8.GetString(eventData.GetBytes());

			}

			return data;

		}

	}

}

\`\`\`

<div style="page-break-after: always;"></div>

\## Summary

\* We have seen how easy it is to get the Device **Primary Key** 

\* By using an Azure Function App with a **HTTP Trigger**, we are able to call it from another service or a Logic App.

<div style="page-break-after: always;"></div>

\# CHAPTER 16 - TIME SERIES INSIGHTS

Time Series Insights is built for storing, visualizing, and querying large amounts of time series data, such as that generated by IoT devices.

\## Time Series Insights has four key jobs:

\* First, it's fully integrated with cloud gateways like Azure IoT Hub and Azure Event Hubs. It easily connects to these event sources and parses JSON from messages and structures that have data in clean rows and columns. It joins metadata with telemetry and indexes your data in a columnar store.

\* Second, Time Series Insights manages the storage of your data. To ensure data is always easily accessible, it stores your data in memory and SSD’s for up to 400 days. You can interactively query billions of events in seconds – on demand.

\* Third, Time Series Insights provides out-of-the-box visualization via the TSI explorer.

\* Fourth, Time Series Insights provides a query service, both in the TSI explorer and by using APIs that are easy to integrate for embedding your time series data into custom applications.

\## TimeSeriesInsightsQuery Function App

Our \`TimeSeriesInsightsQuery\` Function App accepts a \`GET\` HTTP Operation and returns the data as formated JSON. We are quering for Aggregates.

\### Steps

1\. Use a \`StringBuilder\` to store the data.

2\. Pass in the \`applicationClientId\` to our **Query** operaton.

3\. Obtain list of environments and get environment FQDN for the environment of interest. 

4\. Obtain availability data for the environment and get availability range.

5\. Assume data for the whole availablility range is requested.

6\. Get events for the environment.

7\. Get aggregates for the environment: group by Event Source Name and calculate number of events in each group.

8\. Return \`StringBuilder.toString()\`.

\### Source code 

\`\`\`csharp

\#region Information

//  

//  MIT License

//  

//  Copyright (c)  2018  Howard Edidin

//  

//  Permission is hereby granted, free of charge, to any person obtaining a copy

//  of this software and associated documentation files (the "Software"), to deal

//  in the Software without restriction, including without limitation the rights

//  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell

//  copies of the Software, and to permit persons to whom the Software is

//  furnished to do so, subject to the following conditions:

//  

//  The above copyright notice and this permission notice shall be included in all

//  copies or substantial portions of the Software.

//  

//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR

//  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,

//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE

//  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER

//  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,

//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE

//  SOFTWARE.

\#endregion

\#region

using System;

using System.Collections.Generic;

using System.Configuration;

using System.Diagnostics;

using System.IO;

using System.Linq;

using System.Net;

using System.Net.Http;

using System.Net.WebSockets;

using System.Text;

using System.Threading;

using System.Threading.Tasks;

using Microsoft.Azure.WebJobs;

using Microsoft.Azure.WebJobs.Extensions.Http;

using Microsoft.Azure.WebJobs.Host;

using Microsoft.IdentityModel.Clients.ActiveDirectory;

using Newtonsoft.Json;

using Newtonsoft.Json.Linq;

\#endregion

namespace TimeSeiesInsights

{

	public static class TimeSeriesInsightsQuery

	{

		private static readonly string ApplicationClientSecret = ConfigurationManager.AppSettings\["applicationClientSecret"\];

		//private static readonly string Tenant = ConfigurationManager.AppSettings\["tenant"\];

		public static StringBuilder Dump = new StringBuilder();

		\[FunctionName("TimeSeriesInsightsQuery")\]

		public static async Task<HttpResponseMessage> Run(

			\[HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)\]

			HttpRequestMessage req, TraceWriter log)

		{

			log.Info("C# HTTP trigger function processed a request.");

			if (string.IsNullOrEmpty(ApplicationClientSecret))

				return req.CreateResponse(HttpStatusCode.BadRequest,

					"ApplicationClientSecret is invalid or missing ");

			var applicationClientId = req.GetQueryNameValuePairs()

				.FirstOrDefault(q => string.Compare(q.Key, "ApplicationClientId", StringComparison.OrdinalIgnoreCase) == 0)

				.Value;

			await Query(applicationClientId);

			var result = Dump.ToString();

			return req.CreateResponse(HttpStatusCode.OK, result);

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		public static async Task Query(string applicationClientId)

		{

			// Acquire an access token.

			var accessToken = await AcquireAccessTokenAsync(applicationClientId);

			// Obtain list of environments and get environment FQDN for the environment of interest.

			string environmentFqdn;

			{

				var request = CreateHttpsWebRequest("api.timeseries.azure.com", "GET", "environments", accessToken);

				var responseContent = await GetResponseAsync(request);

				var environmentsList = (JArray) responseContent\["environments"\];

				if (environmentsList.Count == 0)

				{

					// List of user environments is empty, fallback to sample environment.

					environmentFqdn = "10000000-0000-0000-0000-100000000108.env.timeseries.azure.com";

				}

				else

				{

					// Assume the first environment is the environment of interest.

					var firstEnvironment = (JObject) environmentsList\[0\];

					environmentFqdn = firstEnvironment\["environmentFqdn"\].Value<string>();

				}

			}

			// Obtain availability data for the environment and get availability range.

			DateTime fromAvailabilityTimestamp;

			DateTime toAvailabilityTimestamp;

			{

				var request = CreateHttpsWebRequest(environmentFqdn, "GET", "availability", accessToken);

				var responseContent = await GetResponseAsync(request);

				var range = (JObject) responseContent\["range"\];

				fromAvailabilityTimestamp = range\["from"\].Value<DateTime>();

				toAvailabilityTimestamp = range\["to"\].Value<DateTime>();

			}

			// Assume data for the whole availablility range is requested.

			var from = fromAvailabilityTimestamp;

			var to = toAvailabilityTimestamp;

			// Obtain metadata for the environment.

			{

				var inputPayload = new JObject(

					new JProperty("searchSpan", new JObject(

						new JProperty("from", from),

						new JProperty("to", to))));

				var request = CreateHttpsWebRequest(environmentFqdn, "POST", "metadata", accessToken);

				await WriteRequestStreamAsync(request, inputPayload);

				var responseContent = await GetResponseAsync(request);

				DumpMetadata(responseContent);

			}

			// Get events for the environment.

			{

				const int requestedEventCount = 10;

				var contentInputPayload = new JObject(

					new JProperty("take", requestedEventCount),

					new JProperty("searchSpan", new JObject(

						new JProperty("from", from),

						new JProperty("to", to))));

				// Use HTTP request.

				var request = CreateHttpsWebRequest(environmentFqdn, "POST", "events", accessToken, new\[\] {"timeout=PT20S"});

				await WriteRequestStreamAsync(request, contentInputPayload);

				var responseContent = await GetResponseAsync(request);

				DumpEvents(responseContent);

				// Use WebSocket request.

				var inputPayload = new JObject(

					// Send HTTP headers as a part of the message since .NET WebSocket does not support

					// sending custom headers on HTTP GET upgrade request to WebSocket protocol request.

					new JProperty("headers", new JObject(

						new JProperty("x-ms-client-application-name", "TimeSeriesInsightsQuerySample"),

						new JProperty("Authorization", "Bearer " + accessToken))),

					new JProperty("content", contentInputPayload));

				var responseMessagesContent = await ReadWebSocketResponseAsync(environmentFqdn, "events", inputPayload);

				DumpEventsStreamed(responseMessagesContent);

			}

			// Get aggregates for the environment: group by Event Source Name and calculate number of events in each group.

			{

				var contentInputPayload = new JObject(

					new JProperty("aggregates", new JArray(new JObject(

						new JProperty("dimension", new JObject(

							new JProperty("uniqueValues", new JObject(

								new JProperty("input", new JObject(

									new JProperty("builtInProperty", "$esn"))),

								new JProperty("take", 1000))))),

						new JProperty("measures", new JArray(new JObject(

							new JProperty("count", new JObject()))))))),

					new JProperty("searchSpan", new JObject(

						new JProperty("from", from),

						new JProperty("to", to))));

				// Use HTTP request.

				var request = CreateHttpsWebRequest(environmentFqdn, "POST", "aggregates", accessToken, new\[\] {"timeout=PT20S"});

				await WriteRequestStreamAsync(request, contentInputPayload);

				var responseContent = await GetResponseAsync(request);

				DumpAggregates(responseContent);

				// Use WebSocket request.

				var inputPayload = new JObject(

					// Send HTTP headers as a part of the message since .NET WebSocket does not support

					// sending custom headers on HTTP GET upgrade request to WebSocket protocol request.

					new JProperty("headers", new JObject(

						new JProperty("x-ms-client-application-name", "TimeSeriesInsightsQuery"),

						new JProperty("Authorization", "Bearer " + accessToken))),

					new JProperty("content", contentInputPayload));

				var responseMessagesContent = await ReadWebSocketResponseAsync(environmentFqdn, "aggregates", inputPayload);

				DumpAggregatesStreamed(responseMessagesContent);

			}

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static async Task<string> AcquireAccessTokenAsync(string applicationClientId)

		{

			//var authenticationContext = new AuthenticationContext(authString, false);

			var authenticationContext = new AuthenticationContext("[https://login.microsoftonline.com/common](https://login.microsoftonline.com/common "https://login.microsoftonline.com/common")", TokenCache.DefaultShared);

			var clientCred = new ClientCredential(applicationClientId, ApplicationClientSecret);

			var authenticationResult =

				await authenticationContext.AcquireTokenAsync("[https://api.timeseries.azure.com/](https://api.timeseries.azure.com/ "https://api.timeseries.azure.com/")", clientCred);

			var token = authenticationResult.AccessToken;

			return token;

		}

		private static HttpWebRequest CreateHttpsWebRequest(string host, string method, string path, string accessToken,

			string\[\] queryArgs = null)

		{

			var query = "api-version=2016-12-12";

			if (queryArgs != null && queryArgs.Any()) query += "&" + string.Join("&", queryArgs);

			var uri = new UriBuilder("https", host)

			{

				Path = path,

				Query = query

			}.Uri;

			var request = WebRequest.CreateHttp(uri);

			request.Method = method;

			request.Headers.Add("x-ms-client-application-name", "TimeSeriesInsightsQuerySample");

			request.Headers.Add("Authorization", "Bearer " + accessToken);

			return request;

		}

		private static async Task WriteRequestStreamAsync(HttpWebRequest request, JObject inputPayload)

		{

			using (var stream = await request.GetRequestStreamAsync())

			using (var streamWriter = new StreamWriter(stream))

			{

				await streamWriter.WriteAsync(inputPayload.ToString());

				await streamWriter.FlushAsync();

				streamWriter.Close();

			}

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static async Task<JToken> GetResponseAsync(WebRequest request)

		{

			using (var webResponse = await request.GetResponseAsync())

			using (var sr = new StreamReader(webResponse.GetResponseStream()))

			{

				var result = await sr.ReadToEndAsync();

				return JsonConvert.DeserializeObject<JToken>(result);

			}

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static async Task<IReadOnlyList<JToken>> ReadWebSocketResponseAsync(string environmentFqdn, string path,

			JObject inputPayload)

		{

			var webSocket = new ClientWebSocket();

			// Establish web socket connection.

			var uri = new UriBuilder("wss", environmentFqdn)

			{

				Path = path,

				Query = "api-version=2016-12-12"

			}.Uri;

			await webSocket.ConnectAsync(uri, CancellationToken.None);

			// Send input payload.

			var inputPayloadBytes = Encoding.UTF8.GetBytes(inputPayload.ToString());

			await webSocket.SendAsync(

				new ArraySegment<byte>(inputPayloadBytes),

				WebSocketMessageType.Text,

				true,

				CancellationToken.None);

			// Read response messages from web socket.

			var responseMessagesContent = new List<JToken>();

			using (webSocket)

			{

				while (true)

				{

					string message;

					using (var ms = new MemoryStream())

					{

						// Write from socket to memory stream.

						const int bufferSize = 16 * 1024;

						var temporaryBuffer = new byte\[bufferSize\];

						while (true)

						{

							var response = await webSocket.ReceiveAsync(

								new ArraySegment<byte>(temporaryBuffer),

								CancellationToken.None);

							ms.Write(temporaryBuffer, 0, response.Count);

							if (response.EndOfMessage) break;

						}

						// Reset position to the beginning to allow reads.

						ms.Position = 0;

						using (var sr = new StreamReader(ms))

						{

							message = sr.ReadToEnd();

						}

					}

					var messageObj = JsonConvert.DeserializeObject<JObject>(message);

					// Stop reading if error is emitted.

					if (messageObj\["error"\] != null) break;

					// Actual response contents is wrapped into "content" object.

					responseMessagesContent.Add(messageObj\["content"\]);

					// Stop reading if 100% of completeness is reached.

					if (messageObj\["percentCompleted"\] != null &&

					    Math.Abs((double) messageObj\["percentCompleted"\] - 100d) < 0.01)

						break;

				}

				// Close web socket connection.

				if (webSocket.State == WebSocketState.Open)

					await webSocket.CloseAsync(

						WebSocketCloseStatus.NormalClosure,

						"CompletedByClient",

						CancellationToken.None);

			}

			return responseMessagesContent;

		}

		

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static void DumpMetadata(JToken responseContent)

		{

			// Response content has a list of properties under "properties" property.

			var properties = (JArray) responseContent\["properties"\];

			var propertiesCount = properties.Count;

			Dump.Append(@"\\n\\t\\Metadata:  ");

			for (var i = 0; i < propertiesCount; i++)

			{

				var currentProperty = (JObject) properties\[i\];

				Dump.Append("\\n\\t\\tCurrentPropertyType: ").Append(currentProperty.Type);

				foreach (var item in currentProperty)

				{

					Dump.Append("\\n\\t\\t\\tCurrentEvent:Key ").Append(item.Key);

					Dump.Append("\\n\\t\\t\\tCurrentEvent:Value ").Append(item.Value);

				}

			}

		}

		private static void DumpEvents(JToken responseContent)

		{

			// Response content has a list of events under "events" property for HTTP request.

			var events = (JArray) responseContent\["events"\];

			Dump.Append("\\n\\tEvents:  ");

			var eventCount = events.Count;

			for (var i = 0; i < eventCount; i++)

			{

				var currentEvent = (JObject) events\[i\];

				var values = (JArray) currentEvent\["values"\];

				foreach (var item in values) Dump.Append("\\n\\t\\tCurrentEvent: ").Append(item);

			}

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static void DumpEventsStreamed(IEnumerable<JToken> responseMessagesContent)

		{

			// For events stream next message always contains additional events, i.e. new message is additive to the previous one.

			// New message contains new events that were not in the previous message.

			// The previous message should be kept and accumulated with the new message.

			// Response content has a list of events under "events" property both for HTTP and WebSocket requests.

			var events = new JArray(responseMessagesContent.SelectMany(m => (JArray) m\["events"\]));

			Dump.Append("\\n\\tEventsStreamed:  ");

			var eventCount = events.Count;

			for (var i = 0; i < eventCount; i++)

			{

				var currentEvent = (JObject) events\[i\];

				var values = (JArray) currentEvent\["values"\];

				foreach (var item in values) Dump.Append("\\n\\t\\tCurrentEventStreamed: ").Append(item);

			}

		}

		private static void DumpAggregates(JToken responseContent)

		{

			// HTTP response content contains the list of aggregates under "aggregates" property.

			var aggregates = (JArray) responseContent\["aggregates"\];

			Dump.Append("\\n\\t\\tAggregates ").Append(aggregates.GetEnumerator());

			DumpAggregatesInternal(aggregates);

		}

		private static void DumpAggregatesStreamed(IEnumerable<JToken> responseMessagesContent)

		{

			// For aggregates stream next message always contains a replacement (snapshot) of all the values.

			// Previous message can be discarded by the client.

			// WebSocket response represents the list of aggregates.

			var aggregates = (JArray) responseMessagesContent.Last();

			Dump.Append("\\n\\t\\tAggregatesStreamed ").Append(aggregates.GetEnumerator());

			DumpAggregatesInternal(aggregates);

		}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

		private static void DumpAggregatesInternal(JArray aggregates)

		{

			// Number of items corresponds to number of aggregates in input payload.

			// In this sample list of aggregates in input payload contains only 1 item since request contains 1 aggregate.

			var aggregate = (JObject) aggregates\[0\];

			var dimensionValues = (JArray) aggregate\["dimension"\];

			var measures = (JArray) aggregate\["measures"\];

			Dump.Append("\\n\\tAggregates:  ");

			Debug.Assert(

				dimensionValues.Count == measures.Count,

				"Number of measures and dimensions should match.");

			for (var i = 0; i < dimensionValues.Count; i++)

			{

				var currentDimensionValue = (string) dimensionValues\[i\];

				Dump.Append("\\n\\t\\tAggregates:DimensionValue: ").Append(currentDimensionValue);

				var currentMeasureValues = (JArray) measures\[i\];

				Dump.Append("\\n\\t\\tAggregates:MeasureValue: ").Append(currentDimensionValue);

				var currentCount = (double) currentMeasureValues\[0\];

				Dump.Append("\\n\\t\\tAggregates:DimensonCount: ").Append(currentCount);

			}

		}

	}

	

}

\`\`\`

<div style="page-break-after: always;"></div>

\## Summary

\* We can query **Time Series Insights** from a Function App.

\* The data can be returned as formatted JSON and consumed by any application or service.

<div style="page-break-after: always;"></div>

\# CHAPTER 17 - PROCESSING QUEUE MESSAGES

\**Azure Queue storage** is a service for storing large numbers of messages that can be accessed from anywhere in the world via authenticated calls using HTTP or HTTPS. A single queue message can be up to 64 KB in size, and a queue can contain millions of messages, up to the total capacity limit of a storage account.

\**There are times when our Event Processing Services, are pushing messages faster than the services receiving them can handle them.**

\## Insert a message into a queue

The following Function App example shows how easy it is to add a message to the queue.

\`\`\`csharp

\[StorageAccount("AzureWebJobsStorage")\]

   public static class QueueFunctions

   {

       \[FunctionName("QueueOutput")\]

       \[return: Queue("device-items")\]

       public static string QueueOutput(\[HttpTrigger\] dynamic input,  TraceWriter log)

       {

       

           

           log.Info($"C# function processed: {input.Text}");

           return input.Text;

       }

   }

\`\`\`

<div style="page-break-after: always;"></div>

\## De-queue the messages

De-queue the message and insert it into our HL7 FHIR Repository using a Function App

\`\`\`csharp

\[FunctionName("QueueToCosmosDB")\]        

      public static void Run(

          \[QueueTrigger("device-items", Connection = "AzureWebJobsStorage")\] string deviceItem,

          \[DocumentDB("admin", "Devices", Id = "id", ConnectionStringSetting = "myCosmosDB")\] out dynamic document)

      {

          document = new { Text = deviceItem, id = Guid.NewGuid() };

      }

\`\`\`

\## We are also able to read the message Metadata 

The following Function App example shows how easy it is to read the message Metadata

\`\`\`csharp

\[FunctionName("QueueTriggerMetadata")\]

     public static void RunMetadata(\[QueueTrigger("device-items", Connection = "AzureWebJobsStorage")\]CloudQueueMessage myQueueItem, TraceWriter log)

     {

         log.Info("Retrieving Queue metadata");

         log.Info($"Queue ID: {myQueueItem.Id}");

         log.Info($"Queue Insertion Time: {myQueueItem.InsertionTime}");

         log.Info($"Queue Expiration Time: {myQueueItem.ExpirationTime}");

         log.Info($"Queue Payload: {myQueueItem.AsString}");

         

     }

\`\`\`

<div style="page-break-after: always;"></div>

\# CHAPTER 18 - IMPLEMENTING A CUSTOM APPLICATION GATEWAY

When you need to implement an application gateway at the server level and not at the Edge or the gateway may not support IoT Edge due to processing power or memory constraints or due to the lack of container support.

\## Use Case

One of these cases could be a migration scenario where you are unable to change the code running on the devices. In this case you need to implement an intermediary gateway that the devices can connect to without knowing about IoT hub.

!\[\](![](https://i.imgur.com/3AGQGlE.png))

<div style="page-break-after: always;"></div>

Another use for the server gateway for example is to act as a LoRaWAN to IoT Hub connector. The LoRaWAN network server could include this gateway code to forward all the messages from the connected LoRa devices to IoT Hub.

!\[\](![](https://i.imgur.com/M0hmbYL.png))

\-------------------

> !\[\](![](https://i.imgur.com/eSPsKWl.png))   

> The **LoRaWAN™** specification is a Low Power Wide Area (LPWA) network protocol designed to wirelessly connect battery operated ‘things’ to the internet in regional, national or global networks. The protocol includes features that support low-cost, mobile, and secure bi-directional communication for Internet of Things (IoT), machine-to-machine (M2M), smart city & industrial applications. **LoRaWAN protocol is optimized for low power consumption** and is designed to scale from a single gateway installation up to large global networks with billions of devices. Innovative features of the LoRaWAN specification include support for redundant operation, geolocation, low-cost, and low-power: Devices can even run on energy harvesting technologies enabling the mobility and bringing true ease of use to the Internet of Things.

\## Solution Approach

In our example, we created an ASP.NET Core solution to serve as our gateway.

It is crucial that a gateway solution should be able to multiplex device connections instead of creating single connections to IoT Hub per device. IoT Hub supports multiplexing only over \`HTTP\` and \`AMQP\`. For our sample gateway, we decided to use \`AMQP\` for its efficiency. The Azure IoT Hub SDK for .NET already implements support for connection pooling. 

\### Authentication using Tokens

Using the authentication method \`DeviceAuthenticationWithToken\` allows every device to authenticate to IoT hub directly, while still retaining the ability to share the connection among devices with pooling. In this case, the device has the SAS Key and generates a temporary token that it sends through the gateway to IoT Hub. IoT Hub is authenticating the device and not the gateway itself, effectively making it an end-to-end authentication. The time-to-live set for the caching needs to be the same as the one for the SAS token

<div style="page-break-after: always;"></div>

\## Flow

When a device first wants to send a message through the gateway, in our example it calls a REST API, but this could be achieved using any accessible endpoint or protocol that the gateway supports. At this point, the gateway creates a new DeviceClient in order to connect to IoT Hub using the connection pool settings "\`Pooling = true\`" in \`AmqpConnectionPoolSettings\`.

\`\`\`csharp

using System;

using System.Collections.Generic;

using System.IO;

using System.Linq;

using System.Threading.Tasks;

using Microsoft.AspNetCore;

using Microsoft.AspNetCore.Hosting;

using Microsoft.Extensions.Configuration;

using Microsoft.Extensions.Logging;

namespace IoTHubGateway.Server

{

    public class Program

    {

        public static void Main(string\[\] args)

        {

            BuildWebHost(args).Run();

        }

        public static IWebHost BuildWebHost(string\[\] args) =>

            WebHost.CreateDefaultBuilder(args)                

                .UseStartup<Startup>()            

                .Build();

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.Threading.Tasks;

using IoTHubGateway.Server.Services;

using Microsoft.AspNetCore.Builder;

using Microsoft.AspNetCore.Hosting;

using Microsoft.Azure.Devices.Client;

using Microsoft.Extensions.Configuration;

using Microsoft.Extensions.DependencyInjection;

using Microsoft.Extensions.Hosting;

using Microsoft.Extensions.Logging;

using Microsoft.Extensions.Options;

namespace IoTHubGateway.Server

{

    public class Startup

    {

        public Startup(IConfiguration configuration)

        {

            Configuration = configuration;

        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.

        public void ConfigureServices(IServiceCollection services)

        {

            services.AddOptions();

            services.AddMemoryCache();

            // A single instance of registered devices must be kept

            services.AddSingleton<RegisteredDevices>();

            // Resolve server options

            var options = new ServerOptions();

            Configuration.GetSection(nameof(ServerOptions)).Bind(options);

            services.AddSingleton<ServerOptions>(options);

            if (options.CloudMessagesEnabled)

            {

                services.AddSingleton<IHostedService, CloudToMessageListenerJobHostedService>();

            }

\#if DEBUG

            SetupDebugListeners(options);

            

\#endif

            services.AddSingleton<IGatewayService, GatewayService>();

            services.AddMvc();

        }

\#if DEBUG

        private void SetupDebugListeners(ServerOptions options)

        {

            if (options.DirectMethodEnabled && options.DirectMethodCallback == null)

            {

                options.DirectMethodCallback = (methodRequest, userContext) =>

                {

                    var deviceId = (string)userContext;

                    Console.WriteLine($"\[{DateTime.Now.ToString()}\] Device method for {deviceId}.{methodRequest.Name}({methodRequest.DataAsJson}) received");

                    var responseBody = "{ succeeded: true }";

                    MethodResponse methodResponse = new MethodResponse(Encoding.UTF8.GetBytes(responseBody), 200);

                    return Task.FromResult(methodResponse);

                };

            }

            if (options.CloudMessagesEnabled && options.CloudMessageCallback == null)

            {

                options.CloudMessageCallback = (deviceId, message) =>

                {

                    Console.WriteLine($"\[{DateTime.Now.ToString()}\] Cloud message for {deviceId} received");

                };

            }

        }

\#endif

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.

        public void Configure(IApplicationBuilder app, IHostingEnvironment env)

        {

            if (env.IsDevelopment())

            {

                app.UseDeveloperExceptionPage();

            }            

            

            app.UseMvc();

        }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

ausing Microsoft.Azure.Devices.Client;

using System;

using System.Collections.Generic;

using System.Linq;

using System.Threading.Tasks;

namespace IoTHubGateway.Server

{

    /// <summary>

    /// Defines the server options

    /// </summary>

    public class ServerOptions

    {

        /// <summary>

        /// Device operation timeout (in milliseconds)

        /// Default: 10000 (10 seconds)

        /// </summary>

        public int DeviceOperationTimeout { get; set; } =  1000 * 10;

        /// <summary>

        /// IoT Hub host name. Something like xxxxx.azure-devices.net

        /// </summary>

        public string IoTHubHostName { get; set; }

        /// <summary>

        /// The IoT Hub access policy name. A common value is iothubowner

        /// </summary>

        public string AccessPolicyName { get; set; }

        /// <summary>

        /// The IoT Hub access policy key. Get it from Azure Portal

        /// </summary>

        public string AccessPolicyKey { get; set; }

        /// <summary>

        /// The maximum pool size (default is <seealso cref="ushort.MaxValue"/>)

        /// </summary>

        public int MaxPoolSize { get; set; } = ushort.MaxValue;

        /// <summary>

        /// Allows or not the usage of shared access policy keys

        /// If you enable it make sure that access to the API is protected otherwise anyone will be able to impersonate devices

        /// </summary>

        public bool SharedAccessPolicyKeyEnabled { get; set; }

        /// <summary>

        /// Default device client cache in duration in minutes

        /// 60 minutes by default

        /// </summary>

        public int DefaultDeviceCacheInMinutes = 60;

        /// <summary>

        /// Enable/disables direct method (cloud -> device)

        /// </summary>

        public bool DirectMethodEnabled { get; set; } = false;

        /// <summary>

        /// Gets/sets the callback to handle device direct methods

        /// </summary>

        public MethodCallback DirectMethodCallback { get; set; }        

        /// <summary>

        /// Enable/disables cloud messages in the gateway

        /// Cloud messages are retrieved in a background job

        /// Default: false / disabled

        /// </summary>

        public bool CloudMessagesEnabled { get; set; }

        /// <summary>

        /// Degree of parallelism used to check for cloud messages

        /// </summary>

        public int CloudMessageParallelism { get; set; } = 10;

        /// <summary>

        /// Gets/sets the callback to handle cloud messages

        /// </summary>

        public Action<string, Message> CloudMessageCallback { get; set; }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

using System;

using System.Collections.Generic;

using System.Linq;

using System.Threading.Tasks;

namespace IoTHubGateway.Server

{

    public static class Constants

    {

        /// <summary>

        /// Name of request header containing the device sas token

        /// </summary>

        public const string SasTokenHeaderName = "sas_token";

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Services

\`\`\`csharp

using System;

using System.Collections.Generic;

using System.Linq;

using System.Threading.Tasks;

namespace IoTHubGateway.Server.Services

{

    /// <summary>

    /// Maintains a list of registered devices to enabled cloud messages background job to query them

    /// </summary>

    public class RegisteredDevices

    {

        System.Collections.Concurrent.ConcurrentDictionary<string, string> devices = new System.Collections.Concurrent.ConcurrentDictionary<string, string>();

        public void AddDevice(string deviceId)

        {

            devices.AddOrUpdate(deviceId, deviceId, (key, existing) =>

            {

                return deviceId;

            });

        }

        public void RemoveDevice(object key)

        {

            devices.Remove(key.ToString(), out var value);

        }

        public ICollection<string> GetDeviceIdList()

        {

            return this.devices.Keys;

        }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

using System;

using System.Collections.Generic;

using System.Linq;

using System.Threading.Tasks;

namespace IoTHubGateway.Server.Services

{

    /// <summary>

    /// Gateway to IoT Hub service

    /// </summary>

    public interface IGatewayService

    {

        /// <summary>

        /// Sends device to cloud message using device token

        /// </summary>

        /// <param name="deviceId"></param>

        /// <param name="payload"></param>

        /// <param name="sasToken"></param>

        /// <param name="dateTime"></param>

        /// <returns></returns>

        Task SendDeviceToCloudMessageByToken(string deviceId, string payload, string sasToken, DateTime dateTime);

        /// <summary>

        /// Sends device to cloud message using shared access token

        /// </summary>

        /// <param name="deviceId"></param>

        /// <param name="payload"></param>

        /// <returns></returns>

        Task SendDeviceToCloudMessageBySharedAccess(string deviceId, string payload);

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

using Microsoft.Azure.Devices.Client;

using Microsoft.Extensions.Caching.Memory;

using Microsoft.Extensions.Logging;

using Microsoft.Extensions.Options;

using System;

using System.Collections.Generic;

using System.Linq;

using System.Text;

using System.Threading;

using System.Threading.Tasks;

using System.Web;

namespace IoTHubGateway.Server.Services

{

    /// <summary>

    /// IoT Hub Device client multiplexer based on AMQP

    /// </summary>

    public class GatewayService : IGatewayService

    {

        private readonly ServerOptions serverOptions;

        private readonly IMemoryCache cache;

        private readonly ILogger<GatewayService> logger;

        RegisteredDevices registeredDevices;

        /// <summary>

        /// Sliding expiration for each device client connection

        /// Default: 30 minutes

        /// </summary>

        public TimeSpan DeviceConnectionCacheSlidingExpiration { get; set; }

        /// <summary>

        /// Constructor

        /// </summary>

        public GatewayService(ServerOptions serverOptions, IMemoryCache cache, ILogger<GatewayService> logger, RegisteredDevices registeredDevices)

        {

            this.serverOptions = serverOptions;

            this.cache = cache;

            this.logger = logger;

            this.registeredDevices = registeredDevices;

            this.DeviceConnectionCacheSlidingExpiration = TimeSpan.FromMinutes(serverOptions.DefaultDeviceCacheInMinutes);

        }

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

        public async Task SendDeviceToCloudMessageByToken(string deviceId, string payload, string sasToken, DateTime tokenExpiration)

        {

            var deviceClient = await ResolveDeviceClient(deviceId, sasToken, tokenExpiration);

            if (deviceClient == null)

                throw new DeviceConnectionException($"Failed to connect to device {deviceId}");

            try

            {

                await deviceClient.SendEventAsync(new Message(Encoding.UTF8.GetBytes(payload))

                {

                    ContentEncoding = "utf-8",

                    ContentType = "application/json"

                });

                this.logger.LogInformation($"Event sent to device {deviceId} using device token. Payload: {payload}");

            }

            catch (Exception ex)

            {

                this.logger.LogError(ex, $"Could not send device message to IoT Hub (device: {deviceId})");

                throw;

            }

        }

        public async Task SendDeviceToCloudMessageBySharedAccess(string deviceId, string payload)

        {

            var deviceClient = await ResolveDeviceClient(deviceId);

            try

            { 

                await deviceClient.SendEventAsync(new Message(Encoding.UTF8.GetBytes(payload))

                {

                    ContentEncoding = "utf-8",

                    ContentType = "application/json"

                });

                this.logger.LogInformation($"Event sent to device {deviceId} using shared access. Payload: {payload}");

            }

            catch (Exception ex)

            {

                this.logger.LogError(ex, $"Could not send device message to IoT Hub (device: {deviceId})");

                throw;

            }

        }

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

        private async Task<DeviceClient> ResolveDeviceClient(string deviceId, string sasToken = null, DateTime? tokenExpiration = null)

        {

            try

            {

                var deviceClient = await cache.GetOrCreateAsync<DeviceClient>(deviceId, async (cacheEntry) =>

                {

                    IAuthenticationMethod auth = null;

                    if (string.IsNullOrEmpty(sasToken))

                    {

                        auth = new DeviceAuthenticationWithSharedAccessPolicyKey(deviceId, this.serverOptions.AccessPolicyName, this.serverOptions.AccessPolicyKey);

                    }

                    else

                    {

                        auth = new DeviceAuthenticationWithToken(deviceId, sasToken);

                    }

                    var newDeviceClient = DeviceClient.Create(

                       this.serverOptions.IoTHubHostName,

                        auth,

                        new ITransportSettings\[\]

                        {

                            new AmqpTransportSettings(TransportType.Amqp_Tcp_Only)

                            {

                                AmqpConnectionPoolSettings = new AmqpConnectionPoolSettings()

                                {

                                    Pooling = true,

                                    MaxPoolSize = (uint)this.serverOptions.MaxPoolSize,

                                }

                            }

                        }

                    );

                    newDeviceClient.OperationTimeoutInMilliseconds = (uint)this.serverOptions.DeviceOperationTimeout;

                    await newDeviceClient.OpenAsync();

                    if (this.serverOptions.DirectMethodEnabled)

                        await newDeviceClient.SetMethodDefaultHandlerAsync(this.serverOptions.DirectMethodCallback, deviceId);

                    if (!tokenExpiration.HasValue)

                        tokenExpiration = DateTime.UtcNow.AddMinutes(this.serverOptions.DefaultDeviceCacheInMinutes);

                    cacheEntry.SetAbsoluteExpiration(tokenExpiration.Value);

                    cacheEntry.RegisterPostEvictionCallback(this.CacheEntryRemoved, deviceId);

                    this.logger.LogInformation($"Connection to device {deviceId} has been established, valid until {tokenExpiration.Value.ToString()}");

                    registeredDevices.AddDevice(deviceId);

                    return newDeviceClient;

                });

                return deviceClient;

            }

            catch (Exception ex)

            {

                this.logger.LogError(ex, $"Could not connect device {deviceId}");

            }

            return null;

        }

        private void CacheEntryRemoved(object key, object value, EvictionReason reason, object state)

        {

            this.registeredDevices.RemoveDevice(key);

        }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

using System;

using System.Runtime.Serialization;

namespace IoTHubGateway.Server.Services

{

    /// <summary>

    /// Exception raised when the device could not connected

    /// </summary>

    \[Serializable\]

    public class DeviceConnectionException : Exception

    {

        public DeviceConnectionException()

        {

        }

        public DeviceConnectionException(string message) : base(message)

        {

        }

        public DeviceConnectionException(string message, Exception innerException) : base(message, innerException)

        {

        }

        protected DeviceConnectionException(SerializationInfo info, StreamingContext context) : base(info, context)

        {

        }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

using Microsoft.Azure.Devices.Client;

using Microsoft.Extensions.Caching.Memory;

using Microsoft.Extensions.Hosting;

using Microsoft.Extensions.Logging;

using Microsoft.Extensions.Options;

using System;

using System.Collections.Generic;

using System.Linq;

using System.Threading;

using System.Threading.Tasks;

namespace IoTHubGateway.Server.Services

{

    public class CloudToMessageListenerJobHostedService : IHostedService

    {

        private readonly IMemoryCache cache;

        private readonly ILogger<CloudToMessageListenerJobHostedService> logger;

        private readonly RegisteredDevices registeredDevices;

        private readonly ServerOptions serverOptions;

        public CloudToMessageListenerJobHostedService(

            IMemoryCache cache, 

            ILogger<CloudToMessageListenerJobHostedService> logger, 

            RegisteredDevices registeredDevices,

            ServerOptions serverOptions)

        {

            this.cache = cache;

            this.logger = logger;

            this.registeredDevices = registeredDevices;

            this.serverOptions = serverOptions;

        }

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

        /// <summary>

        /// Checks for device messages

        /// </summary>

        /// <returns></returns>

        private void CheckDeviceMessages()

        {

            var deviceIdList = this.registeredDevices.GetDeviceIdList();

            Parallel.ForEach(deviceIdList, new ParallelOptions() { MaxDegreeOfParallelism = serverOptions.CloudMessageParallelism }, (deviceId) =>

            {

                try

                {

                    if (this.cache.TryGetValue<DeviceClient>(deviceId, out var deviceClient))

                    {

                        var message = deviceClient.ReceiveAsync(TimeSpan.FromMilliseconds(1)).GetAwaiter().GetResult();

                        if (message != null)

                        {

                            try

                            {                                

                                this.serverOptions.CloudMessageCallback(deviceId, message);                                

                                deviceClient.CompleteAsync(message).GetAwaiter().GetResult();

                            }

                            catch (Exception handlingMessageException)

                            {

                                logger.LogError(handlingMessageException, $"Error handling message from {deviceId}");

                            }

                        }

                    }

                }

                catch (Exception ex)

                {

                    logger.LogError(ex, $"Error receiving message from {deviceId}");

                }

            });

        }

\`\`\`

<div style="page-break-after: always;"></div>

\`\`\`csharp

        /// <summary>

        /// Starts job

        /// </summary>

        /// <param name="stoppingToken"></param>

        /// <returns></returns>

        public Task StartAsync(CancellationToken stoppingToken)

        {

            logger.LogDebug($"{nameof(CloudToMessageListenerJobHostedService)} is starting.");

            if (this.serverOptions.CloudMessageCallback == null)

            {

                logger.LogInformation($"{nameof(CloudToMessageListenerJobHostedService)} not executing as no handler was defined in {nameof(ServerOptions)}.{nameof(ServerOptions.CloudMessageCallback)}.");

            }

            else

            {

                stoppingToken.Register(() => logger.LogDebug($" {nameof(CloudToMessageListenerJobHostedService)} background task is stopping."));

                while (!stoppingToken.IsCancellationRequested)

                {

                    CheckDeviceMessages();

                }

            }

            logger.LogDebug($"{nameof(CloudToMessageListenerJobHostedService)} background task is stopping.");

            return Task.CompletedTask;

        }

        /// <summary>

        /// Ends the job

        /// </summary>

        /// <param name="stoppingToken"></param>

        /// <returns></returns>

        public Task StopAsync(CancellationToken stoppingToken)

        {

            return Task.CompletedTask;

        }

        

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\### Controllers

\`\`\`csharp

using IoTHubGateway.Server.Services;

using Microsoft.AspNetCore.Mvc;

using Microsoft.Extensions.Options;

using System;

using System.Threading.Tasks;

namespace IoTHubGateway.Server.Controllers

{

    \[Route("api")\]

    public class GatewayController : Controller

    {

        private readonly IGatewayService gatewayService;

        private readonly ServerOptions options;

        private static readonly DateTime epoch = new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc);

        public GatewayController(IGatewayService gatewayService, ServerOptions options)

        {

            this.gatewayService = gatewayService;

            this.options = options;

        }

        /// <summary>

        /// Sends a message for the given device

        /// </summary>

        /// <param name="deviceId">Device identifier</param>

        /// <param name="payload">Payload (JSON format)</param>

        /// <returns></returns>

        \[HttpPost("{deviceId}")\]

        public async Task<IActionResult> Send(string deviceId, \[FromBody\] dynamic payload)

        {

            if (string.IsNullOrEmpty(deviceId))

                return BadRequest(new { error = "Missing deviceId" });

            if (payload == null)

                return BadRequest(new { error = "Missing payload" });

            

            var sasToken = this.ControllerContext.HttpContext.Request.Headers\[Constants.SasTokenHeaderName\].ToString();

            if (!string.IsNullOrEmpty(sasToken))

            {

                var tokenExpirationDate = ResolveTokenExpiration(sasToken);

                if (!tokenExpirationDate.HasValue)

                    tokenExpirationDate = DateTime.UtcNow.AddMinutes(20);

                await gatewayService.SendDeviceToCloudMessageByToken(deviceId, payload.ToString(), sasToken, tokenExpirationDate.Value);

            }

            else

            {

                if (!this.options.SharedAccessPolicyKeyEnabled)

                    return BadRequest(new { error = "Shared access is not enabled" });

                await gatewayService.SendDeviceToCloudMessageBySharedAccess(deviceId, payload.ToString());

            }

            return Ok();

        }

        /// <summary>

        /// Expirations is available as parameter "se" as a unix time in our sample application

        /// </summary>

        /// <param name="sasToken"></param>

        /// <returns></returns>

        private DateTime? ResolveTokenExpiration(string sasToken)

        {

            // TODO: Implement in more reliable way (regex or another built-in class)

            const string field = "se=";

            var index = sasToken.LastIndexOf(field);

            if (index >= 0)

            {

                var unixTime = sasToken.Substring(index + field.Length);

                if (int.TryParse(unixTime, out var unixTimeInt))

                {

                    return epoch.AddSeconds(unixTimeInt);

                }

            }

            return null;

        }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\## Client Example

\`\`\`csharp

using Microsoft.Azure.Devices.Client;

using System;

using System.Net.Http;

using System.Text;

using System.Threading.Tasks;

namespace CSharpClient

{

    class Program

    {

        static async Task Main(string\[\] args)

        {

            Console.WriteLine("<PRESS ENTER TO CONTINUE>");

            Console.ReadLine();

            var hostName = "<enter-iothub-name>";

            var deviceId = "<enter-device-id>";

            var sasToken = new SharedAccessSignatureBuilder()

            {

                Key = "",

                Target = $"{hostName}.azure-devices.net/devices/{deviceId}",

                TimeToLive = TimeSpan.FromMinutes(20)

            }

            .ToSignature();

            using (var client = new HttpClient())

            {

                client.DefaultRequestHeaders.Add("sas_token", sasToken);

                while (true)

                {

                    var postResponse = await client.PostAsync($"http://localhost:32527/api/{deviceId}", new StringContent("{ content: 'from_rest_call' }", Encoding.UTF8, "application/json"));

                    Console.WriteLine($"Response: {postResponse.StatusCode.ToString()}");

                    await Task.Delay(200);

                }

            }

        }

    }

}

\`\`\`

<div style="page-break-after: always;"></div>

\### appsetting.Development.json

\`\`\`json

{

  "Logging": {

    "IncludeScopes": false,

    "LogLevel": {

      "Default": "Debug",

      "System": "Information",

      "Microsoft": "Information"

    }

  },

  "ServerOptions": {

    "IoTHubHostName": "xxx.azure-devices.net",

    "AccessPolicyName": "iothubowner",

    "AccessPolicyKey": "xxxxxxx",

    "SharedAccessPolicyKeyEnabled": true,

    "DirectMethodEnabled": true,

    "CloudMessagesEnabled": true

  }

}

\`\`\`

\# CHAPTER 19 - MICROSERVICES

\## Comparison between application development approaches

!\[\](![](https://i.imgur.com/qbZp86E.png))

\## We will be using Azure Service Fabric

Service Fabric is built with layered subsystems. These subsystems enable us to write applications that are:

\* Highly available

\* Scalable

\* Manageable

\* Testable

The following diagram shows the major subsystems of Service Fabric.  

!\[\](![](https://i.imgur.com/y1KSvXH.png))

> Azure Service Fabric is an orchestrator of services across a cluster of machines, with years of usage and optimization in massive scale services at Microsoft. Services can be developed in many ways, from using the Service Fabric programming models to deploying guest executables. By default, Service Fabric deploys and activates these services as processes. Processes provide the fastest activation and highest density usage of the resources in a cluster. Service Fabric can also deploy services in container images. Importantly, you can mix services in processes and services in containers in the same application.

\### Containers

Containers are encapsulated, individually deployable components that run as isolated instances on the same kernel to take advantage of virtualization that an operating system provides. Thus, each application and its runtime, dependencies, and system libraries run inside a container with full, private access to the container's own isolated view of operating system constructs. Along with portability, this degree of security and resource isolation is the main benefit for using containers with Service Fabric, which otherwise runs services in processes.

<div style="page-break-after: always;"></div>

\# CHAPTER 20 -  MAPPING OUR DEVICE DATA TO FHIR RESOURCES 

\## Mapping our Device Data to HL7 FHIR resources  

HL7 FHIR supports both JSON and XML message formats. Schemas are available for both.

\### There are two methods for mapping our Device Data

\* Transforming XML

\* Transform JSON

\**Before we start we must first create an Integration Account on the Azure Portsl**

\## Integration Account

An integration account provides a way for our enterprise integration apps, specifically logic apps, to access and manage B2B artifacts, for example, trading partners, agreements, maps, schemas, certificates, and so on. To provide this access, we link our integration account to our logic app, after making sure that both the integration account and logic app have the same Azure location. 

\**We will <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-create-integration-account](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-create-integration-account "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-create-integration-account")" target="_blank">create our Integration Account</a> using the Azure Portal.**

<div style="page-break-after: always;"></div>

\## Transform XML

To simplify transforming our data to the HL7 FHIR Resources (Documents), we will be using Visual Studio 2015 2.0 with the <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-overview](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-overview "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-overview")" target="_blank">Azure Logic Apps Enterprise Integration Tools</a>. You can download the tools <a href="[https://www.microsoft.com/en-us/download/details.aspx?id=53016](https://www.microsoft.com/en-us/download/details.aspx?id=53016 "https://www.microsoft.com/en-us/download/details.aspx?id=53016")" target="_blank">here</a>.

\**After we have created our integration account on the Azure Portal, we will create a new Integration Account project in Visual Studio 2015.**

In this project we can add or create schemas, flat file schemas, and maps. We will be adding schemas from the fhir-codegen-xsd that can be <a href="[http://hl7.org/fhir/2018May/fhir-codegen-xsd.zip](http://hl7.org/fhir/2018May/fhir-codegen-xsd.zip "http://hl7.org/fhir/2018May/fhir-codegen-xsd.zip")" target="_blank">downloaded</a> from the FHIR site.

>!\[\](![](https://i.imgur.com/DiPYycK.png))  We will not be using flat file schemas at this time

\### We will add the following schemas to our project

\* devicerequest

\* fhir-base

\* sequence

\* location

\* patient

\* location

\* observation

\* media

\* imagingstudy

\* fhir-xhtml

\* xml

\* device

\* devicecomponent

\* devicemetric 

<div style="page-break-after: always;"></div>

\#### Modifying the FHIR schemas so that they validate.

1\. In **fhir-base.xsd** remove \`<xs:include schemaLocation="fhir-all.xsd"/>\` at line number 40.

2\. Under the **ResourceContaine**r element, remove every   \`<xs:element name="XXXXX" type="XXXXX" minOccurs="0" maxOccurs="1"/>\` except for the following: 

    

    * sequence

    * location

    * patient

    * location

    * observation

    * media

    * imagingstudy

    * device

    * devicecomponent

    * devicemetric  

\-------------

>  !\[\](![](https://i.imgur.com/DiPYycK.png)) 

> NOTE: XML Representation of Resources. You can read more about this <a href="[http://hl7.org/fhir/2018May/xml.html#schema-gen](http://hl7.org/fhir/2018May/xml.html#schema-gen "http://hl7.org/fhir/2018May/xml.html#schema-gen")" target="_blank">here</a>.

In addition to Transforming XML, we can validate, encode and decode flat files, and enrich messages in our Logic Apps. You can read more about this <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-xml](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-xml "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-xml")" target="_blank">here</a>.

\### Steps in Logic App

1\. Receive XML Message

2\. Validate against the schema

3\. Transform

4\. Convert to JSON

<div style="page-break-after: always;"></div>

\### Adding in the HL7 V2.6 XML Schemas

The following figure shows our Visual Studio 2015 Integration Account project items. We have added the HL7 V2.6 schemas

!\[\](![](https://i.imgur.com/hwZ6Ohy.png))

In order to get the schemas to validate, each **ORU** schema imports needs modification. Change it to the following.

\`\`\`xml

<xs:import schemaLocation="segments_26.xsd" namespace="[http://microsoft.com/HealthCare/HL7/2X/2.6/Segments](http://microsoft.com/HealthCare/HL7/2X/2.6/Segments "http://microsoft.com/HealthCare/HL7/2X/2.6/Segments")" />

\`\`\`

\**At this point we can start mapping the ORU to the FHIR Observation schema.** 

<div style="page-break-after: always;"></div>

\## Transform JSON

\### Message manipulation action

Azure Logic Apps supports basic JSON transformations through native data operation actions such as Compose or Parse JSON. Included are built-in actions that can change or manipulate our payload data. The built-in Data Operations connector includes the following actions: 

| Action                              | Description                              |

|-------------------------------------|------------------------------------------|

| Compose                             | Build or generate values or objects to use later, or as you build your workflow. For example, you can author a JSON object with values from multiple steps, or calculate a constant to reference later in a logic app run |

| Create CSV table<br>Create HTML table | Turn an array result-set into a CSV or HTML table. For example, add the CRM "List records" action, and add a filter for records added today. Then, send the results as an HTML table in an email. |

| Filter array (query)                | Filter a result set to the entries that interest you. For example, search all tweets with \`#Azure\`, and then "filter" the returned tweets to only return results that are \`Tweeted_by_followers > 50\`. |

| Join                                | Join an array by some delimiter. For example, the Detect Key Phrases operation returns an array of key phrases. You could "join" them with a , or something similar. So instead of \`\["Some", "Phrase"\]\`, you have \`"Some, Phrase"\`. |

| Parse JSON                          | Parse out and access values from a JSON object in the designer. For example, if your Azure Function returns a JSON payload, then you can parse it to access the JSON properties later in another step. The action also validates that the JSON matches the specified schema at runtime. |

| Select                              | Select certain properties of an array for further processing. If you "List records" from SQL, and it returns 15 columns, then select just a few of those columns for further processing. The output is an array that only contains the properties you select. |

<div style="page-break-after: always;"></div>

\### For advanced JSON transformations, we can use Liquid templates with our logic apps. 

<a href="[https://shopify.github.io/liquid/](https://shopify.github.io/liquid/ "https://shopify.github.io/liquid/")" target="_blank">Liquid</a> is an open-source template language for flexible web apps.

\#### Liquid Logic App Actions

\* Transform JSON to JSON

\* Transform JSON to TEXT

\* Transform XML to JSON

\* Transform XML to TEXT

\### Create a Liquid template or map for your integration account

The complete documentation is available <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform#create-a-liquid-template-or-map-for-your-integration-account](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform#create-a-liquid-template-or-map-for-your-integration-account "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform#create-a-liquid-template-or-map-for-your-integration-account")" target="_blank">here</a>

The following example shows a Liquid template that defines how to transform JSON input 

\`\`\`json

{%- assign deviceList = content.devices | Split: ', ' -%}

   {

     "fullName": "{{content.firstName | Append: ' ' | Append: content.lastName}}",

     "firstNameUpperCase": "{{content.firstName | Upcase}}",

     "phoneAreaCode": "{{content.phone | Slice: 1, 3}}",

     "devices" : \[

     {%- for device in deviceList -%}

         {%- if forloop.Last == true -%}

         "{{device}}"

         {%- else -%}

         "{{device}}",

         {%- endif -%}

     {%- endfor -%}

     \]

   }

\`\`\`

<div style="page-break-after: always;"></div>

\##### Add the Liquid action for JSON transformation

We will add the the action to our Logic App. You can read more about this <a href="[https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform#add-the-liquid-action-for-json-transformation](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform#add-the-liquid-action-for-json-transformation "https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-liquid-transform#add-the-liquid-action-for-json-transformation")" target="_blank">here</a>.

Before we can perform a Liquid transformation in our logic app, we must define the mapping from JSON to JSON with a Liquid map and store that map in our integration account.

\### Transforming XML to JSON

The following is an example.

\`\`\`json

\[{% JSONArrayFor item in content -%}

     {{item}}

 {% endJSONArrayFor -%}\]

\`\`\`

\##### This is the sample input and output  

!\[\](![](https://i.imgur.com/IQKvnN0.png))  

<div style="page-break-after: always;"></div>

\# CHAPTER 21 - HL7 FHIR DEVICE DATA INTEGRATION

\## Roles

These roles are the same as IHE PCD Profiles from \[CHAPTER 8 - PATIENT CARE DEVICE DATA INTEGRATION\](#chapter-8---patient-care-device-data-integration)

\### Point-of-care Device Manager- FHIR client

This is a new actor intended to track implantable devices, associate a patient to their monitoring devices, or help validate medication administration at the point of care using an infusion pump. This point-of-care Device Manager implements several FHIR resources (as a client) in order to:   

\* Start and end a monitoring session as a <a href="[http://hl7-fhir.github.io/procedure.html](http://hl7-fhir.github.io/procedure.html "http://hl7-fhir.github.io/procedure.html")" target="_blank">Procedure</a> "in-progress" and updates the procedure at the end of the monitoring session (and set the status to "complete") 

\* Create a <a href="[http://hl7-fhir.github.io/device.html](http://hl7-fhir.github.io/device.html "http://hl7-fhir.github.io/device.html")" target="_blank">Device</a> resource corresponding to each device implanted during a surgical procedure and creates <a href="[http://hl7-fhir.github.io/procedure.html](http://hl7-fhir.github.io/procedure.html "http://hl7-fhir.github.io/procedure.html")" target="_blank">Procedure</a> to document it. 

> !\[\](![](https://i.imgur.com/dMrFdY8.png)) NOTE: These interactions represent new capabilities to allow a Device Manager to assign devices (implanted or used for treatment) to a patient.  

> 

> Currently an integrator uses an ad-hoc HL7 V2 visit or appointment notification to start and stop a device monitoring session.

\### Device Observation Reporter (DOR)

Send device acquired information to a FHIR server using the device-related resources (i.e., <a href="[https://www.hl7.org/fhir/device.html](https://www.hl7.org/fhir/device.html "https://www.hl7.org/fhir/device.html")" target="_blank">Device</a>, <a href="[https://www.hl7.org/fhir/devicecomponent.html](https://www.hl7.org/fhir/devicecomponent.html "https://www.hl7.org/fhir/devicecomponent.html")" target="_blank">DeviceComponent</a>, <a href="[https://www.hl7.org/fhir/devicemetric.html](https://www.hl7.org/fhir/devicemetric.html "https://www.hl7.org/fhir/devicemetric.html")" target="_blank">DeviceMetric</a>, <a href="[https://www.hl7.org/fhir/observation.html](https://www.hl7.org/fhir/observation.html "https://www.hl7.org/fhir/observation.html")" target="_blank">Observation</a>). 

> !\[\](![](https://i.imgur.com/dMrFdY8.png)) NOTE: This is analogous to the <a href="[http://www.ihe.net/Technical_Frameworks/#pcd](http://www.ihe.net/Technical_Frameworks/#pcd "http://www.ihe.net/Technical_Frameworks/#pcd")" target="_blank">IHE PCD Device Enterprise Communication (DEC) profile DOR actor</a>. **Typically a device or a device "gateway" system**.

 

\### Device Information Server (DIS)

Process updates from a DOR and respond to queries from a DOC. 

\### Device Observation Consumer (DOC)

Retrieve and use device-acquired information from the DIS, e.g. for flow-charting or clinical decision support applications. 

> !\[\](![](https://i.imgur.com/dMrFdY8.png)) NOTE: This is analogous to the <a href="[http://www.ihe.net/Technical_Frameworks/#pcd](http://www.ihe.net/Technical_Frameworks/#pcd "http://www.ihe.net/Technical_Frameworks/#pcd")" target="_blank">IHE PCD Device Enterprise Communication (DEC) profile DOC actor</a>. **Typically an EHR or similar information system**. 

<div style="page-break-after: always;"></div>

\## Scenario

\### Patient-Device and Device-Operator Associations and Management using FHIR Procedure, FHIR Device, and UDI

\* This scenario uses FHIR Procedure and Device to associate a patient to one or more device(s) and operator/nurse/physician who configured or implanted the device. The device will be identified using the UDI scanned at the point of care along the patient id wristband, and the provider's badge.   

\* Create a FHIR Procedure specifying the patient, the device or devices involved, and the operator (i.e. nurse, respiratory therapist, etc) or provider who configured the device, validated its configuration and initiated the procedure.  

\* If the device is implanted, a FHIR Device resource will associate the patient with the implanted device. The Patient Device List will be created by querying the EHR using a Device.search (by patient) operation.  

\* If the device is used monitor the patient or administer medication the procedure indicates duration, the associated medication order. 

<div style="page-break-after: always;"></div>

\#### Patient-to-implementable Association using FHIR Device resource and FHIR Procedure

!\[Patient-to-implementable Association using FHIR Device resource and FHIR Procedure\](![](https://i.imgur.com/g66RbLD.png))

<div style="page-break-after: always;"></div>

\## Implantable Devices

This scenario relies on scanning the UDI in order to convey information about devices implanted to during a procedure. The Medical Device Adapter (FHIR client) submits to the EHR (FHIR server): 

\* One Device record per implantable devices 

\* One Procedure referencing the devices implanted. 

> !\[\](![](https://i.imgur.com/DiPYycK.png)) Coordinated with the<a href="[http://argonautwiki.hl7.org/index.php?title=Implantable_Devices/UDI](http://argonautwiki.hl7.org/index.php?title=Implantable_Devices/UDI "http://argonautwiki.hl7.org/index.php?title=Implantable_Devices/UDI")" target="_blank"> Argonaut Implantable / UDI Project</a>

<div style="page-break-after: always;"></div>

\#### Patient-to-device-to-operator Association using FHIR Encounter

!\[\](![](https://i.imgur.com/gIJxpnz.png))

\## Vital Sign Monitoring Procedure

The Medical Device Records (FHIR client) performs the following operations on the EHR/Clinical Information System (FHIR server):   

1\. The Device Manager "create" a **Procedure** with status \`in-progress\` to associate/link the patient(patient MRN) to the device (UDI) and provider a the point-of-care. 

    * The Device Manager creates a Procedure when a patient starts to be monitored by a vital signs monitor. The Procedure will reference a contained Device resource or externally referenced one. 

    * The Device sets the \`Device.udiCarrier\` to the device UDI. 

2\. The Device Manager updates the Procedure and sets \`Procedure.performed\` (Period) and \`Procedure.status\` to \`"complete"\`. This update effectively "unlinks" the patient-device-provider. 

<div style="page-break-after: always;"></div>

\## Medication Administration BCMA Procedure

The 5+3 rights of of bedside computerized medication administration.

\### 5+3 Rights of Medication Administration

!\[\](![](https://i.imgur.com/L9p3rD8.png))

<div style="page-break-after: always;"></div>

\# APPENDICES

<div style="page-break-after: always;"></div>

\# APPENDIX I - MEDICAL DEVICE AND IMPLANTABLES TRACKING USING UDI

This track is from the January 2018 HL7 FHIR Connectathon.

> !\[\](![](https://i.imgur.com/eSPsKWl.png))  **INFO: <u>It is highly recommended that you attempt to do the Use Cases</u>** 

\## Goals

This track is validating the use FHIR STU3 resources to: 

\* Record device <a href="[https://www.fda.gov/MedicalDevices/DeviceRegulationandGuidance/UniqueDeviceIdentification/](https://www.fda.gov/MedicalDevices/DeviceRegulationandGuidance/UniqueDeviceIdentification/ "https://www.fda.gov/MedicalDevices/DeviceRegulationandGuidance/UniqueDeviceIdentification/")" target="_blank">UDI</a> at the point-of-care and share it with enterprise systems. 

\**Record accurate information about implants (e.g. orthopedic, cardiac) and prosthetics recorded at the point-of-care (e.g. during a procedure).**   

\* Record the Start/end Point-of-care Procedure - at the point-of-care, to avoid documentation errors 

    * Record Implant Procedure in EHR 

    * Update inventory system 

    * Make UDI available to ancillary systems  

    

    

\* **Leverage FHIR “search” Application Programming API to lookup device UDI for a variety of purposes** 

    * Use <a href="[https://www.fda.gov/MedicalDevices/DeviceRegulationandGuidance/UniqueDeviceIdentification/](https://www.fda.gov/MedicalDevices/DeviceRegulationandGuidance/UniqueDeviceIdentification/ "https://www.fda.gov/MedicalDevices/DeviceRegulationandGuidance/UniqueDeviceIdentification/")" target="_blank">UDI</a> for outcome analysis 

    * Product recalls 

    * Patient safety events

<div style="page-break-after: always;"></div>

\## Project/Implementer Group

This track provides a set of simple test cases for Tracking Medical Devices (i.e. diagnostic, therapeutic, monitoring) and Implantables (biologics/tissue/cells and non-biologics/life-supporting/life-sustaining) at point of care addresses current problems relate to information accuracy including procedure-related context. 

This track specifies test cases related FHIR STU3 Procedure (using three PMDT-specific profiles) and FHIR Device resources (one PMDT profile) used to report information from the point-of-care to enterprise system consistent with the IHE PCC Point-of-care Medical Device Tracking Supplement.  

\## Justification

\* Implantable medical devices are costly and concerns about illegitimate (i.e., counterfeit, stolen) products has become a global issue 

\* By 2021 all medical devices that may be used in a point-of-care procedure will be labeled using FDA UDI that will also relate to software/firmware release 

\* Post-market surveillance of implantable medical devices can be challenging 

\* Implantable medical device adverse events and recalls pose a patient safety issue 

\* Acquiring medical device data used at the point-of-care is difficult to retrieve for reuse at a later time 

\* A consistent device identification scheme closes the loop on data acquisition at the point-of-care to support reporting of medical device data   

\**Medical device data used for:** 

\* Continuum of care (e.g., Discharge Summary, Referrals) 

\* Registries (e.g., Total Joint Registry) 

\* Payers (e.g., government provided, private insurance) 

\* Can associate a medical device used for monitoring a disease or symptom of a disease (e.g., vital sign monitors, pulse oximeters, blood glucose monitors) to a patient for querying the device or procedure using the UDI 

\* Increase patient safety 

    * Traceability of medical devices (avoid use of counterfeit or illegitimate products) 

    

    * Quality issues identified (e.g., recalls, adverse events) 

    

\* Increase accurate medical device data capture at the point-of-care 

\* Eliminates human error from manual medical device data entry 

\--------------

!\[\](![](https://i.imgur.com/PLnvxGu.png))

<div style="page-break-after: always;"></div>

\## Roles

The test roles for this track are based on the IHE PCC PMDT Actors below: 

!\[\](![](https://i.imgur.com/R4ngSSQ.png))

\### Point-of-Care Medical Device Reporter

\**Point-of-care client**; it registers medical device, creates/updates point-of-care procedure information, query patient based on identifier scanned at the point-of-care. This role is played by a system used to track information about medical devices used at the point-of-care. The device and procedure information are populated at the point-of-care using scanned (AIDC) UDI and patient identifier to simplify the accurate tracking of this information. 

\### Medical Device Server

\**FHIR server**, exposes Device, Procedure - references Patient. It is used to persist device and procedure information originating at the point of care. This information is made available to other information systems in the enterprise. This role may be played by a Medical Device Registry or an EHR system. 

\### Point-of-Care Medical Device Requester

This role is **played by any information system** that requires to compile an implantable device list for patient, evaluate outcomes based on device type (i.e. DI), respond manufacturers' recalls, and address patient safety events.

<div style="page-break-after: always;"></div>

\## Scenarios

The following is a summary test cases proposed:

1\. **Register (create) device record** using 

   1. UDI and DI only 

   2. UDI and DI and optional attributes 

2\. **Start procedure at the point of care**. This may an infusion procedure, a diagnostic procedure using a device, a monitoring session, etc. The Procedure resource specifies the status as "in-progress" and the start of the procedure as well as the coded procedure type, patient, practitioner, etc. At the minimum the patient, the device, and the practitioner known at the point-of-care should be specified. 

    1. with externally referenced device 

3\. **Complete procedure**; it marks its status to "completed" and specifies the time it was completed. 

    1. with externally referenced device 

4\. **Complete implantable procedure**; this a one-time record that a device, tissue, or tissue products was implated. 

    1. with externally referenced device; it assumes the device was already registered and its UDI and its other identifiers were captures. 

    2. with contained device reference; the device information is included as "contained" resource in the Procedure. 

5\. **Search procedures**; to compute outcomes, compile implantable device lists for transition-of-care, to respond to recall notifications 

    1. by patient 

    2. by other optional search parameters 

6\. **Search devices**; to track device utilization, for quality control, to compute outcomes, compile implantable device lists for transition-of-care, to respond to recall notifications 

   1. by udi-carrier 

   2. by patient 

   3. by udi-di 

   

<div style="page-break-after: always;"></div>

\### Register Medical Device or Implantable from the point-of-care

\* **Action**: The 'Tracker' creates a Device resource (using PUT/update), sets Device.id to a 'GUID', Device.udi.carrierAIDC to the base64-encoded, 'scanned UDI', sets Device.status to 'active'. The Device.patient references is set to external reference previously retrieved Patient resource. (see 'Precondition'). 

\* **Precondition**: The 'Tracker' queried the Patient resource using the Patient.identifier scanned at the point-of-care. This external references is used in Device.patient. 

\* **Success Criteria**: The 'Requester' queries Device by 'udi-carrier' (expression: Device.udi.carrierHRF | Device.udi.carrierAIDC) and retrieves one matching resource containing the correct information. The 'Tracker' can retrieve it by logical id/GUID. 

\* **Bonus point**: specify optional data elements (S, 0..) of the Device resource. 

!\[\](![](https://i.imgur.com/rtW2IZj.png))

<div style="page-break-after: always;"></div>

\### Complete an Implantable Procedure

\* **Action**: The 'Tracker' creates a Procedure resource (using PUT/update), sets the Procedure.id to a GUID, Procedure.status to 'completed', Procedure.focalDevice.manipulated to reference the device registered above. The 'Tracker' set the Procedure.performedDateTime to the current date/time. The 'Tracker' specifies the Procedure.code corresponding the implant procedure performed. 

\* **Precondition**: The 'Tracker' queries the Patient resource using the Patient.identifier scanned at the point-of-care. The 'Tracker' registered a medical device or implantable using a pre-defined GUID as its logical identifier. 

\* **Success Criteria**: The tracker can retrieve the Procedure resource using its _id. The 'Requester' queries Procedure based on the Patient reference. 

\* **Bonus point**: The 'Requester' searches the Procedure resource based on Procedure.code (expression: code). 

!\[\](![](https://i.imgur.com/lBrsQop.png))

<div style="page-break-after: always;"></div>

\### Start a Point-of-Care Procedure

\* **Action**: The 'Tracker' creates a Procedure resource (using PUT/update), sets the Procedure.id to GUID, Procedure.status to 'in-progress', Procedure.focalDevice.manipulated to reference the device registered above. The 'Tracker' set the Procedure.performedPeriod.start to the current date/time. 

\* **Precondition**: The 'Tracker' queries the Patient resource using the Patient.identifier scanned at the point-of-care. The 'Tracker' registered a medical device or implantable using a pre-defined GUID as its logical identifier. 

\* **Success Criteria**: The tracker can retrieve the Procedure resource using its _id. The 'Requester' queries Procedure based on the Patient reference. 

\* **Bonus point**: The 'Requester' searches the Procedure resource based on Procedure.code (expression: code). 

\### Complete a Point-of-Care Procedure

\* **Action**: The 'Tracker' updates the Procedure resource (using PUT/update), it sets the Procedure.id to its previously set GUID, Procedure.status to 'completed', Procedure.focalDevice.manipulated to reference the device registered above. The 'Tracker' set the Procedure.performedPeriod.end to the current date/time. The 'Tracker' specifies the Procedure.code corresponding the implant procedure performed. 

\* **Precondition**: The 'Tracker' queries the Patient resource using the Patient.identifier scanned at the point-of-care. The 'Tracker' registered a medical device or implantable using a pre-defined GUID as its logical identifier. The 'Tracker' created a first version of the Procedure 

\* **Success Criteria**: The tracker can retrieve the Procedure resource using its _id. The 'Requester' queries Procedure based on the Patient reference. 

\* **Bonus point**: The 'Tracker' submits the same transaction using references to the Practitioners involved in the procedure. 

\### Search Devices by UDI, based on associated Condition,

\* **Action**: The 'Requester' retrieves the Device resources using the device identity (DI and lot number) as well as the Condition of the patient. The search API allows the Requestor to create composite searches that search devices by DI/PI and by an associated Condition.code and return the patients who match the composite search.

<a href="[https://www.hl7.org/fhir/search.html](https://www.hl7.org/fhir/search.html "https://www.hl7.org/fhir/search.html")" target="_blank">FHIR Search Application Programming Interface - API</a> sets the 

\* **Success Criteria**: The "Requester" can retrieve the Device resource based on Device DI and lot number 

\* **Bonus point**: The "Requester" can retrieve the Device resource based on Device DI and lot number and by patient's Condition. 

!\[\](![](https://i.imgur.com/p7krPFd.png))

<div style="page-break-after: always;"></div>

\## Goal

To use FHIR to record information about procedures performed at the point-of-care using medical devices (single-use, implantable, monitoring) and record the procedure (e.g. implant, monitoring), identity of the device(s) (e.g. UDI), references to patient, providers,etc.  

This track was intended to validate the FHIR profiles developed to supproto the "Point-of-care Medical Device Tracking" (PMDT) integration profile developed by VA and AORN (Asssociation of PeriOperative Nurses) as an IHE Patient Care Coordination Technical Framework supplement. 

\## Next Steps 

We will add "MedicationAdministration", "Medication", and "Location" to the set resources and profiles we could use to document point-of-care procedures. 

<div style="page-break-after: always;"></div>