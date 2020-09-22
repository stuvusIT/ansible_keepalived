# Role Name

This role installs and configures [keepalived](https://github.com/acassen/keepalived) on Debian.


## Requirements

[Debian](https://www.debian.org/).
Maybe this role also works on other apt based systems.


## Role Variables

| Name                      | Required | Description                                     |
| :------------------------ | :------- | :---------------------------------------------- |
| `keepalived_config_files` | yes      | See [Configuration Files](#configuration-files) |


## Configuration Files

The variable `keepalived_config_files` must contain a list of files to be put on the host in the
directory `/etc/keepalived`.
Each item must be a dict containing the keys and values as expected by the
[copy module](https://docs.ansible.com/ansible/latest/modules/copy_module.html).
The value of [dest](https://docs.ansible.com/ansible/latest/modules/copy_module.html) will be
prepended with the aforementioned directory.
Note that `keepalived_config_files` is required, because you need at least the file
`/etc/keepalived/keepalived.conf`.
See [Example Playbook](#example-playbook) for an example.


## Example Playbook

The following example playbook assumes that you cloned this role to `roles/keepalived`
(i.e. the name of the role is `keepalived` instead of `ansible_keepalived`).

```yml
- hosts: kube01
  roles:
    - role: keepalived
      keepalived_config_files:
        - dest: keepalived.conf
          content: |
            vrrp_script health_check {
              script "/etc/keepalived/health_check.sh"
              interval 2
              weight -2
              fall 2
              rise 2
            }

            vrrp_instance VI_1 {
              state BACKUP
              interface eno2
              virtual_router_id 120
              priority 100
              virtual_ipaddress {
                  1.2.3.4
              }
              track_script {
                  health_check
              }
            }

        - dest: health_check.sh
          mode: 0755
          content: |
            #!/bin/sh

            errorExit() {
              echo "*** $*" 1>&2
              exit 1
            }

            APISERVER_VIP=1.2.3.4
            APISERVER_DEST_PORT=6443

            curl --silent --max-time 2 --insecure https://localhost:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://localhost:${APISERVER_DEST_PORT}/"
            if ip addr | grep -q ${APISERVER_VIP}; then
              curl --silent --max-time 2 --insecure https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/ -o /dev/null || errorExit "Error GET https://${APISERVER_VIP}:${APISERVER_DEST_PORT}/"
            fi
```


## License

This work is licensed under the [MIT License](./LICENSE).


## Author Information

- [Sebastian Hasler (haslersn)](https://github.com/haslersn) _sebastian.hasler at stuvus.uni-stuttgart.de_
