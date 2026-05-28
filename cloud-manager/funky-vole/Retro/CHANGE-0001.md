cloud-manager-api:src/Models/CloudManager.Entities/Models/Host.cs — added bool IsController
cloud-manager-api:src/Models/CloudManager.Entities/Models/PlaybookRun.cs — added string? VaultRunToken
cloud-manager-api:src/Models/CloudManager.Entities/CloudManagerDbContext.cs — Host binding gains is_controller (default false); PlaybookRun binding gains vault_run_token (varchar(256), nullable)
cloud-manager-api:src/Models/CloudManager.Entities/Migrations/20260528222630_AddVaultRunTokenAndControllerHost.cs — generated + appended backfill UPDATE bare_metal.hosts SET is_controller = true WHERE name = 'ubuntu-server'
