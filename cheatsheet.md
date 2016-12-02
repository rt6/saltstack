# Saltstack

### Setup new minion and connect to existing master
===

1) Download and install salt
```sh
curl -o bootstrap-salt.sh -L https://bootstrap.saltstack.com
sudo sh bootstrap-salt.sh
sudo sh bootstrap-salt.sh git v2016.3.2
# check version
sudo salt --version
```

2) Add minion grains
`/etc/salt/grains`:
```yaml
roles:
    - web-server
```

3) On the salt-master, get the salt-master's fingerprint
```sh
$ salt-key -F master
```
3) Add salt-master ip and fingerprint to minion
`/etc/salt/minion`:
```sh
master: 1.1.1.1
master_finger: 'aa:bb:cc:dd:ee:......xx:yy:zz'
```

4) Get the minion's own fingerprint
```sh
$ salt-call key.finger --local
```

4) Try to connect to master (this won't work because master has not accepted the new minion)
```sh
$ sudo salt-minion -l debug
```

5) on the master request for the minion's fingerprint. note: can be found in /etc/salt/minion_id
```sh
$  salt-key -f <minion-id>
```

6) if both fingerprints match , then you can accept the minions key and this will connect the master with the minion
```sh
# accept minion's key
$ salt-key -a <minion-id>

# now the minion will be in the accepted list (green)
$ sudo salt-key -L
```

7) make sure salt-minion has started on the minion
```sh
# -d will run daemon in background without verbose logging
$ sudo salt-minion -d
$ sudo salt-minion -l debug
```
8) test connection on salt-master to minion
```
$ sudo salt '*' test.ping
```

### Only show changes
===
```sh
sudo salt '*test*' state.highstate test=True --state-output=changes
sudo salt '*test*' state.highstate --state-output=changes
```

### Using dictionaries in pillar files (to manage configs easier)
===
**srv/pillar/config.sls**

```yaml
websites:
    web1:
        url: 'www.web1.com'
        ip: '1.2.3.4'
        homepage: 'path/to/home1.html'
    web2:
        url: 'www.web2.com'
        ip: '1.2.3.5'
        homepage: 'path/to/home2.html'
```

**srv/salt/app1/init.sls**

`site` will contain the key (ie. web1 or web2)
`args` will be the dictionary and values accessible by `args.url` and `args.ip` inside the for loop
```jinja
{% for site, args in pillar.get('websites', {}).items() %}
{% set myurl = args.url %}

{{args.homepage}}:
  file.managed:
    content_pillar: homepage_content_pillar

{% endfor %}
```

###
