cloud-manager-api:src/Services/CloudManager.Vault.Client/CloudManager.Vault.Client.csproj — new classlib project, NuGet refs (Microsoft.Extensions.Http/Logging/Configuration/DI Abstractions), added to solution
cloud-manager-api:src/Services/CloudManager.Vault.Client/IVaultClient.cs — interface, VaultPolicy/VaultChildToken records, VaultException
cloud-manager-api:src/Services/CloudManager.Vault.Client/VaultClient.cs — HTTP impl: WritePolicyAsync, DeletePolicyAsync (404 ok), CreateChildTokenAsync, RevokeTokenAsync (204/400 idempotent ok); reads Vault:Address + Vault:Token via IConfiguration
cloud-manager-api:src/Services/CloudManager.Vault.Client/ConfigureServices.cs — DI helper: services.AddHttpClient<IVaultClient, VaultClient>
cloud-manager-api:src/Services/CloudManager.Vault.Client/RedactingLoggerProvider.cs — ILoggerProvider wrapper that runs hvs\.[A-Za-z0-9_\-]+ → hvs.<REDACTED> regex on every formatted message
cloud-manager-api:src/CloudManager.API/Program.cs — register Vault.Client DI; bind VAULT_ADDR/VAULT_TOKEN → Vault:Address/Vault:Token config; wrap every registered ILoggerProvider descriptor with RedactingLoggerProvider
cloud-manager-api:src/CloudManager.API/CloudManager.API.csproj — added project reference to CloudManager.Vault.Client
cloud-manager-api:src/Services/CloudManager.Data.Services/CloudManager.Data.Services.csproj — added project reference to CloudManager.Vault.Client (for PlaybookRunService consumption in Phase 3)
