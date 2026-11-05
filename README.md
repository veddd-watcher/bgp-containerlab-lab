# eBGP Between Two Autonomous Systems (Containerlab + FRR)

Two FRR routers in different autonomous systems peer over eBGP and exchange loopback prefixes.

**Tools:** Containerlab 0.81.1, FRR 10.2.1, OrbStack (Ubuntu) on an M2 Mac

## Topology

| Router | AS | Loopback | Link IP (eth1) |
|--------|-------|-----------|----------------|
| r1 | 65001 | 1.1.1.1/32 | 10.0.12.1/30 |
| r2 | 65002 | 2.2.2.2/32 | 10.0.12.2/30 |

r1:eth1 is connected to r2:eth1.

## Files

- bgp.clab.yml: Containerlab topology
- daemons: enables bgpd in FRR
- r1.conf, r2.conf: router configs
- *.txt: saved command output (evidence)

## Run

```
sudo containerlab deploy -t bgp.clab.yml
docker exec clab-bgp-lab-r1 vtysh -c "show ip bgp summary"
sudo containerlab destroy -t bgp.clab.yml
```


## Results

- eBGP session reached Established
- Each router learned the other's loopback, with AS_PATH 65002 on r1 and 65001 on r2
- r1 installed 2.2.2.2/32 in its routing table as a BGP route, [20/0] via 10.0.12.2
- Ping from 1.1.1.1 to 2.2.2.2: 3/3 received, 0% loss

## What I learned

- eBGP neighbors are configured by hand (neighbor IP + remote-as), unlike OSPF's automatic discovery
- A network statement announces a prefix only if it already exists on the router
- AS_PATH records the ASes a route crossed and prevents loops
- eBGP administrative distance is 20, versus 110 for OSPF
- FRR blocks eBGP routes until a policy is attached. The session came up but showed "discarded due to missing policy" and 0 prefixes. I fixed it with no bgp ebgp-requires-policy for this lab. Production networks use route-maps instead.
