## Contiv

Install Contiv components (netmaster, netplugin, contiv_etcd) on Master and Minion nodes 
```
iptables -t nat -A OUTPUT -p tcp -o contivh1 --src 10.129.0.2 --dst 172.30.0.1 --dport 443 -j DNAT --to-destination 10.129.0.2:8443
iptables -t nat -A OUTPUT -p tcp -o contivh1 --src 10.129.0.2 --dst 172.30.0.1 --dport 53 -j DNAT --to-destination 10.129.0.2:8053
iptables -t nat -A OUTPUT -p udp -o contivh1 --src 10.129.0.2 --dst 172.30.0.1 --dport 53 -j DNAT --to-destination 10.129.0.2:8053
```
## Requirements

* Ansible 2.2
* Centos/ RHEL

## Current Contiv restrictions when used with Openshift

* Only works with default service network
* Openshift Origin only 
* VLAN encap mode only (default for Openshift Ansible)
* Bare metal deployments only
* Requires additional network configuration on the external physical routers (ref. Openshift docs Contiv section)

## Key Ansible inventory configuration parameters

* ``openshift_use_contiv=True``
* ``openshift_use_openshift_sdn=False``
* ``os_sdn_network_plugin_name='cni'``
* ``contiv_netmaster_interface=eth0``
* ``contiv_netplugin_interface=eth1``
* ref. Openshift docs Contiv section for more details

## Example bare metal deployment of Openshift + Contiv 

* Example bare metal deployment

![Screenshot](roles/contiv/contiv-openshift-vlan-network.png)

* contiv241 is a Master + minion node
* contiv242 and contiv243 are minion nodes
* VLANs 1001, 1002 used for contiv container networks
* VLAN 10 used for cluster-internal host network 
* VLANs added to isolated VRF on external physical switch 
* Static routes added on external switch as shown to allow routing between host and container networks
* External switch also used for public internet access 

## Todo

- split Roles,DaemonSet and RS in separate files.
- Use a template for cni_config.
- Redo cleanup tasks.
- Cleanup old files.
- contanerize ovs, based in this config:
```
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    k8s-app: contiv-ovs
  name: contiv-ovs
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: contiv-ovs
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ''
      labels:
        k8s-app: contiv-ovs
    spec:
      containers:
      - command:
        - /scripts/start-ovsdb-server.sh
        image: contiv/ovs:1.2.1
        name: contiv-ovsdb-server
        securityContext:
          privileged: false
        volumeMounts:
        - mountPath: /etc/openvswitch
          name: etc-openvswitch
          readOnly: false
        - mountPath: /var/run
          name: var-run
          readOnly: false
      - command:
        - /scripts/start-ovs-vswitchd.sh
        image: contiv/ovs:1.2.1
        name: contiv-ovs-vswitchd
        securityContext:
          privileged: true
        volumeMounts:
        - mountPath: /etc/openvswitch
          name: etc-openvswitch
          readOnly: false
        - mountPath: /lib/modules
          name: lib-modules
          readOnly: true
        - mountPath: /var/run
          name: var-run
          readOnly: false
      hostNetwork: true
      hostPID: true
      tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
      volumes:
      - hostPath:
          path: /etc/openvswitch
        name: etc-openvswitch
      - hostPath:
          path: /lib/modules
        name: lib-modules
      - hostPath:
          path: /var/run
        name: var-run

```