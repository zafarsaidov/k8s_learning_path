
cURL - client URL

```bash
curl -x POST -H "Hi: ddd" localhost:8080 -vvv
```

Take packets info coming to localhost:8080 tcp
```bash
sudo tcpdump -i lo0 tcp port 8080 -vvv
```

Show gateway
```bash
netstat -nr
```

arp -a

sudo tcpdump -i en0 arp -vvv

strace

ls -lah /proc/<server proc>/fd

## Добавить новый bridge-интерфейс с именем Ьг0

ip link add Ьг0 type bridge

## Подключить eth0 к нашему мосту.

ip link set eth0 master br0

## Подключить veth к нашему мосту. 

ip link set veth master br0

brctl - bridges



# ip netns add net1
# ip netns add net2
# ip link add veth1 netns net1 type veth peer name veth2 netns net2

netfilter.org

