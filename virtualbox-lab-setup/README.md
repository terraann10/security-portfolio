# Building a Pentest Lab on VirtualBox (Windows Host)

## Background

After getting Kali running on my M1 MacBook via UTM, I wanted a second lab setup on my work Windows machine using VirtualBox instead. Same general idea, different hypervisor, and this time with a target machine as well, not just an attack box.

Goal for this round: a Kali attack VM and a Debian 13 (Trixie) VM to later host OWASP Juice Shop via Docker, both isolated from the work network but still able to reach each other and the internet.

## Tools

- **Hypervisor:** VirtualBox
- **Attack VM:** Kali Linux
- **Target VM:** Debian 13 (Trixie), GUI edition

## Networking decision

Before building anything, I thought through the networking setup, since this was on a work machine and I wanted to avoid bridging the VMs onto the actual work network.

Options considered:
- **Bridged** - ruled out immediately, puts VMs directly on the work LAN
- **Host-only** - fully isolated, but no internet access for the VMs on its own
- **NAT** - internet access, but VMs are isolated from each other by default
- **NAT Network** - internet access *and* VMs on the same NAT Network can see each other

Since I needed both internet access (to install Docker and updates) and VM to VM connectivity (Kali attacking Debian), NAT Network was the right fit.

## Bug 1: NAT Network with no name

After setting a VM's network adapter to "Attached to: NAT Network," the "Name" dropdown stayed on "Not selected," and VirtualBox threw:

> Network: Adapter 1 page: No NAT network name is currently specified.

The issue was that a NAT Network hadn't actually been created yet, just selecting the "NAT Network" attachment type doesn't create one automatically.

**Fix:** In the main VirtualBox Manager window (not the individual VM settings), go to File > Tools > Network Manager, open the NAT Networks tab, and create a new network there. Once created, it showed up as a selectable option in the VM's Network settings dropdown, and the name populated automatically once selected.

Both the Kali and Debian VMs were then pointed at this same NAT Network (named `labnet`).

## Updating both VMs

Before installing anything further, updated both VMs to make sure they were building on current packages:

Debian:
```
sudo apt update && sudo apt upgrade -y
```

Kali:
```
sudo apt update && sudo apt full-upgrade -y
```

(Kali uses `full-upgrade` rather than a plain `upgrade`, which handles its rolling release package changes better.)

## Confirming connectivity

With both VMs updated and on `labnet`, checked each VM's IP with:

```
ip a
```

Results:
- Debian: `10.0.2.3`
- Kali: `10.0.2.15`

Both on the same `10.0.2.0/24` subnet, as expected for a NAT Network.

Tested connectivity in both directions:

```
ping -c 4 10.0.2.3    # from Kali to Debian
ping -c 4 10.0.2.15   # from Debian to Kali
```

Both succeeded, confirming the two VMs can reach each other while still isolated from the work network.

## Bug 2: NAT Network gateway stopped responding

Rebuilt both VMs from scratch later on and hit a much bigger issue with the same NAT Network approach. After a few rounds of reattaching VMs to `labnet`, the network's gateway (`10.0.2.1`) stopped responding to ARP requests entirely, for both VMs. They could still see each other directly on the subnet, but neither could reach the gateway, internet, or resolve DNS.

Ruled out in order:
- Guest-side DNS config (`resolv.conf` pointed at a stale nameserver, fixed, still no connectivity)
- - NetworkManager connection state (cycled the connection, no change)
  - - Routing tables (default route was correctly configured, so not a routing issue)
    - - Full host restart (cleared nothing, same failure after reboot)
      - - Recreating the NAT Network object itself under a new name (same result)
        -
        - The fault sat with the NAT Network's underlying virtual router process, which got into a bad state that a straightforward recreation didn't reliably clear.
        -
        - Fix: moved away from a single shared NAT Network entirely. Each VM now runs two adapters instead of one:
        - - Adapter 1: Internal Network (`intlab`), private VM-to-VM traffic only, static IPs assigned manually since Internal Network has no DHCP (Kali: `192.168.50.10`, Debian: `192.168.50.11`)
          - - Adapter 2: plain NAT, independent internet access per VM, no shared router between them
            -
            - This removes the single point of failure that kept breaking, and gives cleaner separation between internal (VM-to-VM) and external (internet-bound) traffic as a side benefit.
            -
            - ## Docker & Juice Shop setup
            -
            - With the new networking in place, installed Docker on the Debian VM via apt:
            -
            - ```
              sudo apt update
              sudo apt install -y docker.io
              sudo systemctl enable --now docker
              ```

              Added my user to the docker group to avoid needing sudo for every command:

              ```
              sudo usermod -aG docker $USER
              ```

              (Needed `newgrp docker` to apply the group change in the current session, rather than logging out and back in.)

              Pulled and ran OWASP Juice Shop:

              ```
              docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
              ```

              Confirmed it was running:

              ```
              docker ps
              ```

              Then confirmed it was reachable from Kali over the Internal Network, at `http://192.168.50.11:3000`, loaded cleanly in Kali's browser.

              Lab is now fully working end to end: Kali and Debian isolated from the work network but able to reach each other, Docker running on Debian, and Juice Shop live as a target to start practicing OWASP Top 10 vulnerabilities against.

              ## Next steps

- Start working through Juice Shop challenges from Kali
- Document findings and writeups as I go

---

*Part of my [security-portfolio](../) repo, documenting hands-on lab setup and the bugs along the way.*
