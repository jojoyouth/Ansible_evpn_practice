### 專案簡介
本專案透過 Ansible 自動化配置，在 NVIDIA Air 模擬平台上從零建置完整的 BGP EVPN Symmetric IRB 網路架構。底層結合 VRF、VLAN、VNI 與 VXLAN 技術，並依靠 EVPN Type-2 實現 Anycast Gateway 與 ECMP 備援。另外在 Border 交換器上額外跑 EVPN Type-5 廣播網段，搭配 Subinterface，成功實作與外部目標 Server 的路由通信。

### 網路拓撲與角色職責
* **Spine**：只跑 Underlay，不處理 VLAN，純粹靠 BGP 轉發流量。每台 Spine 各自獨立運作不跑 MLAG，單純跑 eBGP 交換 EVPN 路由資訊，並透過 BGP Multipath 實現負載均衡。
* **Leaf**：負責接入終端設備。跑 Anycast Gateway（透過 VRR 設定相同的 IP、Global Anycast-MAC 並結合 EVPN Type-2）。利用 VRF 實現多租戶隔離，VRF 內部同樣綁定 VNI 來使用虛擬路由表轉發流量。同時，Leaf 也是整個 VXLAN 網路通道的起點與終點 (VTEP)。
* **Border**：具備 Leaf 的所有功能，並負責處理對外連線。建立新的對外 VLAN SVI，透過靜態路由對接防火牆。重點是利用 EVPN Type-5，向 VRF 內部的其他 Leaf 廣播對外路由。

### 核心 EVPN 功能
* **EVPN Type-2**：主要用來把 MAC 和 IP 丟給 BGP 進行轉送，還有 ARP 抑制的功能。
* **EVPN Type-5**：用來廣播網段。本架構直接在 Border 上利用 Type-5 廣播 `0.0.0.0/0`，讓內部所有對外流量預設都去找 Border，最後再由 Border 將流量轉送給防火牆，達成對外預設出口的功能。

---

### Project Overview
This project uses Ansible to build a complete BGP EVPN Symmetric IRB network from scratch on the NVIDIA Air simulation platform. The underlay combines VRF, VLAN, VNI, and VXLAN. It uses EVPN Type-2 for Anycast Gateway and ECMP redundancy. Additionally, the Border switch uses EVPN Type-5 and subinterfaces to route traffic to external servers.

### Topology & Node Roles
* **Spine**: Runs only the underlay. It does not handle VLANs and only forwards traffic using BGP. Each Spine works independently (no MLAG). It simply runs eBGP to exchange EVPN routes and uses BGP Multipath for load balancing.
* **Leaf**: Connects to end devices. It runs an Anycast Gateway (using VRR for the same IP, Global Anycast-MAC, and EVPN Type-2). It uses VRF for multi-tenant isolation, and binds VNIs inside the VRF to forward traffic using virtual routing tables. The Leaf is also the VTEP (the start and end point of the VXLAN tunnel).
* **Border**: Has all the functions of a Leaf, but also handles external connections. It creates a new external VLAN SVI and uses static routes to connect to the firewall. Most importantly, it uses EVPN Type-5 to broadcast external routes to other Leaves inside the VRF.

### Core EVPN Features
* **EVPN Type-2**: Sends MAC and IP addresses to BGP for forwarding. It also handles ARP suppression.
* **EVPN Type-5**: Broadcasts subnets. In this setup, the Border switch uses Type-5 to broadcast `0.0.0.0/0`. This makes the Border the default gateway for all outbound traffic, which is then routed to the firewall.
