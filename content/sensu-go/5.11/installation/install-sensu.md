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

If you’re trying out Sensu for the first time, we recommend setting up a local sandbox environment using [Docker Compose]() or [Vagrant and VirtualBox]().
If you’re deploying Sensu to your infrastructure, we recommend using one of our supported [packages]() or [Docker images]().

<div>
<img src="/images/install-sensu.svg">
</div>

<p style="text-align:center"><i>Basic Sensu installation</i></p>

Sensu is an agent-based monitoring tool that you install on your organization's infrastructure.
The Sensu agent gives you visibility into everything you care about; the Sensu backend gives you flexible, automated workflows to route metrics and alerts.
You can interact with Sensu from your workstation using the web UI and HTTP API included with the backend or the sensuctl standalone tool.

Sensu Go is available for Linux, Windows (agent and CLI only), macOS (CLI only), and Docker.
See the list of [supported platforms][5] for more information.
Sensu downloads are provided under the [Sensu License][13].

{{< platformBlock "Ubuntu/Debian RHEL/CentOS" >}}

## Install Sensu {#install-the-sensu-backend}
The Sensu backend is available for Ubuntu/Debian, RHEL/CentOS, and [Docker](#deploy-sensu-with-docker).
Backed 

The backend requires four exposed ports, persistent storage, and a local cache.
Sensu Go is backed by a distributed database, and its storage should be provisioned accordingly.  We recommend local storage or something like Throughput Optimized or Provisioned IOPS EBS if local storage is unavailable. The exposed ports are:

- 2380: Sensu storage peer listener (only other Sensu backends need access to this port)
- 3000: Sensu dashboard
- 8080: Sensu API (all users need access to this port)
- 8081: Agent API (all agents need access to this port)

We suggest, but do not require, persistent storage for Sensu backends and Sensu agents using assets. The Sensu agent will cache runtime assets locally for each check, and the Sensu backend will cache runtime assets locally for each handler and mutator. This storage should be unique per sensu-backend/sensu-agent process.

To install Sensu:

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

You can configure Sensu using `sensu-backend start` flags or an /etc/sensu/backend.yml file, the former taking precedence.
At a minimum, the Sensu backend requires the `state-dir` flag, but here are some other useful configs.
For a complete list of config options, see the [backend reference][6].

To configure and start Sensu:

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
    - 2380:2380
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

{{< highlight "/etc/sensu/backend.yml - Linux" >}}
# Copy the config template from the docs
sudo curl https://docs.sensu.io/sensu-go/5.11/files/backend.yml -o /etc/sensu/backend.yml

# Start sensu-backend using a service manager
service sensu-backend start
{{< /highlight >}}

{{< /language-toggle >}}

Now that you've installed the Sensu backend:

- [Install the Sensu agent](#install-the-sensu-agent)
- [Install sensuctl](#install-sensuctl)
- [Sign in to the dashboard][3]

## Install Sensu agents
The Sensu agent is available for Ubuntu/Debian, RHEL/CentOS, Windows, and [Docker](#deploy-sensu-with-docker).
In addition to packages, [binary-only distributions][20] for Linux are available for `amd64`, `arm64`, `armv5`, `armv6`, `armv7`, and `386` architectures and for Windows `amd64` and `386` architectures.

To install the Sensu agent:

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
sudo apt-get install sensu-go-agent
{{< /highlight >}}

{{< highlight "RHEL/CentOS" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash

# Install the sensu-go-backend package
sudo yum install sensu-go-agent
{{< /highlight >}}

{{< highlight "Windows" >}}
# Download the Sensu agent for Windows amd64
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-go-agent_5.11.0.4171_en-US.x64.msi  -OutFile "$env:userprofile\sensu-go-agent_5.11.0.4171_en-US.x64.msi"
# Or for 386
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-go-agent_5.11.0_2380_en-US.x86.msi  -OutFile "$env:userprofile\sensu-go-agent_5.11.0.4171_en-US.x86.msi"

# Start the installation wizard
msiexec.exe /i $env:userprofile\sensu-go-agent_5.11.0.4171_en-US.x64.msi
{{< /highlight >}}

{{< /language-toggle >}}

You can configure the Sensu agent using `sensu-agent start` flags or an /etc/sensu/agent.yml file, the former taking precedence.
At a minimum, the Sensu agent requires the `-backend-url` flag, but here are some other useful configs.
For a complete list of config options, see the [agent reference][].

To configure and start a Sensu agent with the `webserver` subscription:

{{< language-toggle >}}

{{< highlight Docker >}}
# If you are running the agent locally on the same system as the Sensu baclend, add `--link sensu-backend` to your Docker arguments and change the backend URL to `--backend-url ws://sensu-backend:8081`.
docker run -v /var/lib/sensu:/var/lib/sensu -d --name sensu-agent sensu/sensu:latest sensu-agent start --backend-url ws://sensu.yourdomain.com:8081 --subscriptions webserver --cache-dir /var/lib/sensu
{{< /highlight >}}

{{< highlight "Docker Compose" >}}
---
version: "3"
services:
  sensu-agent:
    image: sensu/sensu:latest
    ports:
    - 3031:3031
    volumes:
    - "sensu-agent-data:/var/cache/sensu/sensu-agent"
    command: "sensu-agent start --backend-url ws://sensu-backend:8081 --log-level debug --subscriptions webserver --api-host 0.0.0.0"

volumes:
  sensu-agent-data:
    driver: local
{{< /highlight >}}

{{< highlight "/etc/sensu/agent.yml - Linux" >}}
# Copy the config template from the docs
sudo curl https://docs.sensu.io/sensu-go/5.11/files/agent.yml -o /etc/sensu/agent.yml

# Start sensu-agent using a service manager
service sensu-agent start
{{< /highlight >}}

{{< highlight "/etc/sensu/agent.yml - Windows" >}}
# Copy the example agent config file from `%ALLUSERSPROFILE%\sensu\config\agent.yml.example` (default: `C:\ProgramData\sensu\config\agent.yml.example`) to `C:\ProgramData\sensu\config\agent.yml`.
cp C:\ProgramData\sensu\config\agent.yml.example C:\ProgramData\sensu\config\agent.yml

# Start sensu-backend using a service manager
service sensu-backend start

# Change to the sensu\sensu-agent\bin directory where you've installed Sensu.
cd 'C:\Program Files\sensu\sensu-agent\bin'

# Run the sensu-agent executable.
./sensu-agent.exe

# Install and start the agent.
./sensu-agent service install

{{< /highlight >}}

{{< /language-toggle >}}

Now that you've installed the Sensu agent:

- [Install sensuctl](#install-sensuctl)
- [Create a monitoring event][9]

## Install sensuctl
Sensu Go can be configured and used with the sensuctl command line utility.
Sensuctl is available for Ubuntu/Debian, RHEL/CentOS, Windows, and macOS.

To install sensuctl:

{{< language-toggle >}}

{{< highlight "Ubuntu/Debian" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.deb.sh | sudo bash

# Install the sensu-go-backend package
sudo apt-get install sensu-go-cli
{{< /highlight >}}

{{< highlight "RHEL/CentOS" >}}
# Add the Sensu repository
curl -s https://packagecloud.io/install/repositories/sensu/stable/script.rpm.sh | sudo bash

# Install the sensu-go-backend package
sudo yum install sensu-go-cli
{{< /highlight >}}

{{< highlight "Windows" >}}
# Download sensuctl for Windows amd64
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_windows_amd64.tar.gz  -OutFile C:\Users\Administrator\sensu-enterprise-go_5.11.0_windows_amd64.tar.gz
# Or for 386
Invoke-WebRequest https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_windows_386.tar.gz  -OutFile C:\Users\Administrator\sensu-enterprise-go_5.11.0_windows_386.tar.gz
{{< /highlight >}}

{{< highlight "macOS" >}}
# Download the latest release
curl -LO https://s3-us-west-2.amazonaws.com/sensu.io/sensu-go/5.11.0/sensu-enterprise-go_5.11.0_darwin_amd64.tar.gz

# Extract the archive
tar -xvf sensu-enterprise-go_5.11.0_darwin_amd64.tar.gz

# Copy the executable into your PATH.
sudo cp sensuctl /usr/local/bin/
{{< /highlight >}}

{{< /language-toggle >}}

You must configure sensuctl before it can connect to Sensu Go.
Run `sensuctl configure` to get started:

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

Now that you've installed sensuctl:

- [See the sensuctl quick reference][4]
- [Create a monitoring event pipeline][10]

## Activate licensed-tier features

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
