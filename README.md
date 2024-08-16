
# 05 - Firewalls and intrusion detection

PSI - Pós-Graduação Em Segurança Informática
MÓDULO 1 - Criptografia e Fundamentos de Segurança


## Lessons Learned

#### IPTABLES Manual
```bash
  man iptables
```

#### List all rules and rules numbers
```bash
  iptables -L -nv --line-numbers
```
#### DROP all rules
```bash
  iptables -F
```

#### DROP  rules by number
```bash
  iptables -t filter -D INPUT 1
```
### IPTABLES LOGs
- See firewall logs
```bash
  $ dmesg | more
  
  $ dmesg | grep -w 'DPT=22'
```
- Reules for LOGs
```bash
  $ iptables -A INPUT -j LOG
  
  $ tail -f /var/log/kern.log
  
  ### see logs size on disc
  $ du -chs /var/log 
```
- LOGs With Limits:
```bash
$ iptables -A INPUT -p tcp --dport 80 -m limit --limit 5/m --limit-burst 7 -j LOG --log-prefix "HTTP_TCP_80_LOG: "
$ iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

#### Others links:
- [Make iptables persistent after reboot on Linux](https://linuxconfig.org/how-to-make-iptables-rules-persistent-after-reboot-on-linux) OR [follow this URL 👍](https://www.cyberciti.biz/faq/how-to-save-iptables-firewall-rules-permanently-on-linux/)
- [Iptables Essentials: Common Firewall Rules and Commands](https://www.digitalocean.com/community/tutorials/iptables-essentials-common-firewall-rules-and-commands)
## EXECISES

### 1. Clear all the rules on the system’s firewall configuration

```bash
$ sudo iptables -F
```

### 2. Create a firewall rule to ignore incoming ping requests 
Create a firewall rule to ignore incoming ping requests from hosts on the network 10.1.0.0/24 (network of the server student.dei.uc.pt), 
while authorizing all the remaining IP packets. 

**Note:** ping uses ICMP packets of types **8 (echo request)** and **0 (echo reply)**

```bash
$ iptables -A INPUT -p icmp --icmp-type 8 -s 10.1.0.0/24 -j DROP

$ iptables -R INPUT 4 -p icmp --icmp-type 8 -s 0/0 -j ACCEPT
```
##### More Syntax for exercise 2.:
```
$ iptables -A {INPUT|OUTPUT} -p icmp -j {ACCEPT|REJECT|DROP}
$ iptables -A {INPUT|OUTPUT} -p icmp --icmp-type {0|8}  -j {ACCEPT|REJECT|DROP}
$ iptables -A {INPUT|OUTPUT} -p icmp --icmp-type {echo-reply|echo-request} -j {ACCEPT|REJECT|DROP}
$ iptables -A {INPUT|OUTPUT} -p icmp --icmp-type {echo-reply|echo-request} -m state --state NEW,ESTABLISHED,RELATED -j {ACCEPT|REJECT|DROP}
```
[Linux Iptables allow or block ICMP ping request](https://www.cyberciti.biz/tips/linux-iptables-9-allow-icmp-ping.html)
 ```bash
$ iptables -t filter -A INPUT -p tcp --dport ssh -s 192.168.1.0/24,student.dei.uc.pt -j REJECT
```
### 3. Create firewall rules to authorize the following incoming TCP connections (filter table, INPUT chain), while rejecting (only) other TCP communications:

- **a.** SSH and SMTP connections originated at the server student.dei.uc.pt and from network 192.168.1.0/24
```bash
$ iptables -t filter -A INPUT -p tcp -m multiport --dports ssh,smtp -s 192.168.1.0/24,student.dei.uc.pt -j ACCEPT
```

- **b.** POP3 and IMAP4 connections originated at other hosts on the Lab (network 10.254.0.0/24)
```bash
$ iptables -t filter -A INPUT -p tcp -m multiport --dports pop3,imap -s 10.254.0.0/24 -j ACCEPT
```

**REJECT all others hosts:** 
```bash
$ iptables -t filter -A INPUT -p tcp  -m multiport --dport ssh,smtp -j REJECT
```

### 4. Add to the previous configuration firewall rules to authorize the following outgoing TCP connections (filter table, OUTPUT chain), while rejecting (only) other TCP communications:
- **a.** HTTP and HTTPS connections destined to the server student.dei.uc.pt
```bash
$ iptables -t filter -A OUTPUT -p tcp -m multiport --dports http,https -d student.dei.uc.pt -j ACCEPT
```
- **b.** SSH connections destined to other hosts on the Lab (network 10.254.0.0/24)
```bash
$ iptables -t filter -A OUTPUT -p tcp --dport ssh -d 10.254.0.0/24 -j ACCEPT
```

**REJECT all others hosts:** 
```bash
$ iptables -t filter -A OUTPUT -p tcp  -m multiport --dports http,https -j REJECT
```

### 5. Save the rules to a file
```bash
$ iptables-save > /etc/iptables/rules.v4
```
### 6. Clear all the firewall rules defined in the previous exercises
```bash
$ iptables -F
```

### 7. Use IPTables to authorize the following communications, while denying the remaining IP traffic (policy DROP on both the INPUT and OUTPUT chains):

Denying the remaining IP traffic:
```bash
$ iptables --policy INPUT DROP
$ iptables -P OUTPUT DROP 
```

- **a.** Incoming SSH and HTTP connections
```bash
$ iptables -A INPUT -p tcp -m multiport --dports http,ssh -j ACCEPT 
```

- **b.** Outgoing SSH, HTTP and HTTPS connections
```bash
$ iptables -A OUTPUT -p tcp -m multiport --sports https,http,ssh -j ACCEPT
```

- **c.** DNS queries sent to the server dns.dei.uc.pt and dns2.dei.uc.pt
```bash
$ iptables -A OUTPUT -i eth0 -d dns.dei.uc.pt,dns2.dei.uc.pt -p udp --dport 53 -j ACCEPT

$ iptables -A OUTPUT -i eth0 -d dns.dei.uc.pt,dns2.dei.uc.pt -p tcp --dport 53 -j ACCEPT
```

- **d.** Incoming ping requests from the server student.dei.uc.pt
```bash
$ iptables -A INPUT -p icmp --icmp-type 8 -s student.dei.uc.pt -j ACCEPT

$ iptables -A OUTPUT -p icmp --icmp-type 0 -d student.dei.uc.pt -j ACCEPT
```

- **e.** All IP communications to or from the localhost (127.0.0.1, or interface lo)
```bash
$ iptables -A INPUT -s 127.0.0.1 -j ACCEPT

$ iptables -A OUTPUT -d 127.0.0.1 -j ACCEPT
```

### 8. Activate the previous firewall configuration permanently on the system
[Make iptables persistent after reboot on Linux](https://linuxconfig.org/how-to-make-iptables-rules-persistent-after-reboot-on-linux) OR [follow this URL 👍](https://www.cyberciti.biz/faq/how-to-save-iptables-firewall-rules-permanently-on-linux/)

### 9. Add a rule to log all the pings requests that are performed to the virtual machine.
- **a.** Check the number of packets that have denied and logged
```bash
iptables -A INPUT -p icmp --icmp-type 8 -j LOG
```
- **b.** After the verification of the logging eliminate the rule
```bash
$ iptables -D INPUT 4
```

### 10. Use IPTables to limit the number of connections to the virtual machine, to only accept two simultaneous incoming SSH connections. Consider replacing rule of exercise 7a.
```bash
$ iptables  -R INPUT 5 -p tcp --syn --dport ssh -m connlimit --connlimit-above 2 -j REJECT
```


# For Docker network Helper

### Eliminar regra que rejeita toda entrada para cham DOCKER-INGRESS, definir esta regra em ultimo lugar
```bash
sudo iptables -t filter -D DOCKER-INGRESS -j RETURN
```

### aceitar chamada para porta 8888, somente a partir de container docker
```bash
sudo iptables -A DOCKER-INGRESS -p tcp -m tcp -i docker_gwbridge --dport 8888 -j ACCEPT
```

### aceitar chamada de exterior para porta 8888
```bash
sudo iptables -A DOCKER-INGRESS -p tcp -m tcp --dport 8888 -j ACCEPT
```
```bash
sudo iptables -A DOCKER-INGRESS -p tcp -m tcp --sport 8888 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

### rejeitar todas outrs chamadas a partir desta regra
```bash
sudo iptables -A DOCKER-INGRESS -j RETURN
```

### LOGs para monitorr regra na porta 8888
```bash
sudo iptables -A DOCKER-INGRESS -p tcp -m tcp --dport 8888 -j LOG --log-prefix "HTTP_TCP_8888_LOG: "
```

### Save and Restore rules saved infile
```bash
sudo iptables-save > /opt/iptables.v4.1
```
```bash
sudo iptables-restore < /opt/iptables.v4.1
```

