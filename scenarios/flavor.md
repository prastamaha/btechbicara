# Octavia Flavor

if you don't define flavor, amphora will use the default flavor found in `/ etc / octavia / octavia.conf`

You can customize which flavor Amphora will use by setting the flavorprofile

## Plot

```
[flavorprofile] => [flavor] => [loadbalancer]
```

## Configuration Step

### Flavor list
```
openstack flavor list

+--------------------------------------+--------+------+------+-----------+-------+-----------+
| ID                                   | Name   |  RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+--------+------+------+-----------+-------+-----------+
| 320e80ee-0d66-43db-82f3-a8c0376faf30 | large  | 2048 |   20 |         0 |     4 | True      |
| c7dab6de-9fd1-44fc-88b5-130fa29c26b2 | medium | 1024 |   10 |         0 |     2 | True      |
| e65c460b-546e-4b0b-9dfd-38882d860d36 | small  |  512 |    5 |         0 |     1 | True      |
+--------------------------------------+--------+------+------+-----------+-------+-----------+
```

### Create Flavorprofile

**SINGLE Mode**
```
openstack loadbalancer flavorprofile create --name fp.single.small --provider amphora --flavor-data '{"loadbalancer_topology": "SINGLE", "compute_flavor": "e65c460b-546e-4b0b-9dfd-38882d860d36"}'
openstack loadbalancer flavorprofile create --name fp.single.medium --provider amphora --flavor-data '{"loadbalancer_topology": "SINGLE", "compute_flavor": "c7dab6de-9fd1-44fc-88b5-130fa29c26b2"}'
openstack loadbalancer flavorprofile create --name fp.single.large --provider amphora --flavor-data '{"loadbalancer_topology": "SINGLE", "compute_flavor": "320e80ee-0d66-43db-82f3-a8c0376faf30"}'
```

**ACTIVE_STANDBY Mode**
```
openstack loadbalancer flavorprofile create --name fp.active_standby.small --provider amphora --flavor-data '{"loadbalancer_topology": "ACTIVE_STANDBY", "compute_flavor": "e65c460b-546e-4b0b-9dfd-38882d860d36"}'
openstack loadbalancer flavorprofile create --name fp.active_standby.medium --provider amphora --flavor-data '{"loadbalancer_topology": "ACTIVE_STANDBY", "compute_flavor": "c7dab6de-9fd1-44fc-88b5-130fa29c26b2"}'
openstack loadbalancer flavorprofile create --name fp.active_standby.large --provider amphora --flavor-data '{"loadbalancer_topology": "ACTIVE_STANDBY", "compute_flavor": "320e80ee-0d66-43db-82f3-a8c0376faf30"}'
```

### Create Flavor

**SINGLE Mode**
```
openstack loadbalancer flavor create --flavorprofile fp.single.small --description 'single amphora, 1 vcpu, 512 ram, 5 disk' --enable --name single.small
openstack loadbalancer flavor create --flavorprofile fp.single.medium --description 'single amphora, 2 vcpu, 1024 ram, 10 disk' --enable --name single.medium
openstack loadbalancer flavor create --flavorprofile fp.single.large --description 'single amphora, 4 vcpu, 2048 ram, 20 disk' --enable --name single.large
```

**ACTIVE_STANDBY Mode**
```
openstack loadbalancer flavor create --flavorprofile fp.active_standby.small --description 'high-availability amphora, 1 vcpu, 512 ram, 5 disk' --enable --name active_standby.small
openstack loadbalancer flavor create --flavorprofile fp.active_standby.medium --description 'high-availability amphora, 2 vcpu, 1024 ram, 10 disk' --enable --name active_standby.medium
openstack loadbalancer flavor create --flavorprofile fp.active_standby.large --description 'high-availability amphora, 4 vcpu, 2048 ram, 20 disk' --enable --name active_standby.lage
```

### Create Loadbalancer using flavor

**SINGLE Mode**
```
# Create Loadbalancer
LB_VIP=$(openstack loadbalancer create --flavor single.small  --name single-lb1 --vip-subnet-id private-subnet | awk  '/ vip_address / {print $4}')

# Create Floating ip
openstack floating ip create --floating-ip-address 10.20.150.100 public-net

# Assign floating ip to lb vip
openstack floating ip set --port $(openstack port list | grep $LB_VIP | cut -d '|' -f 3) 10.20.150.100

# Create Listener
openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 single-lb1

# Create Pool
openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP

# Create Health Monitor
openstack loadbalancer healthmonitor create --delay 5 --max-retries 3 --timeout 5 --type HTTP --url-path / pool1

# Add Instance become pool member

openstack loadbalancer member create --subnet-id private-subnet --address 192.168.100.167 --protocol-port 80 pool1
openstack loadbalancer member create --subnet-id private-subnet --address 192.168.100.81 --protocol-port 80 pool1
```

**ACTIVE_STANDBY Mode**
```
# Create Loadbalancer
LB_VIP=$(openstack loadbalancer create --flavor active_standby.small  --name active_standby-lb1 --vip-subnet-id private-subnet | awk  '/ vip_address / {print $4}')

# Create Floating ip
openstack floating ip create --floating-ip-address 10.20.150.100 public-net

# Assign floating ip to lb vip
openstack floating ip set --port $(openstack port list | grep $LB_VIP | cut -d '|' -f 3) 10.20.150.100

# Create Listener
openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 active_standby-lb1

# Create Pool
openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP

# Create Health Monitor
openstack loadbalancer healthmonitor create --delay 5 --max-retries 3 --timeout 5 --type HTTP --url-path / pool1

# Add Instance become pool member

openstack loadbalancer member create --subnet-id private-subnet --address 192.168.100.167 --protocol-port 80 pool1
openstack loadbalancer member create --subnet-id private-subnet --address 192.168.100.81 --protocol-port 80 pool1
```
