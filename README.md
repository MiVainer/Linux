# Запуск и дебаг сервиса  
Копируем файл lf-app.service в каталог /etc/systemd/system.  

##Создаём системного пользователя.  
`sudo useradd --system --user-group --shell /usr/sbin/nologin lfapp`  

##Создаём директорию и файлы сервиса, накидываем права  
```bash
sudo install -d -m 0750 -o root -g lfapp /opt/lf-app  
sudo install -d -m 0750 -o root -g lfapp /opt/lf-app/public  
echo 'Linux Factory working' | sudo tee /opt/lf-app/public/index.html >/dev/null  
sudo chmod 640 /opt/lf-app/public/index.html```  

##Systemd пересчитывает конфигурационные файлы:  
`sudo systemctl daemon-reload`  

##Запуск сервиса:  
`sudo systemctl start lf-app`  

##Проверка работоспособности и просмотр логов:  
`sudo systemctl status lf-app`  

##Для запуска после перезагрузки выполни:  
`sudo systemctl enable --now lf-app.service`  

##Проверяем работает ли сервис:  
`bash check.sh`  
`echo $?`  
если вывело 0 значит все ок.  


##Возможные инциденты и пути решения:  
1. Проблемы с правами на каталогах или файлах сервиса  
`curl -f http://127.0.0.1:8080` выводит сообщение: curl: (22) The requested URL returned error: 404   

###Диагностика:  
`systemctl status lf-app --no-pager -l`   
в логах показывает ошибку 404 файл не найден  
`namei -l /opt/lf-app/public/index.html`  
к каталогу public отказано в доступе, проверяем и корректируем права каталога lf-app  
###Решение:
корректируем права доступа  
`sudo chmod 750 /opt/lf-app/`  
###Проверка:  
`bash check.sh`  
`echo $?`  

2. Порт занят другим сервисом  
вывод команды `systemctl status lf-app.service` указывает на то что сервис не работает, и возникли ошибки  

###Диагностика:  
`pgrep -a -u lfapp python3`  
вывод пуст  

`sudo ss -lntp 'sport = :8080'`   
указывает что порт 8080 занят другим процессом  

### Решение:
`kill PID`  
мягко уничтожаем процесс  

`systemctl start lf-app.service`  
оживляем сервис  

###Проверка:  
`bash check.sh`  
`echo $?`  
