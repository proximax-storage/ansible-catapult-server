# ansible-catapult-server

Ansible Role for Catapult Server. This role will let users configure the properties files and pull the Catapult Server Docker image 
from the specified Docker registry.


The tested platforms are:

* Ubuntu 16.04
* Ubuntu 18.04

## Usage
Specify the role in your `requirements.yml` file

```yaml
# Get the role from GitHub
- src: https://github.com/proximax-storage/ansible-catapult-server.git
  name: ansible-catapult-server
```

Run `ansible-galaxy` install to download the role from Github
```bash
ansible-galaxy install -r requirements.yml
```

## Prerequisites

The NEM Address and Nemesis block must be generated first before running Catapult Server.

Here is a short guide:
#### Generating NEM Addresses

```bash
# Generate raw address in /tmp/raw-addresses.txt
docker run -v /tmp:/tmp techbureau/catapult-tools:gcc-0.1.0.2 /bin/bash -c "/catapult/bin/catapult.tools.address --generate=50 -n mijin-test > /tmp/raw-addresses.txt"
```

#### Generating Nemesis block data

From the generated address, designate specific address to be used in the Catapult configuration 
(i.e. Nemesis Generation Hash, Nemesis Signer, Namespace Sink Account, Mosaic Sink Account, etc.) 

Create the directories and file placeholders 
```bash
mkdir -p /tmp/data/00000
dd if=/dev/zero of=/tmp/data/00000/hashes.dat bs=1 count=64
```

Create a file `/tmp/data/block-properties-file.properties`. Example below, replace it with your own generated addresses.

```text
[nemesis]
networkIdentifier = mijin-test
nemesisGenerationHash = B8A112687EAA8C0ECB43D56B78CC189DD50208267194A25E2FE7DE78D6550A6B 
nemesisSignerPrivateKey = 54DE4B04B1D9EBFA84A3165610B23A2C495D3CE1D4334F30B2625CF51181FFA3  

[output]
cppFile =
binDirectory = /data

[namespaces]
nem = true
eur = false

[namespace>nem]
duration = 0

[namespace>eur]
duration = 123456789

[mosaics]
nem:xem = true
eur:euro = false

[mosaic>nem:xem]
divisibility = 6
duration = 0
supply = 8'999'999'998'000'000
isTransferable = true
isSupplyMutable = false
isLevyMutable = false

[distribution>nem:xem]
SB5FXBNI6VZVGDRKFCPVHIEAI2YO6EINYCFWK6OT = 409'090'909'000'000
SDMOPGMMX3SJ2ROELKTLKGIC2R5CPS6CJUXZ7K6Q = 409'090'909'000'000
SDWDLKHUXBHAU5JTZA2F5Y7J4Z6C7T6YIFALF2WG = 409'090'909'000'000
SCOV23VAWDJH35GWP255R25SJZKQFZJP3P4DVKJK = 409'090'909'000'000
SBQ4Q44O45LAOJYAFMYETZSOVO3C7OCJUW65Y52Z = 409'090'909'000'000
SCQUG4GNUWNGCPJFW2KB7MN2XOBNZZ64OVU6ARP3 = 409'090'909'000'000
SD2H237LZYXARWURM5TEQLGO2GYHSEFSDZXDBCEA = 409'090'909'000'000
SBTCG67L4QX6O63NDIPLFX6WQ2WKJFWCJOH7BYHC = 409'090'909'000'000
SAX4AQ23VXNSGUKJKUSGWXT3ZPMQUMVBPQW6GDWK = 409'090'909'000'000
SAGZ2W2JIQXIXBBOY4XEZKYX2XHNLZETOKQ26JOW = 409'090'909'000'000
SBPV65FNEQ2FKZHXF2PAPSEPZYZAAMZCN2OYDFI2 = 409'090'909'000'000
SCAFEVHYBBKFDYHMSMSPUE7GTBS4ALZH6JEN55U7 = 409'090'909'000'000
SBEYNRVJG7GRIYY5SA42NN5JL4NBGTSATKR44FR7 = 409'090'909'000'000
SAQCRHGYFABZCDTVM5WCQPDEQEZ4RXW4N2O4ZV2W = 409'090'909'000'000
SA44PUILINOIWXUF3XM44KFSRMVJCMRA5SFFEJXY = 409'090'909'000'000
SDEH5OTYM42OU5HLPRWSGZMDGEW2N4QSCHTRUI2Z = 409'090'909'000'000
SCZVP45OCOLP5E73IY6SBANFZSLSLRK2TBKIJK7F = 409'090'909'000'000
SDQLIWIUNKVSMMDB4Z4XXVLQ2DJZHWQIGONGX3KZ = 409'090'909'000'000
SDT6HL7GXHVGY5DTQN5DPSJT42G5NFZW7CKEUOCD = 409'090'909'000'000
SCJTG6ZVICSVI6PS46UHVFGWMQZONFRPJFZLUWCW = 409'090'909'000'000
SDYRX6GKTWVK2FQDXOPBURXDZM2B2GKBSNKCUVK2 = 409'090'909'000'000
SAYZNOITSCHGYADRAXSZM44JT35CLR222VW3HQUG = 409'090'909'000'000

[mosaic>eur:euro]
divisibility = 2
duration = 1234567890
supply = 15'000'000
isTransferable = true
isSupplyMutable = true
isLevyMutable = false
```

Now, generate the nemesis block.
```
docker run -v /tmp/data:/data techbureau/catapult-tools:gcc-0.1.0.2 /bin/bash -c "/catapult/bin/catapult.tools.nemgen --nemesisProperties /data/block-properties-file.properties"

# check files
tree /tmp/data
  /tmp/data
  ├── 00000
  │   ├── 00001.dat
  │   └── hashes.dat
  └── block-properties-file.properties
```

The `data` folder should be copied to your `target host` server and defined in the role variable `catapult_data_directory` as shown in the next section.

## Basic Setup

The code below shows a basic configuration for this role. Do not forget to replace the keys.

```yaml
- name: Catapult Server Simple Setup
  hosts: localhost
  roles:
    - role: ansible-catapult-server
  vars:    
    catapult_config_directory: /opt/catapult-config
    catapult_data_directory: /opt/catapult-config/data
    catapult_user: ubuntu
    catapult_group: ubuntu
    
    catapult_server_repository_name: techbureau/catapult-server
    catapult_server_docker_tag: gcc-0.1.0.2
    
    # config-network.properties
    net_config:
      nemesis_generation_hash_public_key: 0000000000000000000000000000000000000000000000000000000000000000
      nemesis_signer_public_key: 0000000000000000000000000000000000000000000000000000000000000000
      namespace_rental_fee_sink_public_key: 0000000000000000000000000000000000000000000000000000000000000000
      mosaic_rental_fee_sink_public_key: 0000000000000000000000000000000000000000000000000000000000000000
      
    # config-user.properties
    user_config:
      node_boot_private_key: 0000000000000000000000000000000000000000000000000000000000000000
      
    # config-node.properties
    node_config:
      localnode_host: 192.168.1.1
      localnode_friendly_name: peernode1
      
    # config-harvesting.properties
    hv_config:
      harvest_key: 0000000000000000000000000000000000000000000000000000000000000000
      is_auto_harvesting_enabled: true
      
    # peers-p2p.json
    known_peers: [  
    {
    'public_key':'0000000000000000000000000000000000000000000000000000000000000000',
    'endpoint.host':'192.168.1.2',
    'endpoint.port':'7900',
    'metadata.name':'peernode2',
    'metadata.roles':'Peer'
    }]
    
    # peers-api.json
    known_api_peers: [
    {
      'public_key':'0000000000000000000000000000000000000000000000000000000000000000',
      'endpoint.host':'192.168.1.3',
      'endpoint.port':'7900',
      'metadata.name':'apinode1',
      'metadata.roles':'Api'
    }]
```


## Docker Compose

After running the ansible role, a `docker-compose.yml` will be generated based on the set configuration.

```yaml
version: "3.6"
services:
  catapult:
    # Downloads Catapult Server from specified Docker repo
    image: techbureau/catapult-server:gcc-0.1.0.2
    ports:
      - 7902:7902
      - 7900:7900
      - 7901:7901
    volumes:
      - /opt/catapult-config:/catapultconfig
      - /opt/catapult-config/data:/data:rw
    restart: always
    command: sh -c "/catapult/bin/catapult.server /catapultconfig"
```

Run the Docker container in detached mode 
```
docker-compose up -d
```

Stop the Docker container 
```
docker-compose down
```

## Metrics
There are tools developed by TechBureau for checking Catapult Server status

* catapult.tools.health

```bash
docker run -v /opt/catapult-config:/catapultconfig techbureau/catapult-tools:gcc-0.1.0.2 /bin/bash -c "/catapult/bin/catapult.tools.health /catapultconfig"
```

* catapult.tools.network

```bash
docker run -v /opt/catapult-config:/catapultconfig techbureau/catapult-tools:gcc-0.1.0.2 /bin/bash -c "/catapult/bin/catapult.tools.network /catapultconfig"
```

## Work In Progress
* Catapult REST Gateway deployment
