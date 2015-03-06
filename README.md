Vagrant MultiVM with Tomcat and Java Puppet modules
========================

Installation
-----------------

Clone this repository in a directory 

```shell
	git clone https://github.com/oscerd/vagrant-multiVM-tomcat vagrant-multiVM-tomcat
```

Settings
-----------------

This machine contains:

 * The following box: https://atlas.hashicorp.com/alphainternational/boxes/centos-6.5-x64
 * The following box: https://atlas.hashicorp.com/cargomedia/boxes/debian-7-amd64-default
 * The puppet Tomcat module: https://github.com/oscerd/puppet-tomcat-module ver 1.0.6
 * The puppet Java module: https://github.com/oscerd/puppet-java-module ver 1.0.1

In the `/modules/tomcat/files` put the following files:

 * Apache Tomcat 7.0.55 (or other version) package
 * Sample War file from https://tomcat.apache.org/tomcat-7.0-doc/appdev/sample/

In the `/modules/java/files` put the following file:

 * A jdk-7u65-linux-x64.tar.gz Oracle JDK (or other version)

Manifests
-----------------

In this environment we have two different file in the `/manifests/`, one for every server we need to start.
After settings let's take a look to `/manifests/server1.pp`:

```puppet

	$vm_env = "server1"

	Exec {
	  path => ["/bin/", "/sbin/", "/usr/bin/", "/usr/sbin/"] }

	  package { 'tar':
	      ensure => installed
	  }

	  package { 'unzip':
	      ensure => installed
	  }



	java::setup { "java":
	  type => "jdk",
	  family => "7",
	  update_version => "65",
	  architecture => "x64",
	  os => "linux",
	  extension => ".tar.gz",
	  tmpdir => "",
	  alternatives => "yes",
	  export => "yes"
	  }

	tomcat::setup { "tomcat":
	  family => "7",
	  update_version => "55",
	  extension => ".zip",
	  source_mode => "local",
	  installdir => "/opt/",
	  tmpdir => "/tmp/",
	  install_mode => "custom",
	  data_source => "no",
	  driver_db => "no",
	  ssl => "no",
	  users => "yes",
	  access_log => "yes",
	  as_service => "yes",
	  direct_start => "yes"
	  }

	tomcat::deploy { "deploy":
	  war_name => "sample",
	  war_versioned => "no",
	  war_version => "",
	  deploy_path => "/release/",
	  context => "/sample",
	  symbolic_link => "",
	  external_conf => "no",
	  external_dir => "",
	  external_conf_path => "",
	  family => "7",
	  update_version => "55",
	  installdir => "/opt/",
	  tmpdir => "/tmp/",
	  hot_deploy => "yes",
	  as_service => "yes",
	  direct_restart => "no",
	  require => Tomcat::Setup["tomcat"]
	  }

```

and now to `/manifests/server2.pp`:

```puppet

	$vm_env = "server2"

	Exec {
	  path => ["/bin/", "/sbin/", "/usr/bin/", "/usr/sbin/"] }

	  package { 'tar':
	      ensure => installed
	  }

	  package { 'unzip':
	      ensure => installed
	  }



	java::setup { "java":
	  type => "jdk",
	  family => "7",
	  update_version => "65",
	  architecture => "x64",
	  os => "linux",
	  extension => ".tar.gz",
	  tmpdir => "",
	  alternatives => "yes",
	  export => "yes"
	  }

	tomcat::setup { "tomcat":
	  family => "7",
	  update_version => "55",
	  extension => ".zip",
	  source_mode => "local",
	  installdir => "/opt/",
	  tmpdir => "/tmp/",
	  install_mode => "custom",
	  data_source => "no",
	  driver_db => "no",
	  ssl => "no",
	  users => "yes",
	  access_log => "yes",
	  as_service => "yes",
	  direct_start => "yes"
	  }

	tomcat::deploy { "deploy":
	  war_name => "sample",
	  war_versioned => "no",
	  war_version => "",
	  deploy_path => "/release/",
	  context => "/context",
	  symbolic_link => "",
	  external_conf => "no",
	  external_dir => "",
	  external_conf_path => "",
	  family => "7",
	  update_version => "55",
	  installdir => "/opt/",
	  tmpdir => "/tmp/",
	  hot_deploy => "yes",
	  as_service => "yes",
	  direct_restart => "no",
	  require => Tomcat::Setup["tomcat"]
	  }

```

Customize the parameters with correct values if you choose different version of Oracle JDK and/or Apache Tomcat for the two VM.

Customization
-----------------

In _hiera.yaml_ you'll see

```yaml
	---
	:backends:
	  - yaml

	:hierarchy:
	  - "%{::vm_env}.configuration"
	  - "%{::vm_env}.data_source"
	  - "%{::vm_env}.users"

	:yaml:
	  :datadir: '/vagrant/hiera'
```

in `/hiera/server1.data_source.yaml` there are the parameters to customize data source:

```yaml
	---
	tomcat::data_source::ds_resource_name: jdbc/ExampleDB
	tomcat::data_source::ds_max_active: 100
	tomcat::data_source::ds_max_idle: 20
	tomcat::data_source::ds_max_wait: 10000
	tomcat::data_source::ds_username: username
	tomcat::data_source::ds_password: password
	tomcat::data_source::ds_driver_class_name: oracle.jdbc.OracleDriver
	tomcat::data_source::ds_driver: jdbc
	tomcat::data_source::ds_dbms: oracle
	tomcat::data_source::ds_host: 192.168.52.128
	tomcat::data_source::ds_port: 1521
	tomcat::data_source::ds_service: example
	tomcat::data_source::ds_drivername: ojdbc6.jar
```

in `/hiera/server1.configuration.yaml` there are the parameters to customize port and web repository:

```yaml
	---
	tomcat::params::http_port: 8082
	tomcat::params::https_port: 8083
	tomcat::params::ajp_port: 8007
	tomcat::params::shutdown_port: 8001
	tomcat::params::http_connection_timeout: 20000
	tomcat::params::https_max_threads: 150
	tomcat::params::https_keystore: keystore
	tomcat::params::https_keystore_pwd: password
	tomcat::params::web_repository: http://apache.fastbull.org/tomcat/

```

in `/hiera/server1.users.yaml` there are the parameters to customize tomcat users:

```yaml
	---
	tomcat::roles::list:
	      - manager-gui
	      - manager-script
	      - manager-jmx
	      - manager-status
	      - admin-gui
	      - admin-script

	tomcat::users::list:
	      - username: pippo
		    password: paperino
	      - username: topolino
		    password: minnie

	tomcat::users::map:
	      - username: pippo
		    roles: manager-gui,admin-gui
	      - username: topolino
		    roles: manager-gui
```

in `/hiera/server2.data_source.yaml` there are the parameters to customize data source:

```yaml
	---
	tomcat::data_source::ds_resource_name: jdbc/ExampleDB
	tomcat::data_source::ds_max_active: 100
	tomcat::data_source::ds_max_idle: 20
	tomcat::data_source::ds_max_wait: 10000
	tomcat::data_source::ds_username: username
	tomcat::data_source::ds_password: password
	tomcat::data_source::ds_driver_class_name: oracle.jdbc.OracleDriver
	tomcat::data_source::ds_driver: jdbc
	tomcat::data_source::ds_dbms: oracle
	tomcat::data_source::ds_host: 192.168.52.128
	tomcat::data_source::ds_port: 1521
	tomcat::data_source::ds_service: example
	tomcat::data_source::ds_drivername: ojdbc6.jar
```

in `/hiera/server2.configuration.yaml` there are the parameters to customize port and web repository:

```yaml
	---
	tomcat::params::http_port: 8082
	tomcat::params::https_port: 8083
	tomcat::params::ajp_port: 8007
	tomcat::params::shutdown_port: 8001
	tomcat::params::http_connection_timeout: 20000
	tomcat::params::https_max_threads: 150
	tomcat::params::https_keystore: keystore
	tomcat::params::https_keystore_pwd: password
	tomcat::params::web_repository: http://apache.fastbull.org/tomcat/

```

in `/hiera/server2.users.yaml` there are the parameters to customize tomcat users:

```yaml
	---
	tomcat::roles::list:
	      - manager-gui
	      - manager-script
	      - manager-jmx
	      - manager-status
	      - admin-gui
	      - admin-script

	tomcat::users::list:
	      - username: pippo
		    password: paperino
	      - username: topolino
		    password: minnie

	tomcat::users::map:
	      - username: pippo
		    roles: manager-gui,admin-gui
	      - username: topolino
		    roles: manager-gui
```


In both the manifest files the Tomcat installation is set to _custom_ and we need to manage the port forwarding properties in Vagrantfile:

```shell
	config.vm.define :server1 do |node_conf|
	  vm_name= "server1"
	  node_conf.vm.host_name = "#{vm_name}.farm"
	  node_conf.vm.box = "alphainternational/centos-6.5-x64"

	  node_conf.vm.network "forwarded_port", guest: 8082, host: 9902

	...
	...
	...

	config.vm.define :server2 do |node_conf|
	  vm_name= "server2"
	  node_conf.vm.host_name = "#{vm_name}.farm"
	  node_conf.vm.box = "cargomedia/debian-7-amd64-default"

	  node_conf.vm.network "forwarded_port", guest: 8082, host: 9903

	...
	...
	...
```

For more information about customization of Tomcat module see: __https://github.com/oscerd/puppet-tomcat-module__

Usage
-----------------

Now you're ready to do:

```shell
	vagrant up
```

After starting phase vagrant will start provisioning and Puppet will install Java and Tomcat on both the VM. In the server1 the webapp will be visible at __http://localhost:9902/sample/__, while on the server2 will be visible at __http://localhost:9903/context/__

Remember
----------------- 

When Vagrant Machines will be up, from the terminal of the CentOS guest machine 

```shell
	sudo service iptables stop
```

or add in `/manifests/server1.pp` the following code fragment:

```puppet
	class iptables {
	  service {'iptables':
	    ensure => stopped,
	  }
	}

	include iptables
```

otherwise you will not see anything on __http://localhost:9902/sample/__
