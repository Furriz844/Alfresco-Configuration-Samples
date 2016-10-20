##Настройка SSL соединения Alfresco через прокси (NGINX)
Для использования защищенного соединения при работе с Alfresco, а также:

*Доступу по 443 порту

*Редиректу с HTTP на HTTPS

*Редиректу с HOSTNAME на HOSTNAME/alfresco/share

Необходимо сделать следующие шаги:

1) Установить NGINX

Для Ubuntu команда выглядит следующим образом:
```
sudo apt-get install nginx
```
Для RedHat: 
```
yum install nginx
```
2) В моем примере для настройки SSL соединения я использую базовую конфигурацию NGINX, которая содержится в файле:
```
/etc/nginx/sites-available/default 	Для Ubuntu
/etc/nginx/conf.d/default.conf		Для RedHat
```
Очистите данный файл и впишите в него следующую конфигурацию.
```
//Слушаем 80 порт и делаем редирект на 443
server { 
    listen 80;
    return 301 https://$host$request_uri;
}
server {
    listen 443;
    server_name 192.168.1.100;  //Указывается ip адрес или имя сервера, на котором работает веб интерфейс (Share.war)
    ssl_certificate /etc/nginx/keys/rakot.pem; //Ссылка на сертификат в формате PEM
    ssl_certificate_key /etc/nginx/keys/rakot.key.pem; //Ссылка на ключ в формате PEM
    ssl on;
    ssl_session_cache builtin:1000 shared:SSL:10m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;
    access_log /var/log/nginx/access.log;
	rewrite ^/$ /share; //Правило, для того чтобы делать редирект с hostname на hostname/share
    location / {
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      # Fix the “It appears that your reverse proxy set up is broken" error.
      proxy_pass https://localhost:8443;
      proxy_read_timeout 90;
      proxy_redirect https://localhost:8443 https://192.168.1.100:8443; //Также указываем ip адрес или имя сервера, на котором работает Alfresco
    }
  }

```
3) Теперь необходимо отредактировать настройки Alfresco. Для этого необходимо отредактировать файл 
```
path_to_alfresco/tomcat/shared/classes/alfresco-global.properties
```
Отредактируйте следующие параметры:
```
alfresco.host=192.168.1.100 		 //Укажите ip адрес или имя сервера, на котором работает Alfresco
alfresco.port=8443	        		 //Укажите 8443 порт
alfresco.protocol=https	    		 //Укажите https

share.context=share
share.host=192.168.1.100			 //Укажите ip адрес или имя сервера, на котором работает веб интерфейс (Share.war)
share.port=8443						 //Укажите 8443 порт
share.protocol=https				 //Укажите https
```
4) Перезагрузите Alfresco командой:
```
sh path_to_alfresco/alfresco.sh stop
//Дождитесь выполнения предыдущей команды после этого
sh path_to_alfresco/alfresco.sh start
```
5) Перезагрузите NGINX
```
service nginx restart
```
Если все сделано правильно, то при попытке пользователя обратиться к 192.168.1.100 (Адрес или имя сервера), пользователь будет переадресован на https://192.168.1.100/share
