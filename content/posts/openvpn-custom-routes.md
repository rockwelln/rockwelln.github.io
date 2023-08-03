---
title: "Add custom routes to OpenVPN"
date: 2023-08-03T15:05:15+02:00
draft: false
---

When you connect to a VPN, you usually want to access some resources on the VPN network (e.g. a database, a web server, etc...).

But you don't necessarily want to route all your traffic through the VPN (e.g. you don't want to use the VPN to access google.com).

So a proper VPN configuration only routes your company's network through the VPN.

But sometimes a public resource is only accessible if your originating IP is in the VPN network.

In this case, you need to add a custom route to the VPN network.

To manually add a custom route, you can use the `route` command.

```bash
# on MacOS
sudo route -n add -net <public-ip> <gateway>
```

Ok, but that would be annoying to do it manually each time you connect to the VPN.

So you can use the `up` and `down` options of the OpenVPN client configuration to run a script when the VPN is connected or disconnected.

Personally I use Tunnelblick on MacOS, so I can add a script in the `~/Library/Application Support/Tunnelblick/Configurations/<config-name>.tblk/Contents/Resources` folder.

The script will be called `up.sh` and will contain the following:

```bash
#!/bin/bash

# add custom routes to VPN network
/sbin/route -n add -net <public-net>/24 -interface $1
```

The `$1` variable is the name of the network interface created by Tunnelblick.

Mind to secure the script and make it executable:

```bash
chmod 700 up.sh
chown root:wheel up.sh
```

Now, in the OpenVPN configuration, you can add the following at the end:

```bash
...
up up.sh
```

And that's it, you can now connect to your VPN and access the public resource.

## References

- [Add a route on MacOS X](https://blog.remibergsma.com/2012/03/04/howto-quickly-add-a-route-in-mac-osx/)
- [Add a static route](https://support.justaddpower.com/kb/article/320-adding-a-static-route-to-macos/)
- [SO-How to automatically add a local route after OpenVPN start?](https://serverfault.com/a/996743)
