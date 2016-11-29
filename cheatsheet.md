# Saltstack

### Only show changes
```sh
sudo salt '*test*' state.highstate test=True --state-output=changes
sudo salt '*test*' state.highstate --state-output=changes
```

### Using dictionaries in pillar files (to manage configs easier)

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

`route` will contain the key (ie. web1 or web2)
`args` will be the dictionary and values accessible by `args.url` and `args.ip` inside the for loop
```jinja
{% for route, args in pillar.get('websites', {}).items() %}
{% set myurl = args.url %}

{{args.homepage}}:
  file.managed:
    content_pillar: homepage_content_pillar

{% endfor %}
```

###
