[Main](../README.md)
---

# Namespaces and cgroups

## Creating a Namespace

The manual page indicates that it does exactly what we want:

```bash
unshare – run program in new name namespaces
```

I’m currently logged in as a regular user, svk, which has its own user ID, group, and so on, but not root privileges:

```bash
id

output:
uid=1000(svk) gid=1000(svk) groups=1000(svk) 
```

Now I run the following unshare command to create a new namespace with its own user and PID namespaces. I map the root user to the new namespace (in other words, I have root privilege within the new namespace), mount a new proc filesystem, and fork my process (in this case, bash) in the newly created namespace.

```bash
unshare --user --pid --map-root-user --mount-proc --fork bash
```

(For those of you familiar with containers, this accomplishes the same thing as issuing the `<runtime> exec -it </image> /bin/bash command` in a running container.)

The `ps -ef` command shows there are two processes running – `bash` and the `ps` command itself – and the `id` command confirms that I’m root in the new namespace (which is also indicated by the changed command prompt):

```bash
ps -ef
UID PID PPID C STIME TTY TIME CMD
root 1 0 0 14:46 pts/0 00:00:00 bash
root 15 1 0 14:46 pts/0 00:00:00 ps -ef

id
uid=0(root) gid=0(root) groups=0(root) 
```

The crucial thing to notice is that I can see only the two processes in my namespace, not any other processes running on the system. I am completely isolated within my own namespace.

## Example for Network Namespace

🧠 Create and Manage Network Namespaces
```bash
sudo ip netns add ns1
```
➡️ Creates a new network namespace named ns1. This provides a separate networking stack (interfaces, routes, iptables, etc.) from the main system.
```bash
sudo ip netns add ns2
```
➡️ Same as above, but creates a second isolated network namespace ns2.
```bash
sudo ip netns show
```
➡️ Lists all current network namespaces created via ip netns.
```bash
sudo ip netns del ns1
sudo ip netns del ns2
```
➡️ Deletes the ns1 and ns2 namespaces, cleaning up isolated network contexts.

⸻

🔌 Create Virtual Ethernet Pairs
```bash
sudo ip link add veth10 type veth peer name veth11
```
➡️ Creates a pair of virtual Ethernet interfaces (veth10 and veth11) connected like a virtual cable—packets sent on one end are received on the other.
```bash
sudo ip link add veth20 type veth peer name veth21
```
➡️ Another virtual cable: veth20 and veth21. These will be used for ns2.
```bash
sudo ip link show type veth
```
➡️ Lists all veth interfaces on the host.
```bash
sudo ip link del veth10
```
➡️ Deletes the veth10 interface and its peer veth11. Useful for cleanup or resetting.

⸻

🌐 Assign veth Peers to Namespaces
```bash
sudo ip link set veth11 netns ns1
```
➡️ Moves the veth11 end of the virtual Ethernet pair into the ns1 namespace.
```bash
sudo ip link set veth21 netns ns2
```
➡️ Moves veth21 into ns2. Now each namespace has one interface of the pair.

⸻

🔍 Check Network Interfaces Inside Namespaces
```bash
sudo ip netns exec ns1 ifconfig
```
➡️ Runs ifconfig inside ns1 to list interfaces. Only loopback and veth11 should appear.
```bash
sudo ip netns exec ns2 ifconfig
```
➡️ Same as above but for ns2.

⸻

🏗️ Assign IP Addresses to Namespace Interfaces
```bash
sudo ip netns exec ns1 ip addr add 172.16.0.2/24 dev veth11
```
➡️ Assigns a private IP to veth11 inside ns1.
```bash
sudo ip netns exec ns2 ip addr add 172.16.0.3/24 dev veth21
```
➡️ Same for veth21 inside ns2.

⸻

🚦 Bring Interfaces Up
```bash
sudo ip netns exec ns1 ip link set dev veth11 up
sudo ip netns exec ns2 ip link set dev veth21 up
```
➡️ Activates interfaces in both namespaces so they can transmit traffic.

⸻

🕸️ Create and Use a Bridge
```bash
sudo ip link add br0 type bridge
```
➡️ Creates a virtual Ethernet bridge named br0, acting like a virtual switch.
```bash
sudo ip link show type bridge
```
➡️ Shows any bridges currently defined.
```bash
sudo ip link set dev veth10 master br0
sudo ip link set dev veth20 master br0
```
➡️ Connects veth10 and veth20 to the br0 bridge—these are on the host side and act like switch ports.
```bash
sudo ip addr add 172.16.0.1/24 dev br0
```
➡️ Assigns a gateway IP to the bridge. Acts like a default gateway for the namespaces.
```bash
sudo ip link set dev br0 up
```
➡️ Activates the bridge interface.
```bash
sudo ip link set dev veth10 up
sudo ip link set dev veth20 up
```
➡️ Enables the host-side ends of the virtual Ethernet pairs.

⸻

🔎 Inspect Links and Set Loopback Up
```bash
sudo ip netns exec ns1 ip link
```
➡️ Lists all interfaces in ns1, useful to confirm veth11 and lo are present.
```bash
sudo ip netns exec ns1 ip link set lo up
sudo ip netns exec ns2 ip link set lo up
```
➡️ Activates the loopback interfaces, required for internal communications within the namespace.

⸻

🧾 View IPs and Add Routes
```bash
sudo ip netns exec ns1 ip addr
```
➡️ View all IP addresses assigned inside ns1.
```bash
sudo ip netns exec ns1 ip route add default via 172.16.0.1 dev veth11
sudo ip netns exec ns2 ip route add default via 172.16.0.1 dev veth21
```
➡️ Adds default routes for both namespaces pointing to the bridge as the gateway. This enables traffic to leave the namespace.
```bash
sudo ip netns exec ns1 ip route show
sudo ip netns exec ns2 ip route show
```
➡️ Confirms the routing tables in each namespace.

⸻

🔁 Enable Packet Forwarding on Host
```bash
sudo sysctl -w net.ipv4.ip_forward=1 
```
➡️ Enables IPv4 forwarding so the host can route packets between namespaces.

⸻

🧪 Ping and Test External Routing
```bash
sudo ip netns exec ns1 ping -W 1 -c 2 x.x.x.x
```
➡️ Pings an external IP from inside ns1. Replace x.x.x.x with an actual reachable IP (like 8.8.8.8).
```bash
sudo ip route get x.x.x.x
```
➡️ Shows which route and interface would be used to reach the destination from the host.


[Main](../README.md)
