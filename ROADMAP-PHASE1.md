# Jetlag Phase 1 Implementation Plan

## Overview
Implement Phase 1 (Quick Wins & Bug Fixes) of the Jetlag roadmap — 10 issues covering bugs, validation, security, and a small feature.

---

## Issue #760: Fix disconnected SNO IDMS and catalogsource

**Problem**: SNO `sno-post-cluster-install` doesn't apply the `openshift-marketplace` namespace for disconnected clusters when needed, while MNO does it correctly in `mno-post-cluster-install`.

**Analysis**: Comparing the two files:
- MNO (lines 102-115): Disables OperatorHub defaults, applies IDMS, applies catalogSource. Does NOT create marketplace NS.
- SNO (lines 74-94): Disables OperatorHub defaults, creates marketplace NS for OCP >= 4.15, applies IDMS, applies catalogSource.

The MNO is missing the marketplace NS creation for OCP >= 4.15 that SNO already has. The issue title says "fix SNO" but the SNO code actually has MORE logic. Need to ensure both are consistent:
1. Add `openshift-marketplace` namespace creation to `mno-post-cluster-install` for disconnected OCP >= 4.15 deployments
2. Ensure the marketplace namespace template exists in the MNO templates directory

**Files to modify**:
- `ansible/roles/mno-post-cluster-install/tasks/main.yml` — Add marketplace NS creation task before IDMS apply (matching SNO pattern)
- Copy or share the `openshift-marketplace-ns.yaml` template from SNO role to MNO role

---

## Issue #765: Assisted-service pod doesn't pick up new config

**Problem**: After IPv4 deployment, switching to IPv6 doesn't update the assisted-service pod config. The `onprem-environment` file gets re-templated but the pod/containers use `state: started` which doesn't restart already-running containers.

**Fix**: In `bastion-assisted-installer/tasks/main.yml`, ensure the pod and all containers are recreated when the config changes:
1. Add a `state: absent` step for the assisted-service pod before creating it, to force a clean restart
2. This ensures the new `onprem-environment` file is picked up by all containers

**Files to modify**:
- `ansible/roles/bastion-assisted-installer/tasks/main.yml` — Add pod removal step before pod creation (around line 77)

---

## Issue #764: Hypervisor disk2 UUID resilience

**Problem**: `hv-setup-disk2` mounts disk2 using device name (e.g., `/dev/sdb1`) which can change after reboots. Should use UUID for persistent mounting.

**Fix**: After formatting the partition, fetch its UUID via `blkid` and use it in the mount module with `UUID=` source.

**Files to modify**:
- `ansible/roles/hv-setup-disk2/tasks/main.yml` — After formatting, get UUID with `blkid`, then mount using `UUID={{ disk2_uuid }}` instead of device path

---

## Issue #490: Generate ISO files according to inventory boot_iso name

**Problem**: `generate-discovery-iso` always creates `discovery.iso` (default `iso_name`), but BYOL inventories can set `boot_iso` to a different name. The `boot-iso` role then can't find the file.

**Fix**: The `generate-discovery-iso` role already uses `iso_name` variable (default: `discovery`). The SNO deploy already overrides `iso_name` to the SNO hostname. For MNO, `iso_name` defaults to `discovery` and `boot_iso` in inventory is `discovery.iso`, so they match. The issue is specifically about BYOL scenarios where `boot_iso` might be set differently.

The cleanest fix: pass `iso_name` from the first node's `boot_iso` hostvar (stripped of `.iso` extension) when calling the `generate-discovery-iso` role in `mno-deploy.yml`.

**Files to modify**:
- `ansible/mno-deploy.yml` — Add `iso_name` var to `generate-discovery-iso` role invocation, derived from the first controlplane node's `boot_iso`

---

## Issue #669: mno-scale-out version check for OCP 4.17+

**Problem**: `ocp-scale-out` role uses `oc adm node-image create` which requires OCP 4.17+. No version check exists.

**Fix**: Add version check at the start of `ocp-scale-out/tasks/main.yml`. Get the OCP version from the running cluster and fail with a clear message if < 4.17.

**Files to modify**:
- `ansible/roles/ocp-scale-out/tasks/main.yml` — Add task at the top to check OCP version and fail if < 4.17

---

## Issue #666: Validate bastion_lab_interface exists on bastion

**Problem**: If `bastion_lab_interface` doesn't exist on the bastion, the deployment fails late with confusing errors.

**Fix**: Add a validation task in `validate-vars/tasks/main.yml` that checks the interface exists on the bastion using `ansible_facts`. This check should only run when `ansible_facts.interfaces` is available.

**Files to modify**:
- `ansible/roles/validate-vars/tasks/main.yml` — Add validation that `bastion_lab_interface` exists in `ansible_facts.interfaces`

---

## Issue #628: Validate non-existent cloud name

**Problem**: If `lab_cloud` references a non-existent cloud allocation, the inventory download fails with an unclear error.

**Fix**: Add validation in `create-inventory/tasks/main.yml` after downloading the inventory JSON. If the download fails or returns empty/invalid data, fail with a clear message about the cloud name being invalid.

**Files to modify**:
- `ansible/roles/create-inventory/tasks/main.yml` — Add error handling around the inventory download with a clear error message referencing `lab_cloud`

---

## Issue #686: VMNO user variable for disk2 enable/disable

**Problem**: VMNO sets `disk2_enable` automatically based on hardware type, but some machines of that type may not have a second disk (e.g., some r740xd machines only have one disk). Users need an override.

**Fix**: Add a `vmno_disk2_enable` variable (default: unset/auto) that when explicitly set to `false`, overrides the automatic disk2 detection.

**Files to modify**:
- `ansible/vars/all.sample.yml` — Add `vmno_disk2_enable` variable (commented out, with documentation)
- `ansible/roles/create-inventory/tasks/main.yml` — After the automatic disk2 detection block, add an override step that sets `disk2_enable: false` for all HV nodes when `vmno_disk2_enable` is explicitly `false`

---

## Issue #647: Remove internal hostnames from public GitHub

**Problem**: Internal Red Hat hostnames (e.g., `rdu2.scalelab.redhat.com`, `rdu3.labs.perfscale.redhat.com`) are hardcoded in `ansible/vars/lab.yml` and `scripts/self-sched-deploy/README.md`.

**Fix**: The `lab.yml` hostnames are functional configuration used by the tool — they can't simply be removed. The approach is:
1. Remove specific hostnames from documentation/README files where they're shown as examples
2. In `scripts/self-sched-deploy/README.md`, replace the specific hostname example with a generic placeholder

**Files to modify**:
- `scripts/self-sched-deploy/README.md` — Replace specific hostname examples with generic placeholders

**Note**: Fully parameterizing `lab.yml` would require significant refactoring and could break existing workflows. This is a partial fix addressing the documentation exposure. The `lab.yml` file itself contains operational data that's necessary for the tool to function.

---

## Issue #547: Add /etc/hosts entries for BM node hostnames

**Problem**: After deployment, users can't SSH to cluster nodes by hostname from the bastion — they must use IP addresses.

**Fix**: Add `/etc/hosts` entries for each controlplane and worker node on the bastion during post-cluster-install. The `create-ai-cluster` role already adds API/ingress entries; we need to add individual node entries too.

**Files to modify**:
- `ansible/roles/mno-post-cluster-install/tasks/main.yml` — Add a `blockinfile` task to add entries for each controlplane and worker node hostname to IP mapping
- `ansible/roles/sno-post-cluster-install/tasks/main.yml` — Add a similar task for the SNO node

---

## Implementation Order

1. **#666** — Validate bastion_lab_interface (small, self-contained)
2. **#628** — Validate cloud name (small, self-contained)
3. **#686** — VMNO disk2 override (small, one variable + condition)
4. **#764** — Disk2 UUID (small, task modification)
5. **#490** — ISO name from boot_iso (small, playbook change)
6. **#669** — Scale-out version check (small, add guard)
7. **#760** — Fix disconnected SNO/MNO IDMS (small, add missing task)
8. **#765** — Assisted-service pod restart (small, add removal step)
9. **#547** — /etc/hosts node entries (small, add task)
10. **#647** — Internal hostnames in docs (small, README edit)

Each issue will be implemented as a separate commit.
