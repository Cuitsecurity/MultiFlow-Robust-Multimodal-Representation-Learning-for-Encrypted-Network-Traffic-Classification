# 数据集说明

## 1. CSTNET-TLS-1.3
- 描述：CSTNET (China Science and Technology Network) 发布的 TLS 1.3 流量数据集，通常包含正常 TLS 1.3 会话与恶意样本（如命令控制、扫描、DDoS）的192.168.1.0/24等数据。用于加密流量分类与威胁检测。
- 主要内容：PCAP文件、Flow特征、标签（正常/异常）
- 来源：https://drive.google.com/drive/folders/1JSsYmevkxQFanoKOi_i1ooA6pH3s9sDr

## 2. ISCX-VPN
- 描述：ISCX VPN 数据集由加拿大纽芬兰与拉布拉多大学计算机网络安全实验室发布，包含VPN和非VPN流量（包括多种应用协议如HTTP、FTP、Streaming），用于区分加密VPN流量与普通互联网流量。
- 主要内容：PCAP、Flow统计特征、应用标签
- 来源：https://www.unb.ca/cic/datasets/vpn.html

## 3. CICIoT2022
- 描述：CIC（加拿大网络安全研究所）发布的2022年IoT设备流量数据集，涵盖多种IoT设备在正常状态和受到攻击时生成的网络流量。适合IoT分类、异常检测及多模态攻击分析。
- 主要内容：PCAP、NetFlow、特征、恶意/正常标签
- 来源：https://www.unb.ca/cic/datasets/iotdataset-2022.html

## 4. ISCX-Tor
- 描述：ISCX Tor 数据集用于Tor网络流量分析，包含Tor浏览器、非Tor流量，常用于Tor流量检测与隐私评测。包含研究人员模拟的Tor客户端与服务端通信行为。
- 主要内容：PCAP、Flow特征、Tor/非Tor标签
- 来源：https://www.unb.ca/cic/datasets/tor.html

## 5. USTC-TFC
- 描述：USTC (中国科技大学) TFC (Tor Flow Corpus) 数据集，面向加密流量分类（Tor 与 非Tor），包括不同协议下的Tor/正常流量。该数据集常用于对抗性检测、加密流量分类等研究。
- 主要内容：PCAP、流量统计、标签（Tor/Normal）
- https://github.com/davidyslu/USTC-TFC2016
