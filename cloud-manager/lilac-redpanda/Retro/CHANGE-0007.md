cloud-manager-web:src/components/AnsibleSubNav/AnsibleSubNav.tsx — added 5th tab "Collections" → /ansible/collections
cloud-manager-web:src/App.tsx — imports + routes for AnsibleCollectionsPage and AnsibleCollectionDetailPage
cloud-manager-web:src/services/api.ts — ansibleCollections.* namespace: list/get/create/delete catalog; listForHost; requestInstall/Reinstall/Remove; getInstallRun
cloud-manager-web:src/store/slices/collectionsSlice.ts — catalog slice (fetchCollections, fetchCollection, createCollection, deleteCollection)
cloud-manager-web:src/store/slices/hostCollectionsSlice.ts — per-host install records slice (fetchHostCollections, requestInstall/Reinstall/Remove thunks)
cloud-manager-web:src/store/slices/collectionInstallRunsSlice.ts — install-run slice + pollUntilTerminal thunk (polls every 5s, dispatches fetchHostCollections each tick)
cloud-manager-web:src/store/index.ts — register 3 new reducers (collections, hostCollections, collectionInstallRuns)
cloud-manager-web:src/pages/AnsibleCollectionsPage.tsx — list page + modal "Add collection" + state-aware actions (Install / Reinstall+Remove / spinner)
cloud-manager-web:src/pages/AnsibleCollectionDetailPage.tsx — detail page showing install state on the controller host
