# Knowledge Route — Cloud, Account and Devices

Read: `backend/cloud/README.md` → U-02/U-03/U-18 → Stage04 STATUS → candidate AUTH_DEVICES_SECRETS → EXECUTION_MODEL for runner → DATA_OWNERSHIP before moving any data class to cloud.

Current truth: local SQLite/device/Keychain foundation works. Supabase is selected only as a future small cloud component. Exact cloud data, Auth/account/profile/callback/session custody/sync authority remain unresolved in audited commit. No Supabase provisioning/schema/RLS/Auth/account UI/sync/cloud backup is implemented by Stage04. Proposed primary + closed recovery is not replication/automatic failover. Visible Account popover is not an account system.

Second/Home Mac is Owner intent/proposal; enrollment/identity/transport/leases/grants/offline semantics are not built and Settings runner control is disabled.

Post-commit Owner input dated 2026-09-14 wants future sign-in choices Google/GitHub/Apple plus Apple credential-saving UX. Record as `OWNER_STATED_POST_COMMIT`, not audited implementation.