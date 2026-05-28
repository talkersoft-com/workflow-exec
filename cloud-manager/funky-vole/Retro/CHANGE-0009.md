cloud-manager-web:src/services/api.ts — playbookRequirements namespace: list/create/delete
cloud-manager-web:src/store/slices/playbookCollectionRequirementsSlice.ts — slice + 3 thunks (fetchForPlaybook, createRequirement, deleteRequirement)
cloud-manager-web:src/store/index.ts — register playbookCollectionRequirements reducer
cloud-manager-web:src/pages/PlaybookDetailPage.tsx — imports for requirements slice + Modal + Toast; controllerHost resolution via selectedControllerSlice + isController filter; fetchForPlaybook + fetchCollections + fetchHosts + fetchHostCollections wiring; "Required collections" section with table (collection, min, max, status badge, Remove); red unmet banner with Link to /ansible/collections; "+ Add requirement" modal with collection dropdown + version inputs
