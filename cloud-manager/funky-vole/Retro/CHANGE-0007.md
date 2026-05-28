cloud-manager-api:src/Models/CloudManager.DTO/Models/Host.cs — added bool IsController
cloud-manager-api:src/Services/CloudManager.Data.Services/Interface/IHostService.cs — added ResolveControllerHostIdAsync
cloud-manager-api:src/Services/CloudManager.Data.Services/HostService.cs — ResolveControllerHostIdAsync impl: first IsController=true host by Name; throws "no controller host configured" if zero
