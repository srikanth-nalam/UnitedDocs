
# Low-Level Design (LLD): Server Build Request Refactoring

## 1. Catalog Item Name
**Server Build Request**  
**Table**: `sc_req_item` (via `sc_cat_item` reference)

---

## 2. Field Mapping & Client Logic

| UI Field Label               | Variable Name           | Source Table / Logic                     | Logic Details / Notes                                                                 |
|-----------------------------|--------------------------|-------------------------------------------|----------------------------------------------------------------------------------------|
| **Data Center Network Site**| `u_network_site`         | `u_nmp_network_site_info`                | Reference qualifier: `u_automation=true^active=true^function=data_center_subsite`     |
| **Type of Server**          | `u_server_type`          | Static Choice / Table (`u_vm_type`)      | Filtered via `u_server_classification`                                                |
| **Operating System**        | `u_os_version`           | `u_operating_system_versions`            | Conditional UI Policy using `u_os_type_mapping`                                        |
| **Backup Required**         | `u_backup_required`      | `u_backup_options`                       | Static table-driven dropdown                                                           |
| **User Access**             | `u_direct_access_users`  | `u_user_access_modes`                    | Static dropdown driven from new table                                                  |
| **Environment**             | `u_environment_level`    | `u_environment_levels`                   | Used to construct env string like `NDCNG-DEV-HSEC_OTHER`                               |
| **Security Zone**           | `u_security_zone`        | `u_security_zones`                       | Static zone values; used in string derivation                                          |

---

## 3. Key Scripts & Logic

### 3.1. Client Script: Dynamic Filtering
```javascript
function onChange(control, oldValue, newValue) {
  if (newValue === 'NDCNG' || newValue === 'TULNG') {
    g_form.setVisible('u_os_version', false);
  } else {
    g_form.setVisible('u_os_version', true);
  }
}
```

### 3.2. GlideAjax for Environment String
```javascript
function getEnvironmentString(dc, env, secZone) {
  return dc + '-' + env + '-' + secZone;
}
```

---

## 4. Suggested New Custom Tables

| Table Name                   | Purpose                                | Key Fields                            |
|-----------------------------|----------------------------------------|----------------------------------------|
| `u_os_type_mapping`         | Controls allowed OS by DC              | `u_data_center`, `u_os_type`, `u_allowed` |
| `u_security_zones`          | Holds static zone data                 | `u_zone_name`, `u_zone_code`           |
| `u_environment_levels`      | Holds levels (DEV, QA, PROD...)        | `u_level_name`, `u_code`               |
| `u_operating_system_versions`| Allowed OS and version data           | `u_os_family`, `u_version`, `u_status` |
| `u_backup_options`          | Central backup values                  | `u_option`, `u_description`            |
| `u_user_access_modes`       | Direct access modes                    | `u_mode_name`, `u_requires_approval`   |
| `u_vm_type`                 | Standalone, SQL, Cluster types         | `u_type_name`, `u_supported`           |
| `u_server_classification`   | Map server category with supported config | `u_class`, `u_vm_types_allowed`      |

---

## 5. Workflows

- Use **Flow Designer** as per Xanadu support & Yokohama Support
- Each catalog submission → launch server build orchestration flow

### Subflow Examples:
- `Validate OS per Data Center`
- `Call Ansible Tower Job Template`
- `Call vRO Orchestration`
- `Update CMDB record for build`
- `Send build summary email with dynamic env string`

---

## 6. Legacy Compatibility

- Maintain pre-existing `sys_id` in `sc_req_item` / `sc_task`
- Introduce a flag `u_legacy_format=true` to distinguish old vs new requests
- Create transformation map if importing legacy payloads into new structure

---

## 7. Performance Checks

- Servlet memory usage: 1090MB used / 1630MB allocated
- Instance: `xanadu-07-02-2024__patch5-05-21-2025` is Xandu-ready
- MID Server version validated

---

## 8. Next Action Plan

1. Finalize static table population
2. Build update set for:
   - Catalog item refactor
   - GlideAjax scripts
   - Flow Designer integrations
   - UI Policies / Client Scripts
3. Test both NG (NDCNG/TULNG) and legacy zones
4. UAT walkthrough with mockup screens
