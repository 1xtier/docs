# lxc

Этот ресурс создает контейнер Proxmox LXC и управляет им.

## Пример использования

### Базовый пример

```hcl
resource "proxmox_lxc" "basic" {
  target_node  = "pve"
  hostname     = "lxc-basic"
  ostemplate   = "local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz"
  password     = "BasicLXCContainer"
  unprivileged = true

  // Terraform will crash without rootfs defined
  rootfs {
    storage = "local-zfs"
    size    = "8G"
  }

  network {
    name   = "eth0"
    bridge = "vmbr0"
    ip     = "dhcp"
  }
}
```

### Несколько точек монтирования

-> При указании `local-lvm:12` для атрибута `mountpoint.storage` в первом блоке  `mountpoint` расположенном ниже, том
будет автоматически создан для контейнера LXC. Для получения дополнительной информации об этом поведении
см. [Storage Backed Mount Points](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_storage_backed_mount_points).

```hcl
resource "proxmox_lxc" "multiple_mountpoints" {
  target_node  = "pve"
  hostname     = "lxc-multiple-mountpoints"
  ostemplate   = "local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz"
  unprivileged = true
  ostype       = "ubuntu"

  ssh_public_keys = <<-EOT
    ssh-rsa <public_key_1> user@example.com
    ssh-ed25519 <public_key_2> user@example.com
  EOT

  // Terraform will crash without rootfs defined
  rootfs {
    storage = "local-zfs"
    size    = "8G"
  }

  // Storage Backed Mount Point
  mountpoint {
    key     = "0"
    slot    = 0
    storage = "local-lvm"
    mp      = "/mnt/container/storage-backed-mount-point"
    size    = "12G"
  }

  // Bind Mount Point
  mountpoint {
    key     = "1"
    slot    = 1
    storage = "/srv/host/bind-mount-point"
    // Without 'volume' defined, Proxmox will try to create a volume with
    // the value of 'storage' + : + 'size' (without the trailing G) - e.g.
    // "/srv/host/bind-mount-point:256".
    // This behaviour looks to be caused by a bug in the provider.
    volume  = "/srv/host/bind-mount-point"
    mp      = "/mnt/container/bind-mount-point"
    size    = "256G"
  }

  // Device Mount Point
  mountpoint {
    key     = "2"
    slot    = 2
    storage = "/dev/sdg"
    volume  = "/dev/sdg"
    mp      = "/mnt/container/device-mount-point"
    size    = "32G"
  }

  network {
    name   = "eth0"
    bridge = "vmbr0"
    ip     = "dhcp"
    ip6    = "dhcp"
  }
}
```

### LXC с включенными расширенными функциями

```hcl
resource "proxmox_lxc" "advanced_features" {
  target_node  = "pve"
  hostname     = "lxc-advanced-features"
  ostemplate   = "local:vztmpl/ubuntu-20.04-standard_20.04-1_amd64.tar.gz"
  unprivileged = true

  ssh_public_keys = <<-EOT
    ssh-rsa <public_key_1> user@example.com
    ssh-ed25519 <public_key_2> user@example.com
  EOT

  features {
    fuse    = true
    nesting = true
    mount   = "nfs;cifs"
  }

  // Terraform will crash without rootfs defined
  rootfs {
    storage = "local-zfs"
    size    = "8G"
  }

  // NFS share mounted on host
  mountpoint {
    slot    = "0"
    storage = "/mnt/host/nfs"
    mp      = "/mnt/container/nfs"
    size    = "250G"
  }

  network {
    name   = "eth0"
    bridge = "vmbr0"
    ip     = "10.0.0.2/24"
    ip6    = "auto"
  }
}
```

### Базовый пример клонирования

```hcl
resource "proxmox_lxc" "basic" {
  target_node = "pve"
  hostname    = "lxc-clone"
  #id of lxc container to clone
  clone       = "8001"
}
```

## Argument Reference

[Ссылка на атрибуты](lxc_guest.md)

### Required

При использовании этого ресурса должны быть определены следующие аргументы:

* `target_node` - Строка, содержащая имя узла кластера.

### Optional

-> Хотя приведенные ниже аргументы являются необязательными, некоторые из них содержат дочерние аргументы, которые требуются при использовании родительского
аргумента (например, `name` в атрибуте `network` ). Эти дочерние аргументы помечены как "__(required)__".

При использовании этого ресурса необязательно могут быть определены следующие аргументы:

* `ostemplate` - [volume identifier](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_volumes) который указывает на
шаблон операционной системы или файл резервной копии.
* `arch` - Устанавливает тип архитектуры контейнерной операционной системы. По умолчанию используется `amd64`.
* `bwlimit` - Число для установки ограничения пропускной способности переопределяемого ввода-вывода (in KiB/s).
* `clone` - Идентификатор lxc vmid для клонирования
* `clone_storage` - Целевое хранилище для полного клонирования.
* `cmode` -Настраивает режим консоли. `tty` пытается установить соединение с одним из доступных устройств tty. "console"
 вместо этого пытается подключиться к `/dev/console`. `shell` просто вызывает оболочку внутри контейнера (без входа в систему). По умолчанию
используется `tty`.
* `console` - Логическое значение для подключения консольного устройства к контейнеру. По умолчанию используется значение `true`.
* `cores` - Количество ядер, назначенных контейнеру. По умолчанию контейнер может использовать все доступные ядра.
* `cpulimit` - Число, на которое можно ограничить использование процессора. Значение по умолчанию - `0`.
* `cpuunits` - Количество ресурсов процессора, которыми обладает контейнер. Значение по умолчанию - `1024`.
* `description` - Задает описание контейнера, отображаемое в веб-интерфейсе.
* `features` - Объект, позволяющий контейнеру получать доступ к расширенным функциям.
    * `fuse` - A boolean for enabling FUSE mounts.
    * `keyctl` - Логическое значение для включения системного вызова `keyctl()`.
    * `mount` - Определяет типы файловых систем (разделенные точкой с запятой), которые разрешено монтировать.
    * `nesting` - Логическое значение, разрешающее вложенную виртуализацию.
* `force` - Логическое значение, позволяющее перезаписывать ранее существовавшие контейнеры.
* `full` -При клонировании создайте полную копию всех дисков. Это всегда делается при клонировании обычного CT. Для
шаблона CT по умолчанию создается связанный клон.
* `hastate` - идентифицированное состояние HA для ресурса. Одно из значений "запущен", "остановлен", "включен", "отключен" или "игнорируется". Смотрите
 [docs about HA](https://pve.proxmox.com/pve-docs/chapter-ha-manager.html#ha_manager_resource_config) для получения дополнительной
информации.
* `hagroup` - Идентификатор группы HA, к которой принадлежит ресурс (требуется задать значение "hastate"!). Смотрите
 [docs about HA](https://pve.proxmox.com/pve-docs/chapter-ha-manager.html#ha_manager_resource_config) для получения дополнительной
информации.
* `hookscript` -  Строка, содержащая [a volume identifier to a script](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_hookscripts_2)
  который будет выполняться на различных этапах в течение всего срока службы контейнера. Скрипт должен быть исполняемым файлом.
* `hostname` - Указывает имя хоста контейнера.
* `ignore_unpack_errors` - Логическое значение, определяющее, будут ли игнорироваться ошибки извлечения шаблона при
создании контейнера.
* `lock` - Строка для блокировки или разблокировки виртуальной машины.
* `memory` - Число, содержащее объем оперативной памяти, который необходимо назначить контейнеру (в МБ).
* `mountpoint` - Объект для определения тома, который будет использоваться в качестве точки монтирования контейнера. Может быть указан несколько раз.
    * `mp` __(required)__ - Путь к точке монтирования, видимый изнутри контейнера. Путь не должен содержать
символических ссылок по соображениям безопасности.
    * `size` __(required)__ - Размер базового тома. Должно заканчиваться на T, G, M или K (например, "1T", "1G", "1024M").
 , `"1048576K"`). Обратите внимание, что это значение доступно только для чтения.
    * `slot` __(required)__ - Строка, содержащая номер, который идентифицирует точку монтирования (т.е. "n"
в [`mp[n]`](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pct_mount_points)).
    * `key` __(required)__ -Номер, который идентифицирует точку монтирования (т.е. "n"
в [`mp[n]`](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pct_mount_points)).
    * `storage` __(required)__ - строка, содержащая [volume](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_storage_backed_mount_points), [directory](https:/pve.proxmox.com/pve-docs/pve-admin-guide.html#_bind_mount_points),
      или [device](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_device_mount_points) для установки в
контейнер (по пути, указанному `mp`). Например, `local-lvm`, `local-zfs`, `local` и т.д.
    * `acl` - Логическое значение для включения поддержки ACL. По умолчанию используется значение `false`.
    * `backup` - Логическое значение для включения точки монтирования в резервные копии. Значение по умолчанию - `false`.
    * `quota` - Логическое значение для включения пользовательских квот внутри контейнера для этой точки монтирования. Значение по умолчанию - `false`.
    * `replicate` - Логическое значение для включения этого тома в задание реплики хранилища. Значение по умолчанию - `false`.
    * `shared` - Логическое значение для обозначения тома как доступного на всех узлах. Значение по умолчанию - `false`.
* `nameserver` - IP-адрес DNS-сервера, используемый контейнером. Если не указаны ни `nameserver`, ни `searchdomain`
, по умолчанию будут использоваться значения хоста Proxmox.
* `network` - Объект, определяющий сетевой интерфейс для контейнера. Может быть указан несколько раз.
    * `name` __(required)__ - Название сетевого интерфейса, видимое изнутри контейнера (например, "eth0").
    * `bridge` - Мост, к которому подключается сетевой интерфейс (например, "vmbr0").
    * `firewall` - Логическое значение для включения брандмауэра в сетевом интерфейсе.
    * `gw` - IPv4-адрес, принадлежащий шлюзу сетевого интерфейса по умолчанию.
    * `gw6` - IPv6-адрес шлюза сетевого интерфейса по умолчанию.
    * `hwaddr` - Строка для задания общего MAC-адреса с не заданным битом ввода/вывода (индивидуальный/групповой). 
Определяется автоматически, если не задан.
    * `ip` - IPv4-адрес сетевого интерфейса. Может быть статическим IPv4-адресом (в обозначении CIDR), `dhcp`
или "вручную".
    * `ip6` - IPv6-адрес сетевого интерфейса. Может быть статическим IPv6-адресом (в обозначении CIDR), `автоматическим"
 , "dhcp" или "ручным".
    * `mtu` - Строка для установки MTU в сетевом интерфейсе.
    * `rate` - Число, устанавливающее ограничение скорости на сетевом интерфейсе (Мбит/с).
    * `tag` - Номер, который определяет тег VLAN сетевого интерфейса. Определяется автоматически, если не задан.
* `onboot` - Логическое значение, определяющее, запустится ли контейнер при загрузке. Значение по умолчанию - `false`.
* `ostype` - Тип операционной системы, используемый LXC для установки и конфигурирования контейнера. Определяется автоматически, если
не задан.
* `password` - Устанавливает пароль root внутри контейнера.
* `pool` - Имя пула ресурсов Proxmox, в который нужно добавить этот контейнер.
* `protection` - Логическое значение, которое включает флажок защиты для этого контейнера. Блокирует удаление/обновление контейнера и его диска
. Значение по умолчанию - `false`.
* `restore` - Логическое значение, обозначающее создание/обновление контейнера как задачу восстановления.
* `rootfs` - Объект для настройки корневой точки монтирования контейнера. Может быть указан только один раз.
    * `size` __(required)__ - Размер базового тома. Должно заканчиваться на T, G, M или K (например, `1T`, `1G`, `1024M`,`1048576K`). Обратите внимание, что это значение доступно только для чтения.
    * `storage` __(required)__ - Строка, содержащая
      [volume](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_storage_backed_mount_points)
      , [directory](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_bind_mount_points),
      или  [device](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#_device_mount_points) которые будут установлены в
контейнер (по пути, указанному в `mp`). Например, `local-lvm`, `local-zfs`, `local` и т.д.
* `searchdomain` - Устанавливает домены поиска DNS для контейнера. Если не указаны ни `nameserver` , ни `searchdomain`
, по умолчанию будут использоваться значения хоста Proxmox.
* `ssh_public_keys` -Многострочная строка открытых ключей SSH, которая будет добавлена в контейнер. Может быть определена
с помощью [heredoc syntax](https://www.terraform.io/docs/configuration/expressions/strings.html#heredoc-strings).
* `start` - Логическое значение, определяющее, будет ли запущен контейнер после создания. Значение по умолчанию - `false`.
* `startup` -
  The [startup and shutdown behaviour](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#pct_startup_and_shutdown)
  of the container.
* `swap` - Число, задающее объем памяти подкачки, доступной для контейнера. Значение по умолчанию - `512`.
* `tags` - Теги контейнера, разделенные точкой с запятой (например, "terraform;test"). Это только метаинформация.
* `template` - Логическое значение, определяющее, является ли этот контейнер шаблоном.
* `tty` - Число, указывающее количество доступных для контейнера данных. По умолчанию используется значение `2`.
* `unique` - Логическое значение, определяющее, назначен ли контейнеру уникальный случайный ethernet-адрес.
* `unprivileged` - Логическое значение, которое позволяет контейнеру запускаться от имени непривилегированного пользователя. Значение по умолчанию - `false`.
* `vmid` - Число, задающее VMID контейнера. Если задано значение "0", используется следующий доступный VMID. По умолчанию используется значение `0`.
* `current_node` __(computed)__ - Строка, которая показывает, на каком узле находится гость LXC.|
