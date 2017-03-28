# Ansible NSM tools for arch linux

## Bro

[https://www.bro.org/](https://www.bro.org/)

* Installs dependencies (cmake, swig, cron)
* Installs bro from github [https://github.com/bro/bro.git](https://github.com/bro/bro.git)
* Configures bro
* Runs 'broctl install' && 'broctl restart'
* Adds cron entry to run 'broctl cron' every 5 minutes

Configure at var/localhost.yml
```
bro_interface: '{{ ansible_default_ipv4.interface }}'
bro_log_dir: /var/log/bro
```

Note:

You'll want to add:
```
export PATH=/usr/local/bro/bin:$PATH
```
to your ~/.bashrc or ~/.zshrc

## Snort

### Pulledpork

### Barnyard2

### Local MySQL (for storing alerts?)

