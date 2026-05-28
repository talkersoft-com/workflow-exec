cloud-manager-api:src/CloudManager.API/Controllers/AnsibleCollectionsController.cs — GET/list, POST/create, GET/{cid}, DELETE/{cid} under /api/v1/ansible/collections (4 endpoints)
cloud-manager-api:src/CloudManager.API/Controllers/HostCollectionsController.cs — GET list-for-host, POST request-install, POST reinstall, DELETE request-remove (4 endpoints, publishes AMQP after DB write)
cloud-manager-api:src/CloudManager.API/Controllers/CollectionInstallRunsController.cs — GET/{cid}, PATCH/{cid} (worker callback) (2 endpoints)
