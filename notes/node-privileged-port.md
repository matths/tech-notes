---
title: Node and privileged ports
tags: node, server, port, network, access, linux
---

# Node and privileged ports

When running a node server using privileged ports (below 1024, e.g. http-80 and https-443) in user space, one can give the node binary the right to do so, independent of the user running node. So basically every user of node can take advantage of that and bind to such port.

```sh
sudo setcap cap_net_bind_service=+ep `readlink -f \`which node\``
```

**Attention**

You have to run this command everytime you update node (change to node binary file).
If you forget, you might wonder why your service isn't coming up again after a restart.


chances are high you need to install another package before

```sh
sudo apt-get install libcap2-bin
```


for more information I recommend this gist https://gist.github.com/firstdoit/6389682