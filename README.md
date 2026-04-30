# 📦 Woodpecker Docker Compose Stack

[![MIT License](https://img.shields.io/github/license/Vantasin/Woodpecker?style=flat-square)](LICENSE)
[![Woodpecker CI](https://img.shields.io/badge/Woodpecker%20CI-self--hosted-green?logo=drone&style=flat-square)](https://woodpecker-ci.org/)
[![ZFS](https://img.shields.io/badge/ZFS-OpenZFS-blue?style=flat-square)](https://openzfs.org/)

This repository contains a minimal and production-ready [Woodpecker CI](https://woodpecker-ci.org/) stack using Docker Compose.  
**Woodpecker CI** is a lightweight, self-hosted Continuous Integration (CI) and Continuous Deployment (CD) system, designed to automatically build, test, and deploy your code whenever you push changes to your source code repository.

---

## 📁 Directory Structure

```bash
tank/
├── docker/
│   ├── compose/
│   │   └── Woodpecker/             # Git repo lives here
│   │       ├── docker-compose.yml  # Main Docker Compose config
│   │       ├── .env                # Runtime environment variables and secrets (gitignored!)
│   │       ├── env.example         # Example .env file for reference
│   │       └── README.md           # This file
│   └── data/
│       └── Woodpecker/             # Volume mounts and persistent data
```

---

## 🧰 Prerequisites

* Docker Engine
* Docker Compose V2
* Git
* (Optional) ZFS on Linux for dataset management

> ⚠️ **Note:** These instructions assume your ZFS pool is named `tank`. If your pool has a different name (e.g., `rpool`, `zdata`, etc.), replace `tank` in all paths and commands with your actual pool name.

---

## ⚙️ Setup Instructions

1. **Create the stack directory and clone the repository**

   If using ZFS:
   ```bash
   sudo zfs create -p tank/docker/compose/woodpecker
   cd /tank/docker/compose/woodpecker
   sudo git clone https://github.com/Vantasin/Woodpecker.git .
   ```

   If using standard directories:
   ```bash
   mkdir -p ~/docker/compose/woodpecker
   cd ~/docker/compose/woodpecker
   git clone https://github.com/Vantasin/Woodpecker.git .
   ```

2. **Create the runtime data directory** (optional)

   If using ZFS:
   ```bash
   sudo zfs create -p tank/docker/data/woodpecker
   ```

   If using standard directories:
   ```bash
   mkdir -p ~/docker/data/woodpecker
   ```

3. **Configure environment variables**

   Copy and modify the `.env` file:

   ```bash
   sudo cp env.example .env
   sudo nano .env
   sudo chmod 600 .env
   ```

   > **Note:** Be sure to update the `WOODPECKER_HOST`, `WOODPECKER_ADMIN`, `WOODPECKER_GITEA_URL`, `WOODPECKER_EXPERT_FORGE_OAUTH_HOST`, `WOODPECKER_GITEA_CLIENT`, `WOODPECKER_GITEA_SECRET`, `WOODPECKER_AGENT_SECRET` and if necessary the `WOODPECKER_DATA_DIR`.

     - Create the `WOODPECKER_HOST` using [Nginx Proxy Manager](https://github.com/Vantasin/Nginx-Proxy-Manager.git) as a reverse proxy for HTTPS certificates via Let's Encrypt.

         > **Proxy Host:**
         >  - **Domain Name:** `https://woodpecker.example.com`
         >  - **Scheme:** `http`
         >  - **Forward Hostname/IP:** `woodpecker-server`
         >  - **Forward Port:** `8000`
         >
         > **SSL:**
         >  - Check **Enable SSL**
         >  - From the **Certificate** dropdown select your `*.example.com` certificate
         >  - Enable **Force SSL** to redirect all HTTP → HTTPS

     - The `WOODPECKER_ADMIN` is/are the Gitea username(s) allowed as admin.

     - The `WOODPECKER_GITEA_URL` is the internal URL Woodpecker uses to reach Gitea over Docker networking. If your Gitea container is named `gitea` and listens on port `3000`, use:

         ```env
         WOODPECKER_GITEA_URL=http://gitea:3000
         ```

     - The `WOODPECKER_EXPERT_FORGE_OAUTH_HOST` is the public Gitea URL users open in the browser for OAuth:

         ```env
         WOODPECKER_EXPERT_FORGE_OAUTH_HOST=https://gitea.example.com
         ```

     - Create a Gitea OAuth2 application for Woodpecker:

         1. In Gitea, open `User Settings` -> `Applications` -> `Manage OAuth2 applications`.
         2. Create a new OAuth2 application using:
            - **Application Name:** `Woodpecker`
            - **Redirect URI:** `https://woodpecker.example.com/authorize`
            - **Confidential Client:** enabled
         3. Replace `https://woodpecker.example.com` with your actual `WOODPECKER_HOST` value. The redirect URI must match exactly, with `/authorize` appended.
         4. After saving, copy the generated credentials into `.env`:

            ```env
            WOODPECKER_GITEA_CLIENT=<client-id>
            WOODPECKER_GITEA_SECRET=<client-secret>
            ```

         Do not commit your real `.env` file or OAuth client secret.

     - The `WOODPECKER_AGENT_SECRET` is a random user generated password of your choice, avoid special characters and symbols.

4. **Start Woodpecker**

   ```bash
   docker compose up -d
   ```

---

## 🌐 Accessing Woodpecker Web UI

Once deployed, access `Woodpecker` using:

- **Web Interface (HTTP):** `https://woodpecker.example.com`.
- **Login:** You'll see a `Login with Gitea` button.
> **Note:** If you don’t see the `Login with Gitea` button, check your `.env` file for errors with `WOODPECKER_GITEA_URL`, `WOODPECKER_GITEA_CLIENT` & `WOODPECKER_GITEA_SECRET`.

---

## 🙏 Acknowledgments

- [ChatGPT](https://openai.com/chatgpt) — for assistance in generating setup scripts and templates.
- [Woodpecker CI](https://github.com/woodpecker-ci/woodpecker) — the official source code and Docker images for the Woodpecker continuous integration system.
- [Woodpecker Documentation](https://woodpecker-ci.org/docs/intro) — the official user guide and admin reference for configuring and troubleshooting Woodpecker.
- [Docker](https://www.docker.com/) — for container orchestration and runtime.
- [OpenZFS](https://openzfs.org/) — for advanced local filesystem features, dataset organization, and snapshotting.
