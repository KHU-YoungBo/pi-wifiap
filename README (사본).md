<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="description" content="라즈베리 파이로 공유기 만들기">
<meta name="keywords" content="Raspberry Pi, RPi, AP, hostapd, 공유기">
<meta name="author" content="Sim Young-Bo (ybsim@khu.ac.kr)">

<link type="text/css" rel="stylesheet" href="./mystyle.css">

<style>

.TERM {
  font-style: italic;
  font-weight: bold;
}

</style>

<title>OpenWrt와 Raspberry PI2를 이용한 공유기 만들기</title>

<body>

<h2>0. 서론</h2>

<p>
Raspberry Pi2와 <span class="TERM">OpenWrt</span>를 이용하여 시중에서 구할 수 있는 공유기를 만드는 예제입니다. <br />
</p>

<p>
본 예제에서 구현하고자하는 공유기의 포톨로지와 공유기의 인터페이스 정보는 다음과 같습니다. <br/>
<img src="images/routed.ap_v3.png">
<figcaption>Fig1. Topology</figcaption>
</p><!-- console -->

<p class="console">
root@openwrt:~$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:90:f5:e4:cb:2c  
          inet addr:192.168.1.1  Bcast:192.168.1.255  Mask:255.255.255.0
          inet6 addr: fe80::290:f5ff:fee4:cb2c/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:823383 errors:0 dropped:5 overruns:0 frame:0
          TX packets:99326 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:271224907 (271.2 MB)  TX bytes:9310318 (9.3 MB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:6436 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6436 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:760775 (760.7 KB)  TX bytes:760775 (760.7 KB)

wlan0     Link encap:Ethernet  HWaddr 2c:d0:5a:61:5c:f4
          inet addr:192.168.100.1  Bcast:192.168.100.255  Mask:255.255.255.0  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
</p><!-- console -->
<figcaption>Fig2. Interface</figcaption>

<h2>1. 준비물</h2>
 <ol>
   <li>크로스 케이블</li>
   <li>데스크탑(64bit, Ubuntu 14.04 LTS)</li>
   <li>Raspberry Pi2</li>
   <li>WiFi USB 동글</li>
 </ol>

<h2>2. OpenWrt 설치</h2>

<p>
  <ol>
    <li>이미지 다운로드</li>
    <p>
      <ul>
        <p>
          <li>커스텀 이미지</li>
          본 예제에서 사용된 이미지입니다. <a href="http://dab-embedded.com/en/blogs/openwrt-on-arm-based-platform-raspberry-pi-2/">여기</a> 하단부에서 받으실 수 있습니다. <br />
          각종 유틸리티와 무선 랜 드라이버가 기본 내장되있으므로 <strong> 본 이미지를 사용한다고 가정하고 진행하겠습니다. </strong> <br />
          혹은 커스텀 이미지를 직접 빌드 <a href="http://dab-embedded.com/en/blogs/openwrt-on-arm-based-platform-raspberry-pi-2/">여기</a>를 참조 하세요.
        </p>

        <p>
          <li>공식 배포 이미지</li>
          <a href="https://downloads.openwrt.org/">여기</a>에서 필요한 이미지를 받을 수 있습니다.<br />
          Raspberry Pi1의 경우: brcm2708/brcm2708/ <br />
          Raspberry Pi2의 경우: brcm2708/brcm2709/ <br />
          하위 디렉토리에서 해당 sdcard 이미지를 받을 수 있습니다. 해당 이미지를 사용할 경우 <br />
          드라이버 등을 따로 잡아줘야 할 경우가 있습니다.
        </p>
        
      </ul>
    </p><!-- 이미지 다운로드-->

    <p>
    </p>

    <p>
      <li>sd 카드에 이미지 넣기</li>
      <a href="https://www.raspberrypi.org/documentation/installation/installing-images/linux.md">여기</a>를 참조하세요.<br />
    </p>
    
  </ol>

  더욱 자세한 정보를 얻으실려면 <a href="http://wiki.openwrt.org/toh/raspberry_pi_foundation/raspberry_pi">OpenWrt Wiki/Raspberry Pi</a> 항목을 참조 하십시오.
</p>

<h2>3. 로그인 및 초기 설정</h2>

<h3>3.1 로그인</h3>
<span class="TERM">OpenWrt</span> 최초 설치 되었을 때 기본값으로 192.168.1.1의 주소를 갖습니다. <br />
또한 root 계정의 비밀번호를 변경 하기 전까지 ssh 접속과 https 접속이 불가능 합니다. <br />
더욱 자세한 정보는 <a href="http://wiki.openwrt.org/doc/howto/firstlogin">firstlogin</a> 를 참조하시길 바랍니다. <br />

<ol>
  <p>
    <li>호스트 컴퓨터와 Raspberry Pi를 크로스 케이블로 연결합니다. </li>
    일반적인 USB 키보드로는 입력이 불가능하므로 크로스 케이블을 이용하여 로그인 하는 법을 설명합니다. <br />
    (<a href="https://shop.pimoroni.com/products/usb-to-uart-serial-console-cable">UART-USB console cable</a> 가지고 계시면 바로 로그인 하면 됩니다.)
  </p>

  <p>
    <li>호스트 컴퓨터의 네트워크 설정을 바꾸어 줍니다.</li>
    <p class="console">
MCLAB@KHU:~$ sudo service network-manager stop
MCLAB@KHU:~$ sudo ifconfig eth0 192.168.1.2/24
    </p>
  </p>

  <p>
    <li>telnet을 이용하여 접속을 합니다.</li>
    앞에 언급했듯, <span class="TERM">OpenWrt</span>는 최초 로그인시 192.168.1.1의 주소를 갖습니다. <br />
    telnet을 이용하여 아래와 같이 접속을 시도 합니다.<br />
    <p class="console">
MCLAB@KHU:~$ telnet 192.168.1.1
    </p>

    아래와 같은 화면이 출력되면 성공입니다.
    <p class="console">
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
 === IMPORTANT ============================
  Use 'passwd' to set your login password
  this will disable telnet and enable SSH
 ------------------------------------------


BusyBox v1.23.2 (2015-07-03 17:09:21 CEST) built-in shell (ash)

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 CHAOS CALMER (15.05-rc3, r46163)
 -----------------------------------------------------
  * 1 1/2 oz Gin            Shake with a glassful
  * 1/4 oz Triple Sec       of broken ice and pour
  * 3/4 oz Lime Juice       unstrained into a goblet.
  * 1 1/2 oz Orange Juice
  * 1 tsp. Grenadine Syrup
 -----------------------------------------------------

root@openwrt:~$
    </p>
  </p>

  <p>
    <li>패스워드를 바꿔 줍니다.</li>
    <p class="console">
root@openwrt:~$ passwd
Changing password for root
New password:
Retype password:
Password for root changed by root
root@openwrt:~$
</p>
  </p>

이제부터 telnet으로 접속 할 수 없으며, ssh 나 웹인터페이스로 접속해서 <br />
설정 해야 합니다.

</ol>

<h3>3.2 필수 패키지 설치</h3>
<p>
2.1.1에서 사용한 커스텀 이미지를 이용하면, 대부분의 드라이버를 포함하고 있습니다. <br />
따라서 드라이버를 제외한, 필수 패키지인 hostapd 경량버전인 <strong>wpad</strong>를 설치 하도록 합니다. <br />

  <ol>
    <p>
      <li>wpad 다운로드</li>
      호스트 컴퓨터에서 <a href="https://downloads.openwrt.org/chaos_calmer/15.05-rc3/brcm2708/bcm2709/packages/base/wpad_2015-03-25-1_brcm2708.ipk">wpad (for Raspberry Pi2 package)</a>를 받습니다.
    </p>

    <p>    
      <li>Raspberry Pi2로 업로드 하기</li>
      호스트 컴퓨터에서 Raspberry Pi2 wpad를 업로드 합니다.
      <p class="console">
MCLAB@KHU:~$ scp ./wpad_2015-03-25-1_brcm2708.ipk root@192.168.1.1:/
      </p>
    </p>

    <p>
      <li>wpad 설치하기</li>
      <ol>
        <li> wpa_supplicant 이름을 변경합니다.(파일 충돌 예방) </li>
        <li> opkg를 통하여 wpad를 설치합니다. </li>
      </ol>
      <p class="console">
MCLAB@KHU:~$ ssh root@192.168.1.1
BusyBox v1.22.1 (2015-03-29 09:12:13 PDT) built-in shell (ash)
Enter 'help' for a list of built-in commands.

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 CHAOS CALMER (Bleeding Edge, r45140)
 -----------------------------------------------------
  * 1 1/2 oz Gin            Shake with a glassful
  * 1/4 oz Triple Sec       of broken ice and pour
  * 3/4 oz Lime Juice       unstrained into a goblet.
  * 1 1/2 oz Orange Juice
  * 1 tsp. Grenadine Syrup
 -----------------------------------------------------

root@OpenWrt:/# mv /usr/sbin/wpa_supplicant /usr/sbin/wpa_supplicant2
root@OpenWrt:/# opkg install wpad_2015-03-25-1_brcm2708.ipk 
Installing wpad (2015-03-25-1) to root...
Configuring wpad.
root@OpenWrt:/#
      </p>
    </p>
  </ol>
</p>

<h2>4. 공유기로 설정</h2>

<p>
<span class="TERM">OpenWrt</span>의 주요 설정은 UCI System(<em><strong><em class="u">U</em></strong>nified <strong><em class="u">C</em></strong>onfiguration <strong><em class="u">I</em></strong>nterface</em>)의해 이루어 집니다.  <br />
UCI System의 모든 설정 파일은 /etc/config/ 디렉토리 아래에 있습니다. <br />
<br />
!!! 만약 <em>linux physical interface</em> 및 <em>linux virtual interface</em>를 모르신다면 <a href="http://wiki.openwrt.org/doc/networking/network.interfaces">여길</a> 를 먼저 참조하세요. <br />

</p><!-- 4. 공유기로 설정 -->

<h3>4.1 /etc/config/wireless 설정</h3>

<span class="TERM">iw</span>를 대체하는 UCI System 설정파일로, 무선 랜 어뎁터를 설정합니다. <em>wifi-device</em>, <em>wifi-iface</em> 두 가지 항목을 수정해야합니다. <br />
아래는 무선 랜 동글을 연결 했을 때 기본으로 작성되어있는 기본 설정입니다.
<p class="console">
config wifi-device  radio0
        option type     mac80211
        option channel  11
        option hwmode   11g
        option path     'platform/bcm2708_usb/usb1/1-1/1-1.5/1-1.5:1.0'
        option htmode   HT20
        # REMOVE THIS LINE TO ENABLE WIFI:
        option disabled 1

config wifi-iface
        option device   radio0
        option network  lan
        option mode     ap
        option ssid     OpenWrt
        option encryption none
</p>
<p>

<ol>
  <li>wifi-device section</li>
  <em>linux physical interface</em> 에 해당 하는 것으로, iw phy 명령을 통해서 확인 할 수 있습니다. <br />
  이 항목은 시스템에 의해서 설정 파일에 자동으로 type, hwmode, path, htmode 등이 설정되므로 channel의 정보만 수정해 봅시다.<br />

<p class="console">
config wifi-device  radio0
        option type     mac80211
        option channel  6
        option hwmode   11a
        option path     pci0000:00/0000:00:00.0
        option htmode   HT20
</p><!-- console -->

<p>
  <dl>
    <dt>config wifi-device</dt>
    <dd>UCI System 내부적으로 사용될 wifi-device타입의 식별자를 정의합니다.</dd>

    <dt>option channel</dt>
    <dd>어뎁터가 사용할 물리 체널을 지정합니다.</dd>

    <dt>option disabled</dt>
    <dd>디바이스의 기본 동작을 '끔' 상태로 설정합니다. 해당 항목은 삭제합니다.</dd>
  </dl>
</p>

  <li>wifi-iface section</li>
  <em>linux virtual interface</em> 에 해당하는 것으로, iw dev info 통해서 확인 할 수 있습니다.<br />
  무선 설정은 /etc/config/wireless 수정을 통해 이루어 집니다. 아래와 같이 추가 합니다.<br />

<p class="console">
config wifi-iface
        option device   radio0
        option network  wlan
        option mode     ap
        option ssid     OpenWrt
        option encryption none
</p><!-- console -->

<p>
  <dl>
    <dt>option device</dt>
    <dd>wifi-device section에 정의된 wifi-device타입의 식별자를 지정합니다.</dd>

    <dt>option network</dt>
    <dd>무선에 연결될 interface 식별자를 지정합니다. interface는 다음 항목에서 자세히 설명됩니다.</dd>

    <dt>option mode</dt>
    <dd>무선 어뎁터의 동작 모드를 지정합니다. 공유기로 사용 될 것이므로 ap라고 적습니다.</dd>

    <dt>option ssid</dt>
    <dd>무선 공유기가 브로드캐스트 할 ssid를 지정합니다. </dd>

    <dt>option encryption</dt>
    <dd>암호화 방식을 선택합니다. 간단한 예제이므로 none으로 설정합니다.</dd>
  </dl>
</p>

다른 상세한 옵션을 보려면 <a href="http://wiki.openwrt.org/doc/uci/wireless">여기</a>를 참고하세요.
  
</ol>

</p><!-- 4.1 /etc/config/wireless 설정 -->

<h3>4.2 /etc/config/network 설정</h3>
<span class="TERM">ifconfig</span>를 대체하는 UCI System 설정파일로, 네트워크를 설정합니다. <br />

<p class="console">
config interface lan
        option ifname eth0
        option proto static
        option ipaddr 192.168.1.1
        option netmask 255.255.255.0

config interface wlan
        option proto static
        option ipaddr 192.168.100.1
        option netmask 255.255.255.0
</p><!-- console -->

<p>
  <dl>
    <dt>config interface</dt>
    <dd>UCI System 내부적으로 사용되는 interface 타입의 식별자 이름입니다</dd>

    <dt>option ifname</dt>
    <dd>ethernet의 경우 물리 인터페이스를 지정합니다.</dd>
    <dd>wifi의 경우 생략합니다.</dd>

    <dt>option proto</dt>
    <dd>static 혹은 dhcp(or dhcpv6)가 될 수 있습니다.</dd>

  </dl>
</p>

<h3>4.3 /etc/config/dhcp 설정</h3>
DHCP 설정입니다. 할당해 줄 주소의 범위와 시간 등을 지정해 줄 수 있습니다.

<p class="console">
config dhcp wlan
        option interface   wlan
        option start       100
        option limit	   150
        option leasetime   12h
</p><!-- console -->

<h3>4.4 /etc/config/firewall</h3>
<p>
<span class="TERM">iptables</span>를 대체하는 UCI System 설정파일로, 방화벽 설정 및 NAT 설정합니다. 이번 항목은 공유기로 작동하기 위한<br />
기본 설정만 하도록 하고 자세한 설명은 생략 하도록 하겠습니다.<br />
만약 보다 상세한 설정을 하시려면 <a href="http://wiki.openwrt.org/doc/uci/firewall"><em>Firewall configuration</em></a>을 참조하십시오.

<p class="console">
config zone
        option name       lan
        list   network    lan
        option input      ACCEPT
        option output     ACCEPT
        option forward    REJECT

config zone
        option name       wlan
        list   network    wlan
        option input      ACCEPT
        option output     ACCEPT
        option forward    REJECT
        option masq       1 

config forwarding
        option src        lan
        option dest       wlan

config forwarding
        option src        wlan
        option dest       lan 
</p><!-- console -->

</p>

<h2>5. 변경사항 적용 </h2>
쉘에서 아래와 같이 입력합니다.
<p class="console">
root@openwrt:~$ ifdown wlan
root@openwrt:~$ ifup wlan
root@openwrt:~$ /etc/init.d/firewall restart
root@openwrt:~$ /etc/init.d/dnsmasq restart
</p>

<p>
!!!주의: <em>ifdown</em>, <em>ifup</em> 뒤의 인자는 4.2절에서 정의된 식별자입니다. 커널 인터페이스 이름이 아닙니다. <br />
</p>

<h2>6. 동작 확인</h2>

<p>
Raspberry Pi2의 OpenWrt상에서 제대로 동작하는지 확인해 봅시다.

<p class="console">
root@OpenWrt:/etc/config# iw dev
phy#0
        Interface wlan0
                ifindex 10
                wdev 0x7
                addr 00:e0:4d:a0:30:65
                ssid OpenWrt
                type AP
                channel 11 (2462 MHz), width: 20 MHz, center1: 2462 MHz
</p>


</body>

