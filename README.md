# Ansible: Ubuntu 24.04 Secure Setup

An idempotent Ansible playbook to perform a secure, initial setup of a fresh Ubuntu 24.04 server. This project automates best practices for hardening a new system, creating a non-root user, and configuring essential security tools.

The playbook uses Ansible tags to separate one-time bootstrapping from ongoing configuration, ensuring it can be run safely against both new and existing servers.

---

## 🛡️ Features

* Updates all system packages.
* Installs essential security packages (`fail2ban`, `ufw`).
* Creates a new `sudo` user for administration.
* Adds your local SSH public key for passwordless login.
* Hardens SSH by disabling root login, disabling password authentication, and changing the default port.
* Correctly handles systems with SSH socket activation.
* Configures UFW (Uncomplicated Firewall) to allow only SSH, HTTP, and HTTPS traffic.

---

## ✅ Requirements

### On Your Control Node (Your Laptop)

* A modern version of Ansible (`ansible-core >= 2.16`).
* The `passlib` Python library for password hashing.
* An SSH key pair (the playbook uses `~/.ssh/id_rsa.pub`).

To install the requirements:

```bash
python3 -m pip install --user ansible passlib
```

### On the Target Node

* A fresh Ubuntu 24.04 installation.
* Access to the `root` user with a password for the initial run.

---

## ⚙️ Configuration

1. **Clone the repository:**

    ```bash
    git clone [https://github.com/farzad-khb/ansible-ubuntu-bootstrap.git](https://github.com/farzad-khb/ansible-ubuntu-bootstrap.git)
    cd ansible-ubuntu-bootstrap
    ```

2. **Create your private inventory file:**
    Copy the example template and edit it with your server's IP address. For the first run, ensure the `ansible_port` is commented out or set to `22`.

    ```bash
    cp inventory.yml.example inventory.yml
    nano inventory.yml
    ```

3. **Configure Variables:**
    Edit `group_vars/servers.yml` to set your desired administrative username (`bootstrap_user`) and the new SSH port (`new_ssh_port`).

---

## 🚀 Usage

Running this playbook is a two-stage process controlled by tags.

### 1. Initial Server Provisioning

This command runs all plays, including the `bootstrap` plays, to set up a new server from scratch. It connects as `root` on the default port `22`.

```bash
ansible-playbook site.yml --user root --ask-pass -K
```

You will be prompted for passwords for `root` login and to create your new user.

**After this run is successful**, you must update your `inventory.yml` to use the new SSH port (e.g., `ansible_port: 2222`).

### 2. Subsequent Configuration Updates

For all future runs, you must **skip the bootstrap tasks**, as root login will now be disabled. This command uses `--skip-tags` to run only the ongoing configuration plays.

```bash
ansible-playbook site.yml -K --skip-tags bootstrap
```

This command will connect as your new admin user (via its SSH key) and will only prompt for the `BECOME password:` (your user's sudo password).
