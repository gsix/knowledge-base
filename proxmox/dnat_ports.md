# Проброс портов снаружи в контейнер/виртуалку

**Требования**: SSH доступ к рутовой Proxmox машине

**Решение** В консоль bash на сервере:

    sudo iptables -A PREROUTING -t nat -i enp2s0 -p tcp -d ${PROXMOX_EXTERNAL_IP} --dport ${EXTERNAL_PORT} -j DNAT --to ${INTERNAL_BOX_IP}:${INTERNAL_BOX_PORT}

* `${PROXMOX_EXTERNAL_IP}` - внешний IP самого Proxmox (константа)
* `${EXTERNAL_PORT}` - внешний порт, на который будут подключаться снаружи
* `${INTERNAL_BOX_IP}` - IP-адрес контейнера/виртуалки во внутренней сети
* `${INTERNAL_BOX_PORT}` - порт контейнера/виртуалки, на который будет происходить перенаправление снаружи

# Удаление перенаправление

* Получить список

```
iptables -t nat -nL --line-number
```

Список будет походить на нижеследующий:

```
Chain PREROUTING (policy ACCEPT)
num  target     prot opt source     destination         
1    DNAT       tcp  --  0.0.0.0/0      0.0.0.0      tcp dpt:2425 to:10.211.5.5:22
2    DNAT       tcp  --  0.0.0.0/0      0.0.0.0      tcp dpt:2426 to:10.211.5.7:22
3    DNAT       tcp  --  0.0.0.0/0      0.0.0.0      tcp dpt:2822 to:10.211.5.70:22
```

* Удалить 2-е правило

```
iptables -D -t nat 2
```
