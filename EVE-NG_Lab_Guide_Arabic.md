# دليل تصميم لاب EVE-NG - شبكة متعددة الأجزاء

## نظرة عامة

يقدم هذا الدليل توثيقاً شاملاً لتصميم وتنفيذ شبكة مؤسسية صغيرة متعددة الأجزاء (Multi-Segment Network) باستخدام منصة **EVE-NG**. يركز المشروع على تطبيق المفاهيم الأساسية في بناء الشبكات المؤسسية، مثل البنية الهرمية، وبروتوكولات التوجيه المتقدمة، وتقسيم الشبكة باستخدام VLANs، وضمان استمرارية الشبكة من خلال تقنيات المرونة.

---

## الأهداف التعليمية

يهدف هذا المشروع إلى تحقيق الأهداف التعليمية التالية:

1. **فهم البنية الهرمية للشبكات** (Core, Distribution, Access)
2. **تطبيق بروتوكول التوجيه OSPF** في بيئة متعددة المناطق (Multi-area)
3. **إنشاء شبكات VLANs** لعزل التدفق (Traffic) بين المستخدمين والخوادم
4. **اختبار مرونة الشبكة** (Resilience) من خلال إعداد روابط احتياطية (Redundant Links) باستخدام EtherChannel
5. **بناء بيئة معملية** (Lab) جاهزة للتوسع ودمج مشاريع مستقبلية مثل جدار الحماية (Firewall) أو أنظمة التحكم في الوصول (NAC)

---

## الرسم التوضيحي للشبكة (Network Topology)

![EVE-NG Lab Topology](EVE-NG_Multi-Segment_Network_Lab.png)

يوضح الرسم أعلاه البنية الكاملة للشبكة مع جميع الأجهزة والاتصالات وعناوين IP.

---

## جدول الأجهزة والمكونات

| الجهاز | النوع | الوظيفة الأساسية | ملاحظات |
|---|---|---|---|
| **R1** | Router | Core Router (الفرع الرئيسي) | يعمل في OSPF Area 0 والربط مع الفروع الأخرى |
| **R2** | Router | Branch Router (الفرع الثانوي) | كبة الرئيسية عبر WAN ويعمل في OSPF Area 1 |
| **SW-Core1** | L3 Switch | Distribution/Core Switch | المحلية (Inter-VLAN Routing) ويعمل كنقطة تجميع لـ Access Switches |
| **SW-Access1** | L2/L3 Switch | Access Switch | النهائية بالشبكة ويدير الـ VLANs الخاصة بهم |
| **Server1** | Server | DHCP/DNS Server | DHCP وترجمة أسماء النطاقات (DNS) |
| **Server2** | Server | Web/Backup Server | خادم الويب ويعمل كخادم نسخ احتياطي |
| **PC1** | End Device | أجهزة مستخدمين | في VLAN 30 (Sales) لاختبار الاتصال والعزل |
| **PC2** | End Device | أجهزة مستخدمين | في VLAN 40 (HR) لاختبار الاتصال والعزل |
| **PC3** | End Device | أجهزة مستخدمين | جهاز إضافي لاختبار الشبكة |

---

## خطة العنونة (IP Addressing Plan)

### جدول الشبكات الفرعية (VLANs & Subnets)

| VLAN ID | اسم الشبكة | الوصف | عنوان الشبكة | قناع الشبكة | البوابة الافتراضية | نطاق DHCP |
|---|---|---|---|---|---|---|
| **10** | Management | شبكة إدارة الأجهزة | 192.168.10.0 | 255.255.255.0 | 192.168.10.1 | - |
| **20** | Servers | شبكة الخوادم | 192.168.20.0 | 255.255.255.0 | 192.168.20.1 | - |
| **30** | Users-Sales | شبكة مستخدمي قسم المبيعات | 192.168.30.0 | 255.255.255.0 | 192.168.30.1 | 192.168.30.100-200 |
| **40** | Users-HR | شبكة مستخدمي قسم الموارد البشرية | 192.168.40.0 | 255.255.255.0 | 192.168.40.1 | 192.168.40.100-200 |
| **N/A** | WAN Link | الربط بين R1 و R2 | 10.0.0.0 | 255.255.255.252 | - | - |

### جدول عناوين IP للأجهزة

| الجهاز | الواجهة | عنوان IP | قناع الشبكة | الوصف |
|---|---|---|---|---|
| **R1** | GigabitEthernet0/0 | 192.168.10.254 | 255.255.255.0 | الاتصال بـ SW-Core1 |
| **R1** | Serial0/0 | 10.0.0.1 | 255.255.255.252 | WAN Link إلى R2 |
| **R2** | GigabitEthernet0/0 | 192.168.50.254 | 255.255.255.0 | الاتصال بشبكة الفرع |
| **R2** | Serial0/0 | 10.0.0.2 | 255.255.255.252 | WAN Link إلى R1 |
| **SW-Core1** | VLAN 10 (SVI) | 192.168.10.1 | 255.255.255.0 | بوابة شبكة الإدارة |
| **SW-Core1** | VLAN 20 (SVI) | 192.168.20.1 | 255.255.255.0 | بوابة شبكة الخوادم |
| **SW-Core1** | VLAN 30 (SVI) | 192.168.30.1 | 255.255.255.0 | بوابة شبكة المبيعات |
| **SW-Core1** | VLAN 40 (SVI) | 192.168.40.1 | 255.255.255.0 | بوابة شبكة الموارد البشرية |
| **SW-Core1** | GigabitEthernet0/0 | 192.168.10.2 | 255.255.255.0 | الاتصال بـ R1 |
| **Server1** | eth0 | 192.168.20.10 | 255.255.255.0 | خادم DHCP/DNS |
| **Server2** | eth0 | 192.168.20.20 | 255.255.255.0 | خادم الويب والنسخ الاحتياطي |
| **PC1** | eth0 | DHCP | - | جهاز قسم المبيعات (VLAN 30) |
| **PC2** | eth0 | DHCP | - | جهاز قسم الموارد البشرية (VLAN 40) |
| **PC3** | eth0 | DHCP أو Static | - | جهاز اختبار إضافي |

---

## التكوينات الكاملة للأجهزة

### 1. تكوين R1 – Core Router

```
enable
configure terminal
!
hostname R1
!
! === تكوين الواجهات ===
interface GigabitEthernet0/0
 description Link to SW-Core1
 ip address 192.168.10.254 255.255.255.0
 no shutdown
!
interface Serial0/0
 description WAN Link to R2
 ip address 10.0.0.1 255.255.255.252
 clock rate 128000
 no shutdown
!
! === تكوين OSPF ===
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
!
end
write memory
```

**شرح الأوامر:**
- `hostname R1`: تعيين اسم الجهاز
- `ip address 192.168.10.254 255.255.255.0`: تعيين عنوان IP للواجهة
- `clock rate 128000`: تعيين سرعة الساعة للواجهة Serial (مطلوب على جهة DCE)
- `router ospf 1`: تفعيل بروتوكول OSPF بمعرف العملية 1
- `router-id 1.1.1.1`: تعيين معرف فريد للموجه في OSPF
- `network 192.168.10.0 0.0.0.255 area 0`: إعلان الشبكة في OSPF Area 0

---

### 2. تكوين R2 – Branch Router

```
enable
configure terminal
!
hostname R2
!
! === تكوين الواجهات ===
interface GigabitEthernet0/0
 description Link to Branch Network
 ip address 192.168.50.254 255.255.255.0
 no shutdown
!
interface Serial0/0
 description WAN Link to R1
 ip address 10.0.0.2 255.255.255.252
 no shutdown
!
! === تكوين OSPF ===
router ospf 1
 router-id 2.2.2.2
 network 192.168.50.0 0.0.0.255 area 1
 network 10.0.0.0 0.0.0.3 area 0
!
end
write memory
```

**شرح الأوامر:**
- `network 192.168.50.0 0.0.0.255 area 1`: إعلان شبكة الفرع في OSPF Area 1
- `network 10.0.0.0 0.0.0.3 area 0`: إعلان رابط WAN في OSPF Area 0

---

### 3. تكوين SW-Core1 – Distribution/Core Switch

```
enable
configure terminal
!
hostname SW-Core1
!
! === تفعيل التوجيه ===
ip routing
!
! === إنشاء VLANs ===
vlan 10
 name Management
vlan 20
 name Servers
vlan 30
 name Users-Sales
vlan 40
 name Users-HR
!
! === تكوين SVIs للتوجيه بين VLANs ===
interface Vlan10
 description Management VLAN
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
interface Vlan20
 description Servers VLAN
 ip address 192.168.20.1 255.255.255.0
 no shutdown
!
interface Vlan30
 description Users-Sales VLAN
 ip address 192.168.30.1 255.255.255.0
 no shutdown
!
interface Vlan40
 description Users-HR VLAN
 ip address 192.168.40.1 255.255.255.0
 no shutdown
!
! === الاتصال بـ R1 ===
interface GigabitEthernet0/0
 description Link to R1
 no switchport
 ip address 192.168.10.2 255.255.255.0
 no shutdown
!
! === تكوين EtherChannel ===
interface Port-channel1
 description EtherChannel to SW-Access1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
!
interface range GigabitEthernet0/1 - 2
 description Member of Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
 channel-group 1 mode desirable
 no shutdown
!
! === تكوين OSPF ===
router ospf 10
 router-id 3.3.3.3
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
!
end
write memory
```

**شرح الأوامر:**
- `ip routing`: تفعيل التوجيه على المبدل L3
- `interface Vlan10`: إنشاء واجهة افتراضية (SVI) للـ VLAN 10
- `no switchport`: تحويل المنفذ من وضع المبدل إلى وضع الموجه
- `switchport trunk encapsulation dot1q`: تحديد نوع التغليف للـ Trunk
- `channel-group 1 mode desirable`: إضافة الواجهة إلى Port-channel بوضع PAgP

---

### 4. تكوين SW-Access1 – Access Switch

```
enable
configure terminal
!
hostname SW-Access1
!
! === إنشاء VLANs ===
vlan 20
 name Servers
vlan 30
 name Users-Sales
vlan 40
 name Users-HR
!
! === تكوين EtherChannel ===
interface Port-channel1
 description EtherChannel to SW-Core1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
!
interface range GigabitEthernet0/0 - 1
 description Member of Port-channel1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 20,30,40
 channel-group 1 mode desirable
 no shutdown
!
! === منافذ الوصول للأجهزة الطرفية ===
interface GigabitEthernet0/2
 description To PC1 (Sales)
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet0/3
 description To PC2 (HR)
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
 no shutdown
!
interface GigabitEthernet1/0
 description To PC3
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
 no shutdown
!
end
write memory
```

**شرح الأوامر:**
- `switchport mode access`: تعيين المنفذ كمنفذ وصول (Access Port)
- `switchport access vlan 30`: تعيين المنفذ إلى VLAN 30
- `spanning-tree portfast`: تفعيل PortFast لتسريع الاتصال

---

### 5. تكوين Server1 – DHCP/DNS Server (Linux)

#### تكوين الواجهة الشبكية:
```bash
sudo ip addr add 192.168.20.10/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.20.1
```

#### تثبيت وتكوين DHCP Server:
```bash
sudo apt-get update
sudo apt-get install -y isc-dhcp-server
```

#### محتوى ملف `/etc/dhcp/dhcpd.conf`:
```
# DHCP Configuration for VLAN 30 - Sales
subnet 192.168.30.0 netmask 255.255.255.0 {
  range 192.168.30.100 192.168.30.200;
  option routers 192.168.30.1;
  option domain-name-servers 192.168.20.10;
  option domain-name "lab.local";
  default-lease-time 600;
  max-lease-time 7200;
}

# DHCP Configuration for VLAN 40 - HR
subnet 192.168.40.0 netmask 255.255.255.0 {
  range 192.168.40.100 192.168.40.200;
  option routers 192.168.40.1;
  option domain-name-servers 192.168.20.10;
  option domain-name "lab.local";
  default-lease-time 600;
  max-lease-time 7200;
}
```

#### تشغيل خدمة DHCP:
```bash
sudo systemctl restart isc-dhcp-server
sudo systemctl enable isc-dhcp-server
sudo systemctl status isc-dhcp-server
```

#### تثبيت وتكوين DNS Server (BIND9):
```bash
sudo apt-get install -y bind9 bind9utils bind9-doc
sudo systemctl start bind9
sudo systemctl enable bind9
```

---

### 6. تكوين Server2 – Web/Backup Server (Linux)

#### تكوين الواجهة الشبكية:
```bash
sudo ip addr add 192.168.20.20/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 192.168.20.1
```

#### تثبيت وتكوين Apache Web Server:
```bash
sudo apt-get update
sudo apt-get install -y apache2
sudo systemctl start apache2
sudo systemctl enable apache2
```

#### إنشاء صفحة ويب اختبارية:
```bash
echo "<h1>مرحباً بك في شبكة المعمل - Server2</h1>" | sudo tee /var/www/html/index.html
```

---

## خطوات التحقق والاختبار

### 1. التحقق من الاتصال الأساسي (Ping Tests)

#### من R1:
```
R1# ping 10.0.0.2
R1# ping 192.168.10.1
R1# ping 192.168.20.10
R1# ping 192.168.50.254
```

#### من R2:
```
R2# ping 10.0.0.1
R2# ping 192.168.10.1
R2# ping 192.168.20.10
```

#### من PC1 (VLAN 30):
```bash
ping 192.168.40.1
ping 192.168.20.20
ping 192.168.30.1
ping 8.8.8.8
```

#### من PC2 (VLAN 40):
```bash
ping 192.168.30.1
ping 192.168.20.10
ping 192.168.40.1
```

---

### 2. التحقق من OSPF

#### على R1:
```
R1# show ip ospf neighbor
R1# show ip route ospf
R1# show ip ospf interface brief
R1# show ip ospf database
```

**النتيجة المتوقعة:**
- يجب أن تظهر علاقة جوار (Neighbor) مع R2 و SW-Core1
- يجب أن تظهر المسارات من Area 1 (192.168.50.0/24) في جدول التوجيه

#### على R2:
```
R2# show ip ospf neighbor
R2# show ip route ospf
R2# show ip protocols
```

#### على SW-Core1:
```
SW-Core1# show ip ospf neighbor
SW-Core1# show ip route ospf
SW-Core1# show ip protocols
```

---

### 3. التحقق من EtherChannel

#### على SW-Core1:
```
SW-Core1# show etherchannel summary
SW-Core1# show etherchannel port-channel
SW-Core1# show interfaces port-channel 1
SW-Core1# show interfaces trunk
```

**النتيجة المتوقعة:**
```
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)         PAgP      Gi0/1(P)    Gi0/2(P)
```

#### على SW-Access1:
```
SW-Access1# show etherchannel summary
SW-Access1# show interfaces port-channel 1
```

---

### 4. التحقق من VLANs

#### على SW-Core1:
```
SW-Core1# show vlan brief
SW-Core1# show interfaces trunk
SW-Core1# show ip interface brief
```

#### على SW-Access1:
```
SW-Access1# show vlan brief
SW-Access1# show interfaces trunk
SW-Access1# show interfaces status
```

---

### 5. التحقق من DHCP

#### على PC1 و PC2 و PC3:

**في Linux:**
```bash
sudo dhclient -r eth0
sudo dhclient -v eth0
ip addr show eth0
ip route show
cat /etc/resolv.conf
```

**في Windows:**
```cmd
ipconfig /release
ipconfig /renew
ipconfig /all
```

**النتيجة المتوقعة:**
- PC1 يجب أن يحصل على عنوان IP من النطاق `192.168.30.100-200`
- PC2 يجب أن يحصل على عنوان IP من النطاق `192.168.40.100-200`
- البوابة الافتراضية (Default Gateway) يجب أن تكون صحيحة
- خادم DNS يجب أن يكون `192.168.20.10`

#### على Server1 (للتحقق من عمليات DHCP):
```bash
sudo tail -f /var/log/syslog | grep dhcpd
sudo journalctl -u isc-dhcp-server -f
```

---

### 6. التحقق من خادم الويب

#### من PC1 أو PC2 أو PC3:
```bash
curl http://192.168.20.20
# أو من المتصفح
firefox http://192.168.20.20
```

**النتيجة المتوقعة:**
- يجب أن تظهر صفحة الويب الاختبارية

---

## ملاحظات التنفيذ في EVE-NG

### متطلبات الأجهزة في EVE-NG:

| نوع الجهاز | الصورة الموصى بها | ملاحظات |
|---|---|---|
| **Routers (R1, R2)** | vIOS L3 / IOL | `vios-adventerprisek9-m` |
| **L3 Switch (SW-Core1)** | IOL L3 | `i86bi-linux-l3-adventerprisek9` |
| **L2 Switch (SW-Access1)** | IOL L2/L3 | `i86bi-linux-l2-adventerprisek9` |
| **Servers** | Linux | Ubuntu Server 20.04 أو Debian |
| **PCs** | VPCS أو Linux | TinyCore Linux أو VPCS |

### خطوات الإعداد في EVE-NG:

1. إنشاء Lab جديد في EVE-NG
2. إضافة الأجهزة حسب الرسم التوضيحي
3. توصيل الأجهزة باستخدام الكابلات الشبكية (Network Cables)
4. تشغيل جميع الأجهزة
5. تطبيق التكوينات المذكورة أعلاه على كل جهاز
6. حفظ التكوينات باستخدام `write memory`
7. إجراء اختبارات التحقق

---

## استكشاف الأخطاء وإصلاحها

### مشكلة: OSPF Neighbor لا يظهر

**الحل:**
```
! تحقق من تطابق معرف المنطقة (Area ID)
show ip ospf interface

! تحقق من الاتصال الأساسي
ping [neighbor-ip]

! تحقق من تكوين OSPF
show running-config | section ospf
```

### مشكلة: EtherChannel لا يعمل

**الحل:**
```
! تحقق من تطابق الإعدادات على الطرفين
show etherchannel summary

! تحقق من وضع الـ Trunk
show interfaces trunk

! إعادة تكوين EtherChannel
no interface port-channel 1
interface range gi0/1-2
 no channel-group 1
 shutdown
 no shutdown
 channel-group 1 mode desirable
```

### مشكلة: DHCP لا يعمل

**الحل:**
```bash
# على Server1
sudo systemctl status isc-dhcp-server
sudo journalctl -u isc-dhcp-server -n 50

# تحقق من التكوين
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

# على PC
sudo dhclient -v eth0
```

---

## الخلاصة

يقدم هذا المشروع أساساً قوياً لبناء شبكة مؤسسية موثوقة وقابلة للتطوير. من خلال تطبيق OSPF، VLANs، و EtherChannel، تمكنا من إنشاء بيئة معملية تعكس أفضل الممارسات في تصميم الشبكات. يمكن استخدام هذا المعمل كنقطة انطلاق للمشاريع المتقدمة القادمة، مثل:

- إضافة جدران الحماية (Firewalls)
- تطبيق أنظمة كشف التسلل (IDS/IPS)
- إنشاء شبكات VPN
- تطبيق تقنيات QoS (Quality of Service)
- إضافة بروتوكولات توجيه إضافية (BGP, EIGRP)


