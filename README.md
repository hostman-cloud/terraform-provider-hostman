# Hostman Provider

Terraform enables automated management of multiple resources in Hostman using convenient HCL (HashiCorp Configuration Language) configuration files and detailed change plans.

For detailed information about provider resources, refer to the documentation on the Terraform website or its mirror.

## Documentation

Detailed documentation on the parameters of provider resources and data sources

* Resources

    * [hm_server](docs/resources/server.md) - server;
    * [hm_server_disk](docs/resources/server_disk.md) - additional disk for the server;
    * [hm_server_disk_backup](docs/resources/server_disk_backup.md) - backup of the main and additional server disk;
    * [hm_server_disk_backup_schedule](docs/resources/server_disk_backup_schedule.md) - settings for automatically created backups of the main and additional server disk;
    * [hm_project](docs/resources/project.md) - projects;
    * [hm_s3_bucket](docs/resources/s3_bucket.md) - S3 storage;
    * [hm_s3_bucket_subdomain](docs/resources/s3_bucket_subdomain.md) - subdomains for S3 storage;
    * [hm_database_cluster](docs/resources/database_cluster.md) - managed DBMS cluster;
    * [hm_database_instance](docs/resources/database_instance.md) - database in a managed DBMS cluster;
    * [hm_database_user](docs/resources/database_user.md) - user in a managed DBMS cluster;
    * [hm_database_backup](docs/resources/database_backup.md) - managed DBMS backups;
    * [hm_database_backup_schedule](docs/resources/database_backup_schedule.md) - settings for automatically created DBMS backups;
    * [hm_k8s_cluster](docs/resources/k8s_cluster.md) - managed K8S cluster;
    * [hm_k8s_node_group](docs/resources/k8s_node_group.md) - K8S cluster node groups;
    * [hm_firewall](docs/resources/firewall.md) - managed Firewall;
    * [hm_firewall_rule](docs/resources/firewall_rule.md) - managed Firewall rules;
    * [hm_vpc](docs/resources/vpc.md) - managed VPC;
    * [hm_floating_ip](docs/resources/floating_ip.md) - floating IP address;

* Data sources

    * [hm_configurator](docs/data-sources/configurator.md) - configurators for manual selection of required VDS resources;
    * [hm_os](docs/data-sources/os.md) - OS for server installation;
    * [hm_presets](docs/data-sources/presets.md) - server configuration presets;
    * [hm_projects](docs/data-sources/projects.md) - account projects;
    * [hm_software](docs/data-sources/software.md) - software for server installation;
    * [hm_ssh_keys](docs/data-sources/ssh_keys.md) - SSH keys for server installation;
    * [hm_s3_bucket](docs/data-sources/s3_bucket.md) - S3 storages;
    * [hm_s3_preset](docs/data-sources/s3_preset.md) - S3 storage configuration presets;
    * [hm_database_preset](docs/data-sources/database_preset.md) - database configuration presets;
    * [hm_database_cluster](docs/data-sources/database_cluster.md) - managed DBMS clusters;
    * [hm_k8s_preset](docs/data-sources/k8s_preset.md) - K8S cluster configuration presets;
    * [hm_image](docs/data-sources/image.md) - images for servers;
    * [hm_floating_ip](docs/data-sources/floating_ip.md) - floating IP address;

## Quick Start

### Install Terraform

You can install Terraform by following the [official instructions](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli?in=terraform%2Faws-get-started)

### Create a Terraform Configuration File

1. Create a new directory with any name, e.g., `hostman-terraform`. It will store Terraform configuration files and saved infrastructure states;

2. Create a configuration file with a `.tf` extension in the new directory, e.g., `main.tf`.

### Adding the Provider

1. At the beginning of the `main.tf` configuration file, add the following block:

    ```
    terraform {
      required_providers {
        hm = {
          source = "hostman-cloud/hostman"
        }
      }
      required_version = ">= 0.13"
    }
    ```

   > The provider only supports Terraform versions 0.13 and above. Versions below are not supported.

2. Run `terraform init` to download the provider.

### Setting Up the API Token for Provider Authentication

To work with the Terraform provider, you need to obtain an API token in the [relevant section of the Control Panel](https://hostman.com/my/api-keys) to access your account resources.

After creating the token, you can use it in one of the following ways:

Add the created token to environment variables for use in Terraform:

```sh
export HM_TOKEN=eyJhbGc...
```

or

Add the token as a provider parameter:

```terraform
provider "hm" {
  token = "eyJhbGci..."
}
```

> For the Terraform provider to work correctly, the token must have server deletion confirmation via Telegram disabled.

### Preparing the Configuration

Terraform can manage various types of resources in Hostman. Detailed information about each resource and data source can be found in separate sections of this documentation.

As an example, we'll use the configuration below to create a virtual machine named  `Example server` with a 25 GB NVMe disk, 1 CPU core, 1 GB RAM, and Ubuntu 22.04 OS in the `us-2` location.

```terraform
terraform {
  required_providers {
    hm = {
      source = "hostman-cloud/hostman"
    }
  }
  required_version = ">= 0.13"
}

data "hm_configurator" "configurator" {
  location = "us-2"
}

data "hm_os" "os" {
  name = "ubuntu"
  version = "22.04"
}
resource "hm_server" "example-server" {
  name = "Example server"
  os_id = data.hm_os.os.id

  configuration {
    configurator_id = data.hm_configurator.configurator.id
    disk = 1024 * 25
    cpu = 1
    ram = 1024
  }
}
```

> This example requires the HM_TOKEN environment variable to be set.

To access the created VDS, you'll need a password, which will be sent to your email after server creation. As an alternative, you can use SSH access. Learn more in the documentation for the `hm_server` resource and `hm_ssh_keys` data source..


### Validating Configuration Files

Validate the configuration by running:

```sh
terraform validate
```

If the configuration is valid, you'll see:

```
Success! The configuration is valid.
```

> For other `terraform` commands, refer to the [official Terraform documentation](https://developer.hashicorp.com/terraform/cli)

### Applying Configuration Files

1. To generate an execution plan, run:

    ```sh
    terraform plan
    ```

   The terminal will display a list of planned changes. No changes will be applied at this stage.

2. To apply the configuration and create resources, run:

    ```sh
    terraform apply
    ```

3. Resource Creation Confirmation

Before applying changes, Terraform will display the execution plan again and request manual confirmation:

  ```sh
  Do you want to perform these actions?
    Terraform will perform the actions described above.
    Only 'yes' will be accepted to approve.

    Enter a value:
  ```

Type `yes` and press *Enter*.

After applying the configuration, Terraform will display the result, which you can verify in the [Control Panel](https://hostman/my)

### Deleting Created Resources

To delete all resources created via Terraform:

1. Run;
    ```sh
    terraform destroy
    ```
   During execution, Terraform will display a plan for deleting all created resources and request confirmation:

    ```sh
    Do you really want to destroy all resources?
      Terraform will destroy all your managed infrastructure, as shown above.
      There is no undo. Only 'yes' will be accepted to confirm.

      Enter a value:
    ```

2. Type `yes` and press *Enter*