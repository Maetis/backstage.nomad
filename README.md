# Deploying Backstage with Nomad
This guide helps you to get Backstage up and running with Nomad. It's inspired from [Deploying with Kubernetes](https://backstage.io/docs/deployment/k8s) guide.

**This example is not for production deployment!**

## Before starting
* Install [Nomad](https://www.nomadproject.io/downloads) (version >= 1.4.x)
* Install [Docker](https://docs.docker.com/get-docker/)

## Creating a namespace
Deployments in Nomad are commonly assigned to their own namespace to isolate services in a multi-tenant environment.

This can be done through Nomad CLI directly:
```
nomad namespace apply backstage
```

## Creating the PostgreSQL database
### Creating PostgreSQL secrets
First, create a [Nomad Variable](https://developer.hashicorp.com/nomad/docs/concepts/variables) for the PostgreSQL username and password. This will be 
used by both the PostgreSQL database and Backstage deployments:
```
# spec.nv.hcl
path = "nomad/jobs"
namespace = "backstage"

items {
  postgres_user = "YmFja3N0YWdl"
  postgres_password = "aHVudGVyMg=="
}
```

The secrets can now be applied to Nomad:
```shell
nomad var put @spec.nv.hcl
```

### Creating a PostgreSQL persistent volume
In order to persist PostgreSQL data we will create a [host volume](https://developer.hashicorp.com/nomad/docs/configuration/client#host_volume-stanza),
maybe in a production scenario a [CSI volume](https://developer.hashicorp.com/nomad/docs/commands/volume/register) will be more appropriate.
```
client {
    host_volume "postgres" {
        path      = "/mnt/data"
        read_only = false
    }    
}
```
