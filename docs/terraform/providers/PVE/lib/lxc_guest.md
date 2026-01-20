# LXC Guest Resource

Этот ресурс управляет контейнером LXC.

Чтобы начать работу, это минимальный пример:
??? minimal-example
    ```hcl
    resource "proxmox_lxc_guest" "minimal-example" {
        name         = "minimal-example"
        power_state  = "running"
        node         = "pve-1"
        unprivileged = true
        password     = "yourpassword"
        template {
            file    = "alpine-3.22-default_20250617_amd64.tar.xz"
            storage = "local"
        }
        cpu {
            cores = 1
        }
        features {
            unprivileged {
                nesting = true
            }
        }
        memory = 1024
        swap   = 512
        pool   = "my-pool"
        root_mount {
            size    = "4G"
            storage = "local-lvm"
        }
        network {
            id = 0
            name = "eth0"
            bridge = "vmbr0"
            ipv4_address = "192.168.1.100/24"
            ipv4_gateway = "192.168.1.1"
        }
        startup_shutdown {}
    }
    ```

## Ссылки на аргумент

| Argument            | Type    | Default Value            | Description |
|:--------------------|---------|--------------------------|:------------|
| `clone`             | `nested`|                          | **Принудительное восстановление**: Конфигурация клонирования, см. [Clone Reference](#clone-reference).|
| `cpu_architecture`  | `string`|                          | **Вычисляемый**: Архитектура центрального процессора.|
| `cpu`               | `nested`|                          | Конфигурация процессора, смотрите в разделе [CPU Reference](#cpu-reference).|
| `description`       | `string`| `"Managed by Terraform."`| Описание гостевого контейнера.|
| `dns`               | `nested`|                          | Конфигурация DNS, смотрите в разделе [DNS Reference](#dns-reference).|
| `features`          | `nested`|                          | Конфигурация функций, смотрите в разделе [Features Reference](#features-reference).|
| `guest_id`          | `int`   |                          | **Принудительно восстанавливает**, **Вычисляется**: Числовой идентификатор гостевого контейнера, также известный как "vmid". Если он не указан, идентификатор будет присвоен автоматически.|
| `memory`            | `int`   | `512`                    | Объем памяти, выделяемой гостю, в мегабайтах.|
| `mount`             | `array` |                          | Хранилища монтируются как отдельные элементы массива, см. [Mount Reference](#mount-reference).|
| `mounts`            | `nested`|                          | Монтирование хранилища, настроенное как вложенные вложенные элементы, см. [Mounts Reference](#mounts-reference).|
| `name`              | `string`|                          | **Обязательно**: название контейнера.|
| `network`           | `array` |                          | Сетевые интерфейсы, сконфигурированные как отдельные элементы массива, см. [Network Reference](#network-reference).|
| `networks`          | `nested`|                          | Сетевые интерфейсы, настроенные как вложенные подпункты, см. [Networks Reference](#networks-reference).|
| `os`                | `string`|                          | **Вычислено**: Название операционной системы внутри гостевой системы.|
| `password`          | `string`|                          | **Принудительное восстановление**, **Конфиденциально**: Пароль пользователя root внутри гостевого контейнера.|
| `pool`              | `string`|                          | Имя пула, членом которого должен быть гостевой контейнер.|
| `power_state`       | `string`| `"running"`              | Состояние питания гостя может быть `запущено" или `остановлено".|
| `privileged`        | `bool`  |                          | **Forces Recreation**: Является ли гость привилегированным или непривилегированным. Может быть только `true" или "unset". Взаимоисключающие значения "непривилегированный".|
| `root_mount`        | `nested`|                          | **Требуется**: Настройка корневого/загрузочного диска/диска для установки в гостевой контейнер. **Примечание:** Размер может быть только увеличен, но не уменьшен.|
| `ssh_public_key`    | `string`|                          | **Forces Recreation** Открытый SSH-ключ пользователя root внутри гостевого контейнера.|
| `start_at_node_boot`| `bool`  | `false`                  | Должен ли гостевой сервер запускаться автоматически при загрузке узла Proxmox.|
| `startup_shutdown`  | `nested`|                          | Настройка запуска и выключения гостевой системы приведена в разделе [Startup and Shutdown Reference](#startup-and-shutdown-reference).|
| `swap`              | `int`   | `512`                    | Объем виртуальной памяти гостя, который будет сопоставлен с пространством подкачки на узле PVE.|
| `tags`              | `list`  | `[]`                     | Список тегов, которые нужно присвоить гостевому контейнеру.|
| `target_node`       | `string`|                          | Один узел, на котором должен находиться гость. Если гость находится на другом узле, он будет перенесен на этот.|
| `target_nodes`      | `array` |                          | Список узлов, на которых должен находиться гость. Если гость не находится ни на одном из этих узлов, он будет перенесен на один из них.|
| `unprivileged`      | `bool`  |                          | **Forces Recreation**: Если гость является непривилегированным или привилегированным. Может быть только "true" или "unset". Взаимоисключающие с `privileged`.|

### Clone Reference

Поле "клонировать" используется для настройки параметров клонирования.
**Его можно указать только один раз.**

| Argument       | Type    | Default Value | Description |
|:---------------|---------|---------------|:------------|
| `id`           | `int`   |               | **Forces Recreation**: Числовой идентификатор исходного контейнера для клонирования.|
| `linked`       | `bool`  | `false`       | **Forces Recreation**: Независимо от того, должен ли клон быть связанным.|
| `name`         | `string`|               | **Forces Recreation**: Имя исходного контейнера для клонирования. Любой `id` или `name` должно быть указано конкретно.|

### CPU Reference

Поле `cpu` поле используется для настройки параметров процессора. Его можно указать только один раз.

| Argument | Type | Default Value | Description |
|:---------|------|---------------|:------------|
| `cores`  | `int`| `0`           | Количество процессорных ядер гостевого компьютера, `0` означает неограниченное.|
| `limit`  | `int`| `0`           | Ограничение по количеству процессорных ядер гостевых процессоров, "0" означает неограниченное количество.|
| `units`  | `int`| `100`         | Процессорные блоки гостя.|

### DNS Reference

Поле `dns` используется для настройки параметров DNS. Его можно указать только один раз.

| Argument      | Type    | Default Value | Description |
|:--------------|---------|---------------|:------------|
| `searchdomain`| `string`| `""`          | DNS-поисковый домен гостя наследует конфигурацию узла PVE, когда он пуст.|
| `nameserver`  | `array` | `[]`          | DNS-сервер имен гостя наследует конфигурацию узла PVE, когда он пуст.|

### Features Reference

Поле  `features` используется для настройки параметров функций. Его можно указать только один раз.

| Argument        | Type    | Default Value | Description |
|:----------------|---------|---------------|:------------|
| `privileged`    | `nested`|               | Настройка привилегированных функций, см. [Features Privileged Reference](#features-privileged-reference).|
| `unprivileged`  | `nested`|               | Настройка непривилегированных функций, смотрите в разделе [Features Unprivileged Reference](#features-unprivileged-reference).|

#### Features Privileged Reference

Поле `features.privileged` используется для настройки параметров привилегированных функций. Его можно указать только один раз. `features.privileged`является взаимоисключающим с  `features.unprivileged`. Для его использования требуется `privileged = true` верхнего уровня.

| Argument             | Type   | Default Value | Description |
|:---------------------|--------|---------------|:------------|
| `create_device_nodes`| `bool` | `false`       | Следует ли включить функцию создания узлов устройств.|
| `fuse`               | `bool` | `false`       | Whether FUSE should be enabled.|
| `nesting`            | `bool` | `false`       | Следует ли включать вложенность.|
| `nfs`                | `bool` | `false`       | Следует ли включать NFS.|
| `smb`                | `bool` | `false`       | Следует ли включать SMB.|

#### Features Unprivileged Reference

Поле `features.unprivileged` используется для настройки параметров непривилегированных функций. Его можно указать только один раз.  `features.unprivileged` является взаимоисключающим с `features.privileged`. Высший уровень `unprivileged = true` требуется для использования этого.

| Argument             | Type   | Default Value | Description |
|:---------------------|--------|---------------|:------------|
| `create_device_nodes`| `bool` | `false`       | Следует ли включать функцию создания узлов устройств.|
| `fuse`               | `bool` | `false`       | Whether FUSE should be enabled.|
| `keyctl`             | `bool` | `false`       | Должен ли быть включен keyctl.|
| `nesting`            | `boIf the guest is unprivileged or privileged. Can only be `true` or unset. Mutually exclusive withol` | `false`       | Следует ли включать вложенность.|

### Mount Reference

The `mount` field is used to configure the mount settings. It may be specified multiple times, each instance requires a unique `slot` value. `mount` is mutually exclusive with `mounts`.

| Argument          | Type    | Default Value | Description |
|:------------------|---------|---------------|:------------|
| `acl`             | `string`| `default`     | Mount acl configuration, can be one of `"true"`, `"false"`, `"default"`. Requires `type` = `data`.|
| `backup`          | `bool`  | `true`        | Wheter the mount will be included in backup tasks.|
| `guest_path`      | `string`|               | **Required**: Absolute path of the mount point inside the container guest, example: `"/mnt/data-mount`.|
| `host_path`       | `string`|               | **Required when `type` = `bind`**: Absolute path of the mount point on the PVE host, example: `"/mnt/pve-storage/data-mount`.|
| `option_discard`  | `bool`  | `true`        | Enable discart.|
| `option_lazy_time`| `bool`  | `true`        | Enable lazy time.|
| `option_no_atime` | `bool`  | `true`        | Enable no atime.|
| `option_no_device`| `bool`  | `true`        | Enable no device.|
| `option_no_exec`  | `bool`  | `true`        | Enable no exec.|
| `option_no_suid`  | `bool`  | `true`        | Enable no suid.|
| `quota`           | `bool`  | `false`       | Wheter data quota should be enabled. Requires top level `privileged` = `true`. Requires `type` = `data`.|
| `read_only`       | `bool`  | `false`       | Wheter the mount point is read only.|
| `replicate`       | `bool`  | `false`       | Wheter replication is enabled on the mount point.|
| `size`            | `string`|               | **Required when `type` = `data`**: Size of the mount.|
| `slot`            | `string`|               | **required**: The unique slot id of the mount. Must be prefixed with `mp`, example: `mp0`. Maximum amount of mounts is 256.|
| `storage`         | `string`|               | **Required when `type` = `data`**: Storage of the mount.|
| `type`            | `string`| `"data"`      | The type of mount point. Use `"bind"` for a bind mount to the PVE host and  `"data"` for a normal data disk.|

### Mounts Reference

The `mounts` field is used to configure the mount settings. It may only be specified once. `mounts` is mutually exclusive with `mount`. `mounts` has 256 sub items, with each sub item representing a unique slot, example: `mp0`, `mp1`, etc. Every slot has the following configuration options:

| Argument | Type    | Default Value | Description |
|:---------|---------|---------------|:------------|
| `bind`   | `nested`|               | Bind mount configuration, see [Bind Mounts Reference](#bind-mounts-reference).|
| `data`   | `nested`|               | Data mount configuration, see [Data Mounts Reference](#data-mounts-reference).|

#### Bind Mounts Reference

The `bind` field is used to configure a bind moun, And is mutually exclusive with `data`.

| Argument          | Type    | Default Value | Description |
|:------------------|---------|---------------|:------------|
| `guest_path`      | `string`|               | **Required**: Absolute path of the mount point inside the container guest, example: `"/mnt/data-mount`.|
| `host_path`       | `string`|               | **Required**: Absolute path of the mount point on the PVE host, example: `"/mnt/pve-storage/data-mount`.|
| `option_discard`  | `bool`  | `true`        | Enable discart.|
| `option_lazy_time`| `bool`  | `true`        | Enable lazy time.|
| `option_no_atime` | `bool`  | `true`        | Enable no atime.|
| `option_no_device`| `bool`  | `true`        | Enable no device.|
| `option_no_exec`  | `bool`  | `true`        | Enable no exec.|
| `option_no_suid`  | `bool`  | `true`        | Enable no suid.|
| `read_only`       | `bool`  | `false`       | Wheter the mount point is read only.|
| `replicate`       | `bool`  | `false`       | Wheter replication is enabled on the mount point.|

#### Data Mounts Reference

The `data` field is used to configure a data mount, And is mutually exclusive with `bind`.

| Argument          | Type    | Default Value | Description |
|:------------------|---------|---------------|:------------|
| `acl`             | `string`| `default`     | Mount acl configuration, can be one of `"true"`, `"false"`, `"default"`. Requires top level `privileged` = `true`.|
| `backup`          | `bool`  | `true`        | Wheter the mount will be included in backup tasks.|
| `guest_path`      | `string`|               | **Required**: Absolute path of the mount point inside the container guest, example: `"/mnt/data-mount`.|
| `option_discard`  | `bool`  | `true`        | Enable discart.|
| `option_lazy_time`| `bool`  | `true`        | Enable lazy time.|
| `option_no_atime` | `bool`  | `true`        | Enable no atime.|
| `option_no_device`| `bool`  | `true`        | Enable no device.|
| `option_no_exec`  | `bool`  | `true`        | Enable no exec.|
| `option_no_suid`  | `bool`  | `true`        | Enable no suid.|
| `quota`           | `bool`  | `false`       | Wheter data quota should be enabled. Requires top level `privileged` = `true`.|
| `read_only`       | `bool`  | `false`       | Wheter the mount point is read only.|
| `replicate`       | `bool`  | `false`       | Wheter replication is enabled on the mount point.|
| `size`            | `string`|               | **Required**: Size of the mount.|
| `storage`         | `string`|               | **Required**: Storage of the mount.|

### Network Reference

The `network` field is used to configure the network interfaces. It may be specified multiple times, each instance requires a unique `id` and `name` value. `network` is mutually exclusive with `networks`.

| Argument        | Type    | Default Value | Description |
|:----------------|---------|---------------|:------------|
| `bridge`        | `string`|               | **Required**: Bridge the network interface will be connected to.|
| `connected`     | `bool`  | `true`        | Wheter the network interface will be connected.|
| `firewall`      | `bool`  | `false`       | Wheter the network interface will be protected by the firewall.|
| `id`            | `string`|               | **Required**: The unique id of the network interface. Must be prefixed with `net`, example: `net0`. Maximum amount of network interfaces is 16.|
| `ipv4_address`  | `string`|               | IPv4 address of the network interface.|
| `ipv4_dhcp`     | `bool`  | `false`       | Wheter IPv4 DHCP is enabled on the network interface.|
| `ipv4_gateway`  | `string`|               | IPv4 gateway of the network interface.|
| `ipv6_address`  | `string`|               | IPv6 address of the network interface.|
| `ipv6_dhcp`     | `bool`  | `false`       | Wheter IPv6 DHCP is enabled on the network interface.|
| `ipv6_gateway`  | `string`|               | IPv6 gateway of the network interface.|
| `mac`           | `string`|               | MAC address of the network interface.|
| `mtu`           | `int`   |               | MTU of the network interface.|
| `name`          | `string`|               | **Required**: Name of the network interface inside the guest. The name must be unique example: `eth0`.|
| `rate_limit`    | `int`   |               | Rate limit of the network interface in Kbit/s. `0` means unlimited.|
| `slaac`         | `bool`  | `false`       | Wheter SLAAC is enabled on the network interface. Conflicts with IPv6 settings.|
| `vlan_native`   | `int`   |               | Native VLAN of the network interface.|

### Networks Reference

The `networks` field is used to configure the network interfaces. It may only be specified once. `networks` is mutually exclusive with `network`. `mounts` has 16 sub items, with each sub item representing a unique id, example: `net0`, `net1`, etc. Every slot has the following configuration options:

| Argument     | Type    | Default Value | Description |
|:-------------|---------|---------------|:------------|
| `bridge`     | `string`|               | **Required**: Bridge the network interface will be connected to.|
| `connected`  | `bool`  | `true`        | Wheter the network interface will be connected.|
| `firewall`   | `bool`  | `false`       | Wheter the network interface will be protected by the firewall.|
| `ipv4`       | `nested`|               | IPv4 configuration, see [IPv4 Reference](#ipv4-reference).|
| `ipv6`       | `nested`|               | IPv6 configuration, see [IPv6 Reference](#ipv6-reference).|
| `mac`        | `string`|               | MAC address of the network interface.|
| `mtu`        | `int`   |               | MTU of the network interface.|
| `name`       | `string`|               | **Required**: Name of the network interface inside the guest. The name must be unique example: `eth0`.|
| `rate_limit` | `int`   |               | Rate limit of the network interface in Kbit/s. `0` means unlimited.|
| `vlan_native`| `int`   |               | Native VLAN of the network interface.|

#### IPv4 Reference

| Argument     | Type    | Default Value | Description |
|:-------------|---------|---------------|:------------|
| `address`    | `string`|               | IPv4 address of the network interface.|
| `gateway`    | `string`|               | IPv4 gateway of the network interface.|
| `dhcp`       | `bool`  | `false`       | Wheter IPv4 DHCP is enabled on the network interface. Mutually exclusive with `address` and `gateway`|

#### IPv6 Reference

| Argument     | Type    | Default Value | Description |
|:-------------|---------|---------------|:------------|
| `address`    | `string`|               | IPv6 address of the network interface.|
| `gateway`    | `string`|               | IPv6 gateway of the network interface.|
| `dhcp`       | `bool`  | `false`       | Wheter IPv6 DHCP is enabled on the network interface. Mutually exclusive with `address`, `gateway` and `slaac`.|
| `slaac`      | `bool`  | `false`       | Wheter SLAAC is enabled on the network interface. Conflicts with IPv6 settings. Mutually exclusive with `address`, `gateway` and `dhcp`.|

### Startup and Shutdown Reference

The `startup_shutdown` field is used to configure the startup and shutdown settings. It may only be specified once.

| Argument            | Type | Default Value | Description |
|:--------------------|------|---------------|:------------|
| `order`             | `int`| `-1`          | Startup order `-1` means any.|
| `shutdown_timeout`  | `int`| `-1`          | Shutdown timeout in seconds, `-1` means default.|
| `startup_delay`     | `int`| `-1`          | Startup delay in seconds, `-1` means default.|