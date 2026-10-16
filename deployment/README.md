# n8n Ubuntu / Docker reference

The included `docker-compose.example.yml` is a sanitised **illustrative starting point**, not a byte-for-byte export from the running Ubuntu VM. It uses a persistent volume and binds n8n to port 5678 for the private VirtualBox lab.

To try it in a separate test environment, copy `.env.example` to a new private `.env`, replace placeholder values, then run Docker Compose. Do **not** overwrite the running VM's existing `.env`, encryption key or persistent volume. Losing the original encryption key can make saved credentials unrecoverable.

Do not publish or expose the non-TLS HTTP deployment on the public internet. Use private network access and HTTPS/security hardening for real environments.
