# Investigation Report: `system_id` and `capability_id` Definitions

**Project:** A-GRA UCI Interface Specification (v6.0.a)  


---

## Executive Summary

The project (a UCI — Unified Command and Control — message schema) uses **camelCase**: `SystemID` and `CapabilityID`.

Both are defined as complex types in the XSD schema `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd`, both extending a common base type `ID_Type` (which carries a UUID and an optional human-readable descriptive label). The assumtion that their values come from **a sensor system that generates tracks** — is **confirmed** by both the schema and the documentation PDFs.

---

## Scope of Investigation


| Path | Contents |
|------|----------|
| `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd` | Primary UCI message schema (v006.0.ASK) |
| `Schema/A-GRA_SecurityMarkings_v6_0_a.xsd` | Security markings schema |
| `Documentation/` | 7 interface volume PDFs (C2, ICD, MA, MD, MP, MS, Peer, VI) |
| Root | `ASK 6.0a Start Here Guide.pdf` |

All files were searched: the XSD schema was grepped directly; all 8 PDFs were extracted to text with `pdftotext` and then grepped.

---

## 1. `system_id` → `SystemID` (Type `SystemID_Type`)

### Schema Definition

**File:** `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd`, line 112787

```xml
<xs:complexType name="SystemID_Type" uci:version="000.000.000.000">
    <xs:annotation>
        <xs:documentation>The ID type that refers to an aggregation of Subsystems.</xs:documentation>
    </xs:annotation>
    <xs:complexContent>
        <xs:extension base="uci:ID_Type" />
    </xs:complexContent>
</xs:complexType>
```

### Base Type: `ID_Type`

**File:** `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd`, line 41700

```xml
<xs:complexType name="ID_Type" uci:version="000.000.000.000">
    <xs:annotation>
        <xs:documentation>The common base type for all UCI IDs. All extension types should end in "ID_Type".</xs:documentation>
    </xs:annotation>
    <xs:sequence>
        <xs:element name="UUID" type="uci:UniversallyUniqueIdentifierType">
            <xs:annotation>
                <xs:documentation>Universally Unique Identifier (UUID). A UUID is a 128-bit number (32 hexadecimal digits, 16 bytes) that is conformant to a version and variant of IETF RFC 4122.</xs:documentation>
            </xs:annotation>
        </xs:element>
        <xs:element name="DescriptiveLabel" type="uci:VisibleString256Type" minOccurs="0">
            <xs:annotation>
                <xs:documentation>Human readable text label with content meaningful to human operators.</xs:documentation>
            </xs:annotation>
        </xs:element>
    </xs:sequence>
</xs:complexType>
```

So a `SystemID` carries:
1. **`UUID`** — a RFC 4122 conformant universally unique identifier
2. **`DescriptiveLabel`** (optional) — human-readable text for operators

### Usage in Schema

`SystemID` (type `SystemID_Type`) appears 60+ times across the schema, including as:
- `SourceSystemID`, `DestinationSystemID`, `ControllingSystemID`, `ControllerSystemID`, `ReportingSystemID`, `AllocatedToSystemID`, `ReferenceSystemID`, `ApproverSystemID`, `ApproveeSystemID`, `ProtectedEntityID`, etc.

### Usage in Documentation

The MS L1 Interface Volume documents a concrete example: a SystemID of `3b39Bb83ee-d994-5f64-ae73-e62df8ec3c72` becomes a DDS topic name of `id_3b39Bb83ee_d994_5f64_ae73_e62df8ec3c72`. This confirms SystemID values are UUIDs used for topic routing and pub/sub messaging.

---

## 2. `capability_id` → `CapabilityID` (Type `CapabilityID_Type`)

### Schema Definition

**File:** `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd`, line 14346

```xml
<xs:complexType name="CapabilityID_Type" uci:version="000.000.000.000">
    <xs:annotation>
        <xs:documentation>The ID type that refers to a commandable and/or autonomous Capability of a Subsystem.</xs:documentation>
    </xs:annotation>
    <xs:complexContent>
        <xs:extension base="uci:ID_Type" />
    </xs:complexContent>
</xs:complexType>
```

Like `SystemID_Type`, `CapabilityID_Type` extends `ID_Type`, so it also carries a `UUID` and optional `DescriptiveLabel`.

### Related Type: `SubsystemID_Type`

**File:** `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd`, line 110854

```xml
<xs:complexType name="SubsystemID_Type" uci:version="000.000.000.000">
    <xs:annotation>
        <xs:documentation>The ID type that refers to an aggregation of Capabilities.</xs:documentation>
    </xs:annotation>
    <xs:complexContent>
        <xs:extension base="uci:ID_Type" />
    </xs:complexContent>
</xs:complexType>
```

This establishes the hierarchy: **System → Subsystem → Capability**.

### Usage in Schema

`CapabilityID` appears 100+ times, including variants:
- `CapabilityInstanceID`, `CapabilityDependencyID`, `ContributorCapabilityID`, `DetectorCapabilityID`, `GatewayCapabilityID`, `CommCapabilityID`, `DigitalPayload_ID`, `SupportCapabilityID`, etc.

---

## 3. Sensor Systems and Tracks

### Schema-Level Evidence

The type `MeasurementSourceIdentityType` (line 66298) directly ties `SystemID` and `CapabilityID` to sensor-generated measurements:

```xml
<xs:complexType name="MeasurementSourceIdentityType" uci:version="002.001.001.000">
    <xs:sequence>
        <xs:element name="CapabilityID" type="uci:CapabilityID_Type" minOccurs="0">
            <!-- "Indicates a Capability that generated the measurements." -->
        </xs:element>
        <xs:element name="TaskID" type="uci:TaskID_Type" minOccurs="0"/>
        <xs:element name="ActivityID" type="uci:ActivityID_Type" minOccurs="0"/>
        <xs:element name="SystemID" type="uci:SystemID_Type" minOccurs="0">
            <!-- "ID of the System that generated the measurements." -->
        </xs:element>
        <xs:element name="SubsystemID" type="uci:SubsystemID_Type" minOccurs="0">
            <!-- "ID of the Subsystem that generated the measurements." -->
        </xs:element>
        <xs:element name="SensorSystemIdentity" type="xs:unsignedShort" minOccurs="0">
            <!-- "The unique identifier for the sensor system producing the measurements." -->
        </xs:element>
    </xs:sequence>
</xs:complexType>
```

The related type `SourceKinematicsElementIdentityType` (line 107385) defines:
- `SystemID` — "ID of the System which is associated with the sibling kinematics."
- `SubsystemID` — "ID of the Subsystem which is associated with the sibling kinematics."
- `ComponentID` — "ID of the component which is associated with the sibling kinematics."

The `ObservationMeasurementSourceType` (line 70969) includes `SourceIdentity` (of type `MeasurementSourceIdentityType`) and a `SourceType` enum indicating "the category of sensor that generated the MeasurementReport."

### Documentation-Level Evidence

#### MS L1 Interface Volume — "Distribute Single Sensor Track Data to Perform Local Fusion" (pp. 54–55)

> "This interaction describes how MS provides tracks from onboard sensors to MA to enable local fusion. This is accomplished via the **EntityMT** message, which contains the kinematic data of the track. Additionally, the **ObservationMeasurementReportMT** message contains the sensor data that is used to generate the track or entity. Track data sources include both onboard cooperative sensors (e.g., Mode 5 IFF, ADS-B) and onboard non-cooperative sensors."

The document identifies two paths within `ObservationMeasurementReportMT → MeasurementSource`:

1. `ElementDetails → SourceElementIdentifier → SystemID + SubsystemID` — "These messages provide the system/subsystem that's the source of the OMR's kinematics."
2. `SourceIdentity → CapabilityID + SystemID + SubsystemID` — "These messages provide the system/subsystem and capability for the source of identity in the OMR (it's expected this will usually be the same system/subsystem as the kinematics)."

#### Peer L1 Interface Volume — "Distribute Sensor Track Data to Facilitate Package Fusion and Update COP" (pp. 81–82)

> "Un-fused tracks, that is, tracks that have not yet been fused together, are the result of an onboard sensor tracker associating a series of observation measurement reports (OMRs) and linking them to an entity... Once these sensor trackers have produced a set of entities, they may then be run through the sensor fusion engine, which will ingest all available entities and associate, fuse, and/or deconflict them. The output is what is called a fused entity (or track)."

Both volumes confirm that **SystemID** and **CapabilityID** values in observation/measurement reports identify the sensor system and sensor capability that produced the track-generating data.

---

## 4. Summary Table

| Query term | Actual schema name | Type name | Definition from schema | Base type | Sensor/track context |
|------------|-------------------|-----------|------------------------|-----------|----------------------|
| `system_id` | `SystemID` | `SystemID_Type` | "The ID type that refers to an aggregation of Subsystems" | `ID_Type` | ID of the System that generated sensor measurements / track kinematics |
| `capability_id` | `CapabilityID` | `CapabilityID_Type` | "The ID type that refers to a commandable and/or autonomous Capability of a Subsystem" | `ID_Type` | ID of the sensor Capability that generated the measurements |

**Base type `ID_Type`** provides:
- `UUID` (required, RFC 4122)
- `DescriptiveLabel` (optional, human-readable)

**Related identifier in hierarchy:** `SubsystemID` (type `SubsystemID_Type`) — "The ID type that refers to an aggregation of Capabilities."

---

## 5. Conclusion

1. The project uses **camelCase**: `SystemID` and `CapabilityID`.
2. **Both are defined** in `Schema/A-GRA_MessageDefinitions_v6_0_a.xsd` as complex types extending the common `ID_Type` base (UUID + optional DescriptiveLabel).
3. **Your assumption is confirmed.** In the sensor/track use case, `SystemID` and `CapabilityID` are populated by the sensor system that generates tracks. They appear inside `ObservationMeasurementReportMT → MeasurementSource`, where they identify which system and which capability onboard a platform produced the observation measurement reports (OMRs) that are associated into tracks (entities). The documentation explicitly describes this in the "Distribute Single Sensor Track Data" and "Distribute Sensor Track Data" interaction sections.
