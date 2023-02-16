---
title:  "5G Network Channel(Downlink)"
excerpt: "Comprehand 5G downlink channel"

categories:
  - cellular network
tags:
  - [PDCCH, PDSCH, NR, 5G, LTE,]

toc: true
toc_sticky: true
 
date: 2022-02-16
last_modified_at: 2022-02-16
---


# 1. 5G Downlink Channel

## 1.1 PDCCH (Physical Downlink Control Channel)
---
### 1.1.1. Allocation
This concept is very simple. Don't be afraid, believe me.

PDCCH is dowlink channel in PHY layer.

It looks like this &darr;

![image](https://user-images.githubusercontent.com/18244590/219415270-c5e2cd73-b89e-4e43-a99e-de47e873c398.png)


Yellow area is PDCCH area. It is located within CORESET.

You don't have to know what is CORESET here, just remind that PDCCH is located at specific area called CORESET.

You are done.

**PDCCH is just pre-allocated radio resource within CORESET.**

That's all.

---
### 1.1.2. Concept
Why did we decided to use that amount of radio resource for PDCCH?

The reason is to carry [DCI](). I recommand to read what is DCI firstly.

Very breifly, DCI is used to schedule downlink channel and uplink channel.

By putting DCI at front slot, we can know how slot looks like.

For example, to find PDSCH area we should look at PDCCH area first which contains DCI.

Furthermore I guess you can naturally understand why PDCCH is starting from first symbol.

#
### 1.1.3. Parameter
There is three significant parameter in PDCCH.

monitoringSymbolWithInSlot : This parameter is 14 bits string. It represents 1 slot. When it is set to "1000 0000 0000 00" it means PDCCH starts at first Symbol.

startSymbol : This parameter is array. It represent where PDCCH symbol starts at. It has same information as monitoringSymbolWithInSlot.

duration : This parameter is interger. It represent duration of PDCCH symbol.


For example when startSymbolSet is set to [0] which means monitoringSymbolWithInSlot is set to "1000 0000 0000 00". And if duration is 3, this can be illustrated as picture I've attach above(yellow area). 



## 1.2 PDSCH (Physical Downlink Shared Channel)
#
### 1.2.1. Allocation
This concept is also simple.

PDSCH is dowlink channel in PHY layer.

Let's look at the picture again. &darr;

![image](https://user-images.githubusercontent.com/18244590/219415130-e943a51e-2279-4081-a1bf-4a3857f3f3bf.png)



Turquoise area is PDSCH area. This area was allocated by DCI.

It starts right after where PDCCH ends.

#
### 1.2.2. Concept
PDSCH exists to send user data.

UE try to find PDSCH are using DCI which was carried in PDCCH.

You are done. That is PDSCH.

#
### 1.1.3. Parameter
There is one significant parameter in PDSCH.

SLIV : This parameter is integer. It represent where PDSCH start and how many symbol does PDSCH is using.

![image](https://user-images.githubusercontent.com/18244590/219415691-c8fda778-32c7-4271-aecf-e36d403aeb56.png)

For example when startSymbolSet is set to [0] which means monitoringSymbolWithInSlot is set to "1000 0000 0000 00". And if duration is 3 this can be illustrated as picture I've attach above(yellow area).
This means that PDSCH will be start from starting Symbol 3.

PDSCH is using 8 symbol, so we can choose 101 as SLIV value.