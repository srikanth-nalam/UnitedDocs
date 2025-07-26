
**High-Level Design (HLD) Document: Server Build Request Refactoring**

**Project Title:** Refactoring Server Build Request Catalog Item (Legacy to Best Practice Model)  
**Instance:** ServiceNow Vancouver Xandu Version.

**Primary Stakeholders:** Platform Admins, Cloud Ops, ServiceNow Developers, Requesters

---

**1. Objective:**  
Refactor the existing Server Build Request catalog item into a modular, reusable, and manageable structure based on ServiceNow Vancouver/Washington best practices. The goal is to maintain existing integrations and CMDB structure while aligning with modern standards.

**2. Key Goals:**
- Redesign Server Build form UI for ease of use and better validation.
- Retain and reuse existing CMDB (`u_nmp_network_site_info`, etc.) and custom tables.
- Integrate seamlessly with vRO workflows.
- Modularize logic based on form input combinations.
- Use Flow Designer where feasible for maintainability.
- Maintain backward compatibility for data migration.

**3. Scope:**
- Service Catalog > Maintain Items: Redesign "Server Build Request" item.
- Update Catalog Client Scripts, UI Policies, and Flow logic.
- Integrate with VRO and AAP playbooks (existing endpoints to remain).
- Update reference qualifiers and dependent dropdown behavior.
- Implement business rules or data policies for legacy support.
- Consider and support Vancouver build version: `xanadu-07-02-2024__patch5-05-21-2025`
- Utilize instance stats from `/stats.do` for resource assessment (e.g., servlet memory, active sessions).

**4. Not In Scope:**
- Changes to CMDB schema.
- Major refactoring of backend integrations.
- UI Pages or Portals overhaul.

**5. Technical Architecture:**
- **Catalog Item**: Server Build Request
- **Tables Involved**:
  - CMDB: `u_nmp_network_site_info`, `cmdb_ci_server`, `cmn_location`
  - Custom: `u_build_request`, `u_server_build_logs`
  - 
    - `u_os_type_mapping`: to manage allowed OS per data center (e.g., restrict SQL for NDCNG/TULNG)
    - `u_environment_levels`: reference table for available environment level values
    - `u_security_zones`: manage static security zone metadata
    - `u_operating_system_versions`: manage OS and version compatibility
    - `u_backup_options`: static options used in backup section
    - `u_user_access_modes`: manages access types for direct server access
    - `u_vm_type`: defines available VM types based on server classification
    - `u_server_classification`: maps logical groupings for logic enforcement
- **Workflow Engine**:
  - **Primary:** Flow Designer (Vancouver-compliant)
  - **Secondary (Fallback):** Legacy Workflow (existing logic to be encapsulated if needed)
- **External Integrations**:  
  - VMware vRealize Orchestrator (vRO) via MID server or REST

**6. Key Form Logic and Enhancements:**
| Field Group | Changes |
|------------|---------|
| Data Center Selection | Add `NDCNG`, `TULNG` with updated reference qualifiers in "Data Center Network Site" to reflect only automation=true, network_site_function=data_center, status=active |
| Environment String | Update client script to reflect full prefix (`NDCNG-DEV-HSEC_OTHER`) instead of defaulting to NDC/TUL |
| O/S VM Type Logic | Conditionally show only Standalone Linux/Windows for NDCNG/TULNG; hide SQL. Manage logic using `u_os_type_mapping` |
| Dependency Fields | Use UI Policies for mandatory/visibility; avoid DOM manipulation |
| Auto-populated Fields | Use GlideAjax or g_scratchpad as required for dynamic data loading |

**7. Data Migration Plan:**
- Use update set for capturing client script, UI policy, catalog item config
- Export old catalog item and fields for archival
- Retain legacy sys_ids using new flags or custom field to track pre/post change requests
- Backward compatibility preserved using data flags or migration scripts

**8. Reporting & SLA:**
- Retain SLA logic for server build duration
- Add reporting tags for NG vs Legacy DC types for analytics

**9. Risks & Mitigations:**
| Risk | Mitigation |
|------|------------|
| Existing integrations may break | Retain endpoint formats, test against staging |
| Legacy users confused by new fields | Provide tooltips/documentation inline |
| Large number of dependent catalog items | Incremental rollout with backwards compatibility |
| Instance memory overload during updates | Ensure max memory and active session counts (as seen in stats.do) are within operating thresholds |

**10. Next Steps:**
- Confirm catalog item sys_id and baseline logic
- Implement update set with refactored structure
- Validate in lower environment (Dev/Test)
- Export XML for final UAT + migration

---


