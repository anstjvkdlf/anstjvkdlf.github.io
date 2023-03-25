---
title:  "gNB architecture"
excerpt: "Carreier aggregation detail"
permalink: /ca_detail/

categories:
  - cellular network
tags:
  - [NSA, ED-DC, ENDC, CA, NR, 5G, LTE, Scell, Pcell, Spcell]

toc: true
toc_sticky: true
 
date: 2022-03-19
last_modified_at: 2022-03-19
---

# 1. gNB
In my defination, gNB is server receiving data from client.

We called client as "UE" in telecommunication field.

## 1.1. gNB architecture
As like other server, gNB is also seperated into three components.

Each component has its role.

Very simply,
- DU is unit welcoming UE.
- CU-CP is unit to deliver UE data to Core network.
- CU-UP is a unit providing internet access to UE
```
+----------+   +----------+   +----------+
| gNB-CU-CP| + | gNB-CU-UP| + |  gNB-DU  |
+----------+   +----------+   +----------+
```
---

## 1.2. gNB interface
```
+----------+            +----------+
| gNB-CU-CP| --- E1 --- | gNB-CU-UP|
+----------+            +----------+

+----------+            +----------+
| gNB-CU-CP| --- F1 --- |  gNB-DU  |
+----------+            +----------+

+----------+            +----------+
| gNB-CU-CP| --- NG --- | AMF(CORE)|
+----------+            +----------+

+----------+            +----------+
| gNB-CU-CP| --- X2 --- |    eNB   |
+----------+            +----------+

+----------+            +----------+
| gNB-CU-CP| --- Xn --- |    gNB   |
+----------+            +----------+

+----------+            +----------+
|  gNB-DU  | --- RRC ---|    UE    |
+----------+            +----------+
```

## 1.3. How is UE attached?
Attach means UE is connect to gNB server.

Process
+ UE is attached to gNB-DU via RRC interface.
+ gNB-DU forward UE data to gNB-CU-CP via F1 interface.
+ gNB-CU-CP give UE data to AMF(CORE).
+ AMF tries to make a lot of configurations to allow UE to use celluler network service
```
+----------+            +----------+            +----------+            +----------+
|    UE    | --- RRC ---|  gNB-DU  | --- F1 --- | gNB-CU-CP| --- NG --- |    AMF   |
+----------+            +----------+            +----------+            +----------+
```


## 1.4. UE attach process


```
AMF                        CU-CP                        DU                          UE
|                            |                           |     RRC Connect Req       |
|                            |                           |<--------------------------|
|                            |  F1 : UL-RRC-msg transfer |                           |
|                            |<--------------------------|                           |
|                            |  F1 : DL-RRC-msg transfer |                           |
|                            |-------------------------->|                           |
|                            |                           |     RRC ConnectSetup      |
|                            |                           |-------------------------->|
|                            |                           |RRC ConnectSetup complete  |
|                            |                           |<--------------------------|
|   NG: Ininitial UE Msg     |                           |                           |
|<---------------------------|                           |                           |
|NG: Initial UE context Setup|                           |                           |
|--------------------------->|                           |                           |
|                            |  F1 : UEContextSetupReq   |                           |
|                            |-------------------------->|                           |
|                            |  F1 : UEContextSetupResp  |                           |
|                            |<--------------------------|                           |
|                            |                           |    RRC Reconfiguration    |
|                            |                           |-------------------------->|
|                            |                           |    RRC Reconf complete    |
|                            |                           |<--------------------------|
|                            |  F1 : UL-RRC-msg transfer |                           |
|                            |<--------------------------|                           |
|                            |  F1 : UEContextModifReq   |                           |
|                            |-------------------------->|                           |
|                            |  F1 : UEContextModifRes   |                           |
|                            |<--------------------------|                           |
|NG: UE contextSetup resopnse|                           |                           |
|<---------------------------|                           |                           |
```