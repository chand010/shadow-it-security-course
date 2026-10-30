# Ubuntu and Docker Lab Setup

## Purpose
Prepare a small, isolated training environment. Assumptions: Ubuntu, 16 GB RAM, Docker and VirtualBox. This is a learning lab, not production.

## 1. Verify the host
Run and review:

    lsb_release -a
    uname -m
    free -h
    df -h
    docker --version
    docker compose version
    docker ps

Do not share secrets or unrelated personal information in command output. If Docker requires elevated privileges, use sudo docker unless your account is deliberately configured for Docker access. Membership in the Docker group is highly privileged.

## 2. Create a working directory

    mkdir -p ~/shadow-it-lab/{evidence,reports,scripts}
    chmod 700 ~/shadow-it-lab
    cd ~/shadow-it-lab

Keep working evidence outside this public Git repository. Publish only sanitized synthetic outputs.

## 3. Resource planning
With 16 GB RAM, start with one application and its database. Avoid running Nextcloud, Keycloak, MinIO and Wazuh simultaneously. Shut down services when not in use. Check docker stats and free -h if the host slows down. Use VirtualBox only when a separate guest OS is needed.

## 4. Network safety
- Bind web interfaces to 127.0.0.1 for local-only access unless a lab requirement explicitly needs otherwise.
- Use a dedicated Docker network.
- Do not expose test services to the public internet.
- Do not reuse production credentials.
- Do not mount host directories containing sensitive files into containers.
- Review running containers, published ports and volumes before each exercise.

## 5. Optional Nextcloud lab — plan before deployment
Nextcloud is used to demonstrate cloud-storage governance. Before deployment:
- Use maintained image and database versions; pin versions rather than using latest.
- Generate local-only passwords and protect the environment file.
- Bind the HTTP port to loopback.
- Use fictional users and synthetic documents.
- Use named volumes for application and database data.
- Document stop and cleanup procedures before deleting anything.

No deployment is claimed by this guide. Verify the host and inspect existing containers before deployment. Follow current official Nextcloud container documentation for image-specific variables and initialization behavior.

## 6. Verification commands

    docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'
    docker network ls
    docker volume ls
    docker stats --no-stream
    free -h

## 7. Cleanup principles
Stop only containers created for this lab. Avoid broad prune commands on machines hosting other work. Identify volume contents and confirm they contain no evidence you need before deleting them.

## Official documentation
- Docker Engine: https://docs.docker.com/engine/
- Docker Compose: https://docs.docker.com/compose/
- Nextcloud Docker: https://github.com/nextcloud/docker
