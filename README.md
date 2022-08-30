# aksdnsfallback
Fix for AKS clusters running systemd 237-3ubuntu10.54 (bug 1988119)


https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1988119

A bug with systemd makes resolved populate /run/systemd/resolve/resolv.conf with no nameservers.
Because /etc/resolv.conf is a symbolic link to the above file, the OS will have no nameservers to use

A reboot should address the issue but when this is not possible/feasible we can rely on systemd-resolved itself and configure a fallback nameserver which will ensure there will always be a nameserver.

This fix will deploy a privileged pod which will make the necessary changes to the configuration files and restart the resolved service. It uses an image that AKS worker nodes already have pre-pulled (mcr.microsoft.com/oss/nginx/nginx:1.13.12-alpine, see https://github.com/Azure/AKS/blob/4e0f71eee390ab3b36003e52c66e542830aa8ad9/vhd-notes/aks-ubuntu/AKSUbuntu-1804/2020.03.05.txt#L103 ) as with any other image this would fail since with no DNS, the container runtime wouldn't be able to resolve any container registry.

If you're using your own custom DNS servers, please replace 168.63.129.16 with the ip of your DNS server either if you're using the individual node fix or the daemonset for all nodes


# Fix for individual nodes
```
wget https://raw.githubusercontent.com/joaguas/aksdnsfallback/main/individualnode.yaml
edit line nr 7 and replace it with the node you wish to fix
kubectl apply -f individualnode.yaml
```

# Fix for all worker nodes
```
kubectl apply -f https://raw.githubusercontent.com/joaguas/aksdnsfallback/main/dnsfixdaemonset.yaml
```
