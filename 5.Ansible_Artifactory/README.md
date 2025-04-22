JFrog Artifactory OSS Ansible Role
==================================
The simpliest Ansible role for JFrog Artifactory OSS 7 that works on Ubuntu 19.10 hosts with init.d.

Role includes all *key Artifactory config files* in [templates/](https://github.com/eugene-krivosheyev/ansible-artifactory-role/tree/master/templates/) so you can widely customize your Artifactory installation.


Role requirements
-----------------
- _Unzip_. This role requires unzip package and installs it automatically by itself. If you don't need it, you can remove it in post_tasks section of your playbook.
- _Production Database_. Artifactory uses simple lightweigth database called Derby by default. If you don't plan to host high-load/fail-over solution with your Artifactory instance, Derby could be enough. If you plan to use production level database, you have to install and cofigure it yourself in your playbook. Already done Ansible Galaxy roles will help, e.g. geerlingguy.postgresql, geerlingguy.mysql, etc. After you describe database installation and configuration with your playbook, [it required to configure Artifactory](https://www.jfrog.com/confluence/display/JFROG/Configuring+the+Database) to work with your database. After you add driver downloading, creating database itself and database user to your playbook, you're required to configure Atrifactory role to work with your just configured database by [setting properties artifactory_database_*](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/defaults/main.yml) to correct values. As an example you can take a look at the [test playbook with Postgres](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/tests/test.yml).


[Role variables](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/defaults/main.yml)
----------------
Variable 'ansible_become_method' is used to generate service script. It have default value 'sudo' so if your target host don't have sudo command you have to explicitly set it to 'su'. Because it depends on target hosts linux distr better set it on per-host basis in your inventory file. See [example](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/tests/inventory.yml) of setting 'ansible_become_method' variable for inventory hosts.

Other variables that could be customized described in [defaults/main.yml](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/defaults/main.yml).


[Example _requirements.yml_](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/tests/requirements.yml) to add this role to your playbook
------------------------------------------------------------
```yml
- eugene_krivosheyev.artifactory
```


[Example playbook](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/tests/test.yml)
------------------
```yml
- hosts: ci_server
=======
# Ansible Artifactory Setup

This directory contains Ansible configuration for deploying and managing JFrog Artifactory OSS, a universal binary repository manager.

## Overview

The Ansible Artifactory setup provides an automated solution for deploying JFrog Artifactory OSS on Ubuntu servers. It leverages the `eugene_krivosheyev.artifactory` role from Ansible Galaxy to handle the installation and configuration process.

Artifactory serves as a central repository for managing binary artifacts such as:
- Maven/Gradle dependencies
- Docker images
- NPM packages
- Python packages
- Generic binary files

## Directory Structure

```
5.Ansible_Artifactory/
└── ansible-artifactory-role/
    ├── defaults/           # Default variable values
    ├── handlers/           # Service handlers
    ├── meta/               # Role metadata
    ├── tasks/              # Implementation tasks
    ├── templates/          # Configuration templates
    ├── tests/              # Test configurations
    ├── vars/               # Role variables
    ├── ansible.cfg         # Ansible configuration
    ├── inventory.yml       # Target server inventory
    ├── playbook.yml        # Main deployment playbook
    ├── requirements.yml    # Role dependencies
    └── README.md           # Original role documentation
```

## Configuration

The deployment is configured through the `playbook.yml` file, which specifies:

```yaml
- hosts: server1
  become: yes
  vars:
    artifactory_download_url: "https://releases.jfrog.io/artifactory/jfrog-artifactory-oss-release/com/jfrog/artifactory/oss/jfrog-artifactory-oss/7.3.2/jfrog-artifactory-oss-7.3.2-linux.tar.gz"
>>>>>>> a9301e4fb39b03574680b8c91aeda89624f85277
  roles:
    - role: eugene_krivosheyev.artifactory
      artifactory_port: 8081
      artifactory_ui_port: 8082
      artifactory_username: admin
      artifactory_password: P@ssw0rd
```
<<<<<<< HEAD
Also you can refer [test role usage](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/tests/test.yml) for example.


Known issues
============
1. You may face issue if target host don't have command 'sudo'
-------------------------------------------------------------- 
In this case just set default priviledge escalation command to 'su'. Because it depends on target hosts linux distr better set it on per-host basis in your inventory file. See [example](https://github.com/eugene-krivosheyev/ansible-artifactory-role/blob/master/tests/inventory.yml) of setting 'ansible_become_method' variable for inventory hosts.

2. You may face issue with 'su' command timeouts if it set as default privilege escalation command for some hosts
-----------------------------------------------------------------------------------------------------------------
```bash
TASK [geerlingguy.postgresql : Ensure PostgreSQL Python libraries are installed.] ********
fatal: [test_host]: FAILED! => {"msg": "Timeout (12s) waiting for privilege escalation prompt: "}
```
In this case set default priviledge escalation command to 'sudo' with 'ansible_become_method' variable explicitely. Better set it on per-host basis in your inventory file. 
Or just skip setting 'ansible_become_method' at all because of its default is 'sudo' already. 

3. Artifactory tries to start but then get permanent error at web interface
---------------------------------------------------------------------------
```json
{
  "errors" : [ {
    "status" : 500,
    "message" : "Artifactory failed to initialize: check Artifactory logs for errors."
  } ]
}
```
And in logs (e.g. */opt/jfrog/artifactory/var/log/console.log*) you have
```log
.
Could not validate router Check-url: http://::1:8082/router/api/v1/system/ping
.
.
.
Registration with router on URL http://localhost:8046 failed with error: UNAVAILABLE: io exception.
Registration with router on URL http://localhost:8046 failed with error: UNAVAILABLE: io exception.
Registration with router on URL http://localhost:8046 failed with error: UNAVAILABLE: io exception.
.
```
This means that one of Artifactory component uses 'localhost' landed to IPv6's "localhost" called '::1'.

To fix you can try [force Java to use IPv4](https://superuser.com/questions/453298/how-to-force-java-to-use-ipv4-instead-ipv6) or 
manually remove binding of IPv6 from 'localhost' at your */etc/hosts*:
```/etc/hosts
127.0.0.1       localhost   
::1             localhost  # <-- Remove any type of such binding "::1 -> localhost" at any string 
::1             ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
```
In case of reproducing issue just [turn IPv6 off](https://www.techrepublic.com/article/how-to-disable-ipv6-through-grub-in-linux/).

License
=======
BSD
=======

Key configuration parameters:
- **artifactory_port**: Main HTTP port (8081)
- **artifactory_ui_port**: Web UI port (8082)
- **artifactory_username**: Admin username
- **artifactory_password**: Admin password

## Prerequisites

- Ansible 2.9 or higher
- Target Ubuntu server with:
  - SSH access
  - Sudo privileges
  - At least 50GB disk space (for repositories)

## Usage

### Installation

1. Install the required Ansible role:
   ```bash
   ansible-galaxy install -r ansible-artifactory-role/requirements.yml
   ```

2. Update the inventory file with your target server details:
   ```yaml
   all:
     hosts:
       server1:
         ansible_host: your-server-ip
         ansible_user: your-ssh-user
   ```

3. Deploy Artifactory:
   ```bash
   cd ansible-artifactory-role
   ansible-playbook -i inventory.yml playbook.yml
   ```

### Accessing Artifactory

After successful deployment, access the Artifactory web interface at:
```
http://your-server-ip:8081/artifactory
```

Log in with the credentials specified in the playbook:
- Username: admin
- Password: P@ssw0rd (or your custom password)

## Database Configuration

By default, Artifactory uses an embedded Derby database, which is suitable for testing but not recommended for production. The role supports configuring various production databases:

- PostgreSQL
- MySQL
- Oracle
- MS SQL
- MariaDB

To configure a production database, modify the variables in your playbook or in the defaults/main.yml file. For example, to use PostgreSQL:

```yaml
artifactory_database_type: 'postgresql'
artifactory_database: 'localhost:5432/artifactory'
artifactory_database_user: "artifactory_user"
artifactory_database_password: "secure_password"
```

## Customization

The role includes all key Artifactory configuration files as templates, allowing for extensive customization:

- System configuration: `templates/system.yaml`
- Security settings: `templates/security.yaml`
- Repository configurations: `templates/artifactory.config.import.xml`

## Troubleshooting

### Common Issues

1. **IPv6 Connectivity Issues**:
   If Artifactory fails to initialize with errors about router connectivity, it may be due to IPv6 issues. Solutions include:
   - Force Java to use IPv4
   - Remove IPv6 binding from 'localhost' in /etc/hosts
   - Disable IPv6 through GRUB

2. **Database Connection Issues**:
   Check the logs at `/opt/jfrog/artifactory/var/log/artifactory.log` for database connection errors.

### Log Locations

- Main log: `/opt/jfrog/artifactory/var/log/artifactory.log`
- Console log: `/opt/jfrog/artifactory/var/log/console.log`
- Access log: `/opt/jfrog/artifactory/var/log/access.log`

## Maintenance

### Backup

To back up Artifactory:
```bash
cp -r /opt/jfrog/artifactory/var/etc /path/to/backup/etc
cp -r /opt/jfrog/artifactory/var/data /path/to/backup/data
```

### Service Management

```bash
# Start Artifactory
sudo service artifactory start

# Stop Artifactory
sudo service artifactory stop

# Check status
sudo service artifactory status
```
