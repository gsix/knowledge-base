# Vault c zookeeper

### Error initializing Dev mode: failed to initialize barrier: failed to persist keyring: zk: not authenticated

**Причина**: отсутствие параметров авторизации.

**Контекст**: `Dev`-режим включён по умолчанию при запуске контейнера. В `Dev`-режиме предполагается одноразовое хранилище и **не предполагается** использовать `storage` отличные от `file`. Поэтому, либо работаете в `Dev`-режиме (на своей машине) и не переопределяете `storage` либо работаете в `Server`-режиме и можно настраивать любой другой `storage`. Суть ошибки заключается в отсутствии `znode_owner` и `auth_info`, которые определяют `ACL`. Нет параметров определяющий `ACL` → нет `ACL` →нет доступа.

**Решение**: надо определить `znode_owner` и `auth_info` в `/vault/config`.

**Пример** `/vault/config`:

    ui = true
    default_lease_ttl = "168h"
    max_lease_ttl = "720h"
    
    listener "tcp" {
      address = "0.0.0.0:8200"
      tls_cert_file = "/vault/certs/vault.crt"
      tls_key_file  = "/vault/certs/vault.key"
    }
    
    storage "zookeeper" {
      address = "zookeeper:2181"
      path    = "vault1/"
      znode_owner = "digest:vaultUser:raxgVAfnDRljZDAcJFxznkZsExs="
      auth_info   = "digest:vaultUser:abc"
    }

Параметры подключения к `Zookeeper` можно посмотреть тут [Zookeeper Storage Backend](https://vaultproject.io/docs/configuration/storage/zookeeper.html).

В `Zookeeper` по умолчанию **нет авторизации**. И данные в `znode_owner` и `auth_info` больше ни где не указаны (только в файле `/vault/config`). `znode_owner` и `auth_info` нужны для установки `ACL` в `zookeeper`.

**Пример** `docker-compose.yml`

    services:
      vault: # hub.docker.com/_/vault/
        cap_add:
          - IPC_LOCK
        image: vault:0.10.4
        entrypoint:
          - vault
          - server
          - -config
          - /vault/config/config
        expose:
         - 8200
        ports:
          - '8200:8200'
        environment:
          VAULT_ADDR: http://127.0.0.1:8200   #for cli client to point to non https
          VAULT_REDIRECT_ADDR: http://vault:8200
        volumes:
          - ./config:/vault/config
          - ./certs:/vault/certs
          - ./logs:/logs
    
      zookeeper: # hub.docker.com/_/zookeeper/
        image: zookeeper:3.4.13
        ports:
          - '2181:2181'
        volumes:
          - ./data:/data
          - ./zoo.cfg:/conf/zoo.cfg
    
    version: '3.4'

**Пример**: `zoo.cfg`:

    # http://hadoop.apache.org/zookeeper/docs/current/zookeeperAdmin.html
    
    # The number of milliseconds of each tick
    tickTime=2000
    # The number of ticks that the initial
    # synchronization phase can take
    initLimit=10
    # The number of ticks that can pass between
    # sending a request and getting an acknowledgement
    syncLimit=5
    # the directory where the snapshot is stored.
    dataDir=/data
    # Place the dataLogDir to a separate physical disc for better performance
    # dataLogDir=/disk2/zookeeper
    
    # the port at which the clients will connect
    clientPort=2181
    
    # specify all zookeeper servers
    # The fist port is used by followers to connect to the leader
    # The second one is used for leader election
    # server.1=zookeeper1:2888:3888
    #server.2=zookeeper2:2888:3888
    #server.3=zookeeper3:2888:3888
    
    # To avoid seeks ZooKeeper allocates space in the transaction log file in
    # blocks of preAllocSize kilobytes. The default block size is 64M. One reason
    # for changing the size of the blocks is to reduce the block size if snapshots
    # are taken more often. (Also, see snapCount).
    #preAllocSize=65536
    
    # Clients can submit requests faster than ZooKeeper can process them,
    # especially if there are a lot of clients. To prevent ZooKeeper from running
    # out of memory due to queued requests, ZooKeeper will throttle clients so that
    # there is no more than globalOutstandingLimit outstanding requests in the
    # system. The default limit is 1,000.ZooKeeper logs transactions to a
    # transaction log. After snapCount transactions are written to a log file a
    # snapshot is started and a new transaction log file is started. The default
    # snapCount is 10,000.
    #snapCount=1000
    
    # If this option is defined, requests will be will logged to a trace file named
    # traceFile.year.month.day.
    #traceFile=
    
    # Leader accepts client connections. Default value is "yes". The leader machine
    # coordinates updates. For higher update throughput at thes slight expense of
    # read throughput the leader can be configured to not accept clients and focus
    # on coordination.
    #leaderServes=yes