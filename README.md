Ansible Role: Setup and Teardown of BIG-IP HA cluster
=========

Performs a series of steps needed to setup a functioning HA cluster on the BIG-IP. Can also perform a first setup,
where it configure individual device's HA settings prior to creating HA. It can also remove the configured HA settings
from the BIG-IP system.

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below. For their default values, see `defaults/main.yml`:

    bigip_ha_cluster_server: localhost
    bigip_ha_cluster_server_port: 443
    bigip_ha_cluster_user: admin
    bigip_ha_cluster_password: secret
    bigip_ha_cluster_validate_certs: false

Establishes initial connection to the specified **BIG-IP** to configure HA cluster. These values are substituted into
your ``provider`` module parameter.

    bigip_ha_cluster_state: present

When set to ``present`` the HA cluster will be created on the BIG-IP.
The ``absent`` value will remove the HA cluster from the BIG-IP
    
    bigip_ha_cluster_peer_server: localhost

The IP address of the peer device, used when performing first setup or setting up device trust. 

    bigip_ha_cluster_peer_user: admin

The username on the peer device, used when performing first setup or setting up device trust.    
    
    bigip_ha_cluster_peer_password: secret

The password on the peer device, used when performing first setup or setting up device trust. 

    bigip_ha_cluster_peer_server_port: 443
    bigip_ha_cluster_peer_validate_certs: false
    
The common connection parameters used when performing first setup on the peer device.

    bigip_ha_cluster_first_setup
    
When set to `true` the role will preform first_setup of both devices in HA, which includes setting up hostname and 
HA connection information.

    bigip_ha_cluster_config_sync_ip

Used during first setup, specifies config sync ip on the first **BIG-IP** in the cluster. This parameter is mandatory

    bigip_ha_cluster_unicast_failover

Used during first setup, specifies a list of addresses and ports used for fail-over detection on the first 
**BIG-IP** in the cluster. This parameter is mandatory.

    bigip_ha_cluster_mirror_primary_address
    bigip_ha_cluster_mirror_secondary_address

Optional parameters used during first setup, specifies primary and secondary address for connection mirroring on the
first **BIG-IP** device.

    bigip_ha_cluster_peer_config_sync_ip

Used during first setup, specifies config sync ip on the second **BIG-IP** in the cluster. This parameter is mandatory

    bigip_ha_cluster_peer_unicast_failover

Used during first setup, specifies a list of addresses and ports used for fail-over detection on the second 
**BIG-IP** in the cluster. This parameter is mandatory.

    bigip_ha_cluster_peer_mirror_primary_address
    bigip_ha_cluster_peer_mirror_secondary_address

Optional parameters used during first setup, specifies primary and secondary address for connection mirroring on the
second **BIG-IP** device.

    bigip_ha_cluster_set_trust 

When set to `true` the role will setup trust between two **BIG-IPs** before setting up HA configuration.

    bigip_ha_cluster_hostname
    bigip_ha_cluster_peer_hostname

The hostname for first and second **BIG-IP** in the HA cluster. It is also used during first setup to
 configure these values on the respective **BIGIPs** . This is a mandatory parameter.

    bigip_ha_cluster_device_group

The name of the device group to be configured or removed from the **BIG-IP**.

    bigip_ha_cluster_auto_sync

Device group parameter that specifies if auto-sync should be enabled, used during HA creation.

    bigip_ha_cluster_sync_to_group

Parameter that specifies if first sync to device group should be performed after HA is set.

    bigip_ha_cluster_traffic_group

Defines the name of the traffic-group that needs to be created or configured. 
It is also used to remove the traffic group during HA teardown. **Only non-default traffic groups can be removed.**

    bigip_ha_cluster_ha_group

Specifies the HA group failover method that needs to be associated with the traffic-group.

    bigip_ha_cluster_ha_load_factor

Configures system relative load that the associated traffic-group presents to other traffic-groups. 
Used when Load Aware failover method is desired. 

    bigip_ha_cluster_ha_order
    
Specifies HA order as the failover method, the parameter is a list of **BIG-IP** hostnames. The order on the list 
matters and represents the order in which devices should be come Active/Standby.

    bigip_ha_cluster_auto_failback

Specifies whether the traffic group fails back to the initial device configured in `bigip_ha_cluster_ha_order`

    bigip_ha_cluster_auto_failback_time

Specifies the number of seconds the system delays before failing back to the initial device 
configured in `bigip_ha_cluster_ha_order`


Dependencies
------------

* A licensed **BIG-IP** with pre-configured **VLANs**, **self-IPs**, **DNS** and **NTP** information.
* A BIG-IP TMOS version **12.0.0** and **above**.

Example Playbook
----------------

* Configure HA with traffic-group HA order

        - name: Configure HA and Traffic Group
          hosts: configure
          any_errors_fatal: true
        
          tasks:
            - name: Configure HA group, setup TG HA order, preform first sync
              include_role:
                name: ansible-role-bigip_ha_cluster
              vars:
                bigip_ha_cluster_sync_to_group: true
                bigip_ha_cluster_device_group: "test_dg_failover"
                bigip_ha_cluster_traffic_group: "test_tg"
                bigip_ha_cluster_hostname: "bigip1.local"
                bigip_ha_cluster_peer_hostname: "bigip2.local"
                bigip_ha_cluster_ha_order:
                - "{{ bigip_ha_cluster_hostname }}"
                - "{{ bigip_ha_cluster_peer_hostname }}"

* Configure HA with first setup

        - name: First Setup, configure HA and traffic group
          hosts: configure
          any_errors_fatal: true
        
          tasks:
            - name: First Setup, configure HA group, setup TG HA order, preform first sync
              include_role:
                name: ansible-role-bigip_ha_cluster
              vars:
                bigip_ha_cluster_first_setup: true
                bigip_ha_cluster_sync_to_group: true
                bigip_ha_cluster_device_group: "test_dg_failover"
                bigip_ha_cluster_traffic_group: "test_tg"
                bigip_ha_cluster_hostname: "bigip1.local"
                bigip_ha_cluster_peer_hostname: "bigip2.local"
                bigip_ha_cluster_ha_order:
                - "{{ bigip_ha_cluster_hostname }}"
                - "{{ bigip_ha_cluster_peer_hostname }}"
                bigip_ha_cluster_config_sync_ip: "192.168.1.1"
                bigip_ha_cluster_unicast_failover:
                  - address: "192.168.1.1"
                    port: 1099
                  - address: "management-ip"
                bigip_ha_cluster_peer_config_sync_ip: "192.168.1.2"
                bigip_ha_cluster_peer_unicast_failover:
                  - address: "192.168.1.2"
                    port: 1099
                  - address: "management-ip"

* Remove HA and TG

        - name: Teardown HA and Traffic Group
          hosts: remove
          any_errors_fatal: true
        
          tasks:
            - name: Remove HA group and Traffic group
              include_role:
                name: ansible-role-bigip_ha_cluster
              vars:
                bigip_ha_cluster_device_group: "test_dg_failover"
                bigip_ha_cluster_traffic_group: "test_tg"
                bigip_ha_cluster_hostname: "bigip1.local"
                bigip_ha_cluster_peer_hostname: "bigip2.local"


## License

Apache

## Author Information

This role was created in 2019 by [Wojciech Wypior](https://github.com/wojtek0806).

[1]: https://galaxy.ansible.com/f5devcentral/bigip_ha_cluster
