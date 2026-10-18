# Configs

BGP sections from the final, working state (with `next-hop-self` on R1 and R4). Back to the [README](README.md).

## R5 (AS 100)

```
router bgp 100
 bgp log-neighbor-changes
 network 10.10.10.10 mask 255.255.255.255
 network 5.5.5.0 mask 255.255.255.0
 neighbor 5.5.5.1 remote-as 200
```

## R6 (AS 300)

```
router bgp 300
 bgp log-neighbor-changes
 network 30.30.30.30 mask 255.255.255.255
 network 4.4.4.0 mask 255.255.255.0
 neighbor 4.4.4.4 remote-as 200
```

## R1 (AS 200, edge toward AS 100)

```
router bgp 200
 bgp log-neighbor-changes
 network 1.1.1.0 mask 255.255.255.0
 network 2.2.2.0 mask 255.255.255.0
 network 3.3.3.0 mask 255.255.255.0
 neighbor 2.2.2.2 remote-as 200
 neighbor 2.2.2.2 next-hop-self
 neighbor 3.3.3.3 remote-as 200
 neighbor 3.3.3.3 next-hop-self
 neighbor 3.3.3.4 remote-as 200
 neighbor 3.3.3.4 next-hop-self
 neighbor 5.5.5.5 remote-as 100
```

## R2 (AS 200, internal)

```
router bgp 200
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 200
 neighbor 2.2.2.4 remote-as 200
 neighbor 3.3.3.3 remote-as 200
```

## R3 (AS 200, internal)

```
router bgp 200
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 200
 neighbor 2.2.2.2 remote-as 200
 neighbor 3.3.3.4 remote-as 200
```

## R4 (AS 200, edge toward AS 300)

```
router bgp 200
 bgp log-neighbor-changes
 neighbor 1.1.1.1 remote-as 200
 neighbor 1.1.1.1 next-hop-self
 neighbor 2.2.2.2 remote-as 200
 neighbor 2.2.2.2 next-hop-self
 neighbor 3.3.3.3 remote-as 200
 neighbor 3.3.3.3 next-hop-self
 neighbor 4.4.4.6 remote-as 300
```

## Reproducing the problem state

Remove `next-hop-self` from R1 and R4, then refresh:

```
R1(config)# router bgp 200
R1(config-router)# no neighbor 2.2.2.2 next-hop-self
R1(config-router)# no neighbor 3.3.3.3 next-hop-self
R1(config-router)# no neighbor 3.3.3.4 next-hop-self
R1# clear ip bgp * soft out
```

Then on R3, `show ip bgp 10.10.10.10` shows next hop `5.5.5.5 (inaccessible)`, and the route is missing from `show ip route`.
