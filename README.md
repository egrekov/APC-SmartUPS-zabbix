# APC SmartUPS 1000 - мониторинг

Для получения параметров ИБП, требуется установить демон ```apcupsd```, который будет общаться с APC SmartUPS
 и получать от него данные
* Установка ```apcupsd```
```bash
apt install apcupsd apcupsd-cgi
```
* Настройка
```bash
# cat /etc/apcupsd/apcupsd.conf | grep -v '#' | grep -v ^$
UPSCABLE usb
UPSTYPE apcsmart
DEVICE /dev/ttyUSB1
LOCKFILE /var/lock
SCRIPTDIR /etc/apcupsd
PWRFAILDIR /etc/apcupsd
NOLOGINDIR /etc
BATTERYLEVEL -1
MINUTES -1
TIMEOUT -1
ANNOY 0
ANNOYDELAY 60
NOLOGON disable
KILLDELAY 0
NETSERVER on
NISIP 127.0.0.1
NISPORT 3551
EVENTSFILE /var/log/apcupsd.events
EVENTSFILEMAX 10
UPSCLASS standalone
UPSMODE disable
STATTIME 0
STATFILE /var/log/apcupsd.status
LOGSTATS off
DATATIME 0
```
```BATTERYLEVEL```, ```MINUTES```, ```TIMEOUT``` выставил в ```-1```, так если сервер выключится, то сам он уже 
не поднимется, а это плохо...
* Разрешаем запуск ```apcupsd```, для этого нужно изменить значение параметра ```ISCONFIGURED``` на ```yes``` 
в файле ```/etc/default/apcupsd``` и запустить
```bash
service apcupsd start
```
* Проверка конфигурации
```bash
apcaccess
```
Должны получить, что то вроде:
```bash
APC      : 001,051,1209
DATE     : 2019-06-13 14:51:55 +0500
HOSTNAME : servername
VERSION  : 3.14.14 (31 May 2016) debian
UPSNAME  : UPS_IDEN
CABLE    : Custom Cable Smart
DRIVER   : APC Smart UPS (any)
UPSMODE  : Stand Alone
STARTTIME: 2019-06-11 14:29:11 +0500
MODEL    : Smart-UPS 1000
STATUS   : ONLINE
LINEV    : 223.2 Volts
LOADPCT  : 44.8 Percent
BCHARGE  : 100.0 Percent
TIMELEFT : 1854.0 Minutes
MBATTCHG : -1 Percent
MINTIMEL : -1 Minutes
MAXTIME  : -1 Seconds
MAXLINEV : 227.5 Volts
MINLINEV : 217.4 Volts
OUTPUTV  : 223.2 Volts
SENSE    : Low
DWAKE    : 0 Seconds
DSHUTD   : 180 Seconds
DLOWBATT : 2 Minutes
LOTRANS  : 208.0 Volts
HITRANS  : 253.0 Volts
RETPCT   : 0.0 Percent
ITEMP    : 22.0 C
ALARMDEL : 5 Seconds
BATTV    : 27.7 Volts
LINEFREQ : 50.0 Hz
LASTXFER : Unacceptable line voltage changes
NUMXFERS : 1
XONBATT  : 2019-06-11 19:07:32 +0500
TONBATT  : 0 Seconds
CUMONBATT: 781 Seconds
XOFFBATT : 2019-06-11 19:20:33 +0500
SELFTEST : NO
STESTI   : OFF
STATFLAG : 0x05000008
REG1     : 0x00
REG2     : 0x00
REG3     : 0x00
MANDATE  : 06/27/07
SERIALNO : AS0726221015
BATTDATE : 06/14/19
NOMOUTV  : 230 Volts
NOMBATTV : 24.0 Volts
EXTBATTS : 16
FIRMWARE : 652.13.I
END APC  : 2019-06-13 14:52:11 +0500
```
* Устанавливаем ```zabbix-agent```
```bash
apt install zabbix-agent
```
```bash
# cat /etc/zabbix/zabbix_agentd.d/apcupsd.conf
UserParameter=apc.mon[*],apcaccess -p $1 -u

cat /etc/zabbix/zabbix_agentd.d/server.conf
Server=zabbix_server_ip
ServerActive=zabbix_server_ip
```
* Ну и самый сложный этап ))))) Импортируем шаблон ```zabbix-template.xml``` в Zabbix и назначаем его хосту, 
смотрим и радуемся красивым графикам. 