# Home network: Self-hosting Docker containers with Caddy Reverse Proxy

_This project sets up n8n, a workflow automation tool, with Caddy as a reverse proxy using Docker Compose. Caddy handles routing and provides HTTPS, making n8n accessible via a custom domain. To host other services, see "Adding more services to Caddy"_.

## What it Does

* **n8n:** Runs the n8n workflow automation platform.
* **Caddy:** Acts as a reverse proxy, routing traffic from `n8n.contextual.local` to the n8n service.
* **HTTPS:** Caddy automatically provides HTTPS using Let's Encrypt.
* **Local Development:** Designed for local development, leveraging an `/etc/hosts` entry.
* **Persistent Data:** n8n data is stored in a Docker volume for persistence.

## Getting Started

1.  **Prerequisites:**
    * Docker and Docker Compose installed.
    * An entry in your `/etc/hosts` file: `127.0.0.1 n8n.contextual.local`.
2.  **Clone the Repository (if applicable):**
    ```bash
    git clone <your-repository-url>
    cd <your-repository-directory>
    ```
3.  **Create the `Caddyfile`:**
    * Ensure you have a `Caddyfile` in the same directory as `docker-compose.yml` with the following content:
        ```caddyfile
        n8n.contextual.local {
            reverse_proxy n8n:5678
        }
        ```
4.  **Start the Services:**
    ```bash
    docker-compose up -d
    ```
5.  **Access n8n:**
    * Open your web browser and go to `https://n8n.contextual.local`.

## HOWTO

### Accessing n8n Logs

1.  **View n8n Logs:**
    ```bash
    docker-compose logs n8n
    ```

### Accessing Caddy Logs

1.  **View Caddy Logs:**
    ```bash
    docker-compose logs caddy
    ```

### Stopping the Services

1.  **Stop the Services:**
    ```bash
    docker-compose down
    ```

### Adding more services to Caddy

1.  **Edit the Caddyfile:**
    * Add a new block to the Caddyfile with the new domain, and the new service name and port.
2.  **Add the new service to the docker-compose.yml file:**
    * Add the new service to the docker-compose.yml file.
3.  **Restart the Services:**
    ```bash
    docker-compose down && docker-compose up -d
    ```


