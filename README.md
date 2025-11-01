# Instructions-for-host-configure-Ite-RU

# Шаг 1: Проверка установленных агентов Zabbix
```
# Polish agents
ls -la /opt | grep zab

# Russian agents
ls -la /etc | grep zab
```
_Вариант 2_<br>
`systemctl status zabb` + `tab`

<strong>Также необходимо убедиться, что на системе ещё не стоит нужный агент, путём проверки наличия разных инстанций SAP</strong><br>
`ls -la /usr/sap | grep -o '[A-Z]\{3\}$'`

# Шаг 2: Установка и настройка заббикс агента версии 2
```
# Repo installation
rpm -Uvh --nosignature https://repo.zabbix.com/zabbix/7.4/release/sles/15/noarch/zabbix-release-latest-7.4.sles15.noarch.rpm
zypper --gpg-auto-import-keys refresh 'Zabbix Official Repository'
# Check if agent is there 
zypper se zabbix-agent2
# Zabbix agent 2 installation
zypper in zabbix-agent2
```
Проверка, установился ли агент `ls -la /etc/zabbix`

_Настройка агента_
```
# Move to zabbix dir
cd /etc/zabbix/
# Configure agent
vim ./zabbix_agent2.conf
```
- Заменить после `Server=127.0.0.1` на IP Zabbix proxy
- Заменить поле `# ListenPort=10050` на `ListenPort=10055`
- Закомментировать `ServerActive=127.0.0.1`
- Заменить `Hostname=Zabbix server` на значение из таблицы (либо на значение из команды `hostname -f`)

# Шаг 3: Перенос программы netmath на хост и настройка прав
- Через WinSCP перенести программу в предварительно созданную `mkdir zabbix_dir` субдиректорию пользователя plNNNN
- Создать директорию для программы `mkdir /usr/sbin/zabbix-agent2-plugin`
- Перенести zabbix-agent2-plugin-netmath в директорию `/usr/sbin/zabbix-agent2-plugin`, например, из созданной субдиректории `mv ./zabbix-agent2-plugin-netmath /usr/sbin/zabbix-agent2-plugin/`
- Изменить полномочия программы `chmod 755 /usr/sbin/zabbix-agent2-plugin/zabbix-agent2-plugin-netmath`
- Перейти в директорию `cd /etc/zabbix/zabbix_agent2.d/plugins.d/` и скопировать в неё необходимые UserParameters.conf-файлы ('netmath.conf') по образцу из сконфигурированной системы
- Изменить `SID` и `NR` в `netmath.conf` при необходимости
- Обновить дату изменения всех *.conf файлов с помощью команды `touh <filename>`
- Растиражировать файл `/etc/sudoers/grzabbix` с сконфигурированной системы, изменив `SID`, `sidadm` и `NR` при необходимости

# Шаг 4: Конфигурация PSK на хосте
- Создать файл с ключом шифрования `touch /etc/zabbix/tls.psk`; `echo <key> > /etc/zabbix/tls.psk`
- Изменить полномочия файла с ключом `chmod 400 /etc/zabbix/tls.psk`; `chown zabbix:zabbix /etc/zabbix/tls.psk`
- Добавить файл конфигурации psk:
```
touch /etc/zabbix/zabbix_agent2.d/tls.conf
vim /etc/zabbix/zabbix_agent2.d/tls.conf
# Inside add configuration
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=<psk_id>
TLSPSKFile=/etc/zabbix/tls.psk 
```

# Шаг 5: Запуск агента
- Запуск агента `systemctl start zabbix-agent2.service`
- Автозапуск агента `systemctl enable zabbix-agent2.service`

# Шаг 6: Конфигурация хоста со стороны сервера Zabbix
Сконфигурировать хост в веб-интерфейсе Zabbix, добавив psk шифрование, шаблон netmath_monitor, а также стандартные шаблоны для Linux по образцу сконфигурированных хостов.
