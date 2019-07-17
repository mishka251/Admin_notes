## **Установка и настройка nginx**

**Описание процесса установки nginx для Django приложения в связке с UWSGI и Daphne(для обработки протоколов ws и wss)**

Версия системы: Ubuntu Server 16.04/18.04

`sudo apt-get install nginx` 

Заменить <site_name> на название вашего сайта 
`sudo nano /etc/nginx/sites-available/<site_name>`

Вставляем конфиг, **с соответствующими изменениями для вашего сайта**:   

`server { 

    listen 80; 

    server_name <ip-adres>; 

    server_name <site_name>; 
    # Необязательно, но случается в жизни, когда заголовки становятся больше дефолтных
    large_client_header_buffers 4 16k; 

 

    location = /favicon.ico { access_log off; log_not_found off; } 

    location /static/ { 

        alias /<path_to_staticfiles>/staticfiles/; 

    } 

    location /media/ { 

        alias /<path_to_media>/media/; 

    } 

 

    location / { 

        include         uwsgi_params; 

        uwsgi_pass      unix:/run/uwsgi/<socket_name>.sock; 

        client_max_body_size 10M; 

        uwsgi_read_timeout 60; 

        client_body_buffer_size 10M; 

        client_body_timeout 10000; 

    } 

    location /ws/ { 

       rewrite /ws/(.*) /$1 break; 

       proxy_pass http://localhost:8000; 

       proxy_http_version 1.1; 

       proxy_read_timeout 86400; 

       proxy_set_header Upgrade websocket; 

       proxy_set_header Connection upgrade; 

    } 

    send_timeout                1200; 

    uwsgi_read_timeout          1200; 

}`

Если не планируете работать с веб-сокетами - удаляйте последний location.  

Особенности работы данного конфига с сокетами:
- Когда nginx работает в режиме reverse-proxy он благополучно отбразывает заголовки, их нужно явно прописывать  
- Конфиг предполагает, что к приложению идут и http и ws запросы. НО! nginx не допускает одинаковых location, так что пришлось выкручиваться.
Когда на клиенте формируется запрос на открытие соединения по ws, в урле я так и указываю:
ws://<site_name>/**ws**/<ws_request>. ОБычные же http запросы продолжают работать, и можноспокойно обращаться, например:
http://<site_name>/admin/
- Зачем тогда обрезать **ws** rewrit-ом? Отличный вопрос, вы можете не обрезать, я так делаю, из за особенностей моего routing - a 
в проекте

Создаем симплинк этого файла в sites-enabled:  
`sudo ln -s /etc/nginx/sites-available/<site_name> /etc/nginx/sites-enabled`

Проверяем конфиг на валидность:  
`sudo nginx -t`

Щедро даем права:  
`sudo ufw allow 'Nginx Full'`  

Разрешаем работать:  
`sudo systemctl enable nginx`  

Запускаем:  
`sudo systemctl restart nginx`

В последтвии, если поправили конфиги, достаточно reload:
`sudo nginx -s reload`
