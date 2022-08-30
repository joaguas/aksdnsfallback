# aksdnsfallback
Fix for AKS clusters running systemd 237-3ubuntu10.54 (bug 1988119)


https://bugs.launchpad.net/ubuntu/+source/systemd/+bug/1988119

A bug with systemd makes resolved populate /run/systemd/resolve/resolv.conf with no nameservers.
Because /etc/resolv.conf is a symbolic link to the above file, the OS will have no nameservers to use

A reboot should address the issue but when this is not possible/feasible  we can use dhclient to renew dhcp information and get the DNS server.


This fix will deploy a privileged pod which will force dhclient to renew dns configuration. It uses an image that AKS worker nodes already have pre-pulled (mcr.microsoft.com/oss/nginx/nginx:1.13.12-alpine, see https://github.com/Azure/AKS/blob/4e0f71eee390ab3b36003e52c66e542830aa8ad9/vhd-notes/aks-ubuntu/AKSUbuntu-1804/2020.03.05.txt#L103 ) as with any other image this would fail since with no DNS, the container runtime wouldn't be able to resolve any container registry. It will then force dhclient to renew.

# Fix for individual nodes
```
wget https://raw.githubusercontent.com/joaguas/aksdnsfallback/main/nodedhclient.yaml
edit line nr 7 and replace it with the node you wish to fix
kubectl apply -f individualnode.yaml
```

# Fix for all worker nodes
```
kubectl apply -f https://raw.githubusercontent.com/joaguas/aksdnsfallback/main/dsdhclient.yaml
```


# If the above method fails, another alternative is to configure resolved to use a fallback DNS server which we can hardcode in its configuration.

If you're using your own custom DNS servers, please replace 168.63.129.16 with the ip of your DNS server either if you're using the individual node fix or the daemonset for all nodes


-- Fix for individual nodes
```
wget https://raw.githubusercontent.com/joaguas/aksdnsfallback/main/nodefallback.yaml
edit line nr 7 and replace it with the node you wish to fix
kubectl apply -f individualnode.yaml
```

--  Fix for all worker nodes
```
kubectl apply -f https://raw.githubusercontent.com/joaguas/aksdnsfallback/main/dsfallback.yaml


Please note that the fallback method will make permanent changes to your node's resolved configuration until node gets reimaged. 

For both solutions, once the issue has been addressed and because no new worker nodes should be affected by this issue (canonical has since removed the systemd package that triggers the bug, you can delete the daemonset/pod)
```
