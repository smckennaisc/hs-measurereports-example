Overview

Organizations that need to capture, store, and share patient-level quality or risk adjustment measure results via FHIR can do so by extending HealthShare SDA and enabling transformation to the FHIR MeasureReport resource.

This article describes how to configure HealthShare to support the Da Vinci Risk Adjustment (RA) Implementation Guide by enabling storage and FHIR-based exchange of MeasureReport resources for individual patients.

    Da Vinci Risk Adjustment IG:                   https://build.fhir.org/ig/HL7/davinci-ra/index.html
    FHIR MeasureReport Resource (R4):     https://hl7.org/fhir/R4/measurereport.html

The outcome of this configuration is the ability to persist MeasureReport data in SDA and expose it via FHIR R4 in compliance with the Da Vinci RA profile.

 
Solution Summary

This solution introduces a custom SDA3 container to store MeasureReport data for each patient and configures HealthShare to transform that data into a FHIR R4 MeasureReport resource.

The deliverables include:

    A custom SDA container and related supporting classes
    A custom DTL to transform SDA to FHIR
    Configuration updates to register and enable the transformation
    A test patient to validate end-to-end functionality

Once completed, HealthShare will:

    Store MeasureReport data per patient in SDA
    Automatically use the custom container
    Expose the MeasureReport through the FHIR R4 endpoint

Implementation Steps
1. Download, Import and Compile Custom Code

Download MeasureReportCode.xml from GitHub: 

Import and compile the contents of MeasureReportCode.xml into:

    UCR
    Edge Gateway
    ODS
    CV

Note: Applies to HealthShare v2025.1
Included Components in file

i. Custom SDA Container

    HS.Local.SDA3.ZContainer.cls

ii. Custom SDA3 MeasureReport Classes

    HS.Local.SDA3.ZMeasureReport.cls
    HS.Local.SDA3.ZMeasureReportGroup.cls
    HS.Local.SDA3.ZMeasureReportExtension.cls
    HS.Local.SDA3.Streamlet.ZMeasureReport.cls
    HS.Local.SDA3.CodeTableDetail.ZMeasureReportCategory.cls
    HS.Local.SDA3.CodeTableDetail.ZGroupCode.cls
    HS.Local.SDA3.CodeTableDetail.ZccRemark.cls

iii. Custom DTL

    HS.Local.FHIR.DTL.SDA3.vR4.MeasureReport.MeasureReport

2. Register the Custom SDA Container

 

In HSREGISTRY, add the following Configuration Registry entry:
\CustomSDA3Container = HS.Local.SDA3.ZContainer

This ensures that HealthShare uses the custom SDA container when processing SDA3 data.
3. Enable the SDA-to-FHIR Transformation

 

In HSODS, update the following global:

set ^HS.XF.Transform("SDA3","vR4", "HS.Local.SDA3.ZMeasureReport", "HS.Local.FHIR.DTL.SDA3.vR4.MeasureReport.MeasureReport") = 1 

This explicitly enables the transformation from the custom SDA3 MeasureReport container to the FHIR R4 MeasureReport resource.
4. Update XFTransform.xml

 

Locate XFTransform.xml in the instance directory:
/dev/fhir/gbl/

    You may need to temporarily set the file to read/write.

Add the following XML node after the SDA4 and vR4 nodes and before HS.SDA3.Address (typically around line 7):
<Node> <Sub>HS.Local.SDA3.ZMeasureReport</Sub> <Node> <Sub>HS.Local.FHIR.DTL.SDA3.vR4.MeasureReport.MeasureReport</Sub> <Data>1</Data> </Node> </Node>

This step ensures the transformation is recognized during runtime.
Testing and Validation

 
Load Test SDA Data

    Download TestMeasurePatient.txt from GitHub: 
    If running a HealthShare demo environment, drop TestMeasurePatient.txt into the SDAIn directory of HSEDGE1
    This can be any Edge configured to accept SDA3 messages

Validate via FHIR API

Using Postman or another API testing tool, issue the following requests:
Query by Patient and Profile

https://<your-host>/hs20251ucr/csp/healthshare/hsods/fhir/r4/MeasureReport ?patient=100000012 &_profile=http://hl7.org/fhir/us/davinci-ra/StructureDefinition/ra-measurereport 

    Replace <your-host> with your HealthShare URL
    Replace 100000012 with the patient’s MPIID

Query All MeasureReports

https://<your-host>/hs20251ucr/csp/healthshare/hsods/fhir/r4/MeasureReport 

Expected Result:

    A single MeasureReport resource should be returned

Validate in Clinical Viewer (CV)

 

    Look up the patient “Test MeasureReport”
    Confirm the SDA record includes the custom MeasureReport container

Outcome

After completing these steps, your HealthShare environment will be capable of:

    Persisting patient-level MeasureReport data in SDA
    Transforming SDA data into a FHIR R4 MeasureReport resource
    Sharing MeasureReport resources via FHIR in alignment with the Da Vinci Risk Adjustment Implementation Guide

This enables standards-based risk adjustment reporting and interoperability with downstream analytics, payers, and regulatory systems
