# Multi-Site Enterprise Network

**Cisco Packet Tracer를 활용한 멀티사이트 기업 네트워크 구축 프로젝트**

Seoul HQ, Busan Branch, Daegu Branch로 구성된 3개 사이트의 기업 네트워크를 시뮬레이션한 프로젝트입니다. VLAN을 활용한 네트워크 분리, 두 가지 Inter-VLAN Routing 방식, 중앙 집중식 DHCP, 사이트 간 OSPF Routing, Layer 2 보안 기능을 구현했습니다.

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [네트워크 토폴로지](#2-네트워크-토폴로지)
3. [VLAN 구성](#3-vlan-구성)
4. [IP 주소 구성](#4-ip-주소-구성)
5. [구현](#5-구현)

   * [5.1 Seoul HQ](#51-seoul-hq)
   * [5.2 Busan Branch](#52-busan-branch)
   * [5.3 Daegu Branch](#53-daegu-branch)
6. [Routing (OSPF)](#6-routing-ospf)
7. [DHCP](#7-dhcp)
8. [Layer 2 Security](#8-layer-2-security)
9. [테스트 및 검증](#9-테스트-및-검증)
10. [파일 구성](#10-파일-구성)

---

## 1. 프로젝트 개요

Cisco Packet Tracer를 활용하여 소규모 멀티사이트 기업 네트워크를 구축했습니다.

각 사이트는 부서별 VLAN으로 네트워크를 분리했으며, Seoul HQ에는 HR, Sales, IT, Management VLAN을 구성했습니다. 세 사이트는 Point-to-Point Serial Link로 연결하고, 사이트 간 Dynamic Routing을 위해 OSPF를 사용했습니다.

또한 Seoul HQ에 중앙 DHCP Server를 구성하여 DHCP Relay를 통해 세 사이트의 모든 VLAN에 IP 주소를 자동으로 할당하도록 구현했습니다.

### Inter-VLAN Routing 구성

서로 다른 두 가지 Inter-VLAN Routing 방식을 비교하고 구현했습니다.

* **Seoul HQ**: Layer 2 Switch를 사용하며, Router의 Subinterface를 활용한 **Router-on-a-Stick** 방식으로 Inter-VLAN Routing을 구성했습니다.
* **Busan Branch / Daegu Branch**: Layer 3 Switch를 사용하며, Switch의 **SVI (Switched Virtual Interface)***를*통해 Inter-VLAN Routing을 구성했습니다.

---

## 2. 네트워크 토폴로지

[네트워크 토폴로지 보기](https://github.com/aidana-net/multi-user-enterprise-network/blob/main/topology.png)

---

## 3. VLAN 구성

| Site  | VLAN | Name       | Network          |
| ----- | ---: | ---------- | ---------------- |
| Seoul |   10 | HR         | 192.168.10.0/24  |
| Seoul |   20 | Sales      | 192.168.20.0/24  |
| Seoul |   30 | IT         | 192.168.30.0/24  |
| Seoul |   99 | Management | 192.168.99.0/24  |
| Busan |   10 | HR         | 192.168.210.0/24 |
| Busan |   20 | Sales      | 192.168.220.0/24 |
| Busan |   30 | IT         | 192.168.230.0/24 |
| Daegu |   10 | HR         | 192.168.110.0/24 |
| Daegu |   20 | Sales      | 192.168.120.0/24 |
| Daegu |   30 | IT         | 192.168.130.0/24 |

---

## 4. IP 주소 구성

### Point-to-Point (Serial) Link

| Link          | Network     | Seoul Side   | Remote Side          |
| ------------- | ----------- | ------------ | -------------------- |
| Seoul ↔ Busan | 10.0.0.0/30 | .1 (Se0/1/0) | .2 (r1 Se0/1/0)      |
| Seoul ↔ Daegu | 10.0.0.4/30 | .5 (Se0/1/1) | .6 (Router2 Se0/1/0) |

### Branch Router ↔ L3 Switch Uplink

| Link                      | Network     |
| ------------------------- | ----------- |
| Busan r1 ↔ L3 Switch      | 10.0.1.0/30 |
| Daegu Router2 ↔ L3 Switch | 10.0.2.0/30 |

---

## 5. 구현

### 5.1 Seoul HQ

* **Switch2** — Layer 2 Switch로 VLAN 10 (HR), VLAN 20 (Sales), VLAN 30 (IT), VLAN 99 (Management)을*구성했습니다. DHCP Server (`Server0`)는 Management VLAN에 위치합니다.
* **Router** — Gig0/1을 통해 Switch2와 Trunk로 연결하고, Router의 Subinterface를 구성하여 **Router-on-a-Stick 방식의 Inter-VLAN Routing**을 구현했습니다. 또한 Se0/1/0, Se0/1/1 Serial Link를 통해 Busan과 Daegu에 연결했습니다.

### 5.2 Busan Branch

* **L3 Switch** — VLAN 10 (HR), VLAN 20 (Sales), VLAN 30 (IT)을*구성하고,*&#xAC01; VLAN의 SVI를 통해 로컬 Inter-VLAN Routing을 구현했습니다.
* **r1** — Serial Link를 통해 Seoul HQ와 연결하고, Gig0/0–Gig0/1 Routed Uplink를 통해 L3 Switch와 연결했습니다. Uplink Network는 `10.0.1.0/30`을 사용했습니다.

### 5.3 Daegu Branch

* **L3 Switch** — VLAN 10 (HR), VLAN 20 (Sales), VLAN 30 (IT)을*구성하고,*&#xAC01; VLAN의 SVI를 통해 로컬 Inter-VLAN Routing을 구현했습니다.
* **Router2** — Serial Link를 통해 Seoul HQ와 연결하고, Gig0/0–Gig0/1 Routed Uplink를 통해 L3 Switch와 연결했습니다. Uplink Network는 `10.0.2.0/30`을 사용했습니다.

---

## 6. Routing (OSPF)

세 사이트의 Router 간에 OSPF를 구성했습니다.

Seoul Router, Busan `r1`, Daegu `Router2`가 각각의 VLAN Network와 Point-to-Point Link Network를 OSPF를 통해 광고하도록 구성하여 세 사이트 간의 전체적인 네트워크 통신이 가능하도록 구현했습니다.

---

## 7. DHCP

Seoul의 Management VLAN (99)에 DHCP Server를 구성했습니다.

Busan과 Daegu를 포함한 세 사이트의 모든 VLAN이 중앙 DHCP Server를 통해 IP 주소를 할당받을 수 있도록 각 VLAN의 Gateway Interface에 `ip helper-address`를 설정하여 **DHCP Relay**를 구성했습니다.

---

## 8. Layer 2 Security

| Site       | Port Security           | DHCP Snooping | Dynamic ARP Inspection (DAI)      |
| ---------- | ----------------------- | ------------- | --------------------------------- |
| Seoul (L2) | ✅                       | ✅             | ✅ (Router 연결 Uplink를 Trusted로 설정) |
| Busan (L3) | ✅ (Violation: Restrict) | ❌ 제거          | ❌ L3에서는 적용하지 않음                   |
| Daegu (L3) | ✅ (Violation: Protect)  | ❌ 제거          | ❌ L3에서는 적용하지 않음                   |

DHCP Snooping과 DAI는 Switchport 및 VLAN 환경에 기반한 Layer 2 기능이므로 Seoul HQ의 Layer 2 Switch에서만 전체적으로 구현했습니다.

Busan과 Daegu의 Layer 3 Switch에서는 DHCP Snooping을 처음에 구성하려고 시도했지만 이후 제거했습니다.

---

## 9. 테스트 및 검증

네트워크가 정상적으로 동작하는지 확인하기 위해 각 장비의 Configuration과 Command Output을 확인했습니다.

관련 자료는 [`/configs`](https://github.com/aidana-net/multi-user-enterprise-network/tree/main/configs) 및 [`/results`](https://github.com/aidana-net/multi-user-enterprise-network/tree/main/results)에서 확인할 수 있습니다.

### Configuration

각 장비의 `show running-config` 결과를 저장했습니다.

* `configs/Seoul-Router.txt`
* `configs/Seoul-Switch.txt`
* `configs/Busan-Router.txt`
* `configs/Busan-Switch.txt`
* `configs/Daegu-Router.txt`
* `configs/Daegu-Switch.txt`

### 검증 결과

* `ospf-neighbors-and-routes.txt` — `show ip ospf neighbor` 및 `show ip route`를 통해 세 Router의 OSPF Neighbor 상태와 사이트 간 Routing 정보를 확인했습니다.
* `port-security.txt` — `show port-security`를 통해 세 Switch의 Sticky MAC 학습 및 Port Security 설정을 확인했습니다.
* `seoul-dhcp-snooping.txt` — Seoul Switch에서 `show ip dhcp snooping`을 통해 DHCP Snooping 설정을 확인했습니다.
* `dhcp-verification.txt` — 세 사이트의 PC에서 `ipconfig /all`을 실행하여 중앙 DHCP Server를 통한 IP 주소 할당을 확인했습니다.
* `site-to-site-ping.txt` — Busan ↔ Seoul, Busan ↔ Daegu, Seoul ↔ Daegu 간의 Site-to-Site Ping Test를 수행했습니다.

### Ping Test 참고 사항

대부분의 Site-to-Site Ping Test에서 첫 번째 Packet이 Timeout되는 현상이 발생했습니다.

이는 송신 장비가 목적지의 MAC Address를 아직 ARP Cache에 저장하지 않아 발생하는 **ARP Resolution Delay** 때문입니다.

ARP Entry가 학습된 이후에는 후속 Packet이 정상적으로 전달되었으며, Packet Loss는 0%로 확인되었습니다.

---


## 10. 파일 구성

* `multi-site enterprise network .pkt` — Cisco Packet Tracer Network Topology 파일
* `topology.png` — 네트워크 토폴로지 다이어그램
* `configs/` — 모든 장비의 `show running-config` 결과
* `results/` — OSPF, Port Security, DHCP Snooping, DHCP Verification 및 Ping Test 결과

---

## Tools

* Cisco Packet Tracer

