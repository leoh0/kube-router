# User Guide

## Try Kube-router with cluster installers

The best way to get started is to deploy Kubernetes with Kube-router is with a cluster installer.

### kops
Please see the [steps](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kops.md) to deploy Kubernetes cluster with Kube-router using [Kops](https://github.com/kubernetes/kops)

### bootkube
Please see the [steps](https://github.com/cloudnativelabs/kube-router/tree/master/contrib/bootkube) to deploy Kubernetes cluster with Kube-router using [bootkube](https://github.com/kubernetes-incubator/bootkube)

### kubeadm
Please see the [steps](https://github.com/cloudnativelabs/kube-router/blob/master/docs/kubeadm.md) to deploy Kubernetes cluster with Kube-router using [Kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

### generic
Please see the [steps](https://github.com/cloudnativelabs/kube-router/blob/master/docs/generic.md) to deploy kube-router on manually installed clusters

## deployment

Depending on what functionality of kube-router you want to use, multiple deployment options are possible. You can use the flags `--run-firewall`, `--run-router`, `--run-service-proxy` to selectively enable only required functionality of kube-router.

Also you can choose to run kube-router as agent running on each cluster node. Alternativley you can run kube-router as pod on each node through daemonset.

## command line options

```
Usage of kube-router:
      --advertise-cluster-ip             Add Cluster IP of the service to the RIB so that it gets advertises to the BGP peers.
      --advertise-external-ip            Add External IP of service to the RIB so that it gets advertised to the BGP peers.
      --advertise-loadbalancer-ip        Add LoadbBalancer IP of service status as set by the LB provider to the RIB so that it gets advertised to the BGP peers.
      --advertise-pod-cidr               Add Node's POD cidr to the RIB so that it gets advertised to the BGP peers. (default true)
      --bgp-graceful-restart             Enables the BGP Graceful Restart capability so that routes are preserved on unexpected restarts
      --bgp-port uint16                  The port open for incoming BGP connections and to use for connecting with other BGP peers. (default 179)
      --cache-sync-timeout duration      The timeout for cache synchronization (e.g. '5s', '1m'). Must be greater than 0. (default 1m0s)
      --cleanup-config                   Cleanup iptables rules, ipvs, ipset configuration and exit.
      --cluster-asn uint                 ASN number under which cluster nodes will run iBGP.
      --cluster-cidr string              CIDR range of pods in the cluster. It is used to identify traffic originating from and destinated to pods.
      --enable-cni                       Enable CNI plugin. Disable if you want to use kube-router features alongside another CNI plugin. (default true)
      --enable-ibgp                      Enables peering with nodes with the same ASN, if disabled will only peer with external BGP peers (default true)
      --enable-overlay                   When enable-overlay set to true, IP-in-IP tunneling is used for pod-to-pod networking across nodes in different subnets. When set to false no tunneling is used and routing infrastrcture is expected to route traffic for pod-to-pod networking across nodes in different subnets (default true)
      --enable-pod-egress                SNAT traffic from Pods to destinations outside the cluster. (default true)
      --enable-pprof                     Enables pprof for debugging performance and memory leak issues.
      --hairpin-mode                     Add iptable rules for every Service Endpoint to support hairpin traffic.
      --health-port uint16               Health check port, 0 = Disabled (default 20244)
  -h, --help                             Print usage information.
      --hostname-override string         Overrides the NodeName of the node. Set this if kube-router is unable to determine your NodeName automatically.
      --iptables-sync-period duration    The delay between iptables rule synchronizations (e.g. '5s', '1m'). Must be greater than 0. (default 5m0s)
      --ipvs-sync-period duration        The delay between ipvs config synchronizations (e.g. '5s', '1m', '2h22m'). Must be greater than 0. (default 5m0s)
      --kubeconfig string                Path to kubeconfig file with authorization information (the master location is set by the master flag).
      --masquerade-all                   SNAT all traffic to cluster IP/node port.
      --master string                    The address of the Kubernetes API server (overrides any value in kubeconfig).
      --metrics-path string              Prometheus metrics path (default "/metrics")
      --metrics-port uint16              Prometheus metrics port, (Default 0, Disabled)
      --nodeport-bindon-all-ip           For service of NodePort type create IPVS service that listens on all IP's of the node.
      --nodes-full-mesh                  Each node in the cluster will setup BGP peering with rest of the nodes. (default true)
      --override-nexthop                 Override the next-hop in bgp routes sent to peers with the local ip.
      --peer-router-asns uints           ASN numbers of the BGP peer to which cluster nodes will advertise cluster ip and node's pod cidr. (default [])
      --peer-router-ips ipSlice          The ip address of the external router to which all nodes will peer and advertise the cluster ip and pod cidr's. (default [])
      --peer-router-multihop-ttl uint8   Enable eBGP multihop supports -- sets multihop-ttl. (Relevant only if ttl >= 2)
      --peer-router-passwords strings    Password for authenticating against the BGP peer defined with "--peer-router-ips".
      --peer-router-ports uints          The remote port of the external BGP to which all nodes will peer. If not set, default BGP port (179) will be used. (default [])
      --routes-sync-period duration      The delay between route updates and advertisements (e.g. '5s', '1m', '2h22m'). Must be greater than 0. (default 5m0s)
      --run-firewall                     Enables Network Policy -- sets up iptables to provide ingress firewall for pods. (default true)
      --run-router                       Enables Pod Networking -- Advertises and learns the routes to Pods via iBGP. (default true)
      --run-service-proxy                Enables Service Proxy -- sets up IPVS for Kubernetes Services. (default true)
  -v, --v string                         log level for V logs (default "0")
  -V, --version                          Print version information.
```

## requirements

- Kube-router need to access kubernetes API server to get information on pods, services, endpoints, network policies etc. The very minimum information it requires is the details on where to access the kubernetes API server. This information can be passed as `kube-router --master=http://192.168.1.99:8080/` or `kube-router --kubeconfig=<path to kubeconfig file>`.

- If you run kube-router as agent on the node, ipset package must be installed on each of the nodes (when run as daemonset, container image is prepackaged with ipset)

- If you choose to use kube-router for pod-to-pod network connectivity then Kubernetes controller manager need to be configured to allocate pod CIDRs by passing `--allocate-node-cidrs=true` flag and providing a `cluster-cidr` (i.e. by passing --cluster-cidr=10.1.0.0/16 for e.g.)

- If you choose to run kube-router as daemonset, then both kube-apiserver and kubelet must be run with `--allow-privileged=true` option

- If you choose to use kube-router for pod-to-pod network connecitvity then Kubernetes cluster must be configured to use CNI network plugins. On each node CNI conf file is expected to be present as /etc/cni/net.d/10-kuberouter.conf .`bridge` CNI plugin and `host-local` for IPAM should be used. A sample conf file that can be downloaded as `wget -O /etc/cni/net.d/10-kuberouter.conf https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/cni/10-kuberouter.conf`

## running as daemonset

This is quickest way to deploy kube-router (**dont forget to ensure the requirements**). Just run

```
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kube-router-all-service-daemonset.yaml
```

Above will run kube-router as pod on each node automatically. You can change the arguments in the daemonset definition as required to suit your needs. Some samples can be found at https://github.com/cloudnativelabs/kube-router/tree/master/daemonset with different argument to select set of the services kube-router should run.

## running as agent

You can choose to run kube-router as agent runnng on each node. For e.g if you just want kube-router to provide ingress firewall for the pods then you can start kube-router as
```
kube-router --master=http://192.168.1.99:8080/ --run-firewall=true --run-service-proxy=false --run-router=false
```

## cleanup configuration

Please delete kube-router daemonset and then clean up all the configurations done (to ipvs, iptables, ipset, ip routes etc) by kube-router on the node by running below command.

```
docker run --privileged --net=host cloudnativelabs/kube-router --cleanup-config
```

## trying kube-router as alternative to kube-proxy

If you have a kube-proxy in use, and want to try kube-router just for service proxy you can do
```
kube-proxy --cleanup-iptables
```
followed by
```
kube-router --master=http://192.168.1.99:8080/ --run-service-proxy=true --run-firewall=false --run-router=false
```
and if you want to move back to kube-proxy then clean up config done by kube-router by running
```
 kube-router --cleanup-config
```
and run kube-proxy with the configuration you have.
- [General Setup](/README.md#getting-started)

## Hairpin Mode

Communication from a Pod that is behind a Service to its own ClusterIP:Port is
not supported by default.  However, It can be enabled per-service by adding the
`kube-router.io/service.hairpin=` annotation, or for all Services in a cluster by
passing the flag `--hairpin-mode=true` to kube-router.

Additionally, the `hairpin_mode` sysctl option must be set to `1` for all veth
interfaces on each node.  This can be done by adding the `"hairpinMode": true`
option to your CNI configuration and rebooting all cluster nodes if they are
already running kubernetes.

Hairpin traffic will be seen by the pod it originated from as coming from the
Service ClusterIP if it is logging the source IP.

### Hairpin Mode Example

10-kuberouter.conf
```json
{
    "name":"mynet",
    "type":"bridge",
    "bridge":"kube-bridge",
    "isDefaultGateway":true,
    "hairpinMode":true,
    "ipam": {
        "type":"host-local"
     }
}
```

To enable hairpin traffic for Service `my-service`:
```
kubectl annotate service my-service "kube-router.io/service.hairpin="
```

## Direct server return

Please read below blog on how to user DSR in combination with `--advertise-external-ip` to build highly scalable and available ingress.
https://cloudnativelabs.github.io/post/2017-11-01-kube-high-available-ingress/

You can enable DSR(Direct Server Return) functionality per service. When enabled service endpoint
will directly respond to the client by passing the service proxy. When DSR is enabled Kube-router 
will uses LVS's tunneling mode to achieve this.

To enable DSR you need to annotate service with `kube-router.io/service.dsr=tunnel` annotation. For e.g.

```
kubectl annotate service my-service "kube-router.io/service.dsr=tunnel"
```

**In the current implementation when annotation is applied on the service, DSR will be applicable only to the external IP's.**

**Also when DSR is used, current implementation does not support port remapping. So you need to use same port and target port for the service**

You will need to enable `hostIPC: true` and `hostPID: true` in kube-router daemonset manifest.
Also host path `/var/run/docker.sock` must be made a volumemount to kube-router.

Above changes are required for kube-router to enter pod namespeace and create ipip tunnel in the pod and to 
assign the external IP to the VIP. 

For an e.g manifest please look at [manifest](../daemonset/kubeadm-kuberouter-all-features-dsr.yaml) with DSR requirements enabled.

## Load balancing Scheduling Algorithms

Kube-router uses LVS for service proxy. LVS support rich set of [scheduling alogirthms](http://kb.linuxvirtualserver.org/wiki/IPVS#Job_Scheduling_Algorithms). You can annotate 
the service to choose one of the scheduling alogirthms. When a service is not annotated `round-robin` scheduler is selected by default

```
For least connection scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=lc"

For round-robin scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=rr"

For source hashing scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=sh"

For destination hashing scheduling use:
kubectl annotate service my-service "kube-router.io/service.scheduler=dh"
```

## LoadBalancer IPs

If you want to also advertise loadbalancer set IPs
(`status.loadBalancer.ingress` IPs), e.g. when using it with MetalLb,
add the `--advertise-loadbalancer-ip` flag (`false` by default).

To selectively disable this behaviour per-service, you can use
the `kube-router.io/service.skiplbips` annotation as e.g.:
`$ kubectl annotate service my-external-service "kube-router.io/service.skiplbips=true"`

In concrete, unless the Service is annotated as per above, the
`--advertise-loadbalancer-ip` flag will make Service's Ingress IP(s)
set by the LoadBalancer to:
* be locally added to nodes' `kube-dummy-if` network interface
* be advertised to BGP peers

FYI Above has been successfully tested together with
[MetalLB](https://github.com/google/metallb) in ARP mode.

## HostPort support

If you would like to use `HostPort` functionality below changes are required in the manifest.

- By default kube-router assumes CNI conf file to be `/etc/cni/net.d/10-kuberouter.conf`. Add an environment variable `KUBE_ROUTER_CNI_CONF_FILE` to kube-router manifest and set it to `/etc/cni/net.d/10-kuberouter.conflist`
- Modify `kube-router-cfg` ConfigMap with CNI config that supports `portmap` as additional plug-in
```
    {
       "cniVersion":"0.3.0",
       "name":"mynet",
       "plugins":[
          {
             "name":"kubernetes",
             "type":"bridge",
             "bridge":"kube-bridge",
             "isDefaultGateway":true,
             "ipam":{
                "type":"host-local"
             }
          },
          {
             "type":"portmap",
             "capabilities":{
                "snat":true,
                "portMappings":true
             }
          }
       ]
    }
```
- Update init container command to create `/etc/cni/net.d/10-kuberouter.conflist` file
- Restart the container runtime

For an e.g manifest please look at [manifest](../daemonset/kubeadm-kuberouter-all-features-hostport.yaml) with necessary changes required for `HostPort` functionality.

## BGP configuration

[Configuring BGP Peers](bgp.md)

## Metrics

[Configure metrics gathering](metrics.md)
