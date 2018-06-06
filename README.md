# What the plugin does

The plugin provides in the Zabbix the information about the vulnerabilities of the entire infrastructure: the scope of the impact, and a list of affected hosts and ways to fix them.

# How the plugin works

Transmits this data to Vulners, and in return receives information on the vulnerabilities of each server.

Processes the received information, aggregates it and display it in Zabbix in the following form:
- CVSS score for each server.
- Command to fix all detected vulnerabilities of the server (where vulnerabilities were detected).
- List of security bulletins with the description of the vulnerabilities of the packages for the whole infrastructure.
- List of the packages that are vulnerable for the whole infrastructure.

Information about the security bulletins and packages  is presented in a following form:
- CVSS score of package or bulletins.
- Number of affected servers.
- Index of the impact on the infrastructure
- A detailed list of affected hosts.
- Hyperlink to the description of the bulletin.

Sometimes it is quite difficult to update all packages on all servers to a version that fixes vulnerabilities. The offeredformat of the information representation helps to work selectively with both needed servers and particular packages.

# Installation

**On zabbix-server host:**

- Move the ```zbx-vulners``` directory to /opt/monitoring/
- touch /var/log/zbxvulners.log
- chown -R zabbix:zabbix /opt/monitoring/zbx-vulners
- chown zabbix:zabbix /var/log/zbxvulners.log

**For all servers that require a vulnerability scan:**

- Move the ```os-report``` directory to /opt/monitoring/
- chown -R zabbix:zabbix /opt/monitoring/os-report


# Сonfiguration

## Vulners

Now you should get Vulners api-key. Log in to vulners.com, go to userinfo space https://vulners.com/userinfo. Then you should choose "apikey" section.
Choose "scan" in scope menu and click "Generate new key". You will get an api-key, which looks like this:
**RGB9YPJG7CFAXP35PMDVYFFJPGZ9ZIRO1VGO9K9269B0K86K6XQQQR32O6007NUK**

## Configuration file

You wll need to write Vulners api-key into configuration (parameter ```vuln_api_key```). Configuration is located in file  /opt/monitoring/zbx-vulners/zbxvulners_settings.py

Enter the following to connect to Zabbix:
-	The URL, username and password for connection with API. The User should have rights to create groups, hosts and templates in Zabbix.
-	Address and port of the Zabbix-server for pushing data using the zabbix-sender.

Here is example of config file:
```
vuln_api_key = 'RGB9YPJG7CFAXP35PMDVYFFJPGZ9ZIRO1VGO9K9269B0K86K6XQQQR32O6007NUK'

zbx_pass = 'yourpassword'
zbx_user = 'yourlogin'
zbx_url = 'https://zabbixfront.yourdomain.com'

zbx_server = 'zabbixserver.yourdomain.com'
zbx_port = '10051'
```

## Zabbix

You need to create these objects in Zabbix:
- A template; through which data will be collected from the servers.
- Zabbix hosts; for obtaining data on vulnerabilities.
- Dashboard; for their display.

To do this, run the ```/opt/monitoring/zbx-vulners/create_zbxobj.py``` script.
This will create all the necessary objects in Zabbix using the API.

Following this step. Using the Zabbix web interface, it is necessary to link the "Template Vulners" template to the hosts that you are doing a vulnerabilities scan on.

# Execution

Every day at 6 am, Zabbix will automatically receive the name, version and installed packages of the operation system of all servers.
Data processing is performed by script /opt/monitoring/zbx-vulners/zbxvulners.py.
This script is launched by the zabbix-agent every day at 7 am via the item "Service item" on the host "Vulners - Statistics".