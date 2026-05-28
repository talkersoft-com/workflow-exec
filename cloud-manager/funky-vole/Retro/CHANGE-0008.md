cloud-manager-web:src/store/slices/selectedControllerSlice.ts — slice with localStorage persistence (key cm.selectedControllerHostId)
cloud-manager-web:src/store/index.ts — registered selectedController reducer
cloud-manager-web:src/types/index.ts — Host gains optional isController
cloud-manager-web:src/pages/AnsibleCollectionsPage.tsx — replaces hosts[0] with controllerHosts filter + selectedControllerId; renders Controller dropdown; defaults selection on first load
cloud-manager-web:src/pages/AnsibleCollectionDetailPage.tsx — same selector pattern; new "All controllers" summary table with status + version per controller
