### Установка и настройка UWSGI для Django приложения

Подготовка   
`sudo apt-get install python3-dev`  
`sudo -H pip3 install uwsgi`  

Ставлю глобально, потому что на одном серваке вполне могут крутиться несколько проектов. Если кажется не правильным - ставьте в venv каждого проекта, только с путями внимательно)   
   
Создание папки и первого конфига проекта  
`sudo mkdir -p /etc/uwsgi/sites`  
`sudo nano /etc/uwsgi/sites/<project_name>.ini`   

Копируем в `project_name>.ini`, внимательно подставляя **свои** значения.    
```
[uwsgi]
project = <project_name>
uid = <your_user>
base = /home/%(uid)

chdir = %(base)/%(project)
home = %(base)/venv/%(project)
module = %(project).wsgi:application

master = true
processes = 5

socket = /run/uwsgi/%(project).sock
chown-socket = %(uid):www-data
chmod-socket = 660
vacuum = true
```   

Создаем сервис
`sudo nano /etc/systemd/system/uwsgi.service`

Заполняем, опять же очень внимательно. Внимание, связь с nginx не по 8000-му порту, а по сокету!   
```
[Unit]
Description=uWSGI Emperor service

[Service]
ExecStartPre=/bin/bash -c 'mkdir -p /run/uwsgi; chown <your_user>:www-data /run/uwsgi'
ExecStart=/usr/local/bin/uwsgi --emperor /etc/uwsgi/sites
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

Релоад демонов, и запуск.   
`sudo systemctl daemon-reload`  
`sudo systemctl enable uwsgi`   
`sudo systemctl start uwsgi`
 
Обязательно проверяем как работает, ну и если заломилось, сначала можно здесь посмотреть)  
`sudo systemctl status uwsgi`   
`sudo journalctl -u uwsgi`
