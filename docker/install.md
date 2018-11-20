### Ставим `Docker` и `Docker compose` на Ubuntu 17.04 (zesty) x64 

  * [Установка](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#uninstall-old-versions) Docker
  
    > wget -qO- https://get.docker.com/ | sh

    ```
    apt-get install -y apt-transport-https ca-certificates curl software-properties-common --no-install-recommends
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    # apt update # /etc/apt/sources.list
    apt install -y docker-ce --no-install-recommends
    ```

    
  * [Установка](https://docs.docker.com/compose/install/#install-compose) Docker compose

    ```
    curl -L https://github.com/docker/compose/releases/download/1.23.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    ```
    
    > Отсутствие прав к `docker` привдетё к тому что `Docker compose` не найдёнт `docker`. В этом случае можно попробовать выполнить действие с `sudo`.

### Add deploy user

*  Добавить пользователя `deploy` в группу `docker` 

    > `deploy` - пользователь, от лица которого будет проходить запуск цепи в Ubuntu

    ```
    usermod -aG docker deploy
    ```

* Add your key to `authorized_keys`:

```
mkdir /home/deploy/.ssh
nano /home/deploy/.ssh/authorized_keys
```

* Create `/etc/sudoers.d/deploy`:

```
echo 'deploy ALL=(ALL:ALL) ALL' > /etc/sudoers.d/deploy
```

* Change `/etc/ssh/sshd_config`:

```
nano /etc/ssh/sshd_config
```

- `PermitRootLogin` no
- `PasswordAuthentication` no

* Restart `ssh` daemon:

```
service ssh restart
```

### Авторизоваться в `docker`

* Создать `Access Token` для доступа к docker-реестру:

  * Зайти в настройки своего профиля на [gitlab.com](https://gitlab.com/profile);
  
  * Перейти в раздел [Access Tokens](https://gitlab.com/profile/personal_access_tokens);
  
  * Указать `имя`;
  
  * Ниже, в `Scopes` поставить 2 галочки: `api` и `read_registry`;
  
  > `api`необходимо чтобы задвигать (push) образы в реестр (ci)
  
  * Нажать кнопку создать `Access Token`;

##### Авторизация

  ```
  docker login registry.gitlab.com
  ```
  
  > Username - `имя`, указанное при создании `Access Token`

  > Password - сам `токен`
