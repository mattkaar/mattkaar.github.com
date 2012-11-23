---
layout: post
title: "Addendum: Amazon EC2 VPN with Windows"
date: 2010-12-08 19:43
comments: true
categories: 
---

[@Mascasa](http://twitter.com/mascasa) recently wrote some excellent instructions on setting up OpenVPN using Amazon Web Services and getting connected with a Mac client:

[www.stratumsecurity.com/blog/2010/12/03/shearing-firesheep-with-the-cloud/](http://www.stratumsecurity.com/blog/2010/12/03/shearing-firesheep-with-the-cloud/)

It turns out connecting with Windows is easy too. Here's how I did it…

1. Download and install the **OpenVPN GUI for Windows** from [here](http://openvpn.se/download.html) (the full Installation Package). Accept all the defaults when installing.
2. Download the same keys and certificates used in the Mac setup (**ca.crt**, **ta.key**,***yourname*.crt** and ***yourname*.key**) to your Windows machine. You might need a client like [WinSCP](http://winscp.net/eng/download.php) to pull them off the EC2 server.
3. Download the client template [here](http://www.stratumsecurity.com/sites/default/files/ec2.conf) and rename it to **EC2_VPN.ovpn**. Open it in WordPad and change the hostname and cert/key filenames (lines 42 and 89-90).
4. Copy all of the files (*.key, *.crt and *.ovpn) to the OpenVPN configuration directory (usually **\Program Files\OpenVPN\config**).
5. Right-click on the OpenVPN icon in the system tray and click **Connect**.

{% img /images/mk/openvpn_connect.png %}

That's it. Visit [IPChicken](http://ipchicken.com/) and check that your Amazon IP appears. And leave a comment if you run into problems.
