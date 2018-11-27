# Проброс портов снаружи в контейнер/виртуалку

**Требования**: SSH доступ к рутовой Proxmox машине

**Решение** В консоль bash на сервере:

    sudo iptables -A PREROUTING -t nat -i enp2s0 -p tcp -d ${PROXMOX_EXTERNAL_IP} --dport ${EXTERNAL_PORT} -j DNAT --to ${INTERNAL_BOX_IP}:${INTERNAL_BOX_PORT}

* `${PROXMOX_EXTERNAL_IP}` - внешний IP самого Proxmox (константа)
* `${EXTERNAL_PORT}` - внешний порт, на который будут подключаться снаружи
* `${INTERNAL_BOX_IP}` - IP-адрес контейнера/виртуалки во внутренней сети
* `${INTERNAL_BOX_PORT}` - порт контейнера/виртуалки, на который будет происходить перенаправление снаружи
