# Man

📟 Привет. Меня зовут [NikolasMelui][nikolasmelui], я занимаюсь разработкой. В последнее время стал замечать, что с регулярностью примерно раз в 2-3 недели гуглю то, что делаю редко, но делаю. Чтобы перестать искать то, что уже находил, решил сделал свой мануал, в котором, со временем, будет всё. Пока на русском, но скоро будет перевод.

## VPS

### Первичное обновление системы и установка пакетов

```bash
sudo apt update && sudo apt upgrade && sudo apt autoclean && sudo apt autoremove
```

```bash
sudo apt install vim git tig curl inxi wget xclip thefuck tmux ranger build-essential checkinstall libssl-dev
```

### LOCALE

- Проверяем установленные локали

```bash
sudo locale -a
```

- Для корректного отображения русского и английского языков в системе нужно поставить недостающие локали (решается обычным реконфигом)

```bash
sudo dpkg-reconfigure locales
```

- Добавляем локаль в конфигурационный файл ***/etc/default/locale***

```bash
LC_ALL="en_US.UTF-8"
```

### SSH_LOCAL

- На локальной машине заходим в

```bash
cd ~/.ssh
```

- Создаем новый ssh ключ для нашего сервера

```bash
ssh-keygen -t rsa -b 4096
```

- Запускаем ssh-agent (если собираемся использовать passphrase)

```bash
eval“$(ssh-agent -s)”
```

- Добавляем наш ключ в агента

```bash
ssh-add -K ~/.ssh/curkey_rsa
```

- Добавляем в конфигурационный файл настройки подключения к серверу

```bash
vim ~/.ssh/config
```

### SSH_REMOTE

- Заходим на сервер по ssh от root юзера через ввод пароля

```bash
ssh root@yourserveradress.com
```

- Генерируем SSH ключ (для доступа в GitLab)

```bash
ssh-keygen
```

- Добавляем нового юзера с именем manager (чтобы не работать из под рута)

```bash
adduser manager
```

> Если что-то пошло не так, удаляем юзера manager и пересоздаем его

```bash
deluser manager
adduser manager
```

> В случае необходимости меняем пароль

```bash
passwd manager
```

- Добавляем юзера manager в группу админов

```bash
usermod -aG sudo manager
```

- Проверяем id юзера

```bash
id manager
```

- Логинемся юзером manager через sudo

```bash
sudo su - manager
```

- Создаем юзеру папку ssh

```bash
sudo mkdir ~/.ssh
```

- Определяем права доступа

```bash
chmod 700 ~/.ssh
```

- Создаем файл для авторизованных ключей

```~/.bash
ssh/authorized_keys

```

- Копируем с локальной машины публичный SSH ключ через утилиту ***ssh-copy-id***

```bash
ssh-copy-id -i ~/.ssh/projectname_manager_rsa.pub manager@yourserveradress.com
```

- Меняем права доступа на файл для авторизованных ключей

```bash
chmod 600 ~/.ssh/authorized_keys
```

- Перезагружаем ssh службу

```bash
sudo service ssh restart
```

- Выходим из сессии юзера и сессии рута, отключаемся от сервера и заходим по ssh с использованием ключа
- Заходим в конфиг файл и закрываем доступ для рута (и возможность заходить на сервер через введение пароля)

```bash
sudo nano /etc/ssh/sshd_config
```

> PermitRootLogin no
> PasswordAuthentication no

- Перезагружаем наш sshd процесс

```bash
sudo systemctl reload sshd
```

### FIREWALL

- Добавляем правило для нашего ssh соединения (нужно указать ваш порт)

```bash
sudo ufw allow 22
```

- Активируем firewall

```bash
sudo ufw enable
```

- Закрываем небезопасные порты (указываем нужные порты)

```bash
sudo ufw deny 3003
```

- Открываем безопасные порты (указываем нужные порты)

```bash
sudo ufw allow 80
```

### GIT

- Генерируем новый ssh ключ с почтой нашего проекта

```bash
ssh-keygen -t rsa -b 4096 “curproject@gmail.com”
```

- Запустим агента

```bash
eval $(ssh-agent -s)
```

- Добавим наш новый ключ в ssh agent

```bash
ssh-add ~/.ssh/curproject_rsa
```

- Внесём необходимые изменения в ssh config файл, чтобы не было проблем с доступом до социальной сети git (Github, Gitlab)

```bash
vim ~/.ssh/config
```

  Host gitlab.com
  Preferredauthentications publickey
  IdentityFile ~/.ssh/curproject_rsa

- Копируем сгенерированный ssh ключ curproject_rsa.pub, после чего добавляем его в раздел Deploy Keys нужной нам социальной сети git

(Settings->Repository->Deploy Keys)

- Клонируем проект git в папку проектов

```bash
cd ~ && mkdir projects && cd projects && git clone git@gitlab.com:company/curproject.git
```

### FAIL2BAN

- Устанавливаем fail2ban и конфигурируем "локальную тюрьму"

```bash
sudo apt install fail2ban
```

- Конфигурируем fail2ban

```bash
sudo vim /etc/fail2ban/jail.local
```

  [DEFAULT]
  maxretry = 5
  bantime = 86400
  action = firewallcmd-ipset
  [ssh]
  enable = true
  port = ssh
  filter = sshd
  action = iptables[name=ssh, port=ssh, protocol=tcp]
  logpath = /var/log/auth.log
  maxretry = 5
  findtime = 600

- Перезагружаем fail2ban службу

```bash
sudo systemctl reload fail2ban
```

> Посмотреть список забаненных адресов

```bash
sudo iptables -S
```

### ZSH

- Устанавливаем ZSH

```bash
sudo apt install zsh
```

- Устанавливаем ohmyzsh

```bash
curl -L http://install.ohmyz.sh | sh
```

- Определяем zsh как темринал по умолчанию

```bash
chsh -s /bin/zsh
```

- Скачиваем и устанавливаем тему spaceship (можно и другую, но зачем?)

```bash
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
```

```bash
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
```

- Вносим необходимые изменения в ~/.zshrc файл

```bash
vim ~/.zshrc
```

```bash
  # Theme
  ZSH_THEME="spaceship"

  # Configs Spaceship
  SPACESHIP_HOST_SHOW=true
  SPACESHIP_HOST_SUFFIX=' ssh '
  SPACESHIP_USER_SHOW=always
  SPACESHIP_USER_PREFIX=
  SPACESHIP_DIR_TRUNC_REPO=false
  SPACESHIP_DIR_TRUNC=0
  SPACESHIP_GIT_PREFIX=
  SPACESHIP_EXIT_CODE_SHOW=true
  SPACESHIP_PROMPT_ORDER=(
    host          # Hostname section
    user          # Username section
    dir           # Current directory section
    line_sep      # Line break
    git           # Git section (git_branch + git_status)
    package       # Package version
    node          # Node.js section
    php           # PHP section
    conda         # conda virtualenv section
    pyenv         # Pyenv section
    venv          # virtualenv section
    golang        # Go section
    docker        # Docker section
    kubecontext   # Kubectl context section
    exec_time     # Execution time
    line_sep      # Line break
    vi_mode       # Vi-mode indicator
    jobs          # Background jobs indicator
    exit_code     # Exit code section
    char          # Prompt character
  )

  # Plugins
  plugins=(
    ansible
    colored-man-pages
    common-aliases
    copydir
    copyfile
    extract
    docker
    docker-compose
    docker-machine
    git
    history
    last-working-dir
    lighthouse
    lol
    node
    npm
    per-directory-history
    perms
    redis-cli
    ssh-agent
    tig
    tmux
    tmuxinator
    vi-mode
    vscode
    web-search
    yarn
    z
    zsh-autosuggestions
    zsh-syntax-highlighting
    zsh-completions
    history-substring-search
    zsh_reload
  )
```

- Скачиваем файлы плагинов, которые не входят в базовую установку

```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/guiferpa/aterminal/master/installer.sh)
```

```bash
sudo apt install thefuck
```

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

```bash
git clone https://github.com/zsh-users/zsh-completions ~/.oh-my-zsh/custom/plugins/zsh-completions
```

```bash
git clone https://github.com/zsh-users/zsh-history-substring-search ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-history-substring-search
```

### NODEJS (ubuntu)

- Проверяем, нет ли у нас старых версий nodejs

```bash
dpkg --get-selections | grep node
```

- Если есть - удалим их

```bash
sudo apt purge nodejs
```

- Скачиваем и установливаем nvm

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
```

- Добавляем путь до zsh shell файла в ~/.zshrc файл

```bash
 source $ZSH/oh-my-zsh.sh
 ```

- Делаем экспорт nvm настроек в ~/.zshrc файл

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

- Устанавливаем нужную нам версию nodejs

``` bash
nvm install --lts
```

### NODEJS (macOS) with npm and yarn

- Для начала попробуем удалить nodejs через brew (потому что чаще всего именно через brew ноду и устанавливают)

```bash
brew uninstall node
```

- Теперь удаляем все старые версии nodejs, npm и yarn

```bash
sudo rm -rf /usr/local/{lib/node{,/.npm,yarn,_modules},bin,share/man}/{npm*,node*,yarn*,man1/node*}
sudo rm -rf /opt/local/bin/node /opt/local/include/node /opt/local/lib/node_modules
sudo rm -rf /usr/local/bin/npm /usr/local/bin/yarn /usr/local/share/man/man1/node.1 /usr/local/lib/dtrace/node.d
```

- Скачиваем и установливаем nvm через brew

```bash
brew install nvm
```

- Создаем директорию для nvm

```bash
mkdir ~/.nvm
```

- Добавляем путь до zsh shell файла

```bash
source $(brew --prefix nvm)/nvm.sh
 ```

- Делаем экспорт наших nvm настроек

```bash
export NVM_DIR=~/.nvm
```

- Перезагружаем терминал

```bash
source ~/.zshrc
```

- Проверяем версию nvm

```bash
nvm --version
```

- Устанавливаем нужную версию nodejs

``` bash
nvm install --lts
```

- Делаем установленную версию дефолтной

``` bash
nvm use default
```

- Проверяем путь до глобальных npm модулей 

``` bash
npm root -g
```

- При необходимости меняем путь до глобальных npm модулей (prefix) 

``` bash
npm config edit
```

- Устанавливаем yarn

``` bash
npm i -g yarn
```

- Проверяем, что в системе нет лишних установок nodejs 

``` bash
where node
```

- Проверяем, что в системе используется нужная нам версия nodejs

``` bash
which node
```

- Проверяем, что в системе нет лишних установок npm 

``` bash
where npm
```

- Проверяем, что в системе используется нужная нам версия npm, которая установлена через nvm

``` bash
which npm
```

- При необходимости обновляем npm до последней версии

``` bash
cd ~/.nvm/versions/node/<latest-node-lts-version>/lib/ && npm i npm
```

- Посмотрим текущую версию nodejs 

``` bash
nvm current
```

### NGINX

- Устанавим nginx

```bash
sudo apt install nginx
```

- Проверяем его работоспособность

```bash
systemctl status nginx
```

- Создадим первый виртуальный хост (скопируем и изменим дефолтный конфиг файл)

```bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/curproject.com
```

- Подготовим директорию для лог- и error- файлов и создадим эти файлы

```bash
sudo mkdir /var/log/nginx/curproject
```

```bash
sudo touch /var/log/nginx/curproject/access.log
```

```bash
sudo touch /var/log/nginx/curproject/error.log
```

- Настроим виртуальный хост

```bash

  # SSL

  server {
    server_name curproject.com www.curproject.com;
    return 301 https://curproject.com$request_uri;
  }

  server {
    listen 443 ssд http2;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/curproject.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/curproject.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    server_name www.curproject.com.ru;
    return 301 https://curproject.com$request_uri;
  }

  server {
    listen 443 ssl http2;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/curproject.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/curproject.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    server_name curproject.com;
    access_log /var/log/nginx/curproject/access.log;
    error_log /var/log/nginx/curproject/error.log;

    location / {
      proxy_pass http://0.0.0.0:0000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_connect_timeout 120;
      proxy_send_timeout 120;
      proxy_read_timeout 180;
    }

    location /secure/ {
      proxy_pass http://0.0.0.0:0000/secure/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_connect_timeout 120;
      proxy_send_timeout 120;
      proxy_read_timeout 180;
      auth_basic "Unauthorized";
      auth_basic_user_file /etc/nginx/.htpasswd/curproject.com;
    }
  }
```

```bash

  # Without SSL

  server {
    server_name www.curproject.com;
    return 301 http://curproject.com$request_uri;
  }

  server {
    listen 80;
    server_name curproject.com;
    access_log /var/log/nginx/curproject/access.log;
    error_log /var/log/nginx/curproject/error.log;

    location / {
      proxy_pass http://0.0.0.0:0000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_connect_timeout 120;
      proxy_send_timeout 120;
      proxy_read_timeout 180;
    }

    location /secure/ {
      proxy_pass http://0.0.0.0:0000/secure/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_connect_timeout 120;
      proxy_send_timeout 120;
      proxy_read_timeout 180;
      auth_basic "Unauthorized";
      auth_basic_user_file /etc/nginx/.htpasswd/curproject.com;
    }
  }
```

- Добавим симлинк созданного виртуального хоста (активиурем виртуальный хост)

```bash
sudo ln -s /etc/nginx/sites-available/curproject.com /etc/nginx/sites-enabled/
```

- Удалим симлинк на дефолтный конфиг (деактивируем дефолтный виртуальный хост)

```bash
sudo rm /etc/nginx/sites-enabled/default
```

- Добавим некоторые важные настройки nignx в ***/etc/nginx/nginx.conf***

```bash
server_names_hash_bucket_size: 64;
```  

- Добавим наше доменное имя в файл ***/etc/hosts***

```bash
127.0.0.1 curproject.com
```

- Если нам необходима SSL конфигурация - установим certbot

```bash
sudo apt install python-certbot-nginx
```

- Добавим сертификаты для выбранных доменных имен ***(доменное имя с www и без него)***

```bash
sudo certbot --nginx -d curproject.com -d www.curproject.com
```

### PM2

- Устанавливаем PM2

```bash
npm i -g pm2
```

- Запускаем проект

```bash
pm2 start ecosystem.config.js
```

***https://pm2.keymetrics.io/***

### MONGO

- Устанавливаем mongo

```bash
sudo apt install mongodb
```

- Проверяем статус

```bash
sudo systemctl status mongodb
```

- Создаём нового root юзера

```bash
mongo
```

```bash
use admin
```

```bash
db.createUser({user:"root", pwd:”rootpassword”, roles:[{role:”root”, db:"admin”}]})
```

- По умолчанию монга открыта для всех, добавим обязательный флаг —auth

```bash
sudo vim /lib/systemd/system/mongodb.service
```

ExecStart=/usr/bin/mongod --auth --unixSocketPrefix=${SOCKETPATH} --config ${CONF} $DAEMON_OPTS

- Перезапустим службы и проверим их статус

```bash
systemctl daemon-reload
```

```bash
sudo systemctl restart mongodb
```

```bash
sudo systemctl status mongodb
```

- Зайдем в mongo от имени созданного root

```bash
mongo -u "root" -p --authenticationDatabase "admin”
```

- Создадим и перейдем в нужную нам базу

```bash
use curproject_db
```

- Создадим нового юзера-админа для только что созданной базы

```bash
db.createUser( { user: “curproject_manager", pwd: “curpassword”, roles: [ { role: "readWrite", db: “curproject_db" } ] } )
```

- Создадим папку для резервных копий

```bash
mkdir /home/curuser/backups/curproject
```

- Сделаем копию и проверим её содержимое

```bash
mongodump -u curproject_manager -p curprojectpassword --db curproject_db --out /home/curuser/backups/curproject`date +"%m-%d-%y"`
```

- Создадим задачу в cron для автоматических бэкапов каждую ночь

```bash
sudo crontab -e
```

```bash
3 3 * * * mongodump -u curproject_manager -p curprojectpassword --db curproject_db --out  /home/curproject_manager/backups/curproject/curproject`date +"%m-%d-%y"`
```

- И добавим задачу, которая будет чистить диск от старых бэкапов

```bash
3 1 * * * find /home/curproject_manager/backups/curproject/ -mtime +7 -exec rm -rf {} \;
```

### POSTGRESQL

#### MacOS:

- Устанавливаем postgres через brew

```bash
brew install postgresql
```

- Запускаем процесс postgres

```bash
brew services start postgresql
```

#### Ubuntu:

- Устанавливаем postgres

```bash
sudo apt install postgresql
```

- Проверяем статус

```bash
sudo systemctl status postgresql
```

- Проверяем прослушиваемые порты postgres

```bash
netstat -nlt
```

- Входим в psql от имени postgres

```bash
sudo -u postgres psql
```
- Создаем новую базу

```bash
CREATE DATABASE db_name;
```

- Создаем нового юзера с паролем

```bash
CREATE USER user_name WITH ENCRYPTED PASSWORD 'password';
```

- Присваиваем юзера к базе и даём ему все права

```bash
GRANT ALL PRIVILEGES ON DATABASE db_name TO user_name;
```

- При необходимости удаляем базу

```bash
DROP DATABASE db_name;
```

- Создаем дамп базы

```bash
sudo -u postgres pg_dump db_name > db_name_dump.bak
```

- Восстанавливаем базу из дампа

```bash
sudo -u postgres psql -d db_name -f db_name_dump.bak
```

- Если необходимо, открываем доступы для всех коннекшенов в конфиг файле **/var/lib/pgsql/<version>/data/postgresql.conf**

```bash
listen_addresses = '*'
```

- Также, если необходимо, добавляем наш айпи с конфигами в разрешенные хосты в конфиг файле **/var/lib/pgsql/<version>/main/pg_hba.conf**

```bash
host    all             my_login         10.0.0.1/32            md5
```

### Новое разрешение экрана в Ubuntu

```bash
xrandr --newmode "2560x1080"  230.37  2560 2728 3000 3440  1080 1081 1084 1118  -HSync +Vsync
```

```bash
xrandr --addmode HDMI-2 2560x1080
```

```bash
xrandr --output HDMI-2 --mode 2560x1080
```

#### Bluetooth Ubuntu:

- Заупскаем bluetoothctl

```bash
sudo bluetoothctl
```

- Включаем устройство (нужно убедиться, что оно установлено)

```bluetoothctl
power on
```

- Регистрируем агента

```bluetoothctl
agent agent_name
```

- Делаем агента агентом по-умолчанию

```bluetoothctl
default-agent
```

- Даём устройству возможность создавать пару

```bluetoothctl
pairable on
```

- Включаем сканирование (ищем устройства, с которыми можно сделать пары)

```bluetoothctl
scan on
```

- Добавляем найденное сканированием устройство в "доверенные" (для обращения к устройству нужно использовать его MAC-адрес)

```bluetoothctl
trust 28:C6:3F:96:CE:12
```

- Создаем с устройством пару

```bluetoothctl
pair 28:C6:3F:96:CE:12
```

#### License

MIT License

Copyright (c) 2018 NikolasMelui

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

**From developers 2 developers.**
[NikolasMelui][nikolasmelui]

[//]: # "These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax"
[nikolasmelui]: https://github.com/NikolasMelui
