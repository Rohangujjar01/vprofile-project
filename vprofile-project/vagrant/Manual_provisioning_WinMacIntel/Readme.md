# VProfile: Manual VM Provisioning

Manual deployment of a 3-tier Java web application across 5 Vagrant VMs. Every service was installed, configured and connected by hand, with no automation scripts.

> **Credit:** The application source is from [hkhcoder/vprofile-project](https://github.com/hkhcoder/vprofile-project).
> **My work:** VM setup, installing and configuring every service, firewall rules, building and deploying the app, and troubleshooting.

## Architecture

```
User → Nginx (web01) → Tomcat (app01) → Memcached (mc01)
                                      → RabbitMQ (rmq01)
                                      → MariaDB (db01)
```

(Add the architecture diagram here: `![Architecture](screenshots/architecture.png)`)

## Stack

| VM | Service | Role | Port |
|----|---------|------|------|
| web01 | Nginx | Reverse proxy / web tier | 80 |
| app01 | Tomcat 10 (Java 17) | Application server | 8080 |
| mc01 | Memcached | Database caching | 11211 |
| rmq01 | RabbitMQ | Message broker | 5672 |
| db01 | MariaDB | SQL database | 3306 |

## Prerequisites

- Oracle VirtualBox
- Vagrant
- Vagrant plugin: `vagrant plugin install vagrant-hostmanager`
- Git Bash (or equivalent terminal)

## Setup

```bash
cd vprofile-manual-vm-provisioning
vagrant up
```

The first run can take a long time. If it stops partway, run `vagrant up` again. The hostmanager plugin updates the hostnames and `/etc/hosts` entries on every VM automatically.

## Provisioning order

Services depend on each other, so they must be set up in this order:

1. **MariaDB (db01):** install, secure the installation, create the `accounts` database and user, import the schema, open port 3306.
2. **Memcached (mc01):** install, change the bind address to `0.0.0.0`, open port 11211.
3. **RabbitMQ (rmq01):** install, create a user with the administrator tag, open port 5672.
4. **Tomcat (app01):** install Java 17 and Tomcat 10, create a `tomcat` user and a systemd service, open port 8080.
5. **Build and deploy (app01):** build the WAR with Maven, point `application.properties` at the backend services, and deploy it as `ROOT.war`.
6. **Nginx (web01):** install, create a reverse-proxy config pointing to `app01:8080`, and remove the default site.

Full step-by-step commands: [docs/VprofileProjectSetupWindowsAndMacIntel.pdf](docs/VprofileProjectSetupWindowsAndMacIntel.pdf)

## Verification

```bash
# From app01: check connectivity to each backend service
curl -v --max-time 5 telnet://mc01:11211
curl -v --max-time 5 telnet://rmq01:5672
curl -v --max-time 5 telnet://db01:3306

# On each VM: check the service
systemctl status <service-name>
```

Then open `http://web01` (or its IP) in a browser and log in to the app.

## Screenshots

![Vagrant status](screenshots/01-vagrant-status.png)
![Login page](screenshots/02-login-page.png)
![App dashboard](screenshots/03-app-dashboard.png)
![Memcached running](screenshots/04-memcached-status.png)

## Troubleshooting

### Memcached: `Connection refused` from app01
- **Symptom:** `curl -v telnet://mc01:11211` returned "Connection refused" even though `mc01` resolved correctly.
- **Cause:** Memcached listened only on `127.0.0.1`, so other VMs could not connect.
- **Fix:** Changed the bind address to `0.0.0.0` in `/etc/sysconfig/memcached`, restarted the service, and opened port 11211 (tcp) and 11111 (udp) in firewalld.
- **Verified with:** `ss -tulnp | grep 11211` and the same `curl` test from app01.

### (Another issue you hit)
- **Symptom:**
- **Cause:**
- **Fix:**

## What I learned

- How the tiers of a web application depend on one another, and why the provisioning order matters
- Configuring systemd services and firewalld rules by hand
- Setting up Nginx as a reverse proxy in front of Tomcat
- Diagnosing connection problems layer by layer (DNS, port, firewall, bind address)

## Notes

- Passwords such as `admin123` are for this local lab only. Never use them in a real environment.
- Next step: automating this setup with provisioning scripts / Ansible, and a Kubernetes version.
