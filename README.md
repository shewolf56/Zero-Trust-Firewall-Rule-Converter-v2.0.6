ZERO TRUST FIREWALL RULE CONVERTER v2.0.6
====================================
Universal staged Windows Firewall migration with application/service discovery,
SHA-256 executable trust, complete policy backup, selective source-rule removal,
and replacement rules using the ZT- prefix only.

IMPORTANT: THE MIGRATION IS A THREE-STAGE WORKFLOW
-------------------------------------------------
Run these options in order. Do not skip the backup stage.

[1] BACK UP EXISTING RULES
    - Exports the complete Windows Firewall policy to Backups\*.wfw.
    - Calculates SHA-256 for the WFW backup and writes a sidecar checksum.
    - Discovers application/service rules in the local PersistentStore.
    - Captures each candidate's exact supported rule filters and metadata to a
      versioned JSON migration plan in Backups\ZT-MigrationPlan_*.json.
    - Captures Program, Package and Service identities and resolves file-backed
      executables/service binaries without hard-coded app installation paths.
    - Calculates SHA-256 for each resolvable executable or service binary.
    - First-seen hashes require the exact confirmation:
          ENROLL NEW SHA256 HASHES
      Hash baselines are not committed until the corresponding new ZT rule verifies.
    - Does not modify or delete any firewall rule.

[2] DELETE OLD APPLICATION/SERVICE RULES
    - Loads the saved option-1 migration plan and verifies the WFW backup SHA-256.
    - Rechecks each live source rule against the saved filter fingerprint.
    - Rehashes every executable immediately before deletion.
    - Offers a batch source-rule approval sub-menu:
          [1] APPROVE ALL RULE
          [2] APPROVE ONLY NEW RULE
          [0] SKIP APPROVAL
      This is a policy-scope approval, not a bypass for missing or changed hashes.
    - Exports a second immediate pre-delete WFW backup.
    - Requires the exact confirmation:
          DELETE DISCOVERED APP SERVICE RULES
    - Deletes only application/service rules that have a resolvable file path,
      a 64-character current SHA-256, a valid saved filter snapshot, and all required
      approvals. The current rule must still match its captured fingerprint.
    - Preserves unsupported, ambiguous, unhashable, changed-hash, unapproved and
      non-app/service Windows rules. These are listed in the conversion report.
    - Saves each deletion result into the migration plan so the process can resume.

[3] IMPLEMENT NEW ZT RULES
    - Loads the exact stage-1 snapshot and verifies the recorded backup checksum.
    - Requires the exact confirmation:
          CREATE ZT RULES
    - Recreates each deleted rule from its captured filters, not from a live source
      rule (the source has already been deleted by stage 2).
    - Verifies the new rule filters and identity against the saved fingerprint.
    - Recalculates the executable hash immediately before creation.
    - Writes SHA256=<hash> in each new ZT rule Description (audit-only), and updates
      Config\Trusted-Executable-SHA256.csv and Config\ZT-Rule-Executable-SHA256.csv
      only after the corresponding ZT rule verifies.
    - Can resume after interruption by reading statuses from the saved plan.

RULE NAMING
-----------
All newly managed rules use:
    Name       : ZT-<24-character deterministic identifier>
    DisplayName: ZT- <original display name>
    Group      : ZeroTrust - Managed Rules
No new rule uses the old ZTWF_AUTO_ or "ZT Auto -" naming style.
Legacy ZTWF_, ZTWF_AUTO_ and WSFM_ rules are treated as old source rules and require
batch review/approval before they can be migrated into the new ZT- family.

WHAT IS NOT DELETED
-------------------
The delete stage is intentionally selective. It does not indiscriminately remove
all Windows firewall rules. Rules without a Program, Package or Service identity,
unsupported filter combinations, unresolved system/service binaries, changed hashes,
source approval holds, and conflicting target names are preserved. This protects
Windows built-in networking, system policy, and rules the importer cannot recreate.
The final counts and exact names are written to Reports\Existing-Rules-Conversion_*.csv.

SHA-256 CONTROLS
----------------
1. Calculates SHA-256 for each resolvable app executable or service-specific binary.
2. Requires explicit first-seen hash enrollment during option 1.
3. Stores trusted file hashes persistently after a verified ZT rule is created.
4. Recalculates and compares current hashes during backup discovery, deletion and
   ZT-rule creation.
5. Holds changed executable hashes for exact Path + current SHA-256 + Reason approval.
6. Writes the hash into each ZT rule Description and a per-rule CSV manifest.
7. Produces full executable inventory and integrity audit CSV reports.

SHA-256 is an audit/deployment integrity control. Windows Defender Firewall does
not natively enforce file hashes at connection time. A Description hash is not a
firewall enforcement condition, publisher signature, malware scan or guarantee that
a file is safe.

RESTORE / ROLLBACK
------------------
If stage 2 has deleted old rules and stage 3 fails, use menu option [7] to import the
pre-delete .wfw backup. Restore replaces the complete local Windows Firewall policy
and can overwrite changes made after that backup. Select the matching backup from
Backups\ and type RESTORE when prompted. Prefer the latest
WindowsFirewall_<timestamp>.wfw created immediately before deletion.

CORE MENU
---------
MIGRATION - RUN IN ORDER
[1] Back up existing rules and create the migration plan
[2] Delete approved, hash-ready old app/service rules
[3] Implement replacement ZT- rules from the saved snapshot

TOOLS
[4] Preview application/service rules (read-only)
[5] Audit ZT- rules, profiles and executable SHA-256 integrity
[6] Export an additional full firewall backup
[7] Restore full firewall policy from backup
[8] Remove only rules owned by this manager (Name ZT- + exact managed Group)
[9] Validate packaged PowerShell scripts and configuration templates
[10] Open executable baseline / approval files
[0] Exit

BATCH APPROVAL SUB-MENU (OPTION 2)
---------------------------------
[1] APPROVE ALL RULE
    Approves all source-rule review candidates with valid file hash/path and no
    target collision, in alphabetical rule-name order.
[2] APPROVE ONLY NEW RULE
    Approves only candidates whose corresponding ZT- rule does not already exist.
[0] SKIP APPROVAL
    Leaves review candidates untouched. This does not approve hash changes or
    missing executable paths.
Every approval is recorded against source Name + exact filter fingerprint, with
sequence, timestamp and original review reason. Later source-filter edits invalidate
old approvals.

FILES / FOLDERS
---------------
Config\Trusted-Executable-SHA256.csv       persistent executable trust baseline
Config\Approved-Executable-SHA256.csv      exact changed-hash approval ledger
Config\ZT-Rule-Executable-SHA256.csv       per-ZT-rule hash mapping
Config\Approved-Source-Rules.csv           source filter fingerprint approvals
Config\Current-Migration-Plan.txt          pointer to the active staged plan
Backups\*.wfw                              complete Windows Firewall backups
Backups\ZT-MigrationPlan_*.json            exact app/service source snapshots
Reports\Executable-SHA256_*.csv            discovered executable hashes
Reports\Executable-Integrity-Audit_*.csv   baseline comparison and integrity status
Reports\Existing-Rules-Conversion_*.csv    conversion/deletion/creation results
Logs\                                      transcripts

REQUIREMENTS
------------
- Windows 10/11 or compatible Windows Server with NetSecurity cmdlets.
- Windows PowerShell 5.1.
- Administrator rights.
- No third-party dependencies.

Run menu option [9] Validate first. Then run [1], inspect the backup and migration
reports, [2], then [3]. Keep the whole folder (especially Config and Backups) together.
Do not delete the migration-plan JSON or the .wfw backup until the migration has
been verified and you no longer need rollback. Do not start a new option-1 plan while
any old source is marked DELETED_WAITING_ZT_CREATE or ZT_CREATE_ERROR; complete
option 3 or restore the recorded .wfw backup first.

RELEASE
-------
Version: 2.0.6
Replaces v2.0.5's one-step clone-with-source-preservation workflow with a staged,
backup-first migration and strict ZT- naming. Windows PowerShell 5.1 runtime and
live firewall deployment testing must be completed on the target computer.
