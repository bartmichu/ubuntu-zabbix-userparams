# ubuntu-zabbix-userparams: Ubuntu system monitoring

## Usage

- Copy the userparams file (`ubuntu-userparams.conf`) to a directory included in your Zabbix Agent configuration file, such as `/etc/zabbix/zabbix_agentd.d/` or `/etc/zabbix/zabbix_agent2.d/`.
- Set the owner and group of the userparams file (`chown root:zabbix ubuntu-userparams.conf`).
- Set the mode of the userparams file (`chmod 0440 ubuntu-userparams.conf`).
- Copy the corresponding sudoers file (`ubuntu-userparams`) to the `/etc/sudoers.d/` directory.
- Set the mode of the sudoers file (`chmod 0440 /etc/sudoers.d/ubuntu-userparams`).
- Restart the Zabbix Agent (`systemctl restart zabbix-agent.service or systemctl restart zabbix-agent2.service`).
- Import the corresponding template file (`ubuntu-userparams-template.yaml`) on the Zabbix Server.
- Link the 'Ubuntu Linux by Zabbix agent active' template to the selected hosts.
- You may need to increase the Timeout value in your Zabbix Agent configuration file, for example, `Timeout=30`.

## System requirements

All the required packages are installed by default on Ubuntu Server 22.04.

- The `update-notifier-common` package needs to be installed for the `reboot.` and `updates.` keys.
- The `needrestart` and `libpam-systemd` packages need to be installed for the `needrestart.*` keys.
- The `distro-info` package needs to be installed for the `support.*` keys.
- The `sudo` package needs to be installed for some of the `needrestart` keys.

## Tested on

- Zabbix Agent 2 on Ubuntu 20.04 and Ubuntu 22.04
- Zabbix Server 6.0 LTS

## Template Macros

- `{$UBUNTUUSERPARAMS.REBOOT.GRACEPERIOD}`

  Grace period for reboot-pending error trigger.
  Default value: `24h`

- `{$UBUNTUUSERPARAMS.SECURITY.GRACEPERIOD}`

  Grace period for security updates error trigger.
  Default value: `24h`

- `{$UBUNTUUSERPARAMS.RELEASE.THRESHOLD1}`

  End-of-Life (EoL) warning threshold for releases (in days).
  Default value: 90

- `{$UBUNTUUSERPARAMS.RELEASE.THRESHOLD2}`

  End-of-Life (EoL) error threshold for releases (in days).
  Default value: 30

## Template Items

- `ubuntu-userparams.userparams-version`

  Return the version number of this file. This field is reserved for future use and intended for versioning within the template.

- `ubuntu-userparams.reboot.required`

  Check if the reboot-required file was created by a previous update or installation of a core library or service. This flag indicates that a system reboot is needed.

  Expected return value:
  - `1`: reboot is required
  - `2`: reboot is not required.

- `ubuntu-userparams.updates.all`

  Get the number of all applicable updates, including security updates.

  Expected return value: number indicating the number of updates.

- `ubuntu-userparams.updates.security`

  Check the number of pending security updates.

  Expected return value: number indicating the number of security updates.

- `ubuntu-userparams.needrestart.kernel`

  Check for obsolete kernel. Requires sudo.

  Expected return value:
  - `0`: unknown or failed to detect,
  - `1`: no pending upgrade,
  - `2`: ABI compatible upgrade pending,
  - `3`: version upgrade pending.

- `ubuntu-userparams.needrestart.libs`

  Get the number of services that are currently using deleted files (e.g., shared libraries).
  These services may need to be restarted after an update. Requires sudo.

  Expected return value: number indicating the number of services.

- `ubuntu-userparams.needrestart.pids`

  Get the number of processes that are currently using deleted files (e.g., shared libraries).
  These processes may need to be restarted after an update. Requires sudo.

  Expected return value: number indicating the number of processes.

- `ubuntu-userparams.support.eol`

  Calculate the number of days remaining until the installed release reaches its End Of Life milestone.

  Expected return value: number indicating the number of days.

- `ubuntu-userparams.support.status`

  Check the support status of the currently installed release.

  Expected return value:
  - `0`: unsupported,
  - `1`: supported.

- `ubuntu-userparams.packages.broken`

  Get the count of installed packages that are not marked as `ok` in the dpkg database.
  These packages are broken and need manual intervention, such as re-installation.

  Expected return value: number indicating the number of packages.

- `ubuntu-userparams.packages.problematic`

  Get the count of installed packages that are in a potentially problematic state (i.e., not `installed` or `config-files`).
  These packages may require additional configuration or installation steps.

  Expected return value: number indicating the number of packages.

- `ubuntu-userparams.graceful-shutdown`

  Check if the system was shut down gracefully prior to the current boot.

  Expected return value:
  - `1`: ungraceful shutdown,
  - `2`: graceful shutdown.

## Template triggers

- Ubuntu: Broken packages (operational data)

  Some of the installed packages are not marked as 'ok' in the dpkg database. These packages are broken and need manual intervention, such as re-installation.

- Ubuntu: Package updates are available (operational data)

- Ubuntu: Pending kernel upgrade - ABI compatible upgrade pending

  needrestart has detected an obsolete kernel.
  An ABI-compatible upgrade is pending, and a system reboot is required to apply these updates. It is recommended to consider rebooting your system.

- Ubuntu: Pending kernel upgrade - status unknown or failed to detect

- Ubuntu: Pending kernel upgrade - version upgrade pending

  needrestart has detected an obsolete kernel.
  A version upgrade is pending, and a system reboot is required to ensure that your system benefits from these updates. It is recommended to consider rebooting your system.

- Ubuntu: Problematic packages (operational data)

  Some of the installed packages are in a potentially problematic state (i.e., not 'installed' or 'config-files').
  These packages may require additional configuration or installation steps.

- Ubuntu: Security updates pending (operational data)

- Ubuntu: Security updates pending for over {$UBUNTUUSERPARAMS.SECURITY.GRACEPERIOD} (operational data)

  Default value: 24 hours

- Ubuntu: Some processes need to be restarted (operational data)

  There are running processes that are still using files and libraries that have been deleted or updated by recent upgrades. It is recommended to restart these processes in order to take advantage of the latest updates.

- Ubuntu: Some services need to be restarted (operational data)

  There are running services that are still using files and libraries that have been deleted or updated by recent upgrades. It is recommended to restart these services in order to take advantage of the latest updates.

- Ubuntu: System reboot is required

  A system reboot is required due to a previous update or installation of a core library or service.

- Ubuntu: System reboot pending for over {$UBUNTUUSERPARAMS.REBOOT.GRACEPERIOD}

  Default value: 24 hours

  A system reboot is required due to a previous update or installation of a core library or service.

- Ubuntu: The installed release is nearing EoL (operational data)

  This is a reminder that the installed Ubuntu release will very soon reach its End Of Life milestone. Once it reaches this milestone, it will no longer receive security patches or software updates and will no longer be supported by the vendor. It is advisable to promptly explore upgrade paths for your system.

- Ubuntu: The installed release is nearing EoL (operational data)

  This is a reminder that the installed Ubuntu release will soon reach its End Of Life milestone. Once it reaches this milestone, it will no longer receive security patches or software updates and will no longer be supported by the vendor. It is recommended to explore upgrade paths for your system.

- Ubuntu: Ungraceful shutdown

  The system was not shut down gracefully prior to the current boot.

- Ubuntu: Unsupported Ubuntu release

  The installed Ubuntu release has reached the End of Life milestone and is no longer supported by its vendor. It will not receive any further security patches or software updates.
