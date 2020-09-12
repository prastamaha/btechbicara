# Octavia Flavor

if you don't define availabilityzone, amphora will be placed in any available availability zone

You can customize which availability zone Amphora will placed by setting the availabilityzoneprofile

## Plot

```
[availabilityzoneprofile] => [availabilityzone] => [loadbalancer]
```

## Configuration Step

### availability zone list
```
openstack availability zone list

+-----------+-------------+
| Zone Name | Zone Status |
+-----------+-------------+
| rack1     | available   |
| rack2     | available   |
| nova      | available   |
| nova      | available   |
+-----------+-------------+
```

### Create Availabilityzoneprofile
```
$ cat /etc/kolla/globals.yml | grep octavia_amp_boot_network_list
octavia_amp_boot_network_list: 4b70165e-1f30-4905-bcef-a52d2c8a38b5
```
```
openstack loadbalancer availabilityzoneprofile create --name azp.rack1 --provider amphora --availability-zone-data '{"compute_zone": "rack1", "management_network": "4b70165e-1f30-4905-bcef-a52d2c8a38b5"}'
openstack loadbalancer availabilityzoneprofile create --name azp.rack2 --provider amphora --availability-zone-data '{"compute_zone": "rack2", "management_network": "4b70165e-1f30-4905-bcef-a52d2c8a38b5"}'
```

### Create Availabilityzone
```
openstack loadbalancer availabilityzone create --availabilityzoneprofile azp.rack1 --description 'rack1 availability zone' --enable --name az.rack1
openstack loadbalancer availabilityzone create --availabilityzoneprofile azp.rack2 --description 'rack2 availability zone' --enable --name az.rack2
```

### Create loadbalancer with Availability zone
```
# Create Loadbalancer
LB_VIP=$(openstack loadbalancer create --availability-zone az.rack1 --name lb1 --vip-subnet-id private-subnet | awk  '/ vip_address / {print $4}')

# Create Floating ip
openstack floating ip create --floating-ip-address 10.20.150.100 public-net

# Assign floating ip to lb vip
openstack floating ip set --port $(openstack port list | grep $LB_VIP | cut -d '|' -f 3) 10.20.150.100

# Create Listener
openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 lb1

# Create Pool
openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP

# Create Health Monitor
openstack loadbalancer healthmonitor create --delay 5 --max-retries 3 --timeout 5 --type HTTP --url-path / pool1

# Add Instance become pool member

openstack loadbalancer member create --subnet-id private-subnet --address 192.168.100.167 --protocol-port 80 pool1
openstack loadbalancer member create --subnet-id private-subnet --address 192.168.100.81 --protocol-port 80 pool1
```
