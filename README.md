
# Terraform Provisioner for Habitat
A [Terraform](https://terraform.io) provisioner to install and configure [Habitat](https://habitat.sh)

This is a development repository for adding features to the [provisioner](https://github.com/hashicorp/terraform/tree/master/builtin/provisioners/habitat). Primary goal at the moment is to add support for installation and configuration of Habitat on Windows.

## Build from source
Provisioner is written in Go language and uses packages from main Hashicorp Terraform [repository](https://github.com/hashicorp/terraform). Recommended to have Terraform repository on your local at `~/go/src/github.com/hashicorp/terraform`. 
After successful build, copy the binary to ~/.terraform.d/plugins/ and make it is executable before use. 

```
git clone https://github.com/chef-partners/terraform-provisioner-habitat.git
cd terraform-provisioner-habitat
go build -o terraform-provisioner-habitat_dev
mv ./terraform-provisioner-habitat_dev ~/.terraform.d/plugins/terraform-provisioner-habitat_dev
chmod +x ~/.terraform.d/plugins/terraform-provisioner-habitat_dev
```

## Requirements
* You must have winrm access to Windows machine with admin rights for installation


## Usage
The following example shows how to install SQL Server Habitat service on a Windows machine

```
provisioner "habitat_dev" {
    peer = ""
    
    service {
      name = "core/sqlserver"
      topology = "standalone"
    }
    
    connection {
      type = "winrm"
      timeout = "10m"
      user = "${var.win_username}"
      password = "${var.win_password}"
    }
}
```

To attach this SQL server with an application as Habitat peer and establish service binding, the following can be used in the application server's provisioning

```
provisioner "habitat_dev" {
    peer = "${aws_instance.ms-hab-sqlserver.private_ip}"
    
    service {
      name = "mwrock/contosouniversity"
      topology = "standalone"
      binds = [
        "database:sqlserver.default"
      ]
    }

    connection {
      type = "winrm"
      timeout = "15m"
      user = "${var.win_username}"
      password = "${var.win_password}"
    }
  }
 ```

## Arguments
There are 2 configuration levels, supervisor and service.  Values placed directly within the `provisioner` block are supervisor configs, and values placed inside a `service` block are service configs.  Services can also take a `bind` block to configure runtime bindings.

### Supervisor
* `permanent_peer`: Whether this supervisor should be marked as a permanent peer. Optional (Defaults to false)
* `listen_gossip`: IP and port to listen for gossip traffic.  Optional (Defaults to "0.0.0.0:9638")
* `listen_http`: IP and port for the HTTP API service.  Optional (Defaults to "0.0.0.0:9631")
* `peer`: IP or FQDN of a supervisor instance to peer with.  Optional (Defaults to none)
* `ring_key`: Key for encrypting the supervisor ring traffic.  Optional (Defaults to none)
* `skip_install`: Skips the installation Habitat, if it's being installed another way.  Optional (Defaults to no)
### Service
* `name`: A package identifier of the Habitat package to start (eg `core/nginx`, `core/nginx/1.11.10` or `core/nginx/1.11.10/20170215233218`).  Required.
* `strategy`: Update strategy to use. Possible values "at-once", "rolling" or "none".  Optional (Defaults to "none")
* `topology`: Topology to start service in.  Possible values "standalone" or "leader".  Optional (Defaults to "standalone")
* `channel`: Channel in a remote depot to watch for package updates.  Optional
* `group`: Service group to join.  Optional (Defaults to "default")
* `url`: URL of the remote Depot to watch.  Optional (Defaults to the public depot)
* `binds`:  Array of binding statements (eg "backend:nginx.default").  Optional
* `user_toml`: TOML formatted user configuration for the service.  Easiest to source from a file (eg `user_toml = "${file("conf/redis.toml")}"`).  Optional

### Bind
* `service`: The target service to bind.
* `group`: The target group to bind.
* `alias`: The alias for the binding.

**This format for declaring bindings is optional.  It can be used in place of or along side the `binds = ["alias:service.group"]` method of declaring binds.  This format might be easier to manage when populating one or more of the bind parameters dynamically.

Example:
```
service {
  name = "core/haproxy"
  group = "${var.environment}"

  bind {
    alias = "backend"
    service = "nginx"
    group = "${var.environment}"
  }
}
```
This block will generate the option `--bind backend:nginx.default` when starting the haproxy service.


