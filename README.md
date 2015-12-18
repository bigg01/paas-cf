[![Build Status](https://api.travis-ci.org/alphagov/paas-cf.svg)](https://travis-ci.org/alphagov/paas-cf)

# paas-cf

This repository contains [Concourse](http://concourse.ci/) pipelines and
related [Terraform](https://terraform.io/) and [BOSH](https://bosh.io/) manifests
that allow provisioning of [CloudFoundy](https://www.cloudfoundry.org/) on AWS.

## Getting started

You need the following prerequisites before you will be able to deploy first pipeline.

* Virtualbox
* Vagrant
* AWS Access

Provide AWS access keys as environment variables.
```
export AWS_ACCESS_KEY_ID=XXXXXXXXXX
export AWS_SECRET_ACCESS_KEY=YYYYYYYYYY
```

## Usage

### Prepare the environment

1. Clone repo and boot the vagrant node with concourse:

    ```
git clone https://github.com/alphagov/paas-cf.git
cd paas-cf
vagrant up
```

2. Install your `fly` binary:

    ```
FLY_CMD_URL=http://192.168.100.4:8080/api/v1/cli?arch=amd64&platform=$(uname | tr '[:upper:]' '[:lower:]')
sudo curl $FLY_CMD_URL -o /usr/local/bin/fly && \
sudo chmod +x /usr/local/bin/fly
```

3. Login on the local concourse:

     ```
fly login --concourse-url http://192.168.100.4:8080 sync
```

### Deploy the deployer concourse

```
CONCOURSE_ATC_PASSWORD=atcpassword CONCOURSE_DB_PASSWORD=dbpassword ./concourse/scripts/create-deployer.sh env_name
```

* CONCOURSE_DB_PASSWORD is the RDS database password. It must be 8 characters long minimum
* CONCOURSE_ATC_PASSWORD is the deployed Concourse web interface password

Also, you can specify log level and branch by setting LOG_LEVEL and BRANCH variables.

```
BRANCH=branch_name LOG_LEVEL=DEBUG CONCOURSE_ATC_PASSWORD=atcpassword CONCOURSE_DB_PASSWORD=dbpassword ./concourse/scripts/create-deployer.sh env_name
```

This script will:

- Use a local Vagrant virtual machine with concourse-lite.
- Use Terraform to create a VPC, subnet and security group inside AWS
- bosh-init is used to deploy a full-blown Concourse instance inside AWS

### Login to deployed Concourse

* Point the browser to http://"environment"-concourse.cf.paas.alphagov.co.uk:8080/
* Login with username admin and password as CONCOURSE_ATC_PASSWORD above

### Destroy the deployer concourse

```
./concourse/scripts/destroy-deployer.sh env_name
```

or

```
LOG_LEVEL=DEBUG BRANCH=branch_name ./concourse/scripts/destroy-deployer.sh env_name
```

Will destroy the resources created in the previous run.
