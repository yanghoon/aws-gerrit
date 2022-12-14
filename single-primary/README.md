# Gerrit Single Primary

This set of Templates provide all the components to deploy a single Gerrit primary
in ECS

## Architecture

Three templates are provided in this example:
* `cf-cluster`: define the ECS cluster and the networking stack
* `cf-service`: defined the service stack running Gerrit
* `cf-dns-route`: defined the DNS routing for the service

### Data persistency

* EBS volumes for:
  * Indexes
  * Caches
  * Data
  * Git repositories

### Deployment type

* Latest Gerrit version deployed using the official [Docker image](https://hub.docker.com/r/gerritcodereview/gerrit)
* Application deployed in ECS on a single EC2 instance

### Logging

* All the logs are forwarded to AWS CloudWatch in the LogGroup with the cluster
  stack name. Please refer to the general [logging documentation](../README.md#logging)
  for further information on logging.

### Monitoring

* Standard CloudWatch monitoring metrics for each component
* Application level CloudWatch monitoring can be enabled as described [here](../Configuration.md#cloudwatch-monitoring)
* Prometheus and Grafana stack is not available for this recipe yet. However the work has been done for
the dual-primary recipe and it could be easily adapted (you can find the relevant issue
[here](https://bugs.chromium.org/p/gerrit/issues/detail?id=13092)).



## How to run it

You can find [on GerritForge's YouTube Channel](https://www.youtube.com/watch?v=zr2zCSuclIU) a
step-by-step guide on how to setup you Gerrit Code Review in AWS.

However, keep reading this guide for a more exhaustive explanation.

### 0 - Prerequisites

Follow the steps described in the [Prerequisites](../Prerequisites.md) section.

Additionally, whilst it is possible to do so by using an admin user, consider
limiting access only to what is required by using a dedicated role or user to do
so.

### 1 - Configuration

Please refer to the [configuration docs](../Configuration.md) to understand how to set up the
configuration and what common configuration values are needed.
On top of that, you might set the additional parameters, specific for this recipe.

#### Environment

Configuration values affecting deployment environment and cluster properties

* `SERVICE_STACK_NAME`: Optional. Name of the service stack. `gerrit-service` by default.
* `GERRIT_INSTANCE_ID`: Optional. Identifier for the Gerrit instance. "gerrit-single-primary" by default.
* `GERRIT_VOLUME_ID` : Optional. Id of an extisting EBS volume. If empty, a new volume
for Gerrit data will be created
* `GERRIT_VOLUME_SNAPSHOT_ID` : Optional. Ignored if GERRIT_VOLUME_ID is not empty. Id of
the EBS volume snapshot used to create new EBS volume for Gerrit data.
* `GERRIT_VOLUME_SIZE_IN_GIB`: Optional. The size of the Gerrit data volume, in GiBs. `10` by default.

### 2 - Deploy

* Create the cluster, service and DNS routing stacks:

```
make [AWS_REGION=a-valid-aws-region] [AWS_PREFIX=some-cluster-prefix] create-all
```

The optional `AWS_REGION` and `AWS_REFIX` allow you to define where it will be deployed and what it will be named.

It might take several minutes to build the stack.
You can monitor the creations of the stacks in [CloudFormation](https://console.aws.amazon.com/cloudformation/home)

* *NOTE*: the creation of the cluster needs an EC2 key pair are useful when you need to connect
to the EC2 instances for troubleshooting purposes. The key pair is automatically generated
and stored in a `pem` file on the current directory.
To use when ssh-ing into your instances as follow: `ssh -i cluster-keys.pem ec2-user@<ec2_instance_ip>`

### Cleaning up

```
make [AWS_REGION=a-valid-aws-region] [AWS_PREFIX=some-cluster-prefix] delete-all
```

The optional `AWS_REGION` and `AWS_REFIX` allow you to specify exactly which stack you target for deletion.

Note that this will *not* delete:
* Secrets stored in Secret Manager
* SSL certificates
* ECR repositories

### Access your Gerrit

The Gerrit instance will be available at two different URLs, to handle HTTP and
SSH traffic respectively:

* HTTP traffic: `https://<HTTP_SUBDOMAIN>.<HOSTED_ZONE_NAME>:443`
* SSH traffic: `ssh://<SSH_SUBDOMAIN>.<HOSTED_ZONE_NAME>:29418`

### External Services

If you need to setup some external services (maybe for testing purposes, such as SMTP or LDAP),
you can follow the instructions [here](../README.md#external-services)

### Docker

Refer to the [Docker](../Docker.md) section for information on how to setup docker or how to publish images

### Permissions

In order to deploy and destroy a single-primary recipe the invoking user needs to
have the relevant permissions to perform actions on AWS resources.

The list of actions can be found [here](resources/permission.policy.json).
The document can be used to create a permission policy directly in AWS.

The policy then needs to be attached to the invoking user group or alternatively
to the invoking user directly.