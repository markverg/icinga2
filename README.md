# icinga2

This repository contains the source for the
[icinga2](https://www.icinga.org/icinga2/) [docker](https://www.docker.com)
image.

## Image details

1. Based on debian:jessie
1. Supervisor, Apache2, MySQL, icinga2, icingacli, icingaweb2, and icingaweb2 director module
1. No SSH. Use docker [exec](https://docs.docker.com/engine/reference/commandline/exec/) or [nsenter](https://github.com/jpetazzo/nsenter)
1. If passwords are not supplied, they will be randomly generated and shown via stdout.

## Automated build

    docker pull jordan/icinga2

## Usage

Start a new container and bind to host's port 80

    sudo docker run -p 80:80 -t jordan/icinga2:latest

Start a new container and supply the icinga and icinga_web password

    sudo docker run -e ICINGA_PASSWORD="icinga" -e ICINGA_WEB_PASSWORD="icinga_web" -t jordan/icinga2:latest

## Icinga Web 2

Icinga Web 2 can be accessed at http://localhost/icingaweb2 with the credentials icingaadmin:icinga

## Icinga Web

Icinga Web (/icinga-web) is no longer included.  All the fun stuff is in Icinga web anyway ;)

## Graphite

The graphite writer can be enabled by setting thez ICINGA2_FEATURE_GRAPHITE variable to true or 1 and also supplying values for ICINGA2_FEATURE_GRAPHITE_HOST and ICINGA2_FEATURE_GRAPHITE_PORT.  This container does not have graphite  and the carbon daemons installed so ICINGA2_FEATURE_GRAPHITE_HOST should not be set to localhost.

Example:

```
sudo docker run --link graphite:graphite -e ICINGA2_FEATURE_GRAPHITE=true -e ICINGA2_FEATURE_GRAPHITE_HOST=graphite -e ICINGA2_FEATURE_GRAPHITE_PORT=2003 -t jordan/icinga2:latest
```

## Icinga Director

The [Icinga Director](https://github.com/Icinga/icingaweb2-module-director) Icinga Web 2 module is installed and enabled by default.  You can disable the automatic kickstart when the container starts by setting the DIRECTOR_KICKSTART variable to false.  To customize the kickstart settings, modify the /etc/icingaweb2/modules/director/kickstart.ini

## Sending Notification Mails

The container has `ssmtp` installed, which forwards mails to a preconfigured static server.

You have to create the files `ssmtp.conf` for general configuration and `revaliases` (mapping from local Unix-user to mail-address).

```
# ssmtp.conf
root=<E-Mail address to use on>
mailhub=smtp.<YOUR_MAILBOX>:587
UseSTARTTLS=YES
AuthUser=<Username for authentication (mostly the complete e-Mail-address)>
AuthPass=<YOUR_PASSWORD>
FromLineOverride=NO
```
**But be careful, ssmtp is not able to process special chars within the password correctly!**

`revaliases` follows the format: `Unix-user:e-Mail-address:server`.
Therefore the e-Mail-address has to match the `root`'s value in `ssmtp.conf`
Also server has to match mailhub from `ssmtp.conf` **but without the port**.

```
# revaliases
root:<VALUE_FROM_ROOT>:smtp.<YOUR_MAILBOX>
nagios:<VALUE_FROM_ROOT>:smtp.<YOUR_MAILBOX>
www-data:<VALUE_FROM_ROOT>:smtp.<YOUR_MAILBOX>
```

These files have to get mounted into the container. Add these flags to your `docker run`-command:
```
-v $(pwd)/revaliases:/etc/ssmtp/revaliases:ro
-v $(pwd)/ssmtp.conf:/etc/ssmtp/ssmtp.conf:ro
```

If you want to change the display-name of sender-address, you have to define the variable `ICINGA2_USER_FULLNAME`.

If this does not work, please ask your provider for the correct mail-settings or consider the [ssmtp.conf(5)-manpage](https://linux.die.net/man/5/ssmtp.conf) or Section ["Reverse Aliases" on ssmtp(8)](https://linux.die.net/man/8/ssmtp).
Also you can debug your config, by executing inside your container `ssmtp -v $address` and pressing 2x Enter.
It will send an e-Mail to `$address` and give verbose log and all error-messages.

## SSL Support

For enabling of SSL support, just add a volume to `/etc/apache2/ssl`, which contains these files:

- `icinga2.crt`: The certificate file for apache
- `icinga2.key`: The corresponding private key
- `icinga2.chain` (optional): If a certificate chain is needed, add this file. Consult your CA-vendor for additional info.

## Environment variables Reference

| Environmental Variable | Default Value | Description |
| ---------------------- | ------------- | ----------- |
| `ICINGA_PASSWORD` | *randomly generated* | MySQL password for icinga |
| `ICINGAWEB2_PASSWORD` | *randomly generated* | MySQL password for icingaweb2 |
| `DIRECTOR_PASSWORD` | *randomly generated* | MySQL password for icinga director |
| `IDO_PASSWORD` | *randomly generated* | MySQL password for ido |
| `DEBIAN_SYS_MAINT_PASSWORD` | *randomly generated* | Password for debian-syst-maint account |
| `ICINGA2_FEATURE_GRAPHITE` | false | Set to true or 1 to enable graphite writer |
| `ICINGA2_FEATURE_GRAPHITE_HOST` | graphite | hostname or IP address where Carbon/Graphite daemon is running |
| `ICINGA2_FEATURE_GRAPHITE_PORT` | 2003 | Carbon port for graphite |
| `ICINGA2_FEATURE_GRAPHITE_URL` | http://${ICINGA2_FEATURE_GRAPHITE_HOST} | Web-URL for Graphite |
| `DIRECTOR_KICKSTART` | true |  Set to false to disable director auto kickstart at container startup |
| `ICINGAWEB2_ADMIN_USER` | icingaadmin | Icingaweb2 Login User<br>*After changing the username, you should also remove the old User in icingaweb2-> Configuration-> Authentication-> Users* |
| `ICINGAWEB2_ADMIN_PASS` | icinga | Icingaweb2 Login Password |
| `ICINGA2_USER_FULLNAME` | Icinga | Sender's display-name for notification e-Mails |

## Volume Reference

All these folders are configured and able to get mounted as volume. The bottom ones are not quite neccessary.

| Volume | ro/rw | Description & Usage |
| ------ | ----- | ------------------- |
| /etc/apache2/ssl | **ro** |  Mount optional SSL-Certificates (see SSL Support) |
| /etc/ssmtp/revaliases | **ro** | revaliases map (see Sending Notification Mails) |
| /etc/ssmtp/ssmtp.conf | **ro** | ssmtp configufation (see Sending Notification Mails) |
| /etc/icinga2 | rw | Icinga2 configuration folder |
| /etc/icingaweb2 | rw | Icingaweb2 configuration folder |
| /var/lib/mysql | rw | MySQL Database |
| /var/lib/icinga2 | rw | Icinga2 Data |
| /var/log/apache2 | rw | logfolder for apache2 (not neccessary) |
| /var/log/icinga2 | rw | logfolder for icinga2 (not neccessary) |
| /var/log/icingaweb2 | rw | logfolder for icingaweb2 (not neccessary) |
| /var/log/mysql | rw | logfolder for mysql (not neccessary) |
| /var/spool/icinga2 | rw | spool-folder for icinga2 (not neccessary) |
| /var/cache/icinga2 | rw | cache-folder for icinga2 (not neccessary) |
