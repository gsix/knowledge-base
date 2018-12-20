# Продвинутая настройка ssh для Ubuntu/Debian

Чтобы не держать все хосты в одном `~/.ssh/config` файле, их можно вынести в отдельные подфайлы, например, `~/.ssh/config.d/project`.

## Как это сделать

0. Убеждаемся в том, что наша версия ssh > 7.3p1 в которой появилась эта возможность
    ```
    dpkg -l | grep openssh
    ```
1. В начале файла `~/.ssh/config` добавляем 
    ```
    Include ~/.ssh/config.d/*
    ```
2. Создаём конфиг `~/.ssh/config.d/project`
    ```
    Host    project-db
            HostName 10.0.0.3
            User root
            IdentityFile ~/.ssh/id_rsa

    Host    project-back
            HostName 10.0.0.2
            User root
            IdentityFile ~/.ssh/id_rsa

    Host    project-front
            HostName 10.0.0.1
            User root
            IdentityFile ~/.ssh/id_rsa
    ```            
3. Добавляем автокомплит в bash. Создаём файл `/etc/ssh/ssh_config` и добавляем:
    ```
    _ssh() 
    {
        local cur prev opts
        COMPREPLY=()
        cur="${COMP_WORDS[COMP_CWORD]}"
        prev="${COMP_WORDS[COMP_CWORD-1]}"
        opts=$(grep '^Host' ~/.ssh/config ~/.ssh/config.d/* 2>/dev/null | grep -v '[?*]' | cut -d ' ' -f 2-)

        COMPREPLY=( $(compgen -W "$opts" -- ${cur}) )
        return 0
    }
    complete -F _ssh ssh    
    ```
    После
    ```
     . /etc/bash_completion.d/ssh
    ```
    Теперь хосты должны находится по нажаитю Tab `ssh project-`
