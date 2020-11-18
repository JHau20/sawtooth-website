# **Using Docker for a Sawtooth Test Network** #

This procedure describes how to use Docker to create a network of five Sawtooth nodes for an application development environment. Each node is a set of Docker containers that runs a validator and related Sawtooth components.

Note

For a single-node environment, see [Using Docker for a Single Sawtooth Node](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/docker.html).

This procedure guides you through the following tasks:

> - Downloading the Sawtooth Docker Compose file
> - Starting the Sawtooth network with docker-compose
> - Checking process status
> - Configuring the allowed transaction types (optional)
> - Connecting to the Sawtooth shell container and confirming network functionality
> - Stopping Sawtooth and resetting the Docker environment

## **About the Docker Sawtooth Network Environment** ##

This test environment is a network of five Sawtooth nodes.

[Docker: Sawtooth network with five nodes](https://github.com/JHau20/sawtooth-website/blob/refresh/docs/core/1.2/images/appdev-environment-multi-node.pdf)
Like the [single-node test environment <docker>](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/docker_test_network.html), this environment uses parallel transaction processing and static peering. However, it uses a different consensus algorithm (Devmode consensus is not recommended for a network). You can choose either PBFT or PoET consensus.

The first node creates the genesis block, which specifies the on-chain settings for the network configuration. The other nodes access those settings when they join the network.

## **Prerequisites** ##

* This application development environment requires Docker Engine and Docker Compose.
	* Windows: Install the latest version of [Docker Engine for Windows](https://docs.docker.com/docker-for-windows/install/) (also installs Docker Compose).
	* macOS: Install the latest version of [Docker Engine for macOS](https://docs.docker.com/docker-for-mac/install/) (also installs Docker Compose).
	* Linux: Install the latest versions of [Docker Engine](https://docs.docker.com/engine/install/ubuntu/) and [Docker Compose](https://docs.docker.com/compose/install/#install-compose). Then follow [Post-Install steps](https://docs.docker.com/engine/install/linux-postinstall/).
* If you created a [single-node Docker environment <docker>](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/docker_test_network.html) that is still running 
shut it down and delete the existing blockchain data and logs. For more information, see [Stop Sawtooth Docker Label](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/).

## **Step 1: Download the Docker Compose File** ##

Download the Docker Compose file for a multiple-node network.
- For PBFT, download [sawtooth-default-pbft.yaml](sawtooth-default-pbft.yaml)
- For PoET, download [sawtooth-default-poet.yaml](sawtooth-default-poet.yaml)

## **Step 2: Start the Sawtooth Network** ##

Note 

The Docker Compose file for Sawtooth handles environment setup steps such as generating keys and creating a genesis block. To learn how the typical network startup 
process works, see [Ubuntu Test Network](http://172.26.96.1:4000/docs/core/1.2/app_developers_guide/ubuntu_test_network.html).

1. Open a terminal window.

2. Change to the directory where you saved the Docker Compose file.

3. Start the Sawtooth network.

	* For PBFT:
	```console
	user@host$ docker-compose -f sawtooth-default-pbft.yaml up
	```
	
	* For PoET:
	```console
	user@host$ docker-compose -f sawtooth-default-poet.yaml up
	```

4. This Compose file creates five Sawtooth nodes named `validator-#` (numbered from 0 to 4). Note the container names for the Sawtooth components on each node:

	`validator-0`:

	> * `sawtooth-validator-default-0`
	> * `sawtooth-rest-api-default-0`
	> * `sawtooth-pbft-engine-default-0` or `sawtooth-poet-engine-0`
	> * `sawtooth-settings-tp-default-0`
	> * `sawtooth-intkey-tp-python-default-0`
	> * `sawtooth-xo-tp-python-default-0`
	> * (PoET only) `sawtooth-poet-validator-registry-tp-0`
	
	`validator-1`:

 	> * `sawtooth-validator-default-1`
 	> * `sawtooth-rest-api-default-1`
 	> * `sawtooth-pbft-engine-default-1` or `sawtooth-poet-engine-1`
 	> * `sawtooth-settings-tp-default-1`
 	> * `sawtooth-intkey-tp-python-default-1`
 	> * `sawtooth-xo-tp-python-default-1`
 	> * (PoET only) `sawtooth-poet-validator-registry-tp-1`

	... and so on.

5. Note that there is only one shell container for this Docker environment:

	> * `sawtooth-shell-default`

## **Step 3: Check the REST API Process** ##

Use these commands on one or more node to confirm that the REST API is running.

1. Connect to the REST API container on a node, such as `sawtooth-poet-rest-api-default-0`.

	```console
	user@host$ docker exec -it sawtooth-rest-api-default-0 bash
	root@b1adcfe0#
	```

2. Use the following command to verify that this component is running.

	```console
	root@b1adcfe0# ps --pid 1 fw
	PID TTY      STAT   TIME COMMAND
	  1 ?        Ssl    0:00 /usr/bin/python3 /usr/bin/sawtooth-rest-api
	  --connect tcp://validator-0:4004 --bind rest-api-0:8008
	```

## **Step 4: Confirm Network Functionality** ##

1. Connect to the shell container.

	> ```console
	> user@host$ docker exec -it sawtooth-shell-default bash
	> root@0e0fdc1ab#
	> ```

2. To check whether peering has occurred on the network, submit a peers query to the REST API on the first node. This command specifies the container name and port for
   the first node's REST API.

	> ```console
	> root@0e0fdc1ab# curl http://sawtooth-rest-api-default-0:8008/peers
	> ```

	If this query returns a 503 error, the nodes have not yet peered with the Sawtooth network. Repeat the query until you see output that resembles the following 
	example:
 
	> ```console
 	> {
 	>  "data": [
 	>    "tcp://validator-4:8800",
 	>    "tcp://validator-3:8800",
	>    ...
	>    "tcp://validator-2:8800",
	>    "tcp://validator-1:8800"
	>  ],
	> "link": "http://sawtooth-rest-api-default-0:8008/peers"
	> ```

3. (Optional) You can run the following Sawtooth commands to show the other nodes on the network.

	1. Run `sawtooth peer list` to show the peer of a particular node. For example, the following command specifies the REST API on the first node, so it displays t	he first node's peers.
	
		```console 
		root@0e0fdc1ab# sawtooth peer list --url http://sawtooth-rest-api-default-0:8008
		tcp://validator-1:8800,tcp://validator-1:8800,tcp://validator-2:8800,tcp://validator-3:8800
		```

	2. Run `sawnet peers list` to display a complete graph of peers on the network (available in Sawtooth release 1.1 and later).
	
		```console
	 	root@0e0fdc1ab# sawnet peers list http://sawtooth-rest-api-default-0:8008
	 	{
	 	"tcp://validator-0:8800": [
	 	"tcp://validator-1:8800",
	 	"tcp://validator-1:8800",
	 	"tcp://validator-2:8800",
	 	"tcp://validator-3:8800"
	 	]
	 	}
	 	```

4. Submit a transaction to the REST API on the first node. This example sets a key named MyKey to the value 999.

 	> ```console
 	> root@0e0fdc1ab# intkey set --url http://sawtooth-rest-api-default-0:8008 MyKey 999
 	> ```
 	> 
 	> The output should resemble this example:
 	> 
 	> ```console
 	> {
 	> "link": "http://sawtooth-rest-api-default-0:8008/batch_statuses?id=dacefc7c9fe2c8510803f8340...
 	> }
	> ```

5. Watch for this transaction to appear on a different node. The following command requests the value of `MyKey` from the REST API on the second node.
You can run this command from the first node's shell container by specifying the URL of the other node's REST API, as in this example.

 	> ```console
 	> root@0e0fdc1ab# intkey show --url http://sawtooth-rest-api-default-1:8008 MyKey
 	> ```
 	>
 	> The output should show the key name and current value:
 	>
 	> ```console
 	> MyKey: 999
 	> ```
	
## **Step 5: Configure the Allowed Transaction Types (Optional)** ##

By default, a validator accepts transactions from any transaction processor. However, Sawtooth allows you to limit the types of transactions that can be submitted.

In this step, you will configure the Sawtooth network to accept transactions only from the transaction processors running in the example environment. Transaction-type 
restrictions are an on-chain setting, so this configuration change is made on one node, then applied to all other nodes.

The [Settings Transaction Family](http://172.26.96.1:4000/docs/core/1.2/transaction_family_specifications/settings_transaction_family.html) handles on-chain 
configuration settings. You will use the `sawset` command to create and submit a batch of transactions containing the configuration change.

Important

You **must** run this procedure from the first validator container, because the example Docker Compose file uses the first validator's key to create and sign the 
genesis block. (At this point, only the key used to create the genesis block can change on-chain settings.) For more information, see [Adding Authorized Users](http://172.26.96.1:4000/docs/core/1.2/sysadmin_guide/adding_authorized_users.html).

1. Connect to the first validator container (`sawtooth-validator-default-0`). The next command requires the validator key that was generated in that container.

	```console
	user@host$ docker exec -it sawtooth-validator-default-0 bash
	root@c0c0ab33#
	```

2. Run the following command from the validator container to specify the allowed transaction families.
	* For PBFT:

		```console
		root@c0c0ab33# sawset proposal create --url http://sawtooth-rest-api-default-0:8008 --key /etc/sawtooth/keys/validator.priv \
	 	sawtooth.validator.transaction_families='[{"family": "intkey", "version": "1.0"}, {"family":"sawtooth_settings", "version":"1.0"}, {"family":"xo", 
		"version":"1.0"}]'
	 	```
	* For PoET:

	 	```console
	 	root@c0c0ab33# sawset proposal create --url http://sawtooth-rest-api-default-0:8008 --key /etc/sawtooth/keys/validator.priv \	
	 	sawtooth.validator.transaction_families='[{"family": "intkey", "version": "1.0"}, {"family":"sawtooth_settings", "version":"1.0"}, {"family":"xo", 
		"version":"1.0"}, {"family":"sawtooth_validator_registry", "version":"1.0"}]'
	 	```

 	This command sets `sawtooth.validator.transaction_families` to a JSON array that specifies the family name and version of each allowed transaction processor (defined in the transaction header of each family's [Transaction Family Specifications](http://172.26.96.1:4000/docs/core/1.2/transaction_family_specifications).

3. After this command runs, a `TP_PROCESS_REQUEST` message appears in the docker-compose output.

 	```console
 	.
 	.
 	.
 	sawtooth-settings-tp-default-0  | INFO  | settings_tp::handler | Setting "sawtooth.validator.transaction_families" changed to "[{\"family\": \"intkey\", 
	\"version\": \"1.0\"}, {\"family\":\"sawtooth_settings\", \"version\":\"1.0\"}, {\"family\":\"xo\", \"version\":\"1.0\"}, 
	{\"family\":\"sawtooth_validator_registry\", \"version\":\"1.0\"}]" sawtooth-settings-tp-default-0  | INFO  | sawtooth_sdk::proces | TP_PROCESS_REQUEST sending 
	TpProcessResponse: OK
 	```

4. Run the following command to check the setting change on the shell container or any validator container. You can specify any REST API on the network; this example uses the REST API on the first node.

 	```console
 	root@0e0fdc1ab# sawtooth settings list --url http://sawtooth-rest-api-default-0:8008
 	```

 	The output should be similar to this example:

 	```console
 	sawtooth.consensus.algorithm.name: {name}
 	sawtooth.consensus.algorithm.version: {version}
 	...
 	sawtooth.publisher.max_batches_per_block=1200
 	sawtooth.settings.vote.authorized_keys: 0242fcde86373d0aa376...
 	sawtooth.validator.transaction_families: [{"family": "intkey...
 	```

## **Step 6: Stop the Sawtooth Network (Optional)** ##

Use this procedure to stop or reset the multiple-node Sawtooth environment.

1. Exit from all open containers (such as the shell, REST-API, validator, and settings containers used in this procedure).

2. Enter CTRL-c in the window where you ran `docker-compose up`.

3. After all containers have shut down, you can reset the environment (remove all containers and data) with the following command:
	* For PBFT:
	```console
	user@host$ docker-compose -f sawtooth-default-pbft.yaml down
	```
	
	* For PoET:
	```console
	 user@host$ docker-compose -f sawtooth-default-poet.yaml down
	 ```
