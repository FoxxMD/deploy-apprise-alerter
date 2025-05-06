# Deploy an Apprise API [Alerter](https://komo.do/docs/resources#alerter)

Part of the [*Komodo Hub* collection.](https://github.com/komodo-hub/komodo-hub)

Deploys an [Alerter](https://komo.do/docs/resources#alerter) client for [Apprise](https://github.com/caronc/apprise) using [Apprise API](https://github.com/caronc/apprise-api).

Docker image built from [foxxmd/komodo-utilities](https://github.com/FoxxMD/komodo-utilities).

## Requirements

* An [Apprise API instance](https://github.com/caronc/apprise-api)
* Any of the following...
  * Comma-separated [Stateless URLs](https://github.com/caronc/apprise-api?tab=readme-ov-file#stateless-solution) for each [notification provider](https://github.com/caronc/apprise/wiki#notification-services) to use
    * or `APPRISE_STATELESS_URLS` env defined on the apprise-api instance
  * Comma-separated [Persistent Keys](https://github.com/caronc/apprise-api?tab=readme-ov-file#persistent-stateful-storage-solution) for each notification provider to use

## Deploying

Use **either** of the methods below.


### Komodo Resource Sync

<details>


Create a [Resource Sync](https://komo.do/docs/sync-resources) with the TOML configuration below to

* create the [Stack](https://komo.do/docs/resources#stack) to run deploy-apprise-alerter
* setup the [Alerter](https://komo.do/docs/resources#alerter) + configuration

Steps:

* Open Komodo Dashboard -> Syncs -> **New Resource Sync**
* Choose Mode -> UI Defined
  * Toggle the following to active:
    * Include Sync Resources
    * Include Sync Variables

Add the below configuration to **Resource File** field and then modify variables for your environment (APPRISE_HOST, endpoint.params.url, etc...)

```toml
[[stack]]
name = "apprise-alerter"
[stack.config]
repo = "foxxmd/deploy-apprise-alerter"
file_paths = [
  "compose.yaml",
]
environment = """
  ## Required

  ## Your Apprise API instance
  APPRISE_HOST=192.168.0.101:8000

  ## Optional Apprise Config ##

  #APPRISE_STATELESS_URLS=gotify://192.168.0.150:80/MyAppToken,ntfy://192.168.0.51/MyTopic

  #APPRISE_PERSISTENT_KEYS=1daf495719d8d9ed8df02af7c,7239e1e76add847e5f46c

  #https://github.com/caronc/apprise-api?tab=readme-ov-file#tagging
  #APPRISE_TAG=dev, qa

  ## Optional Alerter Config ##

  ## Set whether to include Komodo Severity Level in notification title
  #LEVEL_IN_TITLE=true

  ## Send message/content with markdown formatting (default false)
  #MARKDOWN=true

  ## Prefixes messages with a checkmark when the Alert is in the 'Resolved' state
  #INDICATE_RESOLVED=true

  ## Filter if an alert is pushed based on its Resolved status
  ## * leave unset to push all alerts
  ## * otherwise, alerts will only be pushed if Alert is one of the comma-separated states set here
  #ALLOW_RESOLVED_TYPE=resolved,unresolved

  ## Delay alerts with below types for X milliseconds 
  ## and cancel pushing alert if it is resolved within that time
  #UNRESOLVED_TIMEOUT_TYPES=ServerCpu,ServerMem
  #UNRESOLVED_TIMEOUT=20000
"""

[[alerter]]
name = "apprise"
[alerter.config]
enabled = true
endpoint.type = "Custom"
endpoint.params.url = "http://apprise-alerter-ip:7000"
```

**Save** the sync and then **Execute Sync** to create the Alerter.

</details>

### Manual setup

<details>

Create a new [**Stack**](https://komo.do/docs/resources#stack) with the following for `compose.yaml` file.

```yaml
services:
  komodo-apprise-alerter:
    image: foxxmd/komodo-apprise-alerter:latest
    restart: unless-stopped
    env_file:
      - ./.env
    ports:
      - "7000:7000"
```

Add the following to the Stack -> Config -> Environment section:

```ini
## Required

## Your Apprise API instance
APPRISE_HOST=192.168.0.101:8000

## Optional Apprise Config ##

#APPRISE_STATELESS_URLS=gotify://192.168.0.150:80/MyAppToken,ntfy://192.168.0.51/MyTopic

#APPRISE_PERSISTENT_KEYS=1daf495719d8d9ed8df02af7c,7239e1e76add847e5f46c

#https://github.com/caronc/apprise-api?tab=readme-ov-file#tagging
#APPRISE_TAG=dev, qa

## Optional Alerter Config ##

## Set whether to include Komodo Severity Level in notification title
#LEVEL_IN_TITLE=true

## Send message/content with markdown formatting (default false)
#MARKDOWN=true

## Prefixes messages with a checkmark when the Alert is in the 'Resolved' state
#INDICATE_RESOLVED=true

## Filter if an alert is pushed based on its Resolved status
## * leave unset to push all alerts
## * otherwise, alerts will only be pushed if Alert is one of the comma-separated states set here
#ALLOW_RESOLVED_TYPE=resolved,unresolved

## Delay alerts with below types for X milliseconds 
## and cancel pushing alert if it is resolved within that time
#UNRESOLVED_TIMEOUT_TYPES=ServerCpu,ServerMem
#UNRESOLVED_TIMEOUT=20000
```

After deploying the Stack create a new [Alerter](https://komo.do/docs/resources#alerter)

* **Endpoint:** `Custom`
  * In the Endpoint field set the IP:PORT of the `komodo-apprise-alerter` stack you created IE `http://192.168.YOUR.IP:7000`
* Optionally, set any **Alert Types** you may need

**Save** the Alerter and then **Test Alerter** to make sure everything is ready to use.

</details>