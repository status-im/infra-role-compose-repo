# Description

This Ansible role deploys a Docker Compose setup defined in specified repo.

It also can keep this setup up-to-date by [handling GitHub webhooks](https://github.com/status-im/infra-role-github-webhook).

# Configuration

The basic configuration could define following parameters:
```yaml
compose_repo_service_name: 'service-abc'
compose_repo_repo_url: 'https://github.com/example-org/service-abc'
compose_repo_repo_branch: 'master'
compose_repo_webhook_secret: 'super-secret-webhook-secret'
```
The Docker images used are controlled by env variables defined in the `docker-compose.yml` in the source repo.
Optionally images can be monitored and auto-updated using [Watchtower](https://github.com/containrrr/watchtower).
To do that the `com.centurylinklabs.watchtower.enable` __label__ needs to be set to `true`.

The rest of the configuration comes from `.env` file which is symlinked by default to `custom.env` from the repo.

# Management

The setup consists of two services:

* `service-abc-compose` - Runs Docker Compose which starts all containers.
* `service-abc-webhook` - Runs Python script that watches for repo changes.

## Compose

This service simply runs the compose command that starts the containers.
```
 > sudo systemctl status -o cat service-abc-compose
● service-abc-compose.service - "Service that runs Docker Compose for a custom repo."
     Loaded: loaded (/etc/systemd/system/service-abc-compose.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-09-26 14:12:30 UTC; 11min ago
       Docs: https://github.com/status-im/infra-misc/blob/master/ansible/roles/service-abc
   Main PID: 51089 (docker-compose)
      Tasks: 108 (limit: 76929)
     Memory: 47.0M
        CPU: 8.987s
     CGroup: /system.slice/service-abc-compose.service
             └─51089 /usr/bin/python3 /usr/local/bin/docker-compose up
```

## Webhook

This service updates the `service-abc` repo and re-creates the containers:
```
 > sudo systemctl status -o cat service-abc-webhook
● service-abc-webhook.service - "GitHub Webhook server for updating Git repos"
     Loaded: loaded (/lib/systemd/system/service-abc-webhook.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-09-26 14:11:36 UTC; 22min ago
       Docs: https://github.com/status-im/infra-role-github-webhook/tree/master/README.md
   Main PID: 47328 (python3)
      Tasks: 1 (limit: 76929)
     Memory: 21.1M
        CPU: 427ms
     CGroup: /system.slice/service-abc-webhook.service
             └─47328 python3 /home/comprepo/webhook/server.py --port=6060 --repo-url=https://github.com/example-org/service-abc --repo-bra>

INFO - 127.0.0.1 - - [26/Sep/2023 14:29:00] "GET /health HTTP/1.1" 200 -
INFO - 127.0.0.1 - - [26/Sep/2023 14:29:30] "GET /health HTTP/1.1" 200 -
```
