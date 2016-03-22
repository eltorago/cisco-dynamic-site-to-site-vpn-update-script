# cisco-vpn-script
Automating a site-to-site VPN with Cisco equipment where both ends have dynamic ip addresses.

## Requirements:

- Racnid or clogin
- a domain or hosted dynDNS
- DNS hosting if you have a domain - I recommend dns.he.net for free hosting

This is my first script so I know it is going to look terrible and it will be very inefficient. My plans are to make a TCL version so that I can have it run directly on the router and have it triggered from syslog rather than it relying on a cron timer.
