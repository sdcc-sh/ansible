# SDCC Ansible

A collection of Ansible-related config for managing [sdcc.sh](https://sdcc.sh) servers.

## Quickstart

You will need `sshpass` installed (brew install hudochenkov/sshpass)

```
python3 -m venv env
source env/bin/activate
pip install -r requirements.txt
ansible-playbook -i inventory.ini playbook.yml --check
```

## Development

Before raising a pull request you should run a lint check against your changes:

```
ansible-lint
```

## Provisioning a New Server

### OS Installation

1. Order a new VPS with the desired config

2. Mount the `FreeBSD-11.2-bootonly` image 

3. Use the remote console to run through installation using the following settings:

    | Setting                                           | Value                                                                                                                                                 |
    |---------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
    | Hostname                                          | `vps<id>.sdcc.sh`                                                                                                                                     |
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

4. Finalise the installation and reboot the machine. Don't forget to unmount the image before rebooting.

5. Use the remote console to enter the disk decryption password (you will need to do this each time the server is rebooted).

### Bootstrapping

Unfortunately we cannot log in remotely as `root`, so we will need to manually bootstrap the box to get it ready for Ansible.

1. Add entry for host in `inventory.ini`

2. Install pre-requisite packages:

    ```
    ssh ansible@<hostname>
    su -u
    pkg install sudo python3
    ```

3. Disconnect

4. Run initial bootstrapping (this will take a while):

    ```
    ansible-playbook -l <host> -i inventory.ini -u ansible --become-method=su --ask-pass --ask-become-pass playbook.yml
    ```
