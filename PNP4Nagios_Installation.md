Installing and configuring PNP4Nagios in Centos 8 step by step guide.

```
yum install rrdtool
yum install ruby xorg-x11-fonts-Type1 php-xml
```
## CHANGE PATH to temp folder for downloading pnp4nagios

```
cd /tmp/
wget https://sourceforge.net/projects/pnp4nagios/files/PNP-0.6/pnp4nagios-0.6.25.tar.gz
tar -xvf pnp4nagios-0.6.25.tar.gz
cd pnp4nagios-0.6.25
./configure --with-rrdtool=/usr/bin/rrdtool --with-nagios-user=nagios --with-nagios-group=nagcmd
make all
make install-webconf
make install-config
make install-init
make fullinstall
```

## Start and Enable Web & Nagios services
```
systemctl enable npcd
systemctl start npcd
systemctl restart nagios
systemctl restart httpd
```

## DELETE DEFAULT PHP PAGE
```
rm -rf /usr/local/pnp4nagios/share/install.php`
```
## VERIFY PNP4NAGIOS CONFIGURATION

`cd pnp4nagios-0.6.25/scripts`

`./verify_pnp_config_v2.pl -m bulk -c /usr/local/nagios/etc/nagios.cfg -p /usr/local/pnp4nagios/etc/`

## EDIT `vim /usr/loca/nagios/etc/nagios.cfg` file add add below lines at the end
```
# service performance data
#
service_perfdata_file=/usr/local/pnp4nagios/var/service-perfdata
service_perfdata_file_template=DATATYPE::SERVICEPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tSERVICEDESC::$SERVICEDESC$\tSERVICEPERFDATA::$SERVICEPERFDATA$\tSERVICECHECKCOMMAND::$SERVICECHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$\tSERVICESTATE::$SERVICESTATE$\tSERVICESTATETYPE::$SERVICESTATETYPE$
service_perfdata_file_mode=a
service_perfdata_file_processing_interval=15
service_perfdata_file_processing_command=process-service-perfdata-file

#
# host performance data starting with Nagios 3.0
#
host_perfdata_file=/usr/local/pnp4nagios/var/host-perfdata
host_perfdata_file_template=DATATYPE::HOSTPERFDATA\tTIMET::$TIMET$\tHOSTNAME::$HOSTNAME$\tHOSTPERFDATA::$HOSTPERFDATA$\tHOSTCHECKCOMMAND::$HOSTCHECKCOMMAND$\tHOSTSTATE::$HOSTSTATE$\tHOSTSTATETYPE::$HOSTSTATETYPE$
host_perfdata_file_mode=a
host_perfdata_file_processing_interval=15
host_perfdata_file_processing_command=process-host-perfdata-file`
```
## CHANGE below value from 0 to 1
```
process_performance_data=1
enable_environment_macros=1
```

## ERROR MESSAGES and SOLUTIONS
**CRIT]  Command process-service-perfdata-file is not defined**
```
vim /usr/local/nagios/etc/objects/commands.cfg

define command{
       command_name    process-service-perfdata-file
       command_line    /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/service-perfdata
}

define command{
       command_name    process-host-perfdata-file
       command_line    /usr/local/pnp4nagios/libexec/process_perfdata.pl --bulk=/usr/local/pnp4nagios/var/host-perfdata
}
```


## ERROR 2
- **[CRIT]  service_perfdata_command is defined (service_perfdata_command=process-service-perfdata)**
- **[CRIT]  service_perfdata_command is not allowed in mode 'bulk'**
- **[CRIT]  host_perfdata_command is defined (host_perfdata_command=process-host-perfdata)**
- **[CRIT]  host_perfdata_command is not allowed in mode 'bulk'**

### Solution
```
#host_perfdata_command=process-host-perfdata
#service_perfdata_command=process-service-perfdata
```
Edit nagios.cfg file and comment above lines

## ERROR 3
Call to undefined function simplexml_load_file()

### Solution
`yum install php-xml`


## ERROR 4
sizeof(): Parameter must be an array or an object that implements Countable

### Solution
```
/usr/local/pnp4nagios/share/application/models/data.php
Default line number: 979

Change from
if(sizeof($pages) > 0 ){

To
/*if(sizeof($pages) > 0 ){*/
   if(is_array($pages)&&sizeof($pages) > 0){

systemctl restart httpd
systemctl restart nagios
```

## ERROR 5
Deprecated Methods v/ith the same name	as the r class ??ni not De constructors in a future version of PHP	Servlces_JSON has a deprecated constructor 'usr local pnp4nagios share applications ison.php on line 115
Deprecated Methods with the same name	as the?? class *nH not be constructors in a future version of PHP	Services_JSON_Error has a deprecated constructor in 'usr>local'pnp4nagios>'share application/lib json.php on line 781
Deprecated Methods with the same name	as the?? class w* not be constructors in a future version of PHP	Services_JSON_Errot has a deprecated constructor in 'usr'local'pnp4nagios???Share application/lib json.php on line 795

### Solution
edit `vim /usr/local/pnp4nagios/share/application/lib/json.php` file

```
From
class Services_JSON
To
class _Services_JSON

From
class Services_JSON_Error extends PEAR_Error
To
class _Services_JSON_Error extends PEAR_Error

From
class _Services_JSON_Error
To
class _Services_JSON_Error
```
