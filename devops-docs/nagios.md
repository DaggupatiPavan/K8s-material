# Nagios - Monitoring System

## Overview

Nagios is a powerful and widely used open-source monitoring system that enables organizations to identify and resolve IT infrastructure problems before they affect critical business processes. It provides comprehensive monitoring of servers, switches, applications, and services.

### Key Features
- **Comprehensive Monitoring**: Monitor servers, networks, and applications
- **Alerting**: Customizable alerting via email, SMS, or other methods
- **Visualization**: Web interface for monitoring and reporting
- **Plugin Architecture**: Extensible through custom plugins
- **Distributed Monitoring**: Support for distributed monitoring setups
- **Performance Data**: Collect and graph performance data
- **Trending**: Historical data analysis and trending

## Core Concepts

### Nagios Architecture
- **Nagios Core**: Core monitoring engine
- **Nagios Plugins**: Executables that perform checks
- **NRPE (Nagios Remote Plugin Executor)**: Agent for executing plugins on remote systems
- **NSCA (Nagios Service Check Acceptor)**: For passive service checks
- **NDOUtils**: Database utilities for storing Nagios data
- **Web Interface**: Web-based monitoring interface

### Key Components
- **Hosts**: Physical or virtual devices to be monitored
- **Services**: Applications or services running on hosts
- **Contacts**: People to be notified when problems occur
- **Commands**: Executables that perform checks or notifications
- **Time Periods**: Define when monitoring and notifications occur
- **Dependencies**: Define relationships between hosts and services

### Monitoring Types
- **Active Monitoring**: Nagios initiates checks
- **Passive Monitoring**: External applications send results to Nagios
- **Distributed Monitoring**: Multiple Nagios instances monitoring different parts of infrastructure

## Installation & Setup

### Prerequisites
- **Operating System**: Linux (Ubuntu, CentOS, etc.)
- **Web Server**: Apache or Nginx
- **PHP**: PHP 7.0 or later
- **Database**: MySQL or MariaDB (optional)
- **Dependencies**: GCC, GD libraries, etc.

### Installation Methods

#### Using Package Manager (Ubuntu/Debian)
```bash
# Update package index
sudo apt update

# Install prerequisites
sudo apt install -y autoconf gcc libc6 make wget unzip apache2 php libgd-dev

# Download Nagios Core
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz

# Extract Nagios
tar -xzf nagios-4.4.6.tar.gz
cd nagios-4.4.6

# Configure and compile
./configure --with-httpd-conf=/etc/apache2/sites-enabled
make all

# Create user and group
sudo make install-groups-users
sudo usermod -a -G nagios www-data

# Install Nagios
sudo make install
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
sudo make install-webconf

# Install Nagios plugins
sudo apt install -y nagios-plugins

# Configure Apache
sudo a2enmod rewrite
sudo a2enmod cgi
sudo systemctl restart apache2

# Create Nagios admin user
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

# Start Nagios
sudo systemctl start nagios
sudo systemctl enable nagios

# Verify installation
sudo systemctl status nagios
```

#### Using Package Manager (CentOS/RHEL)
```bash
# Install prerequisites
sudo yum install -y gcc glibc glibc-common wget httpd php gd gd-devel perl postfix

# Download Nagios Core
wget https://assets.nagios.com/downloads/nagioscore/releases/nagios-4.4.6.tar.gz

# Extract Nagios
tar -xzf nagios-4.4.6.tar.gz
cd nagios-4.4.6

# Configure and compile
./configure
make all

# Install Nagios
sudo make install
sudo make install-commandmode
sudo make install-daemoninit
sudo make install-config
sudo make install-webconf

# Install Nagios plugins
sudo yum install -y nagios-plugins-all

# Configure SELinux
sudo setenforce 0
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config

# Configure firewall
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload

# Start services
sudo systemctl start httpd
sudo systemctl start nagios
sudo systemctl enable httpd
sudo systemctl enable nagios

# Create Nagios admin user
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

# Verify installation
sudo systemctl status nagios
```

## Nagios Configuration

### Configuration File Structure
```
/usr/local/nagios/etc/
├── nagios.cfg           # Main configuration file
├── cgi.cfg              # CGI configuration
├── resource.cfg         # Resource definitions
├── objects/
│   ├── commands.cfg     # Command definitions
│   ├── contacts.cfg     # Contact definitions
│   ├── contactgroups.cfg
│   ├── hosts.cfg        # Host definitions
│   ├── hostgroups.cfg
│   ├── services.cfg     # Service definitions
│   ├── servicegroups.cfg
│   ├── templates.cfg    # Template definitions
│   ├── timeperiods.cfg  # Time period definitions
│   └── dependencies.cfg
└── conf.d/              # Additional configuration files
```

### Main Configuration (nagios.cfg)
```cfg
# Main Nagios configuration file
log_file=/usr/local/nagios/var/nagios.log
cfg_file=/usr/local/nagios/etc/objects/commands.cfg
cfg_file=/usr/local/nagios/etc/objects/contacts.cfg
cfg_file=/usr/local/nagios/etc/objects/timeperiods.cfg
cfg_file=/usr/local/nagios/etc/objects/templates.cfg
cfg_file=/usr/local/nagios/etc/objects/hosts.cfg
cfg_file=/usr/local/nagios/etc/objects/services.cfg

# Object configuration directory
cfg_dir=/usr/local/nagios/etc/objects

# Lock file
lock_file=/usr/local/nagios/var/nagios.lock

# Temp file
temp_file=/usr/local/nagios/var/nagios.tmp

# Status file
status_file=/usr/local/nagios/var/status.dat

# Log rotation
log_rotation_method=d

# Enable external commands
check_external_commands=1

# Command check interval
command_check_interval=10s

# Interval length
interval_length=60

# Service check timeout
service_check_timeout=60

# Host check timeout
host_check_timeout=30

# Event broker options
broker_module=/usr/local/nagios/lib/ndoutils/ndo2db broker
```

### Host Configuration (hosts.cfg)
```cfg
# Host template
define host {
    name                    linux-server
    use                     generic-host
    check_period            24x7
    check_interval          5
    retry_interval          1
    max_check_attempts      10
    check_command           check-host-alive
    notification_period    24x7
    notification_interval  60
    notification_options    d,r
    contact_groups          admins
    register                0
}

# Individual host
define host {
    use                     linux-server
    host_name               web-server-01
    alias                   Web Server 01
    address                 192.168.1.100
    hostgroups              web-servers
}

# Another host
define host {
    use                     linux-server
    host_name               db-server-01
    alias                   Database Server 01
    address                 192.168.1.101
    hostgroups              db-servers
}
```

### Service Configuration (services.cfg)
```cfg
# Service template
define service {
    name                            generic-service
    use                             local-service
    check_period                    24x7
    max_check_attempts              3
    normal_check_interval           5
    retry_check_interval            1
    notification_period            24x7
    notification_options           w,u,c,r
    contact_groups                  admins
    register                        0
}

# HTTP service
define service {
    use                     generic-service
    host_name               web-server-01
    service_description     HTTP
    check_command           check_http
    notifications_enabled   1
}

# SSH service
define service {
    use                     generic-service
    host_name               web-server-01
    service_description     SSH
    check_command           check_ssh
    notifications_enabled   1
}

# CPU service
define service {
    use                     generic-service
    host_name               web-server-01
    service_description     CPU Load
    check_command           check_local_load!5.0,4.0,3.0
    notifications_enabled   1
}

# Memory service
define service {
    use                     generic-service
    host_name               web-server-01
    service_description     Memory
    check_command           check_local_memory!80!90
    notifications_enabled   1
}
```

### Contact Configuration (contacts.cfg)
```cfg
# Contact template
define contact {
    name                            generic-contact
    service_notification_period     24x7
    host_notification_period        24x7
    service_notification_options    w,u,c,r,f,s
    host_notification_options       d,u,r,f,s
    service_notification_commands   notify-service-by-email
    host_notification_commands      notify-host-by-email
    register                        0
}

# Individual contact
define contact {
    use                     generic-contact
    contact_name            nagiosadmin
    alias                   Nagios Admin
    email                   admin@example.com
    pager                   1234567890@pager.example.com
}

# Contact group
define contactgroup {
    contactgroup_name       admins
    alias                   Nagios Administrators
    members                 nagiosadmin
}
```

### Command Configuration (commands.cfg)
```cfg
# Host check command
define command {
    command_name    check-host-alive
    command_line    $USER1$/check_ping -H $HOSTADDRESS$ -w 3000.0,80% -c 5000.0,100% -p 1
}

# HTTP check command
define command {
    command_name    check_http
    command_line    $USER1$/check_http -I $HOSTADDRESS$ $ARG1$
}

# SSH check command
define command {
    command_name    check_ssh
    command_line    $USER1$/check_ssh $HOSTADDRESS$ $ARG1$
}

# CPU load check command
define command {
    command_name    check_local_load
    command_line    $USER1$/check_load -w $ARG1$ -c $ARG2$
}

# Memory check command
define command {
    command_name    check_local_memory
    command_line    $USER1$/check_memory -w $ARG1$ -c $ARG2$
}

# Disk space check command
define command {
    command_name    check_local_disk
    command_line    $USER1$/check_disk -w $ARG1$ -c $ARG2$ -p $ARG3$
}

# Notification commands
define command {
    command_name    notify-host-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" | /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
}

define command {
    command_name    notify-service-by-email
    command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTNAME$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" | /usr/bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $SERVICEDESC$ on $HOSTNAME$ is $SERVICESTATE$ **" $CONTACTEMAIL$
}
```

### Time Period Configuration (timeperiods.cfg)
```cfg
# 24x7 time period
define timeperiod {
    timeperiod_name 24x7
    alias           24 Hours A Day, 7 Days A Week
    sunday          00:00-24:00
    monday          00:00-24:00
    tuesday         00:00-24:00
    wednesday       00:00-24:00
    thursday        00:00-24:00
    friday          00:00-24:00
    saturday        00:00-24:00
}

# Work hours time period
define timeperiod {
    timeperiod_name workhours
    alias           Normal Work Hours
    monday          09:00-17:00
    tuesday         09:00-17:00
    wednesday       09:00-17:00
    thursday        09:00-17:00
    friday          09:00-17:00
}

# No notifications time period
define timeperiod {
    timeperiod_name none
    alias           No Notifications
}
```

## Nagios Plugins

### Common Plugins

#### check_ping
```bash
# Check host availability
/usr/local/nagios/libexec/check_ping -H 192.168.1.100 -w 3000.0,80% -c 5000.0,100% -p 1

# Check with warning and critical thresholds
/usr/local/nagios/libexec/check_ping -H 192.168.1.100 -w 100.0,20% -c 200.0,50% -p 1
```

#### check_http
```bash
# Check HTTP service
/usr/local/nagios/libexec/check_http -H 192.168.1.100

# Check specific URL
/usr/local/nagios/libexec/check_http -H 192.168.1.100 -u /health

# Check HTTPS
/usr/local/nagios/libexec/check_http -H 192.168.1.100 -S

# Check response time
/usr/local/nagios/libexec/check_http -H 192.168.1.100 -w 5 -c 10
```

#### check_ssh
```bash
# Check SSH service
/usr/local/nagios/libexec/check_ssh -H 192.168.1.100

# Check specific port
/usr/local/nagios/libexec/check_ssh -H 192.168.1.100 -p 2222
```

#### check_disk
```bash
# Check disk space
/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /

# Check specific mount point
/usr/local/nagios/libexec/check_disk -w 5GB -c 2GB -p /var

# Check all mount points
/usr/local/nagios/libexec/check_disk -w 20% -c 10%
```

#### check_load
```bash
# Check CPU load
/usr/local/nagios/libexec/check_load -w 5.0,4.0,3.0 -c 10.0,8.0,6.0

# Check with specific thresholds
/usr/local/nagios/libexec/check_load -w 2.0,1.5,1.0 -c 4.0,3.0,2.0
```

#### check_memory
```bash
# Check memory usage
/usr/local/nagios/libexec/check_memory -w 80% -c 90%

# Check with absolute values
/usr/local/nagios/libexec/check_memory -w 1GB -c 500MB
```

#### check_procs
```bash
# Check process count
/usr/local/nagios/libexec/check_procs -w 200 -c 300

# Check specific process
/usr/local/nagios/libexec/check_procs -C httpd -w 5:10 -c 0:20

# Check zombie processes
/usr/local/nagios/libexec/check_procs -s Z -w 1 -c 5
```

### Custom Plugins

#### Python Plugin Example
```python
#!/usr/bin/env python3
import sys
import requests
import argparse

def check_api_health(url, warning_threshold, critical_threshold):
    try:
        response = requests.get(url, timeout=10)
        response_time = response.elapsed.total_seconds() * 1000
        
        if response.status_code != 200:
            print(f"CRITICAL - API returned status code {response.status_code}")
            return 2
        
        if response_time > critical_threshold:
            print(f"CRITICAL - API response time is {response_time:.2f}ms")
            return 2
        elif response_time > warning_threshold:
            print(f"WARNING - API response time is {response_time:.2f}ms")
            return 1
        else:
            print(f"OK - API response time is {response_time:.2f}ms")
            return 0
            
    except requests.exceptions.RequestException as e:
        print(f"CRITICAL - API check failed: {str(e)}")
        return 2

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Check API health')
    parser.add_argument('--url', required=True, help='API URL to check')
    parser.add_argument('--warning', type=float, default=1000, help='Warning threshold in milliseconds')
    parser.add_argument('--critical', type=float, default=2000, help='Critical threshold in milliseconds')
    
    args = parser.parse_args()
    
    exit_code = check_api_health(args.url, args.warning, args.critical)
    sys.exit(exit_code)
```

#### Shell Plugin Example
```bash
#!/bin/bash
# Custom plugin to check log file for errors

LOG_FILE="/var/log/application.log"
ERROR_PATTERN="ERROR"
WARNING_PATTERN="WARNING"

# Check if log file exists
if [ ! -f "$LOG_FILE" ]; then
    echo "CRITICAL - Log file $LOG_FILE does not exist"
    exit 2
fi

# Count errors and warnings
error_count=$(grep -c "$ERROR_PATTERN" "$LOG_FILE")
warning_count=$(grep -c "$WARNING_PATTERN" "$LOG_FILE")

# Determine status
if [ $error_count -gt 10 ]; then
    echo "CRITICAL - $error_count errors found in $LOG_FILE"
    exit 2
elif [ $error_count -gt 5 ]; then
    echo "WARNING - $error_count errors found in $LOG_FILE"
    exit 1
elif [ $warning_count -gt 20 ]; then
    echo "WARNING - $warning_count warnings found in $LOG_FILE"
    exit 1
else
    echo "OK - $error_count errors, $warning_count warnings found in $LOG_FILE"
    exit 0
fi
```

## NRPE (Nagios Remote Plugin Executor)

### NRPE Installation on Remote Host
```bash
# Install NRPE on Ubuntu/Debian
sudo apt install -y nagios-nrpe-server nagios-plugins

# Configure NRPE
sudo nano /etc/nagios/nrpe.cfg

# Add allowed hosts
allowed_hosts=192.168.1.50  # Nagios server IP

# Add custom commands
command[check_users]=/usr/lib/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib/nagios/plugins/check_load -w 15,10,5 -c 30,25,20
command[check_disk]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10%
command[check_procs]=/usr/lib/nagios/plugins/check_procs -w 250 -c 400

# Restart NRPE
sudo systemctl restart nagios-nrpe-server
sudo systemctl enable nagios-nrpe-server
```

### NRPE Configuration on Nagios Server
```cfg
# Define NRPE command
define command {
    command_name    check_nrpe
    command_line    $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}

# Use NRPE to check remote host
define service {
    use                     generic-service
    host_name               remote-server-01
    service_description     CPU Load
    check_command           check_nrpe!check_load
}

define service {
    use                     generic-service
    host_name               remote-server-01
    service_description     Disk Space
    check_command           check_nrpe!check_disk
}

define service {
    use                     generic-service
    host_name               remote-server-01
    service_description     Users
    check_command           check_nrpe!check_users
}
```

## Nagios Web Interface

### Accessing the Web Interface
- URL: http://nagios-server/nagios
- Username: nagiosadmin
- Password: (as set during installation)

### Main Interface Sections
- **Tactical Overview**: Summary of all hosts and services
- **Map**: Visual representation of network infrastructure
- **Hosts**: List of all monitored hosts
- **Services**: List of all monitored services
- **Problems**: List of current problems
- **Reports**: Performance and availability reports
- **Configuration**: Configuration management interface

### Common Web Interface Operations
- **Acknowledge Problems**: Acknowledge and temporarily disable notifications
- **Schedule Downtime**: Schedule maintenance periods
- **Disable Notifications**: Disable notifications for hosts/services
- **Force Checks**: Force immediate checks of hosts/services
- **View Performance Data**: View historical performance data

## Nagios XI (Enterprise Version)

### Nagios XI Features
- **Advanced Web Interface**: Enhanced web interface with more features
- **Configuration Wizards**: Easy configuration through wizards
- **Advanced Reporting**: Comprehensive reporting and dashboards
- **Customizable Dashboards**: Create custom monitoring dashboards
- **API Access**: REST API for integration
- **Multi-tenancy**: Support for multiple user accounts
- **Advanced Alerting**: Advanced alerting and notification features

### Nagios XI Installation
```bash
# Download Nagios XI
wget https://assets.nagios.com/downloads/nagiosxi/xi-latest.tar.gz

# Extract Nagios XI
tar xzf xi-latest.tar.gz
cd nagiosxi

# Run installer
sudo ./fullinstall

# Follow the installation wizard
# Configure database, admin user, etc.

# Access web interface
# URL: http://nagios-server/nagiosxi
```

## Interview Questions

### Beginner Level
1. **What is Nagios?**
   - Nagios is an open-source monitoring system that monitors IT infrastructure including servers, networks, and applications

2. **What are the main components of Nagios?**
   - Nagios Core, Nagios Plugins, NRPE, NSCA, and the web interface

3. **What is the purpose of Nagios plugins?**
   - Plugins are executables that perform checks on hosts and services, returning status information to Nagios

4. **What is NRPE?**
   - NRPE (Nagios Remote Plugin Executor) is an agent that allows Nagios to execute plugins on remote systems

### Intermediate Level
1. **What is the difference between active and passive monitoring in Nagios?**
   - Active monitoring is initiated by Nagios, while passive monitoring involves external applications sending results to Nagios

2. **How does Nagios handle notifications?**
   - Nagios sends notifications based on configured contact groups, time periods, and notification options

3. **What are Nagios templates?**
   - Templates are reusable configuration objects that define common properties for hosts, services, and contacts

4. **How do you create custom Nagios plugins?**
   - Custom plugins can be written in any language that can return exit codes (0=OK, 1=WARNING, 2=CRITICAL, 3=UNKNOWN)

### Advanced Level
1. **How does Nagios handle distributed monitoring?**
   - Nagios supports distributed monitoring through multiple Nagios instances that can send results to a central server

2. **What is the purpose of Nagios time periods?**
   - Time periods define when monitoring and notifications should occur, allowing for different schedules for different environments

3. **How do you integrate Nagios with other monitoring tools?**
   - Nagios can be integrated through APIs, custom scripts, or by using tools like Nagios Fusion or Nagios XI

4. **What are the best practices for Nagios configuration?**
   - Use templates, organize configurations logically, implement proper security, and maintain documentation

## Best Practices

### Configuration Management
- **Use Templates**: Create and use templates for common configurations
- **Organize Files**: Keep configurations organized in logical file structure
- **Version Control**: Store Nagios configurations in version control
- **Documentation**: Document all configurations and custom plugins
- **Testing**: Test all configurations before deploying to production

### Security
- **Secure Web Interface**: Use SSL/TLS for web interface access
- **Limit Access**: Restrict access to Nagios files and web interface
- **Use Firewalls**: Configure firewalls to limit access to Nagios server
- **Regular Updates**: Keep Nagios and plugins updated
- **Audit Logs**: Regularly audit Nagios logs and configurations

### Performance
- **Optimize Check Intervals**: Set appropriate check intervals based on importance
- **Use Distributed Monitoring**: Implement distributed monitoring for large environments
- **Monitor Nagios Server**: Monitor the Nagios server itself
- **Database Optimization**: Optimize database performance if using NDOUtils
- **Load Balancing**: Use load balancing for web interface access

## Troubleshooting

### Common Issues
```bash
# Check Nagios status
sudo systemctl status nagios

# Check Nagios logs
sudo tail -f /usr/local/nagios/var/nagios.log

# Verify configuration
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg

# Check web server status
sudo systemctl status apache2

# Check NRPE status
sudo systemctl status nagios-nrpe-server

# Test NRPE connection
/usr/local/nagios/libexec/check_nrpe -H remote-host

# Test plugin execution
/usr/local/nagios/libexec/check_ping -H 192.168.1.100
```

### Debugging Tips
- **Check Logs**: Always check Nagios logs for error messages
- **Verify Configuration**: Use nagios -v to verify configuration syntax
- **Test Plugins**: Test plugins manually before adding to Nagios
- **Check Permissions**: Ensure proper file permissions
- **Monitor Resources**: Monitor Nagios server resources

## Resources

### Official Documentation
- [Nagios Documentation](https://www.nagios.org/documentation/)
- [Nagios Plugins Documentation](https://www.nagios-plugins.org/doc/)
- [NRPE Documentation](https://github.com/NagiosEnterprises/nrpe/blob/master/docs/NRPE.pdf)

### Learning Resources
- [Nagios Training](https://www.nagios.com/training/)
- [Nagios Tutorials](https://www.nagios.com/tutorials/)
- [Nagios Community](https://support.nagios.com/)

### Community
- [Nagios Forums](https://support.nagios.com/forum)
- [Nagios GitHub](https://github.com/Nagios)
- [Nagios Stack Overflow](https://stackoverflow.com/questions/tagged/nagios)