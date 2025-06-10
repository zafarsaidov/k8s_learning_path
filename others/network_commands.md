
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
