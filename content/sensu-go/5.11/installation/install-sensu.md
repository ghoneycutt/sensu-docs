---
title: "Installing Sensu"
linkTitle: "Install Sensu"
description: "Sensu Go is available for Linux, Windows (agent and CLI only), macOS (CLI only), and Docker. Read the installation guide to installed the Sensu backend, agent, and sensuctl CLI tool."
weight: 1
version: "5.11"
product: "Sensu Go"
platformContent: false
platforms: ["Ubuntu/Debian", "RHEL/CentOS", "Windows", "macOS", "Docker"]
menu:
  sensu-go-5.11:
    parent: installation
---

## Configuration Management

## Install Sensu

{{< language-toggle >}}

{{< highlight Docker >}}
# All Sensu images contain a Sensu backend and a Sensu agent

# Pull the Alpine-based image
docker pull sensu/sensu

# Pull the RHEL-based image
docker pull sensu/sensu-rhel
{{< /highlight >}}

{{< highlight "Ubuntu/Debian" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash

# Install the sensu-go-backend package
sudo apt-get install sensu-go-backend
{{< /highlight >}}

{{< highlight "RHEL/CentOS" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash

# Install the sensu-go-backend package
sudo yum install sensu-go-backend
{{< /highlight >}}

{{< /language-toggle >}}

Start:

{{< language-toggle >}}

{{< highlight Docker >}}
docker run -v /var/lib/sensu:/var/lib/sensu -d --name sensu-backend \
-p 2380:2380 -p 3000:3000 -p 8080:8080 -p 8081:8081 sensu/sensu:latest \
sensu-backend start --state-dir /var/lib/sensu/sensu-backend --log-level debug
{{< /highlight >}}

{{< highlight "Docker Compose" >}}
---
version: "3"
services:
  sensu-backend:
    image: sensu/sensu:latest
    ports:
    - 3000:3000
    - 8080:8080
    - 8081:8081
    volumes:
    - "sensu-backend-data:/var/lib/sensu/etcd"
    command: "sensu-backend start --state-dir /var/lib/sensu/sensu-backend --log-level debug"

volumes:
  sensu-backend-data:
    driver: local
{{< /highlight >}}

{{< highlight "/etc/sensu/backend.yml" >}}
# Copy the config template from the docs
wget...

# Start sensu-backend using a service manager
sudo service...
{{< /highlight >}}

{{< /language-toggle >}}

## sensuctl



{{< language-toggle >}}

{{< highlight "Ubuntu/Debian" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash

# Install the sensu-go-cli package
sudo apt-get install sensu-go-cli
{{< /highlight >}}

{{< highlight "RHEL/CentOS" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash

# Install the sensu-go-cli package
sudo yum install sensu-go-cli
{{< /highlight >}}

{{< highlight Windows >}}
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.10.2/sensu-enterprise-go_5.10.2_windows_amd64.tar.gz  -OutFile C:\Users\Administrator\sensu-enterprise-go_5.10.2_windows_amd64.tar.gz
{{< /highlight >}}

{{< highlight macOS >}}
# Downlod the latest release
curl -LO https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.10.2/sensu-enterprise-go_5.10.2_darwin_amd64.tar.gz

# Extract the archive.
tar -xvf sensu-enterprise-go_5.10.2_darwin_amd64.tar.gz

#Copy the executable into your PATH.
sudo cp sensuctl /usr/local/bin/
{{< /highlight >}}

{{< /language-toggle >}}





















If you’re trying out Sensu for the first time, we recommend setting up a local sandbox environment using [Docker Compose]() or [Vagrant and VirtualBox]().
If you’re deploying Sensu to your infrastructure, we recommend using one of our supported [packages]() or [Docker images]().

To spin up a Sensu instance, you'll need a Sensu backend


See the list of [supported platforms][5] for more information.
Sensu downloads are provided under the [Sensu License][13].

- [Deploy Sensu with Docker](#deploy-sensu-with-docker)
- [Install Sensu with configuration management tools](#install-sensu-with-configuration-management-tools)
- [Install Sensu packages](#install-sensu-packages)
- [Downloads](#downloads)
- [Build from source](#build-from-source)
- [Install sensuctl on your workstation](#install-sensuctl-on-your-workstation)

<div>
<img src="/images/test.svg">
</div>

<p style="text-align:center"><i>Sensu installation diagram: Standalone configuration</i></p>

1. **Install a Sensu backend**: The Sensu backend is the heart of your Sensu installation. It processes events and schedules service and metric checks. Every Sensu backend includes the Sensu web UI to help you manage your Sensu resources from your browser, the Sensu HTTP API to help your tooling integrate with Sensu, and an embedded etcd datastore to persist configuration and recent event data. The Sensu backend is available for Linux only.
2. **Fan out your monitoring with Sensu agents**: Once you have a Sensu backend, you're ready to fan out your monitoring with Sensu agents. Sensu agents include an agent HTTP API, a statsd listener, and they also execute checks. Platforms.
3. **Set up your workstation with sensuctl**: For a user-friendly way to interact with the Sensu backend and manage your Sensu resources, we recommend installing the sensuctl command-line tool on your workstation.

{{< platformBlock "Docker" >}}

## Deploy Sensu with Docker

Docker images include a Sensu backend and a Sensu agent.
To get started with Sensu and Docker, see the [quickstart guide][].

Alpine

RHEL for OpenShift

Sensu Go can be run via [Docker](https://www.docker.com/) or [rkt](https://coreos.com/rkt) using the [sensu/sensu](https://hub.docker.com/r/sensu/sensu/) image. When running Sensu from Docker there are a couple of things to take into consideration.

The backend requires four exposed ports and persistent storage. This example uses a shared filesystem. Sensu Go is backed by a distributed database, and its storage should be provisioned accordingly.  We recommend local storage or something like Throughput Optimized or Provisioned IOPS EBS if local storage is unavailable. The exposed ports are:

- 2380: Sensu storage peer listener (only other Sensu backends need access to this port)
- 3000: Sensu dashboard
- 8080: Sensu API (all users need access to this port)
- 8081: Agent API (all agents need access to this port)

We suggest, but do not require, persistent storage for Sensu backends and Sensu agents. The Sensu agent will cache runtime assets locally for each check, and the Sensu backend will cache runtime assets locally for each handler and mutator. This storage should be unique per sensu-backend/sensu-agent process.

### Start a Sensu backend
{{< highlight shell >}}
docker run -v /var/lib/sensu:/var/lib/sensu -d --name sensu-backend -p 2380:2380 -p 3000:3000 -p 8080:8080 -p 8081:8081 sensu/sensu:latest sensu-backend start
{{< /highlight >}}

### Start a Sensu agent
In this case, we're starting an agent with the webserver and system subscriptions as an example.
This assumes that the Sensu backend is running on another host named sensu.yourdomain.com.
If you are running these locally on the same system, add `--link sensu-backend` to your Docker arguments and change the backend URL to `--backend-url ws://sensu-backend:8081`.

{{< highlight shell >}}
docker run -v /var/lib/sensu:/var/lib/sensu -d --name sensu-agent sensu/sensu:latest sensu-agent start --backend-url ws://sensu.yourdomain.com:8081 --subscriptions webserver,system --cache-dir /var/lib/sensu
{{< /highlight >}}

_NOTE: You can configure the backend and agent log levels by using the `--log-level` flag on either process. Log levels include `panic`, `fatal`, `error`, `warn`, `info`, and `debug`, defaulting to `warn`._

### sensuctl and Docker

It's best to [install and run sensuctl](#install-sensuctl) locally and point it at the exposed API port for your the Sensu backend.
The sensuctl utility stores configuration locally, and you'll likely want to persist it across uses.
While it can be run from the docker container, doing so may be problematic.

{{< platformBlockClose >}}

## Install Sensu with configuration management tools

### Puppet
The [Puppet][1] Sensu module can be found on the [GitHub][2].
Sensu has partnered with [Learn Puppet][7] to enhance the Puppet module with new features and bug fixes.

### Chef
The [Chef][3] cookbook for Sensu can be found on the [GitHub][4]. Interested in more information on Sensu + Chef? Get some helpful resources [here][12].

### Ansible

## Install Sensu packages

{{< platformBlock "Ubuntu/Debian RHEL/CentOS" >}}

### Install the Sensu backend
The Sensu backend is available for Ubuntu/Debian, RHEL/CentOS, and [Docker](#deploy-sensu-with-docker).

#### 1. Install the package

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian" >}}

##### Ubuntu/Debian
Add the Sensu repository.

{{< highlight shell >}}
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash
{{< /highlight >}}

Install the `sensu-go-backend` package.

{{< highlight shell >}}
sudo apt-get install sensu-go-backend
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "RHEL/CentOS" >}}

##### RHEL/CentOS

Add the Sensu repository.

{{< highlight shell >}}
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash
{{< /highlight >}}

Install the `sensu-go-backend` package.

{{< highlight shell >}}
sudo yum install sensu-go-backend
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian RHEL/CentOS" >}}

#### 2. Create the configuration file

Copy the example backend config file to the default config path.

{{< highlight shell >}}
sudo cp /usr/share/doc/sensu-go-backend-5.11.0/backend.yml.example /etc/sensu/backend.yml
{{< /highlight >}}

_NOTE: The Sensu backend can be configured using a `/etc/sensu/backend.yml` configuration file or using `sensu-backend start` configuration flags. For more information, see the [backend reference][6]._

#### 3. Start the service

Start the backend using a service manager.

{{< highlight shell >}}
sudo service sensu-backend start
{{< /highlight >}}

Verify that the backend is running.

{{< highlight shell >}}
service sensu-backend status
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian RHEL/CentOS Windows" >}}

### Install the Sensu agent
The Sensu agent is available for Ubuntu/Debian, RHEL/CentOS, Windows, and [Docker](#deploy-sensu-with-docker).
In addition to packages, [binary-only distributions][20] for Linux are available for `amd64`, `arm64`, `armv5`, `armv6`, `armv7`, and `386` architectures and for Windows `amd64` and `386` architectures.

#### 1. Install the package

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian" >}}

##### Ubuntu/Debian

Add the Sensu repository.

{{< highlight shell >}}
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash
{{< /highlight >}}

Install the `sensu-go-agent` package.

{{< highlight shell >}}
sudo apt-get install sensu-go-agent
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "RHEL/CentOS" >}}
##### RHEL/CentOS

Add the Sensu repository.

{{< highlight shell >}}
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash
{{< /highlight >}}

Install the `sensu-go-agent` package.

{{< highlight shell >}}
sudo yum install sensu-go-agent
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Windows" >}}

##### Windows {#windows-agent}

Download the Sensu agent for Windows [`amd64`](https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-go-agent_5.11.0.4171_en-US.x64.msi) or [`386`](https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-go-agent_5.11.0_2380_en-US.x86.msi) architectures.

{{< highlight text >}}
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-go-agent_5.11.0.4171_en-US.x64.msi  -OutFile "$env:userprofile\sensu-go-agent_5.11.0.4171_en-US.x64.msi"
{{< /highlight >}}

Start the installation wizard.

{{< highlight text >}}
msiexec.exe /i $env:userprofile\sensu-go-agent_5.11.0.4171_en-US.x64.msi
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian RHEL/CentOS Windows" >}}

#### 2. Create the configuration file

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian RHEL/CentOS" >}}

##### Linux

Copy the example agent config file to the default config path.

{{< highlight shell >}}
sudo cp /usr/share/doc/sensu-go-agent-5.11.0/agent.yml.example /etc/sensu/agent.yml
{{< /highlight >}}

_NOTE: The Sensu agent can be configured using a `/etc/sensu/agent.yml` configuration file or using `sensu-agent start` configuration flags. For more information, see the [agent reference][7]._

{{< platformBlockClose >}}

{{< platformBlock "Windows" >}}

##### Windows

Copy the example agent config file from `%ALLUSERSPROFILE%\sensu\config\agent.yml.example` (default: `C:\ProgramData\sensu\config\agent.yml.example`) to `C:\ProgramData\sensu\config\agent.yml`.

{{< highlight text >}}
cp C:\ProgramData\sensu\config\agent.yml.example C:\ProgramData\sensu\config\agent.yml
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian RHEL/CentOS Windows" >}}

#### 3. Start the service

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian RHEL/CentOS" >}}

##### Linux

Start the agent using a service manager.

{{< highlight shell >}}
sudo service sensu-agent start
{{< /highlight >}}

Verify that the agent is running.

{{< highlight shell >}}
service sensu-agent status
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Windows" >}}

##### Windows

Change to the `sensu\sensu-agent\bin` directory where you've installed Sensu.

{{< highlight text >}}
cd 'C:\Program Files\sensu\sensu-agent\bin'
{{< /highlight >}}

Run the `sensu-agent` executable.

{{< highlight text >}}
./sensu-agent.exe
{{< /highlight >}}

Run the following command to install and start the agent.

{{< highlight text >}}
./sensu-agent service install
{{< /highlight >}}

Verify that the agent is running.

{{< highlight text >}}
sc.exe query SensuAgent
{{< /highlight >}}

{{< platformBlockClose >}}

## Downloads

In addition to packages, [binary-only distributions][20] for Linux are available for `amd64`, `arm64`, `armv5`, `armv6`, `armv7`, and `386` architectures.

## Build from source

Sensu instances built from source are available under the MIT license. They don’t include enterprise features or include free-tier usage limits.

[Link to GitHub]

{{< platformBlock "macOS RHEL/CentOS Ubuntu/Debian Windows" >}}

## Install sensuctl on your workstation

Sensu Go can be configured and used with the sensuctl command line utility.
Sensuctl is available for Ubuntu/Debian, RHEL/CentOS, Windows, and macOS.

### 1. Install the package

{{< platformBlockClose >}}

{{< platformBlock "Ubuntu/Debian" >}}

#### Ubuntu/Debian

Add the Sensu repository.

{{< highlight shell >}}
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash
{{< /highlight >}}

Install the `sensu-go-cli` package.

{{< highlight shell >}}
sudo apt-get install sensu-go-cli
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "RHEL/CentOS" >}}

#### RHEL/CentOS

Add the Sensu repository.

{{< highlight shell >}}
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash
{{< /highlight >}}

Install the `sensu-go-cli` package.

{{< highlight shell >}}
sudo yum install sensu-go-cli
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "Windows" >}}

#### Windows

Download [sensuctl for Windows `amd64`](https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_windows_amd64.tar.gz).

{{< highlight text >}}
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_windows_amd64.tar.gz  -OutFile C:\Users\Administrator\sensu-enterprise-go_5.11.0_windows_amd64.tar.gz
{{< /highlight >}}

Or download [sensuctl for Windows `386`](https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_windows_386.tar.gz).

{{< highlight text >}}
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_windows_386.tar.gz  -OutFile C:\Users\Administrator\sensu-enterprise-go_5.11.0_windows_386.tar.gz
{{< /highlight >}}

See the [verifying Sensu guide][12] to verify your download using checksums.

{{< platformBlockClose >}}

{{< platformBlock "macOS" >}}

#### macOS

Download the latest release. See the [verifying Sensu guide][12] to verify your download using checksums.

{{< highlight shell >}}
curl -LO https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_darwin_amd64.tar.gz
{{< /highlight >}}

Extract the archive.

{{< highlight shell >}}
tar -xvf sensu-enterprise-go_5.11.0_darwin_amd64.tar.gz
{{< /highlight >}}

Copy the executable into your PATH.

{{< highlight shell >}}
sudo cp sensuctl /usr/local/bin/
{{< /highlight >}}

{{< platformBlockClose >}}

{{< platformBlock "macOS RHEL/CentOS Ubuntu/Debian Windows" >}}

### 2. Configure sensuctl

You must configure sensuctl before it can connect to Sensu Go.
Run `sensuctl configure` to get started.

{{< highlight shell >}}
$ sensuctl configure
? Sensu Backend URL: http://127.0.0.1:8080
? Username: admin
? Password: *********
? Namespace: default
? Preferred output format: tabular
{{< /highlight >}}

By default, your Sensu installation comes with a user named `admin` with password `P@ssw0rd!`.
We **strongly** recommended that you change the password immediately.
Once authenticated, you can change the password using the `change-password` command.

{{< highlight shell >}}
$ sensuctl user change-password --interactive
? Current Password:  *********
? Password:          *********
? Confirm:           *********
{{< /highlight >}}

You can change individual values of your sensuctl configuration with the `config` subcommand.

{{< highlight shell >}}
sensuctl config set-namespace default
{{< /highlight >}}

See the [sensuctl reference][4] for more information about using sensuctl.

### 3. Activate licensed-tier features

Sensu Inc. offers support packages for Sensu Go as well as license-activated features designed for monitoring at scale.
To learn more about license-activated features in Sensu Go, [contact the Sensu sales team](https://sensu.io/sales).

If you already have a Sensu license, [log in to your Sensu account](https://account.sensu.io/) and download your license file, then activate your license using sensuctl.

{{< highlight shell >}}
sensuctl create --file sensu_license.json
{{< /highlight >}}

You can use sensuctl to view your license details at any time.

{{< highlight shell >}}
sensuctl license info
{{< /highlight >}}

For more information about license-activated features in Sensu Go, see the [getting started guide](../../getting-started/enterprise).

{{< platformBlockClose >}}

[1]: https://github.com/sensu/sensu-go/releases
[2]: https://github.com/sensu/sensu-go/blob/5.1.1/packaging/files/windows/agent.yml.example
[3]: ../../dashboard/overview
[4]: ../../sensuctl/reference
[5]: ../../installation/platforms
[6]: ../../reference/backend
[7]: ../../reference/agent
[8]: ../../guides/troubleshooting
[9]: ../../guides/monitor-external-resources
[10]: ../../guides/send-slack-alerts
[11]: https://github.com/sensu/sensu-go/blob/master/packaging/files/backend.yml.example
[12]: ../verify
[13]: https://sensu.io/sensu-license
[20]: ../verify
