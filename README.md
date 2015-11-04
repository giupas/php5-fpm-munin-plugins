### This plugin has been forked to show more pools in the same graphs.

### To install on Ubuntu:

````
cd /usr/share/munin/plugins
git clone https://github.com/giupas/php5-fpm-munin-plugins.git
chmod +x php5-fpm-munin-plugins/phpfpm_*
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_average /etc/munin/plugins/phpfpm_average
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_connections /etc/munin/plugins/phpfpm_connections
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_memory /etc/munin/plugins/phpfpm_memory
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_status /etc/munin/plugins/phpfpm_status
ln -s /usr/share/munin/plugins/php5-fpm-munin-plugins/phpfpm_processes /etc/munin/plugins/phpfpm_processes
service munin-node restart
```

For this version of the plugin to work, you must set pool names to: www-POOLNAME and status to /status-POOLNAME.

For the phpfpm_status and phpfpm_connections plugins, you'll need to enable the _status_ feature included in versions 5.3.2+ of PHP-FPM. The directive can be found in the php5-fpm.conf file:
```
pm.status_path = /status-POOLNAME
```

Jérôme Loyet from the Nginx forums provided some useful insight on how to get this working with Nginx. You'll essentially set up the status location directive like this:

```
location ~ ^/(status|ping)-POOLNAME$ {
    include fastcgi_params;
    fastcgi_pass backend;
    fastcgi_param SCRIPT_FILENAME $fastcgi_script_name;
    allow 127.0.0.1:9000;
    allow stats_collector.localdomain;
    allow watchdog.localdomain;
    deny all;
}
```

You'll need to make sure that from within your box, you can curl /status with # curl http://localhost/status. You should get a response similar to this:

```
accepted conn:    40163
pool:             www
process manager:  dynamic
idle processes:   6
active processes: 0
total processes:  6
```

With apache you can configure a virtual host or directory to allow connection only from local ip and use ProxyPassMatch to call the php-fpm
```
	ProxyPassMatch ^/(ping|status)-POOL1$ fcgi://127.0.0.1:9000
	ProxyPassMatch ^/(ping|status)-POOL1$ fcgi://127.0.0.1:9001
	ProxyPassMatch ^/(ping|status)-POOL3$ fcgi://127.0.0.1:9002
	...
```

Configure the munin-node

```
[phpfpm_*]
env.phpbin php-fpm
env.pools www-POOL1 www-POOL2 www-POOL3 ...
env.urls POOL1 POOL2 POOL3 ...
env.ports 80 80 80 ...
```

#### Note: 

The phpfpm_status plugin is particularly useful if you're using dynamic or on-demand process management. You can choose static, dynamic or on-demand in the php5-fpm.conf.

### Environment variables

* env.url: Set a custom url, defaults to _http://127.0.0.1/status_
* env.ports: Set a custom port, defaults to _80_
* env.phpbin: Set a custom php binary name, defaults to _php5-fpm_
* env.phppool: Set a custom php pool, defaults to www

#### Requirements:

libwww-perl