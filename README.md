# Laravan [WIP]

**Ansible Playbooks for Laravel - machine provisioning and app deployment**

Laravan is capable of preparing a fresh Ubuntu 16.04 machine for running laravel apps by setting up the following components using Ansible:

- Nginx
- PHP 7.0
- MariaDB
- Beanstalkd

It also brings some neat, optional stuff:

- Single-command deployment of Laravel applications
- Automatic database backups
- SSL certificates via Letsencrypt
- Supervisor for reliably running queue workers
- Hosting of static pages


## Getting Started

### 1. Install Ansible

Ansible needs to be installed on your **local machine** from which you're going to provision your servers and deploy your Laravel apps. Install instructions: [http://docs.ansible.com/ansible/2.3/intro_installation.html](http://docs.ansible.com/ansible/2.3/intro_installation.html)

### 2. Prepare Server

Boot up a fresh Ubuntu 16.04 (virtual) machine. Set up ssh keys and make sure you can log in as root from your local machine.

The target machine needs to have `python` installed in order to be provisioned by Ansible.

#### Vagrant

A Vagrantfile is provided with laravan, so you can fire up a fresh local vm by just stating `vagrant up` from your laravan directory. You will still need to set up your ssh keys for the root user of the Vagrant box.

### 3. Install Laravan

```bash
git clone {{ repo_url }}
cd laravan
./setup.sh
```

### 4. Link Machines

1. Enter the IP addresses of your hosts to the `hosts` file, each under their respective environment. If you created a vagrant machine using `vagrant up` from the laravan directory, its IP address will be `192.168.0.42`.

### 5. Configure Applications

Open up `group_vars/{env}/apps.yml` and provide at least the following information for your Laravel app:

```yml
---
apps:
  app_name:
    hosts:
      - canonical: app.example.com
    source:
      url: git@github.com:acme/example.git
      version: master
    env:
      APP_KEY: base64:6o/xyz…
      DB_HOST: "127.0.0.1"
      DB_DATABASE: example
      DB_USERNAME: example
      DB_PASSWORD: secret

  another_app:
  	hosts:
  	  - canonical: …
  	…
```

- `app_name` must be replaced with a unique name for the app. It must be a valid python variable name, so use only letters and underscore
- `canonical:` holds the primary domain under which your app will be available
- `env:` those values will eventually be written into an `.env` on the server. **They are also used to configure the database correctly, so don't miss them!**

### 6. Provision

```bash
ansible-playbook provision.yml -e env=development
```

### 7. Deploy

```bash
ansible-playbook deploy.yml -e env=development -e app_name=example
```

### 8. Encrypt vault

In order to prevent your production secrets from ending up as plain text in your git repositories, use the [ansible vault](http://docs.ansible.com/ansible/2.4/vault.html).

1. Create a `.vault_pass` file containing a strong password (eg. `openssl rand -base64 64 > .vault_pass`). This file is gitignored, which you should leave at all cost. Your coworkers will need a copy of the file.
2. Encrypt your vault: `ansible-vault encrypt group_vars/production/vault.yml`
3. **Never again decrypt the vault!**. Use `ansible-vault view <path>` or `ansible-vault edit <path>` to open the vault file. This reduces the risk of the vault ending up plain text in your version control.


## Credits

Credits to the awesome [Trellis](https://github.com/roots/trellis) project, which heavily inspired me to create laravan and from which i also took some code.


## License

This software is released "as it is" without any warranty under the MIT licence. See [LICENSE](LICENSE) for details.
