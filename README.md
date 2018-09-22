# DjangoNginxGunicornSupervisor
Deploy Django with Gunicorn + Nginx + Supervisor

#### Install [Gunicorn](http://gunicorn.org/)

```
$ pip install Gunicorn
```

#### Install [Supervisor](http://supervisord.org)
```
$ sudo apt-get install supervisor
```


#### Supervisor config
Please create a file in `/etc/supervisor/conf.d/`:
```
$ sudo vim /etc/supervisor/conf.d/mysite.conf
```

and add below lines:

```
[program:mysite]
command=/home/mysite/.virtualenvs/mysite/bin/gunicorn mysite.wsgi \
    --bind 0.0.0.0:8000 \
    --access-logfile /home/mysite/log/guni_access.log \
    --error-logfile /home/mysite/log/guni_errors.log \
    --workers 4 \
    --timeout=600 \
    --graceful-timeout=10 \
    --log-level=DEBUG \
    --capture-output

autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=mysite
directory=/home/mysite/mysite/
stdout_logfile=/home/mysite/log/mysite.log
stderr_logfile=/home/mysite/log/mysite.log
redirect_stderr=true


```

And run:
```
$ sudo supervisorctl reread
$ sudo supervisorctl update
```

#### Nginx config
Remove default config:

```
$ sudo rm -f /etc/nginx/site-available/default /etc/nginx/site-enabled/default
```
And create your config:
```
$ sudo vim /etc/nginx/sites-available/mysite.conf
```

```
server {
  listen  80;
  server_name mysite.com;

  location /static/ {
    autoindex on;
    alias /home/mysite/www/static/;
  }

  location /media/ {
    alias /home/mysite/www/media/;
  }

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_redirect off;

    proxy_pass http://127.0.0.1:8000;
  }
}
```
Add config symbol link in nginx's site-enabled:
```
$ sudo ln -s /etc/nginx/sites-available/mysite.conf /etc/nginx/sites-enabled/mysite.conf
```
Restart nginx:
```
$ sudo systemctl restart nginx.service
```

And run `collectstatic` for static files serving:
```
$ python manage.py collectstatic
```



