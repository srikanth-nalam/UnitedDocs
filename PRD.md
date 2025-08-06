
# Next‑Generation Virtual Server Catalog – Product Requirements Document (PRD)

## Overview

This Product Requirements Document (PRD) consolidates all discussions between Tom Steines, Andrew Agos, Adam Henderson, Srikanth, and the wider team regarding the NextGen ServiceNow catalog form. It also incorporates insights from the legacy data schema (`current_snow_payload.json`) and applies ServiceNow best‑practice guidance for catalog design and maintainability. The objective is to deliver a modern, self‑service portal where users can request virtual servers and SQL Availability Group (AG) clusters with the flexibility to span data centres and support Oracle/Red Hat platforms.

## Goals

- **Enable a self‑service ServiceNow portal** for provisioning virtual servers across UAL’s data centres with minimal manual intervention.
- **Support standard virtual machines and SQL AG clusters** (multi‑region or single‑region) with built-in high availability and disaster recovery options.
- **Incorporate legacy features** from the existing payload (e.g., Oracle Linux template, server size mapping, DR classification) to avoid regression.
- **Follow ServiceNow best‑practice:** separate workflows for distinct items and avoid combining divergent paths in a single form.
- **Generate ServiceNow‑compatible JSON mock‑ups** for rapid prototyping and demonstration to stakeholders.


## Stakeholders

| Stakeholder | Role |
| :-- | :-- |
| Tom Steines | Business owner and sponsor for NextGen platform |
| Andrew Agos | Technical lead for SQL AG and automation |
| Srikanth N  | Service Now Dev |
| Adam Henderson | Developer responsible for building workflow logic and tests |
| Database Administrators (DBAs) | Provide guidance on SQL AG listener limitations |
| ServiceNow Admins | Implement catalog items, forms and UI policies |
| Automation Team | Provide orchestration logic and integration with ServiceNow |

## Server Role and Description

- Allow an **additional application‑specific role** (e.g., Primary SQL Node, Reporting, etc.) and a free‑form description of the server’s purpose.
- Incorporate the **domain enumeration** from the schema (ual.com, global.ual.com, etc.).


## Hardware and Storage

- Map legacy server size codes (e.g., `2_cpu_4_gig` → 2 vCPU, 4 GB RAM) to explicit hardware fields.


## Operating System and Template

- See requirements and mapping in feature matrix below.


## SQL AG Cluster Options

- When the enterprise server role is Windows Cluster Node or the application role indicates SQL AG, show additional fields:
    - `listenerRequired`: boolean to indicate whether a SQL AG listener is needed across regions (only possible within a single data centre due to DBA restrictions).
- Integrate with ServiceNow workflows for approvals and orchestrations.
- Provide a comments field for special instructions.


## Usability \& Security

- **Usability:** Forms must be intuitive; variable sets should be grouped logically; avoid unnecessary questions. Use UI policies to hide/show fields conditionally rather than client scripts.
- **Security:** Apply user criteria to restrict who can access each item; validate all fields server‑side.


## Requirements Matrix

| Feature | Purpose | Priority | Notes |
| :-- | :-- | :-- | :-- |
| Enterprise server role | Identify type (web, app, middleware, cluster, etc.) | High | Enum list from schema |
| Application key | Unique CMDB identifier (3 letters) | High | Uppercase, 3 char |
| DR classification | Specify disaster recovery tier | Medium | From legacy schema |
| Data centre \& environment | Determine location (NDC/TUL) \& stage (DEV/QA/PROD) | High | Drives zone logic |
| Security zone | Select allowed zone (HSEC, PCI, CORP, etc.) | High | Auto‑set based on location |
| Domain | Choose domain for AD membership | Medium | Enum from schema |
| Server quantity | Number of identical VMs | High | Range 1–100 |
| vCPU \& memory | Specify compute resources | High | Must be multiples of 2 |
| Additional disks | Optional storage volumes (up to six) | Medium | Integer GB values |
| Operating system type \& version | Choose OS family and version | High | Support Windows, Oracle, RHEL |
| Oracle Linux template | Provide pre‑built Oracle template option | Medium | Must support v9.x 64‑bit |
| Cluster size (2/4) | Define SQL cluster nodes | High | Only on SQL AG form |
| Multi‑region flag | Build stretched cluster across NDC/Tulsa | High | Only on SQL AG form |
| SQL listener requirement | Indicate need for a listener across regions | Medium | Dependent on DBA limitations |
| Deployment ID | Optional override for orchestration rerun | Low | Advanced use only |
| Exit flag | Boolean to trigger workflow exit/ rerun | Low | Hidden field for automation |
| Direct user/sudo access | Request direct access or sudo privileges | Medium | Inputs list of names |
| Sensitive data flags (PCI, PHI, PII, SOX) | Capture compliance requirements | High | Must confirm classifications |
| Comments/Attachments | Provide additional context | Low | Optional |

## Example JSON Schema

```json
{
  "formType": "standardVmRequest",
  "enterpriseServerRole": "Web Server",
  "applicationServerRole": "Primary Web Node",
  "serverRoleDescription": "Web tier for ServiceRadar application",
  "dataCenter": "NDCNG",
  "environmentLevel": "PROD",
  "applicationKey": "EXN",
  "drClassification": "DR-4",
  "securityZone": "HSEC",
  "domain": "global.ual.com",
  "serverHardware": {
    "serverQuantity": 2,
    "vCPU": 4,
    "memoryGB": 16
  },
  "serverStorage": {
    "additionalDisk1GB": 100,
    "additionalDisk2GB": 500
  },
  "operatingSystem": {
    "operatingSystemType": "Oracle Enterprise Linux",
    "operatingSystemVersion": "9.x"
  },
  "billingAutomation": true,
  "requiresDirectAccess": false,
  "sudoAccess": false,
  "dataClassification": {
    "pci": false,
    "phi": false,
    "pii": false,
    "sox": false,
    "other": false
  },
  "confirmDataClassification": true,
  "attachments": [],
  "comments": "Standard VM build for web tier."
}
```


### References

[Best Practices for Designing Service Catalog in ServiceNow | Emergys](https://www.emergys.com/blog/best-practices-for-designing-service-catalog-in-servicenow/)


