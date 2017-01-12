clamav_mirror
=============

Ansible role which helps to install and configure ClamAV mirror.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```
---

- name: ClamAV mirror server
  hosts: clamav-server
  vars:
    # Use more local DB server (e.g from UK)
    clamav_mirror_script_db: db.uk.clamav.net
    # Create new location for the DB files
    nginx_vhost_config__default__custom:
      - "location /db":
        - alias {{ clamav_mirror_script_mirrordir }}
        - autoindex on
  roles:
    - clamav_mirror
    - nginx

- name: ClamAV client
  hosts: clamav-client
  vars:
    # Point freshclam to our mirror server
    clamav_freshclam_database_mirror:
      - 192.168.151.10
    clamav_freshclam_config__custom:
      DatabaseCustomURL:
        # Point to the /db location
        # (Freshclam cannot handle HTTPS!)
        - http://192.168.151.10/db/bytecode.cvd
        - http://192.168.151.10/db/daily.cvd
        - http://192.168.151.10/db/main.cvd
  roles:
    - clamav
```


Role variables
--------------

```
# Mirror script dependencies:
clamav_mirror_script_deps:
  - clamav
  - python-dns

# File name of the mirror script
# (taken from: https://github.com/vrtadmin/clamav-faq/mirrors/)
clamav_mirror_script: clamavmirror.py

# ClamAV DB mirror
clamav_mirror_script_db: database.clamav.net

# ClamAV TXT record server
clamav_mirror_script_txt: current.cvd.clamav.net

# All the *dir variables bellow can also be accompanied with
# *dir_{mode,owner,group} variables which allow to configure the directory
# owner, group and mode.
# (e.g "clamav_mirror_script_mirrordir_owner: nginx" will set the nginx user as
# the owner)

# Directory where to store the script
clamav_mirror_script_bindir: /usr/local/bin

# Directory where to store intermediate downloads
clamav_mirror_script_workdir: /var/spool/clamav_mirror

# Directory where to store the mirrored files
clamav_mirror_script_mirrordir: /srv/www/clamav_mirror

# Directory where to store the lockfile
clamav_mirror_script_lockdir: /var/lock/subsys

# User which should own the mirrored files
clamav_mirror_script_user: nginx

# Group which should own the mirrored files
clamav_mirror_script_group: nginx

# Crontab time settings
clamav_mirror_cron_minute: "*/30"
clamav_mirror_cron_hour: "*"
clamav_mirror_cron_weekday: "*"

# Crontab job
clamav_mirror_cron_job: >-
  "{{ clamav_mirror_script_bindir }}/{{ clamav_mirror_script }}"
  -w "{{ clamav_mirror_script_workdir }}"
  -d "{{ clamav_mirror_script_mirrordir }}"
  -l "{{ clamav_mirror_script_lockdir }}"
  -r {{ clamav_mirror_script_txt }}
  -a {{ clamav_mirror_script_db }}
  -u {{ clamav_mirror_script_user }}
  -g {{ clamav_mirror_script_group }}
```


Dependencies
------------

- [`clamav`](https://github.com/jtyr/ansible-clamav) (optional)
- [`nginx`](https://github.com/jtyr/ansible-nginx) (optional)


License
-------

MIT


Author
------

Jiri Tyr
