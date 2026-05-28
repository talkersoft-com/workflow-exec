cloud-manager-api:src/Models/CloudManager.DTO/Models/PlaybookCollectionRequirement.cs — added CollectionNamespace + CollectionName denormalized fields
cloud-manager-api:src/Services/CloudManager.Data.Services/MappingProfiles/AnsibleCollectionProfile.cs — Ignore CollectionNamespace + CollectionName in PlaybookCollectionRequirement map (service populates)
cloud-manager-api:src/Services/CloudManager.Data.Services/Interface/IPlaybookCollectionRequirementService.cs — interface
cloud-manager-api:src/Services/CloudManager.Data.Services/PlaybookCollectionRequirementService.cs — ListByPlaybook/Create (409 on dup)/Delete impl
cloud-manager-api:src/CloudManager.API/Controllers/PlaybookCollectionRequirementsController.cs — GET /playbooks/{pid}/collection-requirements, POST /playbooks/{pid}/collection-requirements, DELETE /collection-requirements/{pcreqId}
cloud-manager-api:src/Services/CloudManager.Data.Services/ConfigureServices.cs — DI registration
