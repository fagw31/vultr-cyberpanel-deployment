# Vultr CyberPanel setup: which plan to buy, where the admin password is, and which ports to open (one-click image vs manual installer)

Pick the CyberPanel template under **Marketplace Apps** while you deploy a Vultr instance. The server reaches *Running* in about two minutes, the panel answers on `https://your-ip:8090`, the username is `admin`, and the password comes from `cat .litespeed_password` over SSH. That is the entire installation.

The parts that trip people up are elsewhere: how big an instance the panel actually needs, which firewall rules matter before the first login works, and which instructions in older Vultr CyberPanel tutorials you should ignore outright.

> **Quick pick.** One 1 GB High Frequency or High Performance instance at **$6.00/mo** handles the panel and a single low-traffic WordPress site. Move to 2 GB when mail, DNS, and Docker all run on the same box. CyberPanel and OpenLiteSpeed cost nothing, so the Vultr instance is your only real expense.
>
> 👉 [Check current Vultr plans and hourly rates](https://www.vultr.com/pricing/?ref=9738262-9J)

## What Vultr charges for the machine under CyberPanel

Vultr splits compute into a handful of families, and only the first four matter for a control panel. Prices below are the entry tier of each family, billed hourly with a monthly cap.

| Family | Entry price | What the cheapest tier gives you | Billing | Plans |
| --- | --- | --- | --- | --- |
| Cloud Compute – Regular Performance | $2.50/mo | 1 vCPU, 0.5 GB RAM, 10 GB SSD. The bottom tier is **IPv6-only** | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Cloud Compute – High Performance | $6.00/mo | 1 vCPU, 1 GB RAM, NVMe, current-gen AMD or Intel | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Cloud Compute – High Frequency | $6.00/mo | 1 vCPU, 1 GB RAM, NVMe on 3 GHz+ Xeon | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Optimized – General Purpose | $30.00/mo | Dedicated vCPU; smallest plan starts at 4 GB RAM | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Optimized – CPU Optimized | $28.00/mo | Dedicated vCPU, CPU-heavy ratio | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Optimized – Memory Optimized | $40.00/mo | Dedicated vCPU, RAM-heavy ratio | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Optimized – Storage Optimized | $75.00/mo | Dedicated vCPU, NVMe-heavy ratio | Hourly, monthly cap | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |
| Cloud GPU (GH200 / H100) | $4.335/hr / $19.988/hr | 72 vCPU + 4800 GB / 224 vCPU + 32640 GB | Hourly | [View plans](https://www.vultr.com/pricing/?ref=9738262-9J) |

The GPU line is listed for completeness. Nobody runs a hosting panel on an H100.

One detail in that table decides more purchases than it should. The $2.50 Regular Performance tier ships **IPv6 only**, so `https://your-ip:8090` has no IPv4 address to resolve to unless you know your way around an IPv6-only network. If you want the familiar setup, your floor is the $6.00 High Performance or High Frequency tier, both of which include IPv4.

Within Cloud Compute, High Performance and High Frequency are priced identically at every RAM size. The step up from Regular Performance is fixed: +$1.00/mo at 1 GB, +$2.00 at 2 GB, +$4.00 at 4 GB, +$8.00 at 8 GB.

| RAM | Regular Performance | High Performance / High Frequency |
| --- | --- | --- |
| 1 GB | $5.00/mo | $6.00/mo |
| 2 GB | $10.00/mo | $12.00/mo |
| 8 GB | $40.00/mo | $48.00/mo (3 vCPU, 256 GB NVMe) |

High Frequency means a higher-clocked Xeon; High Performance means newer EPYC or Xeon with NVMe. Same price, different hardware. For a PHP panel serving a handful of sites, either is fine.

## How much server does CyberPanel actually need?

Less than the feature list suggests, until you switch features on. The manual installer runs a memory check on the box before it does anything, and a 512 MB instance clears it with room to spare. Third-party deployment write-ups put the practical minimum at 1 vCPU and 1 GB RAM, which matches what a bare panel plus OpenLiteSpeed plus MySQL consumes.

The one-click image installs more than a bare panel. It comes with PowerDNS, a mail stack, Rainloop webmail, a Docker manager, and auto SSL, and it configures the mail and DNS components during provisioning. Idle, that is still light. Under a WordPress site with LSCache, a couple of cron jobs, and an IMAP mailbox or two, 1 GB starts swapping.

A workable rule set:

- **One site, panel used for SSL and one-click WordPress:** 1 GB at $6.00/mo.
- **WordPress plus mail plus DNS on the same instance:** 2 GB at $12.00/mo. Mail is the component that eats memory and reputation at the same time.
- **Several sites, staging copies, or Docker containers:** 4 GB. High Frequency at this size is $24.00/mo.
- **Production where a noisy neighbour would be a real problem:** Optimized General Purpose from $30.00/mo, which gives you dedicated vCPU.

Disk deserves a mention because the panel is talkative. The installer wants at least 10 GB free before it starts, and that is before WordPress, uploads, mail spools, and backups. Check the storage column on the plan you pick rather than assuming the entry tier is enough.

👉 [Deploy a CyberPanel instance from the Marketplace image](https://www.vultr.com/marketplace/apps/cyberpanel?ref=9738262-9J)

## One-click image vs the manual installer script

The manual route still exists and still works. You download the installer, make it executable, run it as root, and answer a sequence of prompts. Those prompts are the reason most people should skip it.

The script asks you to choose between OpenLiteSpeed and LiteSpeed Enterprise, whether to run a full service install (PowerDNS, Postfix, Pure-FTPd), whether to add Memcached and Redis, whether to enable WatchDog, which version to pin, and whether to accept a default admin password, generate a random one, or set your own. Every one of those is a decision you can get wrong at 1 a.m. with no easy undo.

The Marketplace image has already made them. OpenLiteSpeed with LSCache is installed, the mail service and DNS are configured, WordPress, Prestashop, Joomla, and Mautic installers are ready, and HTTP/3 support is on out of the box. It also gives you a one-click conversion from OpenLiteSpeed to LiteSpeed Enterprise later, which removes the main reason to pick Enterprise during installation.

The image is also the only path with a documented password retrieval flow. Manual installs in older documentation walk you through a default password of `1234567`. Do not use it, and do not follow any guide that presents it as the normal way in. Generate a random password or set your own.

Stick with the manual script only if you need a specific component mix, you are reinstalling onto a server you already rent, or you want to pin a particular CyberPanel version. For a fresh Vultr instance, the image is faster and has fewer ways to fail.

## After deployment: the ten-minute checklist

1. **Wait for Running, then SSH in.** `ssh root@your-ip`. A welcome banner appears with the paths you need. If it does not show up, reconnect after a few minutes instead of rebuilding the server.
2. **Pull both passwords.** `cat .litespeed_password` gives you the panel admin password. `cat .db_password` gives you the MySQL password for the `cyberpanel` user. Most guides only mention the first one, and you will want the second the moment you touch a database.
3. **Open the ports.** The image enables its own firewall, and Vultr has a separate network firewall in the control panel. Both need to allow the traffic.
4. **Log in at `https://your-ip:8090`.** HTTPS is mandatory. The certificate is self-signed on first boot, so the browser will warn you; go through *Advanced* and continue. Username is `admin`.
5. **Change the password and set a real admin email** under the user menu before you add a single site.

Ports to allow, inbound and outbound:

| Port | Protocol | Service | Needed for |
| --- | --- | --- | --- |
| 8090 | TCP | CyberPanel | Always |
| 80, 443 | TCP | Web server | Always |
| 443 | UDP | HTTP/3 and QUIC | Only if you want HTTP/3 |
| 21, 40110–40210 | TCP | FTP | Only if you use FTP |
| 25, 587, 465, 110, 143, 993 | TCP | Mail | Only if you host mail |
| 53 | TCP and UDP | PowerDNS | Only if you run DNS |

The FTP row is where people lose an afternoon. Port 21 alone lets the client connect and authenticate, then every file transfer stalls because the passive data ports 40110–40210 are closed. Open the whole range or skip FTP and use the file manager and SFTP over SSH.

## What CyberPanel costs, and what the $300 credit actually covers

The panel is free, with no cap on the number of websites or domains. OpenLiteSpeed underneath it is GPLv3. Nothing in the stack charges per site, per seat, or per domain, and there is no license to renew in year two. Your bill is the Vultr instance, hour by hour, at the rates in the tables above.

That makes the arithmetic simple. A 1 GB High Frequency instance at $6.00/mo costs about $0.009 per hour, roughly $72 a year. An 8 GB instance at $48.00/mo runs about $576 a year. Vultr bills hourly with a monthly cap, so short-lived test servers cost proportionally less, but the pricing page does not spell out what happens to a stopped instance, so do not assume a powered-off server stops costing money.

The $300 credit attached to the link you arrived through is a new-account promotion. It requires a fresh email address and a card on file for verification, and the balance runs on a limited window that third-party write-ups consistently put at 30 days. Whatever is left when the window closes does not carry over.

Run the numbers against your plan before you get excited about the figure. A $6.00/mo instance burns about $6 of that $300 in a month. The credit is generous for testing several configurations in parallel, and close to irrelevant for a single small server. The binding constraint is the calendar.

👉 [Open a Vultr account and read the credit terms on the signup screen](https://bit.ly/VuLtr)

## Where CyberPanel on Vultr gets awkward

**UFW disappears.** On Ubuntu, the CyberPanel installation removes UFW to avoid conflicts with its own firewall. This is confirmed in CyberPanel's community forum, and it lines up with the banner that tells you the firewall is enabled. If you later find UFW missing or disabled, that is expected. Reinstalling it alongside the panel's firewall is how you end up with two rule sets fighting each other.

**Old tutorials describe an old product.** Vultr's own CyberPanel documentation was last updated in March 2020 and targets Ubuntu 18.04 LTS with installer v2.00. The port list and the login flow still hold. The OS recommendation and the default password do not. Check the Marketplace image page for the base system it currently ships before you assume anything.

**Mail is a trap for the unprepared.** The image configures the mail stack, which makes it tempting to start sending from a fresh IP. Deliverability from a new address with no sending history is poor, and fixing that is a weeks-long job involving SPF, DKIM, DMARC, and warm-up. If your site sends a contact form and a password reset, an external SMTP relay will save you a lot of grief.

**Not everyone needs this panel.** A static site, a single Next.js app, or anything you would deploy from a Dockerfile does not benefit from PowerDNS, Postfix, Rainloop, and a file manager sitting on the same box. Those components expand the attack surface and the patch list. If you are not going to use mail, DNS, and FTP, a plain Ubuntu instance with a web server is a smaller thing to maintain.

**It is not cPanel.** If your workflow depends on cPanel-specific extensions, a host that supports cPanel, or an agency that manages cPanel accounts, CyberPanel will not slot in. The feature overlap is real, the ecosystem is not the same.

## Which instance to order

One low-traffic site and you want a browser UI for SSL, WordPress installs, and backups: 1 GB High Frequency at $6.00/mo is the right purchase, and the extra dollar over Regular Performance buys you IPv4 and NVMe.

One box doing WordPress, mail, and DNS for a handful of small sites: 2 GB at $12.00/mo, and expect to move mail off it eventually.

Multiple sites with staging copies or containers: 4 GB High Frequency at $24.00/mo.

Anything where other tenants' CPU usage would show up in your response times: Optimized General Purpose from $30.00/mo.

Two things to confirm on the order screen before you click deploy. First, that the CyberPanel image is available in the region you picked; Marketplace availability varies by location, and you cannot change region after deployment. Second, that the tier you selected includes IPv4, because the cheapest Regular Performance plan does not.

👉 [Start with the CyberPanel one-click image on Vultr](https://bit.ly/VuLtr)
