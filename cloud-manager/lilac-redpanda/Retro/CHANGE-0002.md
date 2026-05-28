cloud-manager-api:src/Models/CloudManager.DTO/Models/AnsibleCollection.cs — new DTO
cloud-manager-api:src/Models/CloudManager.DTO/Models/HostAnsibleCollection.cs — new DTO (with denormalized collection namespace/name + install_path for worker)
cloud-manager-api:src/Models/CloudManager.DTO/Models/AnsibleCollectionInstallRun.cs — new DTO (with denormalized collection ns/name/install_path)
cloud-manager-api:src/Models/CloudManager.DTO/Models/PlaybookCollectionRequirement.cs — new DTO
cloud-manager-api:src/Services/CloudManager.Data.Services/MappingProfiles/AnsibleCollectionProfile.cs — AutoMapper profile mapping all 4 entities → DTOs (Id ← PublicId, Status.ToString, FK fields Ignore for service-side population)
cloud-manager-api:src/Services/CloudManager.Data.Services/Interface/{IAnsibleCollectionService,IHostCollectionService,ICollectionInstallRunService}.cs — service interfaces + CollectionInstallRunPatchRequest record
cloud-manager-api:src/Services/CloudManager.Data.Services/AnsibleCollectionService.cs — catalog list/get/create/delete with uniqueness + in-use guard
cloud-manager-api:src/Services/CloudManager.Data.Services/HostCollectionService.cs — list per host; RequestInstallAsync creates host_collections (Pending) + collection_install_runs (Queued) in one transaction; RequestReinstall delegates with force=true; RequestRemove sets Removing + creates removal run
cloud-manager-api:src/Services/CloudManager.Data.Services/CollectionInstallRunService.cs — GetByIdAsync + PatchFromWorkerAsync that updates run + host_collections row atomically on terminal status
cloud-manager-api:src/Services/CloudManager.Data.Services/ConfigureServices.cs — register 3 new services as transient
