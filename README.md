[English](https://github.com/txn2/kubefwd/blob/master/README.md)|[中文](https://github.com/txn2/kubefwd/blob/master/README_CN.md)

Kubernetes port forwarding for local development.

**NOTE:** Accepting pull requests for bug fixes, tests, and documentation only. 

![kubefwd - kubernetes bulk port forwarding](https://raw.githubusercontent.com/txn2/kubefwd/master/kubefwd-mast2.jpg)

[![Build Status](https://travis-ci.com/txn2/kubefwd.svg?branch=master)](https://travis-ci.com/txn2/kubefwd)
[![GitHub license](https://img.shields.io/github/license/txn2/kubefwd.svg)](https://github.com/txn2/kubefwd/blob/master/LICENSE)
[![Go Report Card](https://goreportcard.com/badge/github.com/txn2/kubefwd)](https://goreportcard.com/report/github.com/txn2/kubefwd)
[![GitHub release](https://img.shields.io/github/release/txn2/kubefwd.svg)](https://github.com/txn2/kubefwd/releases)

# kubefwd (Kube Forward)

Read [Kubernetes Port Forwarding for Local Development](https://mk.imti.co/kubernetes-port-forwarding/) for background and a detailed guide to **kubefwd**. Follow [Craig Johnston](https://twitter.com/cjimti) on Twitter for project updates.

**kubefwd** is a command line utility built to port forward multiple [services] within one or more [namespaces] on one or more Kubernetes clusters. **kubefwd** uses the same port exposed by the service and forwards it from a loopback IP address on your local workstation. **kubefwd** temporally adds domain entries to your `/etc/hosts` file with the service names it forwards.

When working on our local workstation, my team and I often build applications that access services through their service names and ports within a [Kubernetes] namespace. **kubefwd** allows us to develop locally with services available as they would be in the cluster.

![kubefwd - Kubernetes port forward](kubefwd_ani.gif)

<p align="center">
  <img width="654" height="684" src="https://mk.imti.co/images/content/kubefwd-net.png" alt="kubefwd - Kubernetes Port Forward Diagram">
</p>

## OS

Tested directly on **macOS** and **Linux** based docker containers.

## MacOs Install / Update

**kubefwd** assumes you have **kubectl** installed and configured with access to a Kubernetes cluster. **kubefwd** uses the **kubectl** current context. The **kubectl** configuration is not used. However, its configuration is needed to access a Kubernetes cluster.

Ensure you have a context by running:
```bash
kubectl config current-context
```

If you are running MacOS and use [homebrew] you can install **kubefwd** directly from the [txn2] tap:

```bash
brew install txn2/tap/kubefwd
```

To upgrade:
```bash
brew upgrade kubefwd
```

## Windows Install / Update

```batch
scoop install kubefwd
```

To upgrade:
```batch
scoop update kubefwd
```

## Docker

Forward all services from the namespace **the-project** to a Docker container named **the-project**:

```bash
docker run -it --rm --privileged --name the-project \
    -v "$(echo $HOME)/.kube/":/root/.kube/ \
    txn2/kubefwd services -n the-project
```


Execute a curl call to an Elasticsearch service in your Kubernetes cluster:

```bash
docker exec the-project curl -s elasticsearch:9200
```

## Alternative Installs (tar.gz, RPM, deb)
Check out the [releases](https://github.com/txn2/kubefwd/releases) section on Github for alternative binaries.

## Contribute
[Fork kubefwd](https://github.com/txn2/kubefwd) and build a custom version.
Accepting pull requests for bug fixes, tests, stability and compatibility
enhancements, and documentation only.

## Usage

Forward all services for the namespace `the-project`. Kubefwd finds the first Pod associated with each Kubernetes service found in the Namespace and port forwards it based on the Service spec to a local IP  address and port. A domain name is added to your /etc/hosts file pointing to the local IP.

### Recent Updates

**Service List Filtering** (Latest)
- Forward specific services with specific ports using `--service-list` flag
- Service-level forwarding for improved resource efficiency (one IP per service)
- Endpoint readiness checks before establishing port-forwards
- Support for loading service lists from configuration files
- Port-level filtering to forward only needed ports

**Dynamic Service Discovery**
- Namespace-level service monitoring with automatic start/stop of port-forwarding when services are created or deleted
- Pod-level forwarding monitoring with automatic restart when pods are updated or deleted
- Headless service support with forwarding to all pods or selective first-pod forwarding

**Enhanced Stability**
- Debounced pod synchronization to reduce API server load
- Improved error handling for service endpoints
- Better support for service and pod lifecycle events

```bash
sudo kubefwd svc -n the-project
```

Forward all svc for the namespace `the-project` where labeled `system: wx`:

```bash
sudo kubefwd svc -l system=wx -n the-project
```

Forward a single service named `my-service` in the namespace `the-project`:

```
sudo kubefwd svc -n the-project -f metadata.name=my-service
```

Forward more than one service using the `in` clause:
```bash
sudo kubefwd svc -l "app in (app1, app2)"
```

## Service List Filtering

Forward specific services with specific ports using the `--service-list` flag:

```bash
# Forward a single service with a specific port
sudo kubefwd svc --service-list myapp:8080

# Forward a service with multiple ports
sudo kubefwd svc --service-list myapp:8080,8081,9090

# Forward multiple services with specific ports
sudo kubefwd svc --service-list myapp:8080 --service-list api:443

# Specify namespace in the service name
sudo kubefwd svc --service-list myapp.production:8080 --service-list api.staging:443

# Combine with -n flag (service names without namespace use the -n value)
sudo kubefwd svc -n production --service-list myapp:8080 --service-list api:443
```

Supported formats for `--service-list`:
- `service:port` - requires `-n` flag to specify namespace
- `service.namespace:port` - namespace auto-discovered
- `service:port1,port2,port3` - multiple ports for a service

**Note:** `--service-list` is mutually exclusive with `-l` (label selector) and `-f` (field selector).

### Service List in Configuration File

You can also specify the service list in a configuration file:

```bash
sudo kubefwd svc -z path/to/fwdconf.yml
```

Example configuration file:
```yaml
baseUnreservedIP: 127.1.27.1

# Service list - CLI flag --service-list will override this
serviceList:
  - myapp.default:8080,8081
  - api.production:443,8443

serviceConfigurations:
  - name: myapp.default
    ip: 127.1.28.1
  - name: api.production
    ip: 127.1.28.2
```

### Benefits of Service List Filtering

Using `--service-list` provides several advantages:

1. **Efficient Resource Usage**: Only forwards specified services and ports instead of all services in a namespace
2. **Simplified Networking**: One IP address per service (instead of per pod)
3. **Automatic Load Balancing**: Kubernetes routes traffic to healthy pods
4. **Cleaner `/etc/hosts`**: Fewer hostname entries to manage
5. **Service-Level Forwarding**: For non-headless services, forwards directly to the service endpoint
6. **Selective Port Forwarding**: Only forward the ports you need, avoiding conflicts

## Help

```bash
$ kubefwd svc --help

INFO[00:00:48]  _          _           __             _     
INFO[00:00:48] | | ___   _| |__   ___ / _|_      ____| |    
INFO[00:00:48] | |/ / | | | '_ \ / _ \ |_\ \ /\ / / _  |    
INFO[00:00:48] |   <| |_| | |_) |  __/  _|\ V  V / (_| |    
INFO[00:00:48] |_|\_\\__,_|_.__/ \___|_|   \_/\_/ \__,_|    
INFO[00:00:48]                                              
INFO[00:00:48] Version 0.0.0                                
INFO[00:00:48] https://github.com/txn2/kubefwd              
INFO[00:00:48]                                              
Forward multiple Kubernetes services from one or more namespaces. Filter services with selector.

Usage:
  kubefwd services [flags]

Aliases:
  services, svcs, svc

Examples:
  kubefwd svc -n the-project
  kubefwd svc -n the-project -l app=wx,component=api
  kubefwd svc -n default -l "app in (ws, api)"
  kubefwd svc -n default -n the-project
  kubefwd svc -n default -d internal.example.com
  kubefwd svc -n the-project -x prod-cluster
  kubefwd svc -n the-project -m 80:8080 -m 443:1443
  kubefwd svc -n the-project -z path/to/conf.yml
  kubefwd svc -n the-project -r svc.ns:127.3.3.1
  kubefwd svc --service-list myapp:8080 --service-list api:443
  kubefwd svc --service-list myapp.production:8080,8081
  kubefwd svc --all-namespaces

Flags:
  -A, --all-namespaces             Enable --all-namespaces option like kubectl.
  -x, --context strings            specify a context to override the current context
  -d, --domain string              Append a pseudo domain name to generated host names.
  -f, --field-selector string      Field selector to filter on; supports '=', '==', and '!=' (e.g. -f metadata.name=service-name).
  -z, --fwd-conf string            Define an IP reservation configuration
  -h, --help                       help for services
  -c, --kubeconfig string          absolute path to a kubectl config file
  -m, --mapping strings            Specify a port mapping. Specify multiple mapping by duplicating this argument.
  -n, --namespace strings          Specify a namespace. Specify multiple namespaces by duplicating this argument.
  -r, --reserve strings            Specify an IP reservation. Specify multiple reservations by duplicating this argument.
  -l, --selector string            Selector (label query) to filter on; supports '=', '==', and '!=' (e.g. -l key1=value1,key2=value2).
      --service-list strings       Specify services with specific ports to forward (format: service:port or service.namespace:port). Multiple ports: service:port1,port2
  -t, --timeout int                Specify a timeout seconds for the port forwarding. (default 300)
  -v, --verbose                    Verbose output.
```

### License

Apache License 2.0

# Sponsor

Open source utility by [Craig Johnston](https://twitter.com/cjimti), [imti blog](http://imti.co/) and sponsored by [Deasil Works, Inc.]

Please check out my book [Advanced Platform Development with Kubernetes](https://imti.co/kubernetes-platform-book/):
Enabling Data Management, the Internet of Things, Blockchain, and Machine Learning.

[![Book Cover - Advanced Platform Development with Kubernetes: Enabling Data Management, the Internet of Things, Blockchain, and Machine Learning](https://raw.githubusercontent.com/apk8s/book-source/master/img/apk8s-banner-w.jpg)](https://amzn.to/3g3ihZ3)

Source code from the book [Advanced Platform Development with Kubernetes: Enabling Data Management, the Internet of Things, Blockchain, and Machine Learning](https://amzn.to/3g3ihZ3) by [Craig Johnston](https://imti.co) ([@cjimti](https://twitter.com/cjimti)) ISBN 978-1-4842-5610-7 [Apress; 1st ed. edition (September, 2020)](https://www.apress.com/us/book/9781484256107)

Read my blog post [Advanced Platform Development with Kubernetes](https://imti.co/kubernetes-platform-book/) for more info and background on the book.

Follow me on Twitter: [@cjimti](https://twitter.com/cjimti) ([Craig Johnston](https://twitter.com/cjimti))


## Please Help the Children of Ukraine

UNICEF is on the ground helping Ukraine's children, please donate to https://www.unicefusa.org/ <- "like" this project by donating.


[Kubernetes]:https://kubernetes.io/
[namespaces]:https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[services]:https://kubernetes.io/docs/concepts/services-networking/service/
[homebrew]:https://brew.sh/
[txn2]:https://txn2.com/
[Deasil Works, Inc.]:https://deasil.works/
