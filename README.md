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
[program:web]
command=/home/mysite/.virtualenvs/mysite/bin/gunicorn <Project>.wsgi --bind 127.0.0.1:8000
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=mysite
directory=/home/mysite/www/
stdout_logfile=/var/log/mysite.log
stderr_logfile=/var/log/mysite.log
redirect_stderr=true

```

And run:
```
$ sudo supervisorctl reread
$ sudo supervisorctl update
```

#### Nginx config

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
Restart nginx:
```
$ sudo systemctl restart nginx.service
```

And run `collectstatic` for static files serving:
```
$ python manage.py collectstatic
```



