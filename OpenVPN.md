# OpenVPN

1. > daemon
   >
   > dev tun0
   >
   > port 1194
   >
   > proto udp4
   >
   > ​
   >
   > user openvpn
   >
   > group openvpn
   >
   > ​
   >
   > persist-key
   >
   > persist-tun
   >
   > ​
   >
   > keepalive 10 120
   >
   > ​
   >
   > txqueuelen 1000
   >
   > sndbuf 786432
   >
   > rcvbuf 786432
   >
   > push "sndbuf 786432"
   >
   > push "rcvbuf 786432"
   >
   > ​
   >
   > float
   >
   > fast-io
   >
   > mssfix 0
   >
   > compress <!--Add lzo in some cases-->
   >
   > ​
   >
   > remote-cert-tls client
   >
   > ​
   >
   > topology subnet
   >
   > server 10.8.0.0 255.255.255.0
   >
   > push "dhcp-option DNS 84.200.69.80"
   >
   > push "dhcp-option DNS 84.200.70.40"
   >
   > push "redirect-gateway def1 bypass-dhcp" 
   >
   > ​
   >
   > crl-verify crl.pem
   >
   > ca ca.crt
   >
   > cert <main_server.crt-->
   >
   > key server.key
   >
   > ​
   >
   > tls-crypt static.key
   >
   > ​
   >
   > \#ifconfig-pool-persist /home/cipher/ipp.txt 300
   >
   > duplicate-cn
   >
   > ​
   >
   > ncp-ciphers AES-256-GCM:AES-128-GCM <!--Very strong and slow with old cpu, can replace with AES-128-CBC-->
   >
   > tls-server
   >
   > tls-version-min 1.2
   >
   > tls-cipher TLS-ECDHE-RSA-WITH-AES-256-GCM-SHA384:TLS-ECDHE-RSA-WITH-AES-128-GCM-SHA256
   >
   > dh none
   >
   > ecdh-curve secp384r1
   >
   > reneg-sec 1200
   >
   >  
   >
   > chroot /home/cipher/
   >
   > tmp-dir tmp/
   >
   > ​
   >
   > status s.log
   >
   > log v.log
   >
   > \#log /dev/null
   >
   > \#status /dev/null
   >
   > verb 3
   >
   > \#verb 0

2. sudo or root

   > addgroup --system --no-create-home --disabled-login openvpn
   > adduser --system --no-create-home --disabled-login openvpn
   >
   > adduser --disabled-login cipher
   >
   > mkdir /home/cipher/tmp; chmod 1777 /home/cipher/tmp
   >
   > nano /etc/fstab <!--add this --- tmpfs /home/cipher/tmp tmpfs defaults,noatime,nosuid,nodev,noexec,mode=1777,size=32M 0 0-->
   >
   > mount -a
   >
   > df -h

3. > nano /etc/sysctl.conf <!--uncomment this --- net.ipv4.ip_forward=1-->

4. > apt-install install ufw
   >
   > nano /etc/default/ufw <!--change this --- DEFAULT_FORWARD_POLICY --- ti this --- ”ACCEPT”-->
   >
   > nano /etc/ufw/before.rules

   ​	Should be:

   ```
   # rules.before


   # Rules that should be run before the ufw command line added rules. Custom

   # rules should be added to one of these chains:

   # ufw-before-input

   # ufw-before-output

   # ufw-before-forward


   # START OPENVPN RULES

   # NAT table rules

   *nat

   :POSTROUTING ACCEPT [0:0]

   -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j SNAT --to x.x.x.x

   COMMIT

   # END OPENVPN RULES
   ```

5. > ufw allow 1194/udp
   >
   > ufw allow 22/tcp
   >
   > ufw enable

6. > iptables -I FORWARD -i tun0 -o tun0 -j DROP
   >
   > apt-get install iptables-persistent

7. > git clone https://github.com/OpenVPN/easy-rsa.git
   >
   > cp -R easy-rsa/easyrsa3/ /etc/openvpn/
   >
   > cd /etc/openvpn/easyrsa3
   >
   > cp vars.example vars
   >
   > nano vars

   Should be:

   ```
   set_var EASYRSA "$PWD"
   set_var EASYRSA_OPENSSL "openssl"
   set_var EASYRSA_PKI "$EASYRSA/pki"
   set_var EASYRSA_DN "cn_only"
   set_var EASYRSA_KEY_SIZE 4096
   set_var EASYRSA_ALGO rsa
   set_var EASYRSA_CA_EXPIRE 3650
   set_var EASYRSA_CERT_EXPIRE 365
   set_var EASYRSA_CRL_DAYS 180
   set_var EASYRSA_SSL_CONF "$EASYRSA/openssl-1.0.cnf"
   set_var EASYRSA_DIGEST "sha512"
   ```

8. > ./easyrsa init-pki
   >
   > ./easyrsa build-ca
   >
   > ./easyrsa gen-req server nopass
   >
   > ./easyrsa gen-req client nopass

9. > ./easyrsa import-req pki/reqs/server.req main_server
   >
   > ./easyrsa import-req pki/reqs/client.req client1
   >
   > ​
   >
   > ./easyrsa sign server main_server
   >
   > ./easyrsa sign client client1
   >
   > ​
   >
   > ./easyrsa gen-crl
   >
   > ​
   >
   > cd /etc/openvpn
   >
   > openvpn --genkey --secret static.key

10. > cp easyrsa3/pki/issued/main_server.crt .
    >
    > cp easyrsa3/pki/private/server.key .
    >
    > cp easyrsa3/pki/ca.crt .
    >
    > cp easyrsa3/pki/crl.pem /home/cipher/
    >
    > ​
    >
    > chmod 400 *.key
    >
    > chmod 444 /home/cipher/crl.pem

11. > systemctl start openvpn@udp
    >
    > systemctl enable openvpn@udp

12. Client config:

    > client
    >
    >  
    >
    > dev tun
    >
    > proto udp4
    >
    > remote <VPS IP> 1194
    >
    >  
    >
    > resolv-retry infinite
    >
    > nobind
    >
    > persist-key
    >
    > persist-tun
    >
    > compress
    >
    > mute-replay-warnings
    >
    > reneg-sec 0
    >
    > explicit-exit-notify 3
    >
    >  
    >
    > ca ca.crt
    >
    > cert client1.crt
    >
    > key client.key
    >
    >  
    >
    > remote-cert-tls server
    >
    > tls-crypt static.key
    >
    >  
    >
    > \#block-outside-dns
    >
    > \# uncomment the above for Win8 or newer to prevent DNS leaks
    >
    >  
    >
    > verb 3
    >
    > \# change it to verb 0 once you have a stable client connection
    >
    >  
    >
    > \# user openvpn
    >
    > \# group openvpn

