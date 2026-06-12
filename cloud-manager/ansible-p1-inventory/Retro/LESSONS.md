# Lessons: cloud-manager/ansible-p1-inventory

1. **Referenced contracts file missing.** Exec.md/init.md/PLAN.md all cite
   `workflow-plans/master-plans/ANSIBLE-EPIC-CONTRACTS.md` (§1, §3, §4, §6, §11), but the file
   does not exist — only `ANSIBLE-EPIC.md` does, with different section numbering. PLAN.md's
   own Design section carried everything needed (entity/prefix table, routes, event types), so
   execution proceeded against it. For P2–P9: either write the contracts file or fix the
   references before the next exec scaffold is generated.

2. **Wire-shape convention vs plan example.** The C-INV example used `publicId` as the JSON
   field name, but every existing CRUD DTO in the repo maps the public id into `id`. Followed
   the repo convention (binding per init.md) and recorded the deviation in PLAN.md. Epic plans
   should quote a real repo response shape rather than sketching one.

3. **Local API testing on the deploy box requires stopping the live service.** `Program.cs`
   hardcodes `UseUrls(...:5250)`, so a second instance cannot bind beside the deployed one.
   Testing was done in short stop/test/restore windows against a scratch DB (`clouddb_test`,
   created from the full migration chain, dropped afterwards). A `--urls`-respecting Program.cs
   (or env-var port override) would remove the downtime windows for future plans.

4. **AMQP config comes from appsettings.json, not env vars.** The API reads RabbitMQ creds from
   the `AMQP` section of `appsettings.json` resolved against the *current working directory*
   (env vars are only startup validation). Launching the DLL from the repo root fails with
   `ACCESS_REFUSED` even with correct env creds; launch from `src/CloudManager.API/`.

5. **Membership events fold into host_attached/host_detached.** The 9 contract event types have
   no member_added/member_removed; membership mutations record `host_attached`/`host_detached`
   with `scope:"group"` payloads, and group/host renames record `updated` with an
   `entity` discriminator. Worth ratifying in the contracts file for P5's consumers.

6. **EF query filter on Inventory hides soft-deleted timelines.** `GET {id}/event` 404s after
   inventory soft-delete (lookup goes through the filtered set), even though the `deleted` event
   row exists. Accepted for P1; P5/P9 may want `IgnoreQueryFilters` history access.
