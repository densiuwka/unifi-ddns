# Cloudflare DDNS for UniFi OS

A Cloudflare Worker script that provides a UniFi-compatible DDNS API to dynamically update the IP address of a DNS A record.

## Why?

UniFi Dream Machine Pro (UDM-Pro) or UniFi Security Gateway (USG) users may need to update Cloudflare domain name DNS records when their public IP address changes. UniFi does not natively support Cloudflare as a DDNS provider.

### Configuring Cloudflare

Ensure you have a Cloudflare account and your domain is configured to point to Cloudflare nameservers.

#### Install With Click To Deploy

1. Deploy the Worker: [![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/workerforce/unifi-ddns)
2. Navigate to the Cloudflare Workers dashboard.
3. After deployment, note the `\*.workers.dev` route.
4. Create an API token to update DNS records: 
   - Go to https://dash.cloudflare.com/profile/api-tokens.
   - Click "Create token", select "Create Custom Token".
   - Choose **Zone:DNS:Edit** for permissions, and include your zone under "Zone Resources". 
   - Copy your API Key for later use in UniFi OS Controller configuration.

#### Install With Wrangler CLI

1. Clone or download this project.
2. Ensure you have [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/install-and-update/) installed.
3. Log in with Wrangler and run `wrangler deploy`.
4. Note the `\*.workers.dev` route after creation.
5. Create an API token as described above.

### Configuring UniFi OS

1. Log in to your [UniFi OS Controller](https://unifi.ui.com/).
2. Navigate to Settings > Internet > WAN and scroll down to **Dynamic DNS**.
3. Click **Create New Dynamic DNS** and provide:
   - `Service`: Choose `custom` or `dyndns`.
   - `Hostname`: Full subdomain and hostname to update (e.g., `subdomain.mydomain.com` or `mydomain.com` for root domain).
   - `Username`: Domain name containing the record (e.g., `mydomain.com`).
   - `Password`: Cloudflare API Token.
   - `Server`: Cloudflare Worker route `<worker-name>.<worker-subdomain>.workers.dev/update?ip=%i&hostname=%h`.
     - For older UniFi devices, omit the URL path.
     - Remove `https://` from the URL.

#### Testing Changes - UDM-Pro
To test the configuration and force an update on a UDM-Pro:

1. SSH into your UniFi device.
2. Run `ps aux | grep inadyn`.
3. Note the configuration file path.
4. Run `inadyn -n -1 --force -f <config-path>` (e.g., `inadyn -n -1 --force -f /run/ddns-eth4-inadyn.conf`).
5. Check `/var/log/messages` for related error messages.

#### Testing Changes - USG
To test the configuration and force an update on a USG:

1. SSH into your USG device.
2. Run `ls /run/ddclient/` (e.g.: `/run/ddclient/ddclient_eth0.pid`)
3. Note the pid file path as this will tell you what configuration to use. (e.g.: `ddclient_eth0`)
4. Run `sudo ddclient -daemon=0 -verbose -noquiet -debug -file /etc/ddclient/<config>.conf` (e.g., `sudo ddclient -daemon=0 -verbose -noquiet -debug -file /etc/ddclient/ddclient_eth0.conf`).
5. This should output `SUCCESS` when the DNS record is set.

#### Important Notes!

- For subdomains (`sub.example.com`), create an A record manually in Cloudflare dashboard first.
- If you encounter a hostname resolution error (`inadyn[2173778]: Failed resolving hostname https: Name or service not known`), remove `https://` from the `Server` field.

#### Complementary Notes ~ by 51av0sh
I ran into a few issues and took me about a week to troubleshoot and fix them all. Since documentation is limited I wanted to offer my experience in case someone has similar problems.

1. Starting with installation. I was getting an error when running the Deploy Workers script. I found out that the GHA was not running from the forked repo. To force the run all I did was edit he deploy.yml file (added a space and then removed it). That triggered the workflow and it installed the worker in CF.

2. I have a UDM SE and had to remove the https:// from the server field as it was not working

3. I have a wildcard DNS entry and a root domain entry. To update both at he same time you can add a list of comma separated domains to the Host field like domain.com,*.domain.com. Make sure you have no spaces

4. Some said that dyndns is missing from the UDM but it's there for me in my UDM SE running network application 8.0.28. According to some, if you are missing dyndns you can use dyn. Have not confirmed this.

5. If you want to test this script you have a few options
   i. Ask your ISP for a new IP.
   ii. Change the modem mac. You might need to contact your ISP to register the modem again.
   iii. Use inadyn (easiest but needs SSH). See here
   
7. Proxied DNS works fine with his
