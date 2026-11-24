# VProfile: Automated VM Provisioning

Fully automated deployment of a 3-tier Java web application on 5 Vagrant VMs. A single `vagrant up` builds the VMs, installs every service with shell scripts, and deploys the app.

This is the automated counterpart of my [manual provisioning project](../vprofile-manual-vm-provisioning/), where I set up every service by hand.

> **Credit:** The application source and base scripts are from [hkhcoder/vprofile-project](https://github.com/hkhcoder/vprofile-project).
> **My work:** (describe what you did, for example: ran and verified the automated setup, fixed issues, compared it with the manual process, documented it)

## Architecture

```
User → Nginx (web01) → Tomcat (app01) → Memcached (mc01)
                                      → RabbitMQ (rmq01)
                                      → MariaDB (db01)
```

## Stack

| VM | Service | Provisioning script |
|----|---------|---------------------|
| web01 | Nginx | `nginx.sh` |
| app01 | Tomcat 10 + app deploy | `tomcat.sh` / `tomcat_ubuntu.sh` (verify) |
| mc01 | Memcached | `memcache.sh` |
| rmq01 | RabbitMQ | `rabbitmq.sh` |
| db01 | MariaDB | `mysql.sh` |

## Files

| File | Purpose |
|------|---------|
| `Vagrantfile` | Defines the 5 VMs (box, IP, RAM) and runs each script on its VM |
| `mysql.sh` | Installs MariaDB, creates the `accounts` database and user, imports the schema, opens the firewall |
| `memcache.sh` | Installs Memcached, binds it to all interfaces, opens the firewall |
| `rabbitmq.sh` | Installs RabbitMQ, creates the admin user, opens the firewall |
| `tomcat.sh` / `tomcat_ubuntu.sh` | Installs Java and Tomcat, sets up the systemd service (verify which one the Vagrantfile uses) |
| `backend.sh` | Builds the app with Maven and deploys the WAR to Tomcat (verify) |
| `nginx.sh` | Installs Nginx and configures the reverse proxy to Tomcat |
| `application.properties` | App configuration pointing to the backend services |

## Prerequisites

- Oracle VirtualBox
- Vagrant
- Vagrant plugin: `vagrant plugin install vagrant-hostmanager`
- Git Bash (or equivalent terminal)

## Usage

```bash
cd vprofile-automated-vm-provisioning
vagrant up
```

The first run takes a while because it downloads boxes and packages and builds the app. If it stops partway, run `vagrant up` again, or `vagrant provision <vm-name>` to re-run the scripts on one VM.

Then open `http://web01` (or its IP) in a browser.

Other useful commands:

```bash
vagrant status          # VM states
vagrant ssh app01       # log in to a VM
vagrant halt            # stop all VMs
vagrant destroy -f      # delete all VMs
```

## Verification

```bash
vagrant ssh app01
sudo systemctl status tomcat
curl -v --max-time 5 telnet://mc01:11211
curl -v --max-time 5 telnet://rmq01:5672
curl -v --max-time 5 telnet://db01:3306
```

## Screenshots

![Vagrant status](screenshots/01-vagrant-status.png)
![Login page](screenshots/02-login-page.png)
![App dashboard](screenshots/03-app-dashboard.png)

## Manual vs automated

| | Manual | Automated |
|---|--------|-----------|
| Setup | Log in to each VM and run commands | One `vagrant up` |
| Repeatable | Depends on me following the steps | Same result every time |
| Time | Long | Mostly waiting |
| Debugging | Easy to see each step | Need to read script output |
| Best for | Learning how each service works | Rebuilding quickly |

## Troubleshooting

### (Issue you hit)
- **Symptom:**
- **Cause:**
- **Fix:**

## What I learned

- How Vagrant runs shell provisioners per VM
- Turning manual install steps into repeatable scripts
- Why order matters between database, cache, queue, app and web tiers
- (add your own)

## Notes

- Passwords such as `admin123` are for this local lab only.
- Next step: Ansible automation and a Kubernetes version.
