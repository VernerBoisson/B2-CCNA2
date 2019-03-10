# TP 2 - Routage statique et services d'infra

[Enoncé du TP](https://github.com/It4lik/B2-Reseau-2018/blob/master/tp/2/README.md)

## I. Mise en place du lab

### 1. Création des VMs et adressage IP
### 2. Routage statique

Client 1 et Serveur 1 peuvent se ping

`[root@server1 ~]# ping client1`
`PING client1 (10.2.1.10) 56(84) bytes of data.`  
`64 bytes from client1 (10.2.1.10): icmp_seq=1 ttl=62 time=0.879 ms`  

`[root@client1 ~]# ping server1`  
`PING server1 (10.2.2.10) 56(84) bytes of data.`  
`64 bytes from server1 (10.2.2.10): icmp_seq=1 ttl=62 time=0.989 ms`  

Le ping de Routeur 1 à Serveur 1 ne fonctionne pas car le routeur envoie un ping au serveur mais le serveur n'a pas de route afin d'envoyer un pong au réseau 10.2.12.0 .

### 3. Visualisation du routage avec Wireshark

Entre les deux captures, les adresses MAC sont différentes. Elles sont différentes car le routeur change les adresses MAC afin d'envoyer le message au prochain voisin.

## II. NAT et services d'infra

### 1. Mise en place du NAT

Routeur 1 : 
`[root@router1 ~]# curl google.com`
```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>  
```

Routeur 2 :
`[root@router2 network-scripts]# curl google.com`
```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```  
Client 1 : 
`[root@client1 ~]# curl google.com`
```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```  
Serveur 1 :
`[root@server1 ~]# curl google.com`
```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```  

### 2. DHCP server

`[root@client2 ~]# ip a | grep "inet "`
```inet 127.0.0.1/8 scope host lo
inet 10.2.1.50/24 brd 10.2.1.255 scope global noprefixroute dynamic enp0s3
inet 10.2.1.51/24 brd 10.2.1.255 scope global secondary dynamic enp0s3
```
`[root@client2 ~]# curl google.com`
```
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```  

### 3. NTP server

Routeur 1 :
`[root@router1 ~]# chronyc sources`
```
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? enterprise.frmug.org          3   6     1     7  -1122ms[-1122ms] +/-   68ms
^? saphire.uk.to                 3   6     1     7  -1122ms[-1122ms] +/-   53ms
^? flightplandatabase.com        2   6     1     7  -1120ms[-1120ms] +/-   93ms
^? ovh.sqlserver.express         2   6     1     7  -1125ms[-1125ms] +/-   44ms
```
`[root@router1 ~]# chronyc tracking`
```
Reference ID    : 7F7F0101 ()
Stratum         : 10
Ref time (UTC)  : Sun Mar 10 20:40:22 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```
Routeur 2 :
`[root@router2 network-scripts]# chronyc sources`
```
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? obelix.fraho.eu               2   6     1    43  -1753ms[-1753ms] +/-   36ms
^? nsr2.neoserveur.fr            3   6     1    44  -1754ms[-1754ms] +/-   62ms
^? server.bertold.org            2   6     1    44  -1754ms[-1754ms] +/-   75ms
^? ns358812.ip-91-121-154.eu     2   6     1    44  -1761ms[-1761ms] +/-  103ms
```
`[root@router2 network-scripts]# chronyc tracking`
```
Reference ID    : 7F7F0101 ()
Stratum         : 10
Ref time (UTC)  : Sun Mar 10 20:55:26 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```
Serveur 1 : 
`[root@server1 ~]# chronyc sources`
```
210 Number of sources = 4
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^? ip139.ip-5-196-160.eu         2   6     1     5  -1594ms[-1594ms] +/-   30ms
^? ntp-3.arkena.net              2   6     1     6  -1593ms[-1593ms] +/-   66ms
^? pob01.aplu.fr                 2   6     1     6  -1593ms[-1593ms] +/-   70ms
^? 82-64-45-50.subs.proxad.>     1   6     1     7  -1591ms[-1591ms] +/-   33ms
```
`[root@server1 ~]# chronyc trackers`
```
Unrecognized command
[root@server1 ~]# chronyc tracking
Reference ID    : 7F7F0101 ()
Stratum         : 10
Ref time (UTC)  : Sun Mar 10 20:55:05 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```

### 4. Web server

`[root@client2 ~]# curl server1`
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }
            h1 {
                text-align: center;
                margin: 0;
                padding: 0.6em 2em 0.4em;
                background-color: #294172;
                color: #fff;
                font-weight: normal;
                font-size: 1.75em;
                border-bottom: 2px solid #000;
            }
            h1 strong {
                font-weight: bold;
                font-size: 1.5em;
            }
            h2 {
                text-align: center;
                background-color: #3C6EB4;
                font-size: 1.1em;
                font-weight: bold;
                color: #fff;
                margin: 0;
                padding: 0.5em;
                border-bottom: 2px solid #294172;
            }
            hr {
                display: none;
            }
            .content {
                padding: 1em 5em;
            }
            .alert {
                border: 2px solid #000;
            }

            img {
                border: 2px solid #fff;
                padding: 2px;
                margin: 2px;
            }
            a:hover img {
                border: 2px solid #294172;
            }
            .logos {
                margin: 1em;
                text-align: center;
            }
            /*]]>*/
        </style>
    </head>

    <body>
        <h1>Welcome to <strong>nginx</strong> on Fedora!</h1>

        <div class="content">
            <p>This page is used to test the proper operation of the
            <strong>nginx</strong> HTTP server after it has been
            installed. If you can read this page, it means that the
            web server installed at this site is working
            properly.</p>

            <div class="alert">
                <h2>Website Administrator</h2>
                <div class="content">
                    <p>This is the default <tt>index.html</tt> page that
                    is distributed with <strong>nginx</strong> on
                    Fedora.  It is located in
                    <tt>/usr/share/nginx/html</tt>.</p>

                    <p>You should now put your content in a location of
                    your choice and edit the <tt>root</tt> configuration
                    directive in the <strong>nginx</strong>
                    configuration file
                    <tt>/etc/nginx/nginx.conf</tt>.</p>

                </div>
            </div>

            <div class="logos">
                <a href="http://nginx.net/"><img
                    src="nginx-logo.png" 
                    alt="[ Powered by nginx ]"
                    width="121" height="32" /></a>

                <a href="http://fedoraproject.org/"><img 
                    src="poweredby.png" 
                    alt="[ Powered by Fedora ]" 
                    width="88" height="31" /></a>
            </div>
        </div>
    </body>
</html>
```