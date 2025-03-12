### Docker rootless using Ansible on a debian VM

This Ansible playbook configures a running Debian VM with Docker in rootless mode for enhanced security. It can be used with either an existing Debian VM or automatically with Vagrant. The playbook creates a dedicated non-privileged user 'dockeruser' that can only run containers in isolation from the host system. For secure access, 'dockeruser' can only be reached via SSH using the specific key defined in the inventory file, ensuring strict access control.

## Overview

The playbook automates:
1. Initial setup using a user with sudo privileges
2. Creation of a restricted user 'dockeruser'
3. Docker Rootless configuration for this user 'dockeruser'

## Prerequisites

- Ansible 2.9 or higher
- A running Debian VM
- A user with sudo privileges on the target VM
- an ssh key for the becoming new created user 'dockeruser'

## Getting Started

1. Clone this repository:
```bash
$ git clone https://github.com/st3vebrush/debian-docker-rootless.git
$ cd debian-docker-rootless
```

5. Update the inventory file with your VM details and SSH key:
```ini
[debian]
$ # <ip-address>: IP address of your Debian VM
$ # <username>: Your regular user with sudo privileges on the Debian VM
$ # <ssh_key>: Path to YOUR SSH private key to connect as sudo user
$ debian ansible_host=<ip-address> ansible_user=<username> ansible_ssh_private_key_file=<ssh_key>
```

6. Run the Ansible playbook:
```bash
$ # The --ask-become-pass flag will prompt for your sudo password
$ ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

## Role Descriptions

### Common Role
- System updates
- Essential packages installation
- Security configurations

### Docker Role
- Docker CE installation
- Rootless configuration
- Docker daemon settings

### User Role
- User permissions
- Environment variables
- Shell configurations

## Project Structure
```
.
├── README.md
├── ansible.cfg
├── inventory.ini
├── playbook.yml
├── roles
│   ├── common
│   │   └── tasks
│   │       └── main.yml
│   ├── docker
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── user
│       ├── defaults
│       │   └── main.yml
│       └── tasks
│           └── main.yml
└── vars.yml
```

## Variables Configuration (vars.yml)

The `vars.yml` file defines the following global variables:

- `docker_version`: Specifies the Docker version to install (use `apt-cache madison docker-ce` to see available versions. Default: "5:28.0.1-1~debian.12~bookworm")
- `user_name`: Sets the username of the restricted Docker rootless user (default: "dockeruser")
- `user_ssh_key_file`: Defines the path to the SSH public key for 'user_name' authentication (default: "~/.ssh/id_rsa.pub")

Modify these variables in `vars.yml` to customize your deployment.


## Usage

After deployment:

1. SSH into your newly configured Debian VM as dockeruser with your 'dockeruser' ssh key.:
```bash
$ ssh -i ~/.ssh/id_rsa dockeruser@<ip-address>
```

2. Verify Docker installation:
```bash
$ docker --version
$ docker ps
```

3. Run a test container:
```bash
$ docker run hello-world
```

## Testing with Nginx

The playbook includes a ready-to-use Nginx example:

1. After installation, log in as the dockeruser
2. The setup includes:
   - A pre-configured `run-nginx.sh` script
   - An `html` directory with a sample `index.html`

To test the setup:
```bash
$ ./run-nginx.sh
```

You can verify the Nginx server is running by using curl in any shell:
```bash
$ curl <ip-address>:8080
<!DOCTYPE html>
<html>
  <head>
    <title>Hello World</title>
  </head>
  <body>
    <h1>Hello World!</h1>
    <p>This is a test page served by Nginx in rootless Docker.</p>
  </body>
</html>
```

## Troubleshooting

Common issues and solutions:

1. Docker daemon not starting:
```bash
systemctl --user status docker
journalctl --user -u docker
```

2. Permission issues:
```bash
ls -l $XDG_RUNTIME_DIR/docker.sock
dockerd-rootless-setuptool.sh check
```

## Docker Rootless Restrictions

When running Docker in rootless mode as 'dockeruser', be aware of these limitations:

- Port binding below 1024 is not allowed
- Host network mode (`--network=host`) has limited functionality
- Cannot mount host directories without additional configuration
- Some container capabilities are not available
- Limited access to devices and system resources
- Cannot use external volume drivers
- Network features like IPv6 and bridge networks have restrictions

For detailed information about these limitations, refer to the [Docker Rootless documentation](https://docs.docker.com/engine/security/rootless/).

## Security Benefits

- Reduced attack surface by running Docker without root privileges
- Container processes run with regular user permissions
- Improved isolation between host and containers

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the [MIT License](LICENSE).

## Support

For issues and support:

- Check the [Docker Rootless documentation](https://docs.docker.com/engine/security/rootless/)
- Create an issue in the repository



