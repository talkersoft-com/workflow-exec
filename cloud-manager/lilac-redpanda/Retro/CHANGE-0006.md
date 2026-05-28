vorch-service:vorch-service/messagesub/collection_install_sub.go — subscribes to collection-installs queue, manual ack, nack-with-redelivery on error
vorch-service:vorch-service/messagesub/collection_remove_sub.go — subscribes to collection-removes queue, same shape
vorch-service:vorch-service/handlers/collection_install_handler.go — fetch run → PATCH Running → exec ansible-galaxy collection install (with --force if requested) → parse `ansible-galaxy collection list --format json` to get version + install_path → PATCH Succeeded/Failed
vorch-service:vorch-service/handlers/collection_remove_handler.go — fetch run → PATCH Running → safety guard on install_path prefix /usr/share/ansible/collections/ansible_collections/ → os.RemoveAll → verify via `ansible-galaxy collection list` → PATCH Succeeded/Failed
vorch-service:vorch-service/app.go — registration of both subscribers
cloud-manager-api:scripts/vorch/install-vorch-service.py — ReadWritePaths widened to include /usr/share/ansible/collections
