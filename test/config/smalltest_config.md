# Small Test
## Solace PubSub+ Broker 
### AWS Host
m5a.4xlarge ca-central-1b
### Version
9.2.0.14 Evel
### Tuning
200K scaling tier

3 EIP

## Test hosts
### AWS Host
4 x m5a.xlarge ca-central-1a
### Version
4.14.123-111.109.amzn2.x86_64 #1 SMP Mon Jun 10 19:37:57 UTC 2019

 sudo sysctl net.ipv4.ip_local_port_range

 sudo ./connectChurn --cip=mqtt:172.31.11.210 --contexts=16 --sessions=50000 --start=1100 --mr=200 --qos=0 --cleansession=1 --ka=50 --logintimeout=120
 ./sdkperf_mqtt_dl -cip=172.31.8.41 -cc=5 -ptl=test/topic1,test/topic2,test/topic3,test/topic4,test/topic5  -stl=test/topic1,test/topic2,test/topic3,test/topic4,test/topic5 -msa=1024 -mr=1000000 -mn=100000000


 ## NGiNX Plus
### AWS Host
m5a.4xlarge ca-central-1b
### Version
1.15.10 , build:nginx-plus-r18-p1

### Tuning

seed keyval datastore:
curl -X POST http://localhost:80/api/4/stream/keyvals/mqtt_seed -d '{"default":"172.31.8.41:1883 172.31.4.117:1883 172.31.13.100:1883 172.31.11.149:1883"}'

See config files +

ulimit -n
200000

sysctl -p
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = "1025 65534"

ulimit -s  256
ulimit -i  120000
echo 120000 > /proc/sys/kernel/threads-max
echo 600000 > /proc/sys/vm/max_map_count
echo 200000 > /proc/sys/kernel/pid_max
 
##Raise the ability to handle more ports and FDs.  Current perf-hosts can handle 28K client connections, we can bump this to 65K per source address
/# Ref: https://mrotaru.wordpress.com/2013/10/10/scaling-to-12-million-concurrent-connections-how-migratorydata-did-it/
sysctl -w net.ipv4.ip_local_port_range="1024   65535"
echo 3000000 > /proc/sys/fs/nr_open
ulimit -n 2000000


sysctl -w net.netfilter.nf_conntrack_max=262144
echo 65536 > /sys/module/nf_conntrack/parameters/hashsize
echo 200000 > /proc/sys/fs/file-max

 
/##Add secondary addresses to spread client connections across source addresses.  Spread sdkperf across source address to get to 100K
/#Ref: https://www.garron.me/en/linux/add-secondary-ip-linux.html
ip address add 192.168.190.92/19 dev eth1
 
/##Tune system for through-put not latency:
tuned profile throughput-performance


Up from 4K by editing :
/etc/security/limits.d/20-nproc.conf


Problem was within nginx itself, not linux.  Fix:
worker_rlimit_nofile 150000;

Local ports
https://ma.ttias.be/nginx-cannot-assign-requested-address-for-upstream/


https://gist.github.com/denji/8359866

https://stackoverflow.com/questions/3191509/nginx-error-99-cannot-assign-requested-address


Stall at 28K clients:
https://gryzli.info/2018/03/05/nginx-cannot-assign-requested-address/

Edit /etc/sysctl.conf  and add:
net.ipv4.tcp_tw_reuse=1
then issue:

1	sysctl -p 

From <https://gryzli.info/2018/03/05/nginx-cannot-assign-requested-address/> 
 sudo sysctl -w net.ipv4.ip_local_port_range="1024 65535"
