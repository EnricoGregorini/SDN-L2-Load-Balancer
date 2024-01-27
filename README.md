# SDN - OpenFlow load balancing project for course on Network Automation

## Authors

- [Enrico Gregorini](https://gitlab.com/Enrico.Gregorini)
- [Daniele Gjeka](https://gitlab.com/danielegjeka)
- [Mattia Bevilacqua](https://gitlab.com/Mattiabe98)

## Dependencies
Install `pandas` and `matplotlib` dependencies:
```
pip3 install pandas matplotlib
```

Install iPerf3 and D-ITG tools:
```
apt install iperf3 d-itg
```

## Get started
Run the load balancer app and the underlying mininet topology:

```
sudo -v && ryu-manager --observe-links flowmanager/flowmanager.py load-balancer-apps/load_balancer2.py
sudo mn --custom mininet-topologies/topology_2.py --mac --topo mytopo --controller=remote,ip=127.0.0.1,port=6633 --switch ovs,protocols=OpenFlow13

```
If you're encountering weird errors on startup, try restarting OpenvSwitch: `sudo systemctl restart ovs-vswitchd`

## How to use iPerf3 benchmark scripts

Run ```./iperf_server.sh``` with the name of the hosts where you want to start an iPerf3 server:

Example running iPerf server on h1 and h2:
```
sudo ./Scripts/iperf_server.sh h1 h2
```

then run ```./Scripts/iperf_client.sh``` with the following syntax:

```
sudo ./iperf_client.sh timeout number_of_streams h3 1 h4 2
```

Example:
```
sudo ./iperf_client.sh 30 2 h3 1 h4 2
```

Where h3 is the host on which you wish to run the client and '1' is the last digit of the server IP.

In this example, the iPerf server is running on h1 and h2, while the client is running on h3 (connecting to _10.0.0.1_, which is _h1_), and h4 (connecting to _10.0.0.2_, which is _h2_).
_number_of_streams_, in this case, would be 2 (h3 -> 10.0.0.1 and h4 -> 10.0.0.2).

## How to use D-ITG benchmark


Run the receiver on a chosen Mininet host (in this case host 2):
```
/home/ubuntu/tools/mininet/util/m h2 ITGRecv
```
Run the sender on a chosen Mininet host (in this case host 7):
```
/home/ubuntu/tools/mininet/util/m h7 ITGSend -a 10.0.0.2 -C 80000 -c 1000 -t 20000 -l ITGSend.log
```
See full documentation to configure ITGSend as you wish (UDP, TCP, emulate Telnet/DNS, multiple/single flows, etc..): https://traffic.comics.unina.it/software/ITG/manual/index.html#SECTION00042000000000000000

Run ``ITGDec`` to decode the output (either on sender or receiver).

For example, on sender:
```
/home/ubuntu/tools/mininet/util/m h7 ITGDec ITGSend.log
```

Sample output:
```
****************  TOTAL RESULTS   ******************
__________________________________________________________
Number of flows          =             1
Total time               =      1.999493 s
Total packets            =          1448
Minimum delay            =      0.000000 s
Maximum delay            =      0.000000 s
Average delay            =      0.000000 s
Average jitter           =      0.000000 s
Delay standard deviation =      0.000000 s
Bytes received           =        724000
Average bitrate          =   2896.734322 Kbit/s
Average packet rate      =    724.183581 pkt/s
Packets dropped          =             0 (0.00 %)
Average loss-burst size  =             0 pkt
Error lines              =             0
```

## Script for D-ITG
The `itg_recv.sh` script instatiates multiple receivers listening to traffic generated by senders. In the following example 3 receivers are started in host 1-2-3:
```
sudo ./Scripts/itg_recv.sh h1 h2 h3 
```
The `itg_send.sh` script instantiates multiple senders to generate traffic to the receivers. It is executed inside the controller. In the following example, 3 senders are initiated in hosts 6-7-8 and generate 200000 packets per second with a size of 1500 Byte for 60000 ms (60 seconds):
```
sudo ./Scripts/itg_send.sh 200000 1500 60000 3 h8 1 h7 2 h6 3 
```
The `itg_send.sh` script has the same structure of `iperf_client.sh` with additional parameters referring to packet rate, packet size and itg connection timeout.


## Load balancer versions 

The applications are stored in the folder '/load-balancer-apps', which contains 3 different versions of load balancer: 

- **load-balancer0.py**
    - Static version of load balancer which uses the Dijkstra's algorithm to calculate the shortest path from source to destination hosts.

- **load-balancer1.py**
    - Dynamic version of load balancer that modifies the current paths to load the traffic. It also includes the creation of plots to visualize the data stored.

- **load-balancer2.py**
    - It embeds the `iperf` scripts so that the simulation is more automated. The traffic is generated automatically by the controller using the `iperf` scripts described above.

- **load-balancer3.py**
    - It embeds the `itg` scripts so that the simulation is more automated. The traffic is generated automatically by the controller using the `itg` scripts described above.

## Plot generation 
Once all the data has been collected, the script `'create_chart.py'` plots and stores the results in the folder '/Plots'. 
To execute the script:
```
python3 create_chart.py x traffic_type
```
where `x` indicates the topology that used to create the plot and `traffic_type` can be either `itg` or `iperf`, to distinguish the measurements and plots obtained from the simulations performed with D-ITG or iPerf3 respectively.

Depending on the traffic, when the load balancer is activated, the generated graph will show the good behaviour of the load balancer. Comparing this graph to a simulation without load balancing clearly shows the performance improvements of using a load balancer.

