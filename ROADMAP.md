# Jetlag Roadmap

## Context

Jetlag is a mature OpenShift bare-metal deployment automation tool with 29 open issues, active development (30+ PRs merged in recent months), and growing CI/self-scheduling use cases. This roadmap organizes all 29 open GitHub issues into prioritized themes based on impact, effort, and dependencies.

---

## Priority 1: Reliability & Correctness

Bugs and fixes that directly impact deployments today.

| Issue | Title | Effort |
|-------|-------|--------|
| [#782](https://github.com/redhat-performance/jetlag/issues/782) | Fix recursive call pattern (maximum recursion depth exceeded) | Medium |
| [#765](https://github.com/redhat-performance/jetlag/issues/765) | assisted-service pod doesn't pick up new config between IPv4/IPv6 deployments | Small |
| [#760](https://github.com/redhat-performance/jetlag/issues/760) | Fix disconnected SNO to apply proper IDMS and catalogsource | Small |
| [#764](https://github.com/redhat-performance/jetlag/issues/764) | Hypervisor disk2: use UUID instead of device name for resilience | Small |
| [#490](https://github.com/redhat-performance/jetlag/issues/490) | Generate ISO files according to inventory `boot_iso` name | Small |
| [#669](https://github.com/redhat-performance/jetlag/issues/669) | mno-scale-out assumes OCP 4.17+ (add version check or fallback) | Small |
| [#483](https://github.com/redhat-performance/jetlag/issues/483) | Discovery task relies on inventory count vs actual boot count | Medium |

## Priority 2: Input Validation & Error Messages

Prevent misconfigurations from reaching deployment. All are small, high-value changes.

| Issue | Title | Effort |
|-------|-------|--------|
| [#666](https://github.com/redhat-performance/jetlag/issues/666) | Validate `bastion_lab_interface` exists on bastion during setup | Small |
| [#628](https://github.com/redhat-performance/jetlag/issues/628) | Validate non-existent cloud name with clear error message | Small |
| [#686](https://github.com/redhat-performance/jetlag/issues/686) | VMNO: User variable for enabling/disabling secondary disk | Small |
| [#101](https://github.com/redhat-performance/jetlag/issues/101) | Check IBMcloud IPMI privilege level (admin vs operator) | Small |

## Priority 3: Hardware Support & Network Interface Discovery

Critical for expanding lab coverage and reducing manual configuration failures.

| Issue | Title | Effort |
|-------|-------|--------|
| [#752](https://github.com/redhat-performance/jetlag/issues/752) | Add Dell R7725 support + fix Redfish API for newer iDRAC firmware | Large |
| [#690](https://github.com/redhat-performance/jetlag/issues/690) | Handle same server model with different interface naming | Large |
| [#751](https://github.com/redhat-performance/jetlag/issues/751) | Scale lab r630 with `enp129s0f0` instead of `enp3s0f0` | Medium |
| [#637](https://github.com/redhat-performance/jetlag/issues/637) | Outgrown create-inventory.yml and lab.yml for hardware layout | Large |

**Note**: #690 and #751 are related - both stem from interface naming inconsistency within the same hardware model. A MAC-address-based or runtime-discovery approach would solve both. PR #763 (merged) already started using MAC addresses for node-config conditions.

## Priority 4: CI/CD & Code Quality

| Issue | Title | Effort |
|-------|-------|--------|
| [#492](https://github.com/redhat-performance/jetlag/issues/492) | Apply ansible-lint default rules (1391 violations, ~343 with autofix) | Medium |
| [#695](https://github.com/redhat-performance/jetlag/issues/695) | Option to disable permanent networking setup in bastion (CI use case) | Medium |

**Note**: `.github/workflows/ansible-lint.yml` exists but the project has 1391 violations. Starting with `ansible-lint --fix=fqcn` would auto-fix 589+ FQCN violations as a quick win.

## Priority 5: Deployment Features

New capabilities requested by users.

| Issue | Title | Effort |
|-------|-------|--------|
| [#504](https://github.com/redhat-performance/jetlag/issues/504) | Enable optional day-2 operators via Assisted Installer | Medium |
| [#547](https://github.com/redhat-performance/jetlag/issues/547) | Add /etc/hosts entries for BM node hostnames | Small |
| [#577](https://github.com/redhat-performance/jetlag/issues/577) | Automatically clean disks on deployment | Medium |
| [#358](https://github.com/redhat-performance/jetlag/issues/358) | Define user/password for cluster node login via IPMI console | Medium |
| [#250](https://github.com/redhat-performance/jetlag/issues/250) | HAProxy for connected clusters to facilitate easier GUI access | Medium |

## Priority 6: Documentation

| Issue | Title | Effort |
|-------|-------|--------|
| [#785](https://github.com/redhat-performance/jetlag/issues/785) | Create Jetlag roles documentation (variables, flow, results) | Large |
| [#636](https://github.com/redhat-performance/jetlag/issues/636) | Hybrid cluster documentation / quickstart | Medium |
| [#324](https://github.com/redhat-performance/jetlag/issues/324) | Document mounting extra disk (nvme) on /var/lib/containers for bastion | Small |

## Priority 7: Architecture & Refactoring

| Issue | Title | Effort |
|-------|-------|--------|
| [#321](https://github.com/redhat-performance/jetlag/issues/321) | Remove dnsmasq, make coredns the only DNS solution | Medium |
| [#654](https://github.com/redhat-performance/jetlag/issues/654) | Finish implementing SNO + hypervisor inventory | Medium |

## Priority 8: Security

| Issue | Title | Effort |
|-------|-------|--------|
| [#647](https://github.com/redhat-performance/jetlag/issues/647) | Remove internal hostnames from public GitHub | Medium |

## Priority 9: IBMcloud

| Issue | Title | Effort |
|-------|-------|--------|
| [#127](https://github.com/redhat-performance/jetlag/issues/127) | Research 2nd private subnet for IBMcloud MNO clusters | Medium |

---

## Suggested Implementation Phases

### Phase 1: Quick Wins & Bug Fixes
- Fix bugs: #760, #765, #764, #490, #669
- Add validation: #666, #628, #686
- Security: #647
- Small features: #547

### Phase 2: Quality & Stability
- Ansible-lint cleanup: #492
- Fix recursive patterns: #782
- Discovery count fix: #483
- Remove dnsmasq: #321
- CI networking: #695

### Phase 3: Hardware & Interface Discovery
- Interface naming: #690, #751
- Dell R7725: #752
- Inventory modernization: #637

### Phase 4: Features & Documentation
- Optional operators: #504
- HAProxy: #250
- Node access: #358
- Disk cleaning: #577
- Roles documentation: #785
- Hybrid docs: #636
- SNO+HV inventory: #654
- Extra disk docs: #324
- IBMcloud subnet: #127
- IBMcloud IPMI check: #101

---

## Summary
- **29 open issues** organized into 9 priority categories
- **Phase 1** covers 10 small/quick-win items
- **Phase 2** covers stability and code quality (5 items)
- **Phase 3** covers 4 hardware/interface items (largest effort)
- **Phase 4** covers remaining features, docs, and architecture (10 items)
- **Good first issues**: #504, #358, #324, #101
