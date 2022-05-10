# ubuntu-zabbix-userparams: Ubuntu system monitoring

## Usage

- Copy userparams file (```ubuntu-userparams.conf```) to a directory included in your Zabbix Agent configuration file, e.g. ```/etc/zabbix/zabbix_agentd.d/``` or ```/etc/zabbix/zabbix_agent2.d/```
- Set userparams file owner and group (```chown root:zabbix ubuntu-userparams.conf```)
- Set userparams file mode (```chmod 0440 ubuntu-userparams.conf```)
- Copy corresponding sudoers file (```ubuntu-userparams-sudoers```) to ```/etc/sudoers.d/``` directory
- Set sudoers file mode (```chmod 0440 /etc/sudoers.d/ubuntu-userparams-sudoers```)
- Restart Zabbix Agent (```systemctl restart zabbix-agent.service``` or ```systemctl restart zabbix-agent2.service```)
- Import corresponding template file (```ubuntu-userparams-template.yaml```) on Zabbix Server
- Link ```Template ubuntu-userparams by Zabbix agent active``` template to selected hosts
- You may need to increase the Timeout value in your Zabbix Agent configuration file, e.g. ```Timeout=30```

## System requirements

- ```update-notifier-common``` package installed for ```reboot.*``` and ```updates.*``` keys
- ```needrestart``` and ```libpam-systemd``` packages installed for ```needrestart.*``` keys
- ```distro-info``` package installed for ```support.*``` keys
- ```sudo``` installed for some of ```needrestart``` keys

## Tested on

- Zabbix Agent/Zabbix Agent 2 on Ubuntu 20.04 and Ubuntu 22.04
- Zabbix Server 5.x

## Template Macros

- ```{$UBUNTUUSERPARAMS.REBOOT.GRACEPERIOD}```

  Grace period for reboot-pending error. Default value: ```24h```

- ```{$UBUNTUUSERPARAMS.SECURITY.GRACEPERIOD}```

  Grace period for security updates error. Default value: ```24h```

## Template Keys

- ```ubuntu-userparams.userparams-version```

  Return version number of this file. Reserved for future use (versioning) within the template.

- ```ubuntu-userparams.reboot.required```

  Check if the reboot-required file was created by a previous update or install of a core library or service. This flag indicates that a system reboot is needed.

  Expected return value: ```1``` - reboot is required, ```2``` - reboot is not required.

- ```ubuntu-userparams.updates.all```

  Get the number of all applicable updates, including security updates.

  Expected return value: number indicating the number of updates.

- ```ubuntu-userparams.updates.security```

  Check the number of pending security updates.

  Expected return value: number indicating the number of security updates.

- ```ubuntu-userparams.needrestart.kernel```

  Check for obsolete kernel. Requires sudo.

  Expected return value: ```0``` - unknown or failed to detect, ```1``` - no pending upgrade, ```2``` - ABI compatible upgrade pending, ```3``` - version upgrade pending.

- ```ubuntu-userparams.needrestart.libs```

  Get the number of services which are using meanwhile deleted files (e.g. shared libraries). These services may need to be restarted after an update. Requires sudo.

  Expected return value: number indicating the number of services.

- ```ubuntu-userparams.needrestart.pids```

  Get the number of processes which are using meanwhile deleted files. These processes may need to be restarted after an update. Requires sudo.

  Expected return value: number indicating the number of processes.

- ```ubuntu-userparams.support.eol```

  Get the number of days until installed release reaches End Of Life milestone.

  Expected return value: number indicating the number of days.

- ```ubuntu-userparams.support.status```

  Check support status for currently installed release.

  Expected return value: ```0``` - unsupported, ```1``` - supported.

- ```ubuntu-userparams.packages.broken```

  Get the number of installed packages that are not marked as 'ok' in the dpkg database. These packages are broken and require manual intervention (i.e. re-installation).

  Expected return value: number indicating the number of packages.

- ```ubuntu-userparams.packages.problematic```

  Get the number of installed packages that are in potentially problematic state (i.e. different than 'installed' and 'config-files'). These packages may require further configuration or installation steps.

  Expected return value: number indicating the number of packages.

## Template triggers

- Package updates are available (operational data)

- Pending kernel upgrade - ABI compatible upgrade pending

  Obsolete kernel was detected by needrestart. ABI compatible upgrade is pending and reboot is required to ensure that your system benefits from these updates. You should consider rebooting.

- Pending kernel upgrade - status unknown or failed to detect

- Pending kernel upgrade - version upgrade pending

  Obsolete kernel was detected by needrestart. Version upgrade is pending and reboot is required to ensure that your system benefits from these updates. You should consider rebooting.

- Security updates pending (operational data)

- Security updates pending for over {$UBUNTUUSERPARAMS.SECURITY.GRACEPERIOD}

  Default value: 24 hours

- Some processes need to be restarted (operational data)

  There are running processes which still use files deleted or updated by recent upgrades. They should be restarted to benefit from the latest updates.

- Some services need to be restarted (operational data)

  There are running services which still use files and libraries deleted or updated by recent upgrades. They should be restarted to benefit from the latest updates.

- System reboot is required

  System reboot is needed due to previous update or install of a core library or service.

- System reboot pending for over {$UBUNTUUSERPARAMS.REBOOT.GRACEPERIOD}

  Default value: 24 hours

- Ubuntu release nears its End Of Life (operational data)

  This is a reminder that installed Ubuntu release will reach the End Of Life milestone very soon. After that milestone it will not receive security patches or other software updates and will not be supported by its vendor. You should look into upgrade paths immediately if you haven't already.

- Ubuntu release nears its End Of Life (operational data)

  This is a reminder that installed Ubuntu release will soon reach the End Of Life milestone. After that milestone it will not receive security patches or other software updates and will not be supported by its vendor. You should look into upgrade paths.

- Unsupported Ubuntu release

  Installed Ubuntu release reached the End Of Life milestone and is not supported by its vendor. It will not receive security patches or other software updates.

- Broken packages (operational data)

  Some of installed packages are not marked as 'ok' in the dpkg database. These packages are broken and require manual intervention (i.e. re-installation).

- Problematic packages (operational data)

  Some of installed packages are in potentially problematic state (i.e. different than 'installed' and 'config-files'). These packages may require further configuration or installation steps.
