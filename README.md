# SDCC Ansible

[![Deploy](https://github.com/sdcc-sh/sdcc-ansible/actions/workflows/deploy.yml/badge.svg)](https://github.com/sdcc-sh/sdcc-ansible/actions/workflows/deploy.yml)

A collection of Ansible-related config for managing [sdcc.dev](https://sdcc.dev) servers.

## Quickstart

You will need `sshpass` installed (`brew install hudochenkov/sshpass/sshpass`)

```
$ python3 -m venv env
$ source env/bin/activate
$ pip install -r requirements.txt
$ ansible-playbook --ask-become-pass --ask-vault-pass site.yml
```

## Development

Before raising a pull request you should run a lint check against your changes:

```
$ ansible-lint
```

## Provisioning a New Server

### OS Installation

1. Use the remote console to run through installation using the following settings:

    | Setting                                           | Value                                                                                                                                                 |
    |---------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Hostname                                          | `<mac>.sdcc.dev`                                                                                                                                       |
    | Distribution Select                               | `ports`                                                                                                                                               |
    | Partitioning                                      | Auto (ZFS)                                                                                                                                            |
    | ZFS Configuration - Encrypt Disks?                | YES                                                                                                                                                   |
    | ZFS Configuration - Encrypt Swap?                 | YES                                                                                                                                                   |
    | ZFS Configuration - Pool Type                     | stripe                                                                                                                                                |
    | ZFS Configuration - Disk Encryption Password      | `<Generate strong random password>`                                                                                                                   |
    | `root` Password                                   | `<Generate strong random password>`                                                                                                                   |
    | Timezone                                          | UTC                                                                                                                                                   |
    | System Configuration                              | `sshd`                                                                                                                                                |
    | System Hardening                                  | `hide_uids`, `hide_gids`, `hide_jail`, `read_msgbuf`, `proc_debug`, `random_pid`, `clear_tmp`, `disable_syslogd`, `secure_console`, `disable_ddtrace` |
    | Add User Accounts                                 | `Yes`                                                                                                                                                 |
    | Add Users - Username                              | `ansible`                                                                                                                                             |
    | Add Users - Invite `ansible` into other groups?   | `wheel`                                                                                                                                               |
    | Add Users - Use a random password                 | `yes`                                                                                                                                                 |

2. Finalise the installation and reboot the machine. Don't forget to unmount the image before rebooting.

3. Use the remote console to enter the disk decryption password (you will need to do this each time the server is rebooted).

### Bootstrapping

Unfortunately we cannot log in remotely as `root`, so we will need to manually bootstrap the box to get it ready for Ansible.

1. Add inventory entry for host in `hosts`

2. Install pre-requisite packages:

    ```
    $ ssh ansible@<hostname>
    # pkg install sudo python3
    ```

3. Copy across SSH public key for ansible user

4. Disconnect

5. Run initial bootstrapping (this will take a while):

    ```
    $ ansible-playbook -u ansible --become-method=su --ask-pass --ask-become-pass --ask-vault-pass site.yml
    ```
