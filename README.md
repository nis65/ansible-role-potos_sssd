
# Ansible Role - potos\_sssd

This role works for me, but does not satisfy the potos acceptance rules yet:

* the automated checks are currently failing
* meta is not up to date

`sssd` can be used to integrate a machine into a central directory in various
ways. Currently, only `ldap` integration is implemented, some of the
`sssd` config options are currently hardcoded to work for a Univention
LDAP. For this configuration, you need:

* one (or more synced) ldap servers
* a read only technical user to connect to the ldap
* a SMB Server with users/groups synced with the ldap

Please see below for currently supported variables.

## Example Playbook

As this role is tested via Molecule one can use [that
playbook](./molecule/default/converge.yml) as a starting point:

```yaml
---

- name: Converge
  hosts: all
  gather_facts: yes
  tasks:
    - name: run role
      ansible.builtin.include_role:
        name: 'ansible-role-potos_template'
```

## Role Variables

* `potos_sssd_ca_cert_url`: The URL where to download from the CA certificate that signes the ldap servers host certificate.
* `potos_sssd_integration`: Must be set to `ldap`, only accepted value at the moment
* `potos_sssd_force_template`:

  * If set to yes, you have to put the password of your technical ldap user in plaintext into `default_authtok`.
  * If set to no, you can leave a dummy password in `default_authtok`, run the playbook once, edit `/etc/sssd/sssd.conf` once and from then on, `sssd.conf` will be left alone by ansible (**to be improved**).

* `potos_sssd_ldap` is a dict with the following entries:

  * `uri`: comma separated list of ldap servers, eg `"ldaps://ucsmas.example.net:7636,ldaps://ucssmb.example.net:7636"`
  * `search_base`: ldap search base, e.g. `"dc=example,dc=net"`
  * `default_bind_dn`: ldap read only user, e.g. `"uid=ropam,cn=users,dc=example,dc=net"`
  * `default_authtok`: password of `default_bind_dn`, see `potos_sssd_force_template` above
  * `access_provider_simple_allow_groups`: an ldap group name. If set, only members of that group are authenticated successfully.

* `potos_sssd_pam_mount` is a dict with an entry for each type of mounts. Currently only `cifs` supported.

* `potos_sssd_pam_mount.cifs` is a list of samba shares to be automatically mounted when doing a password login with an ldap user. Each entry is a dict with the following entries:

  * `server`: The SMB server name, e.g. `"smb.example.net"`
  * `path`: The name of the share on the server. Use `"%(USER)"` to refer to the "home" directory.
  * `mountpoint`: The absolute path where to mount the share to. Use e.g. `"/home/%(USER)/share"`
  * `options`: The mount options. You should use at lease `"nosuid,nodev,noexec"` on SMB shares.
  * `sgroup`: The mount is only attempted when the user is in that group. Should match the value of `potos_sssd_ldap.access_provider_simple_allow_groups` (at least in my setup).

The default variables are defined in [defaults/main.yml](./defaults/main.yml):

```yaml
---

potos_sssd_force_template: yes
```

Another option is to use `ansible-doc` to read the argument specification:

```sh
ansible-doc --type role -r . main ansible-role-potos_template
```

## Requirements

N/A

## License

See [LICENSE](./LICENSE)

## Author Information

[Project Potos](https://github.com/projectpotos)

