# Fail2Ban

Fail2Ban watches your auth.log for failed auth attempts. When several failed attempts are detected in a range of time... F2B will create a temporary iptables chain to drop request from the offending IP and email you a notice. The number of attempts and duration of the ban is configurable.

### *Install & Config procedures:*

1. update apt `sudo apt-get update`
2. install fail2ban `sudo apt-get install fail2ban`
3. To receive emailed alerts `sudo apt-get install sendmail`
4. copy the jail config (make edits to jail.local) `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
5. open jail.local to make your edits `sudo nano /etc/fail2ban/jail.local`
6. add any ip's you want F2B to ignore (list is space delimited) `ignoreip = 127.0.0.1/8 192.168.14.2`
7. config your ban time (how long the ip is banned), I made mine 1 day `bantime = 86400`
8. config the find time (window of detection, 3 attempts in this window will result in ban). Default is 10 minutes `findtime = 600`
9. config max retry (number of failed attempts before a ban is triggered) `maxretry = 3`
10. email config, set the email address to receive alerts `destemail = your_email@example.com`
11. who you want the alerts to come from `sendername = Fail2Ban`
12. config F2B to use sendmail `mta = sendmail`
13. if you want email alerts, change `action = $(action_)s` -to- `action = $(action_mwl)s` otherwise... just leave it alone. The default is to ban without emailing.
14. reload the fail2ban service to activate your changes `sudo service fail2ban reload`
15. verify your iptables rules have updated `sudo iptables -S`

*should look something like:*  
```
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-N fail2ban-nginx-http-auth
-N fail2ban-ssh
-A INPUT -p tcp -m multiport --dports 80,443 -j fail2ban-nginx-http-auth
-A INPUT -p tcp -m multiport --dports 22 -j fail2ban-ssh
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -j DROP
-A fail2ban-nginx-http-auth -j RETURN
-A fail2ban-ssh -j RETURN
```
