# Chef - Configuration Management Tool

## Overview

Chef is a powerful configuration management and automation platform that transforms infrastructure into code. It enables you to automate the provisioning, configuration, and management of infrastructure through code, making it repeatable, versionable, testable, and human-readable.

### Key Features
- **Infrastructure as Code**: Define infrastructure configuration as code
- **Idempotent Operations**: Ensure consistent state across runs
- **Cross-Platform**: Support for multiple operating systems
- **Test-Driven Infrastructure**: Test infrastructure code before deployment
- **Large Ecosystem**: Extensive collection of cookbooks and resources
- **Enterprise Support**: Commercial support and training available
- **Integration**: Integrates with DevOps tools and cloud platforms

## Core Concepts

### Chef Architecture
- **Chef Workstation**: Development environment for writing and testing cookbooks
- **Chef Server**: Central hub for storing cookbooks, policies, and node information
- **Chef Infra Client**: Agent that runs on managed nodes
- **Cookbooks**: Collection of recipes, attributes, and templates
- **Recipes**: Files containing configuration instructions
- **Resources**: Statements of configuration policy
- **Templates**: ERB templates for configuration files
- **Attributes**: Configuration values for cookbooks

### Key Components
- **Chef Workstation**: Where you write and test cookbooks
- **Chef Server**: Stores and distributes configuration data
- **Chef Infra Client**: Applies configurations to nodes
- **Ohai**: System discovery tool
- **Knife**: Command-line tool for interacting with Chef Server
- **Berks**: Dependency management tool for cookbooks
- **Test Kitchen**: Testing framework for cookbooks
- **Foodcritic**: Linting tool for cookbooks

### Chef Workflow
1. **Write**: Create cookbooks and recipes
2. **Test**: Test cookbooks with Test Kitchen
3. **Upload**: Upload cookbooks to Chef Server
4. **Apply**: Apply configurations to nodes
5. **Verify**: Verify configurations are applied correctly

## Installation & Setup

### Prerequisites
- **Operating System**: Linux, macOS, Windows
- **Ruby**: Chef requires Ruby 2.6 or later
- **Memory**: Minimum 2GB RAM (4GB recommended)
- **Disk**: Minimum 10GB free space
- **Network**: Internet connection for downloading cookbooks

### Installation Methods

#### Using Chef Workstation
```bash
# Download Chef Workstation
# Visit: https://downloads.chef.io/chef-workstation

# Install on Ubuntu/Debian
sudo dpkg -i chef-workstation_<version>_amd64.deb

# Install on CentOS/RHEL
sudo rpm -Uvh chef-workstation_<version>_amd64.rpm

# Install on macOS
sudo installer -pkg chef-workstation-<version>.pkg -target /

# Verify installation
chef --version
```

#### Using Chef Server
```bash
# Download Chef Server
# Visit: https://downloads.chef.io/chef-server

# Install on Ubuntu/Debian
sudo dpkg -i chef-server-core_<version>_amd64.deb

# Install on CentOS/RHEL
sudo rpm -Uvh chef-server-core-<version>.el7.x86_64.rpm

# Configure Chef Server
sudo chef-server-ctl reconfigure

# Create admin user
sudo chef-server-ctl user-create user_name First Last email password --filename user_name.pem

# Create organization
sudo chef-server-ctl org-create org_name "Organization Name" --association_user user_name --filename org_name.pem
```

#### Using Chef Infra Client
```bash
# Install on Ubuntu/Debian
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef-infrastructure-client

# Install on CentOS/RHEL
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef-infrastructure-client

# Install on Windows
. { iwr -useb https://omnitruck.chef.io/install.ps1 } | iex; install -project chef-infrastructure-client

# Verify installation
chef-client -v
```

## Chef Basics

### Chef Repository Structure
```
chef-repo/
├── cookbooks/           # Custom cookbooks
├── roles/               # Role definitions
├── environments/        # Environment definitions
├── data_bags/           # Data bags
├── .chef/               # Chef configuration
├── Berksfile            # Cookbook dependencies
├── Policyfile.rb        # Policy file
└── metadata.rb          # Repository metadata
```

### Cookbook Structure
```
cookbooks/my_cookbook/
├── attributes/          # Attribute files
│   └── default.rb
├── files/               # Static files
│   └── default/
├── libraries/            # Custom libraries
│   └── default.rb
├── recipes/             # Recipe files
│   └── default.rb
├── resources/           # Custom resources
│   └── default.rb
├── templates/           # Template files
│   └── default/
├── test/                # Test files
│   └── integration/
├── metadata.rb          # Cookbook metadata
└── Berksfile            # Cookbook dependencies
```

### Basic Recipe Example
```ruby
# cookbooks/my_cookbook/recipes/default.rb
# Install Apache web server
package 'apache2' do
  action :install
end

# Start Apache service
service 'apache2' do
  action [:enable, :start]
end

# Create website directory
directory '/var/www/my_website' do
  owner 'www-data'
  group 'www-data'
  mode '0755'
  recursive true
end

# Create index.html file
template '/var/www/my_website/index.html' do
  source 'index.html.erb'
  owner 'www-data'
  group 'www-data'
  mode '0644'
  variables({
    :message => 'Hello from Chef!'
  })
end

# Create virtual host configuration
template '/etc/apache2/sites-available/my_website.conf' do
  source 'my_website.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  notifies :reload, 'service[apache2]'
end

# Enable the site
execute 'enable_site' do
  command 'a2ensite my_website.conf'
  notifies :reload, 'service[apache2]'
  not_if { File.exist?('/etc/apache2/sites-enabled/my_website.conf') }
end
```

### Template Example
```erb
<!-- cookbooks/my_cookbook/templates/default/index.html.erb -->
<!DOCTYPE html>
<html>
<head>
    <title>My Website</title>
</head>
<body>
    <h1><%= @message %></h1>
    <p>This page is served by Chef on <%= node['hostname'] %></p>
    <p>IP Address: <%= node['ipaddress'] %></p>
    <p>Platform: <%= node['platform'] %> <%= node['platform_version'] %></p>
</body>
</html>
```

```erb
<!-- cookbooks/my_cookbook/templates/default/my_website.conf.erb -->
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/my_website
    ServerName my_website.local
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

## Chef Resources

### Built-in Resources

#### Package Resource
```ruby
# Install package
package 'nginx' do
  action :install
end

# Remove package
package 'old-software' do
  action :remove
end

# Install specific version
package 'nginx' do
  version '1.18.0-0ubuntu1'
  action :install
end
```

#### Service Resource
```ruby
# Start and enable service
service 'nginx' do
  action [:enable, :start]
end

# Stop service
service 'nginx' do
  action :stop
end

# Restart service
service 'nginx' do
  action :restart
end

# Reload service
service 'nginx' do
  action :reload
end
```

#### File Resource
```ruby
# Create file
file '/etc/motd' do
  content 'Welcome to this server managed by Chef!'
  owner 'root'
  group 'root'
  mode '0644'
end

# Create directory
directory '/opt/myapp' do
  owner 'appuser'
  group 'appuser'
  mode '0755'
  recursive true
end

# Create symbolic link
link '/usr/local/bin/myapp' do
  to '/opt/myapp/bin/myapp'
  owner 'root'
  group 'root'
end
```

#### Template Resource
```ruby
# Create file from template
template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  owner 'root'
  group 'root'
  mode '0644'
  variables({
    :worker_processes => node['cpu']['total'],
    :worker_connections => 1024
  })
end
```

#### Execute Resource
```ruby
# Execute command
execute 'update-packages' do
  command 'apt-get update'
  action :run
end

# Execute command with conditions
execute 'install-custom-package' do
  command 'dpkg -i /tmp/custom-package.deb'
  not_if { File.exist?('/usr/bin/custom-binary') }
end
```

#### User Resource
```ruby
# Create user
user 'appuser' do
  comment 'Application User'
  home '/home/appuser'
  shell '/bin/bash'
  password '$1$xyz$abc123' # hashed password
  action :create
end

# Create group
group 'appgroup' do
  members ['appuser']
  action :create
end
```

### Custom Resources

#### Define Custom Resource
```ruby
# cookbooks/my_cookbook/resources/app_deploy.rb
unified_mode true

property :app_name, String, name_property: true
property :app_user, String, default: 'appuser'
property :app_group, String, default: 'appgroup'
property :app_dir, String, default: '/opt/apps'
property :source_url, String
property :version, String, default: 'latest'

action :deploy do
  # Create application directory
  directory "#{new_resource.app_dir}/#{new_resource.app_name}" do
    owner new_resource.app_user
    group new_resource.app_group
    mode '0755'
    recursive true
  end

  # Download application
  remote_file "#{new_resource.app_dir}/#{new_resource.app_name}/app-#{new_resource.version}.tar.gz" do
    source new_resource.source_url
    owner new_resource.app_user
    group new_resource.app_group
    mode '0644'
  end

  # Extract application
  execute 'extract_app' do
    command "tar -xzf app-#{new_resource.version}.tar.gz"
    cwd "#{new_resource.app_dir}/#{new_resource.app_name}"
    user new_resource.app_user
    group new_resource.app_group
  end
end

action :remove do
  directory "#{new_resource.app_dir}/#{new_resource.app_name}" do
    action :delete
    recursive true
  end
end
```

#### Use Custom Resource
```ruby
# cookbooks/my_cookbook/recipes/default.rb
app_deploy 'my_web_app' do
  app_user 'webuser'
  app_group 'webgroup'
  source_url 'https://example.com/apps/my_web_app.tar.gz'
  version '1.0.0'
  action :deploy
end
```

## Chef Attributes

### Attribute Types
- **Default**: Default attribute values
- **Normal**: Normal attribute values
- **Override**: Override attribute values
- **Automatic**: Automatic attributes from Ohai

### Attribute Files
```ruby
# cookbooks/my_cookbook/attributes/default.rb
# Default attributes
default['my_cookbook']['app_name'] = 'my_app'
default['my_cookbook']['app_user'] = 'appuser'
default['my_cookbook']['app_group'] = 'appgroup'
default['my_cookbook']['app_dir'] = '/opt/apps'
default['my_cookbook']['config_file'] = '/etc/my_app/config.yml'
default['my_cookbook']['log_file'] = '/var/log/my_app.log'
default['my_cookbook']['service_name'] = 'my_app'

# Environment-specific attributes
if node['environment'] == 'production'
  default['my_cookbook']['debug_mode'] = false
  default['my_cookbook']['log_level'] = 'INFO'
else
  default['my_cookbook']['debug_mode'] = true
  default['my_cookbook']['log_level'] = 'DEBUG'
end
```

### Using Attributes in Recipes
```ruby
# cookbooks/my_cookbook/recipes/default.rb
# Use attributes
app_name = node['my_cookbook']['app_name']
app_user = node['my_cookbook']['app_user']
app_dir = node['my_cookbook']['app_dir']
config_file = node['my_cookbook']['config_file']

# Create application directory
directory app_dir do
  owner app_user
  group node['my_cookbook']['app_group']
  mode '0755'
  recursive true
end

# Create configuration file
template config_file do
  source 'config.yml.erb'
  owner app_user
  group node['my_cookbook']['app_group']
  mode '0644'
  variables({
    :app_name => app_name,
    :debug_mode => node['my_cookbook']['debug_mode'],
    :log_level => node['my_cookbook']['log_level']
  })
end
```

## Chef Roles

### Role Definition
```ruby
# roles/web_server.rb
name 'web_server'
description 'Web Server Role'
run_list 'recipe[apache]', 'recipe[php]', 'recipe[my_cookbook]'

default_attributes(
  'apache' => {
    'version' => '2.4',
    'modules' => ['rewrite', 'php']
  },
  'php' => {
    'version' => '7.4'
  },
  'my_cookbook' => {
    'app_name' => 'web_app',
    'debug_mode' => false
  }
)

override_attributes(
  'apache' => {
    'listen_ports' => ['80', '443']
  }
)
```

### Role in JSON Format
```json
{
  "name": "web_server",
  "description": "Web Server Role",
  "json_class": "Chef::Role",
  "default_attributes": {
    "apache": {
      "version": "2.4",
      "modules": ["rewrite", "php"]
    },
    "php": {
      "version": "7.4"
    },
    "my_cookbook": {
      "app_name": "web_app",
      "debug_mode": false
    }
  },
  "override_attributes": {
    "apache": {
      "listen_ports": ["80", "443"]
    }
  },
  "run_list": [
    "recipe[apache]",
    "recipe[php]",
    "recipe[my_cookbook]"
  ],
  "chef_type": "role"
}
```

## Chef Environments

### Environment Definition
```ruby
# environments/production.rb
name 'production'
description 'Production Environment'

default_attributes(
  'my_cookbook' => {
    'debug_mode' => false,
    'log_level' => 'INFO',
    'database_host' => 'prod-db.example.com',
    'cache_enabled' => true
  }
)

override_attributes(
  'my_cookbook' => {
    'max_connections' => 1000,
    'timeout' => 30
  }
)
```

```ruby
# environments/development.rb
name 'development'
description 'Development Environment'

default_attributes(
  'my_cookbook' => {
    'debug_mode' => true,
    'log_level' => 'DEBUG',
    'database_host' => 'dev-db.example.com',
    'cache_enabled' => false
  }
)
```

## Chef Data Bags

### Data Bag Creation
```bash
# Create data bag
knife data bag create users

# Create data bag item
knife data bag create users admin
```

### Data Bag Item Example
```json
{
  "id": "admin",
  "username": "admin",
  "password": "encrypted_password_here",
  "groups": ["admin", "sudo"],
  "shell": "/bin/bash",
  "ssh_keys": [
    "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ... admin@workstation"
  ]
}
```

### Using Data Bags in Recipes
```ruby
# Load data bag item
admin_user = data_bag_item('users', 'admin')

# Create user
user admin_user['username'] do
  comment 'Administrator'
  home "/home/#{admin_user['username']}"
  shell admin_user['shell']
  password admin_user['password']
  action :create
end

# Add user to groups
admin_user['groups'].each do |group|
  group group do
    action :modify
    members admin_user['username']
    append true
  end
end

# Add SSH key
directory "/home/#{admin_user['username']}/.ssh" do
  owner admin_user['username']
  group admin_user['username']
  mode '0700'
end

file "/home/#{admin_user['username']}/.ssh/authorized_keys" do
  content admin_user['ssh_keys'].join("\n")
  owner admin_user['username']
  group admin_user['username']
  mode '0600'
end
```

## Chef Testing

### Test Kitchen

#### Kitchen Configuration
```yaml
# .kitchen.yml
---
driver:
  name: vagrant

provisioner:
  name: chef_zero

platforms:
  - name: ubuntu-20.04
  - name: centos-7

suites:
  - name: default
    run_list:
      - recipe[my_cookbook::default]
    attributes:
      my_cookbook:
        app_name: 'test_app'
        debug_mode: true
```

#### Kitchen Commands
```bash
# List test instances
kitchen list

# Create test instances
kitchen create

# Converge test instances
kitchen converge

# Run tests
kitchen test

# Login to test instance
kitchen login default-ubuntu-2004

# Destroy test instances
kitchen destroy
```

### ChefSpec

#### Spec File Example
```ruby
# spec/recipes/default_spec.rb
require 'spec_helper'

describe 'my_cookbook::default' do
  let(:chef_run) do
    ChefSpec::SoloRunner.new(platform: 'ubuntu', version: '20.04') do |node|
      node.normal['my_cookbook']['app_name'] = 'test_app'
      node.normal['my_cookbook']['app_user'] = 'testuser'
    end.converge(described_recipe)
  end

  it 'installs the apache2 package' do
    expect(chef_run).to install_package('apache2')
  end

  it 'enables the apache2 service' do
    expect(chef_run).to enable_service('apache2')
  end

  it 'starts the apache2 service' do
    expect(chef_run).to start_service('apache2')
  end

  it 'creates the website directory' do
    expect(chef_run).to create_directory('/var/www/test_app')
  end

  it 'creates the index.html file' do
    expect(chef_run).to create_template('/var/www/test_app/index.html')
  end
end
```

## Chef Commands

### Knife Commands
```bash
# Bootstrap a node
knife bootstrap 192.168.1.100 --ssh-user user --ssh-password password --sudo --use-sudo-password --node-name node1 --environment production

# List nodes
knife node list

# Show node details
knife node show node1

# Edit node attributes
knife node edit node1

# Delete node
knife node delete node1

# List cookbooks
knife cookbook list

# Upload cookbook
knife cookbook upload my_cookbook

# Download cookbook
knife cookbook download my_cookbook

# List roles
knife role list

# Create role
knife role create web_server

# Edit role
knife role edit web_server

# List environments
knife environment list

# Create environment
knife environment create production

# List data bags
knife data bag list

# Create data bag
knife data bag create users

# Show data bag item
knife data bag show users admin

# Create data bag item
knife data bag create users admin
```

### Chef Client Commands
```bash
# Run chef-client
sudo chef-client

# Run chef-client with specific log level
sudo chef-client -l debug

# Run chef-client with specific environment
sudo chef-client -E production

# Run chef-client with specific run list
sudo chef-client -o "recipe[apache],recipe[my_cookbook]"

# Run chef-client with why-run mode
sudo chef-client -W

# Run chef-client with specific json attributes
sudo chef-client -j attributes.json
```

## Chef Supermarket

### Using Supermarket Cookbooks
```ruby
# Berksfile
source 'https://supermarket.chef.io'

cookbook 'apache2', '~> 8.0'
cookbook 'php', '~> 7.0'
cookbook 'mysql', '~> 8.0'
cookbook 'postgresql', '~> 7.0'
```

### Install Cookbooks
```bash
# Install cookbook dependencies
berks install

# Upload cookbooks to chef server
berks upload

# Vendor cookbooks
berks vendor cookbooks/
```

## Interview Questions

### Beginner Level
1. **What is Chef?**
   - Chef is a configuration management and automation platform that transforms infrastructure into code

2. **What are the main components of Chef?**
   - Chef Workstation, Chef Server, Chef Infra Client, and various tools like Knife and Test Kitchen

3. **What is a Chef cookbook?**
   - A cookbook is a collection of recipes, attributes, templates, and other resources that configure a piece of software

4. **What is a Chef recipe?**
   - A recipe is a file containing a series of resources that define a system's configuration

### Intermediate Level
1. **What is the difference between Chef and Puppet?**
   - Chef uses Ruby-based DSL, while Puppet uses its own declarative language. Chef is more procedural, while Puppet is more declarative

2. **What are Chef resources?**
   - Resources are statements of configuration policy that describe the desired state of a system component

3. **What is idempotency in Chef?**
   - Idempotency means that running a recipe multiple times will result in the same final state

4. **What is Ohai in Chef?**
   - Ohai is a tool that discovers information about the system and provides it to Chef for use in cookbooks

### Advanced Level
1. **How does Chef handle dependencies?**
   - Chef uses Berkshelf to manage cookbook dependencies and ensure proper versioning

2. **What is the purpose of Test Kitchen?**
   - Test Kitchen is a testing framework that allows you to test cookbooks across multiple platforms

3. **How do you handle secrets in Chef?**
   - Use encrypted data bags, Chef Vault, or integration with secret management tools like HashiCorp Vault

4. **What is the difference between roles and environments in Chef?**
   - Roles define the function of a node, while environments define different deployment stages like development, testing, and production

## Best Practices

### Cookbook Development
- **Modular Design**: Create small, focused cookbooks
- **Use Templates**: Use templates for configuration files
- **Attribute Management**: Use attributes for configuration values
- **Error Handling**: Implement proper error handling
- **Testing**: Test cookbooks thoroughly

### Configuration Management
- **Version Control**: Store all Chef code in version control
- **Documentation**: Document all cookbooks and recipes
- **Naming Conventions**: Use consistent naming conventions
- **Security**: Use proper security practices for credentials
- **Monitoring**: Monitor Chef runs and system state

### Deployment
- **Gradual Rollout**: Roll out changes gradually
- **Backup**: Backup configurations before deployment
- **Testing**: Test in staging environment first
- **Rollback**: Have rollback procedures ready
- **Communication**: Communicate changes to stakeholders

## Troubleshooting

### Common Issues
```bash
# Check chef-client version
chef-client -v

# Run chef-client with debug output
chef-client -l debug

# Check chef-client logs
tail -f /var/log/chef/client.log

# Validate cookbook syntax
knife cookbook test my_cookbook

# Check knife configuration
knife configure -i

# Test connectivity to chef server
knife ssl check
```

### Debugging Tips
- **Check Logs**: Always check Chef logs for error messages
- **Validate Syntax**: Use knife to validate cookbook syntax
- **Test Locally**: Test cookbooks with Test Kitchen
- **Check Dependencies**: Verify all dependencies are available
- **Use Why-Run**: Use why-run mode to see what would change

## Resources

### Official Documentation
- [Chef Documentation](https://docs.chef.io/)
- [Chef Workstation Documentation](https://docs.chef.io/workstation/)
- [Chef Infra Client Documentation](https://docs.chef.io/infra_client/)

### Learning Resources
- [Learn Chef](https://learn.chef.io/)
- [Chef Training](https://training.chef.io/)
- [Chef Blog](https://blog.chef.io/)

### Community
- [Chef Community](https://community.chef.io/)
- [Chef GitHub](https://github.com/chef)
- [Chef Stack Overflow](https://stackoverflow.com/questions/tagged/chef)