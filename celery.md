##Celery service on Ubuntu Server 16.04/18.04

Создаем конфиг Celery, который значительно удовнее редачить, нежели в конфиге сервиса. У меня он лежит в /etc/default/celeryd   

`nano /etc/default/celeryd`      

Вставляем конфиг, заменяя на свои пути и названия:   

```
CELERYD_NODES=2 
 

# Absolute or relative path to the 'celery' command: 

CELERY_BIN="/home/<path_to_venv>/venv/bin/celery" 


# App instance to use 

CELERY_APP="<your_app_name>" 

# or fully qualified: 

#CELERY_APP="proj.tasks:app" 

 

# Where to chdir at start. Where your manage.py is... 

CELERYD_CHDIR="/home/<path_to_your_manage.py>" 

 

# Extra command-line arguments to the worker 

CELERYD_OPTS=" --concurrency=8" 

# Configure node-specific settings by appending node name to arguments: 

#CELERYD_OPTS="--time-limit=300 -c 8 -c:worker2 4 -c:worker3 2 -Ofair:worker1" 

 
# Set logging level to DEBUG 

CELERYD_LOG_LEVEL="ERROR" 


# %n will be replaced with the first part of the nodename. 

CELERYD_LOG_FILE="/var/log/celery/%n%I.log" 

CELERYD_PID_FILE="/var/run/celery/%n.pid" 

 

# Workers should run as an unprivileged user. 

#   You need to create this user manually (or you can choose 

#   a user/group combination that already exists (e.g., nobody). 

CELERYD_USER="<your_user>" 

#CELERYD_GROUP="celery" 

 
# If enabled pid and log directories will be created if missing, 

# and owned by the userid/group configured. 

CELERY_CREATE_DIRS=1 

```
Немного шаманства для дирректории логов:   
```
sudo mkdir /var/log/celery
chown <your_user> -R /var/log/celery
chmod +x -R celery
```
Может показаться бредом, но мне удобно когда я могу найти все логи моих сервисов в одном месте: /var/log   
Вы же можете избежать этого и складировать ваши логи в другое место и не мучиться с правами.
####Celery должен запускаться от того же пользователя что и uwsgi!   
Если хотите, делайте для каждого своего сервиса отдельного пользователя,но эти двое должны работать от одного! 

Создаем демон сельдерея:   
`sudo nano /etc/systemd/system/celery.service`   

Вставляем конфиг, так же с подстановкой собственных значений:   

```
[Unit] 

Description=Celery Service 

After=network.target 

 

[Service] 

Type=forking 

User=sysadmin 

Restart=always 

 

Environment="DJANGO_SETTINGS_MODULE=<your_project>.settings" 

EnvironmentFile=/etc/default/celeryd 

WorkingDirectory=/home/<path_to_your_project/> 

ExecStart=/bin/sh -c '${CELERY_BIN} multi start ${CELERYD_NODES} \ 

  -A ${CELERY_APP} --pidfile=${CELERYD_PID_FILE} \ 

  --logfile=${CELERYD_LOG_FILE} --loglevel=${CELERYD_LOG_LEVEL} ${CELERYD_OPTS}' 

ExecStop=/bin/sh -c '${CELERY_BIN} multi stopwait ${CELERYD_NODES} \ 

  --pidfile=${CELERYD_PID_FILE}' 

ExecReload=/bin/sh -c '${CELERY_BIN} multi restart ${CELERYD_NODES} \ 

  -A ${CELERY_APP} --pidfile=${CELERYD_PID_FILE} \ 

  --logfile=${CELERYD_LOG_FILE} --loglevel=${CELERYD_LOG_LEVEL} ${CELERYD_OPTS}' 

 

[Install] 

WantedBy=multi-user.target 
```

Дальше стандартно:   
```
sudo systemctl daemon-reload
sudo systemctl start celery
```
