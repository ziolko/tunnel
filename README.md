# @ziolko/tunnel - DIY tunneling solution for developers

This is an instruction on setting up a simple tunneling solution that is reliable, flexible and low-cost. It requires a Linux server with a public IP (I use [contabo.com](https://contabo.com/) but any VPS provider would do) and a domain.

## Usage

After you configure the tunneling server (see instruction below), you can start a tunnel from your local machine using `ssh`:

```
ssh -o ServerAliveInterval=60 -R <remote-port>:localhost:<local-port> root@<tunnelling-server-url>
```

Assuming your tunneling server is available at `ssh.proxy.dev.io` and your application locally is running at port 3000 you can run the following command:

```
ssh -o ServerAliveInterval=60 -R 8080:localhost:3000 root@ssh.proxy.dev.io
```

With this setup your application will be publicly available at https://8080.proxy.dev.io. The `8080` part of the address directly refers to the `<remote-port>` part from the command above.  Additionally, you can connect to the same port with e.g. https://one.8080.proxy.dev.io, https://two.8080.proxy.dev.io or https://awesome.8080.proxy.dev.io. The server takes care of getting valid SSL certificates for all the supported domains.

You can tunnel as many applications as you want as long as you tunnel them to different ports.

## Setting up the tunneling server

You need to follow these steps only once and they should take no more than 20min.

1. Create a Linux instance with HTTP and HTTPS traffic allowed
2. [Install Caddy](https://caddyserver.com/docs/install#debian-ubuntu-raspbian)
3. Open Caddy config file: `sudo vim /etc/caddy/Caddyfile` and paste content of the Caddyfile from this repository
4. Reload Caddy configuration with `sudo systemctl reload caddy`
5. Create a wildcard DNS record for your domain e.g. `*.proxy.dev.io` (if you use Cloudflare disable their proxy for this DNS entry)
6. Visit your domain https://8080.proxy.dev.io - it should say that it couldn't connect to the service at this port. This means the reverse proxy server is up and running.

I highly discourage you from running any other services on the tunneling server.

### Optional configuration steps

1. Add your SSH public key to `~/.ssh/authorized_keys` on the server to start tunnels without providing passwords
2. If you want to bind to "well-known ports" on the tunneling server (ports lower than 1024 e.g. standard SMTP ports) and you connect to the tunneling server as a non-root user you need to run `sudo echo 'net.ipv4.ip_unprivileged_port_start=0' >> /etc/sysctl.conf && sudo sysctl -p` on the server.

## Exposing TCP traffic

Sometimes you need to expose TCP port directly (e.g. to temporarily expose local database server). To enable this on the tunneling server run  `sudo echo 'GatewayPorts clientspecified' >> /etc/ssh/sshd_config && sudo systemctl restart ssh.service`. With this setup in place you can control if a port is directly exposed by adding `*:` before `<remote-port>`: 

```
ssh -o ServerAliveInterval=60 -R *:8080:localhost:3000 root@ssh.proxy.dev.io
```

Notice that your service provider/firewall may by default block most of the ports so you might need to do some additional configuration.
