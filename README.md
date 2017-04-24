# nat-testbench
-- a Vagrant configuration for testing STUN and NAT

How do you test your NAT traversal methods? To my surprise, I could not find anything useful on the internet, so here is
my take on it.

This repository contains a Vagrantfile that sets up a STUN server and simulates several clients behind the most common NAT settings (via `iptables`). To use it:

    vagrant up

(note: this will need roughly 9GB of storage)

This starts the following VMs:

| name | public ip | private ip | comment |
| ---- | --------- | ---------- | ------- |
| stun | 192.168.42.10,192.168.42.20 | - | STUN server, running `stund` |
| fullcone | 192.168.42.100 | 10.10.44.2 | full cone NAT |
| restrictedcone | 192.168.42.110 | 10.10.45.2 | restricted cone NAT |
| portrestrictedcone | 192.168.42.120 | 10.10.46.2 | port restricted cone NAT |
| symmetric | 192.168.42.130 | 10.10.47.2 | symmetric NAT |

Each NAT'ted machine has host entries for:
- `my-lan-ip`: points to the "private" ip behind the NAT
- `my-public-ip`: points to the "public" ip of the NAT
- `stun.server.local`: points to the STUN server
- all other NAT hosts by their name (e.g. `fullcone`)

Per default, all VMs have [`pystun`](https://github.com/jtriley/pystun) installed. To test the different NAT types, you
can do

    vagrant ssh <box name>
    pystun -H stun.server.local -i my-lan-ip

where `<box name>` is one of `fullcone`, `restrictedcone`, `portrestrictedcone`, `symmetric`.

If you want to make adjustments to the installed packages (i.e. install different STUN clients, test your own traversal
methods, ...), find the `GENERAL PROVISIONING SECTION` in the `Vagrantfile`.
