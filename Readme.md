## Abstract
Web scanners and bots automatically attack every website they can get access to without the owners even knowing.
This can lead to security issues like data leaks, website takeovers and more.
However, most attacks can be prevented by blocking unnatural user behavior.
Blocking strange users can lead to a safer website.
## Web scanners and bot attacks - what are they how do they work?
Web scanners, also knows as bots, are automated scripts that try to find vulnerabilities in as many websites as possible, be it a PHP website or some other type.
Looking through the logs of our Hacker News close, we discovered a few "users" that were trying to access pages that they shouldn't. A couple of exameples from the logs:
* [client 178.62.195.55:44528] script '/var/www/html/muieblackcat.php' not found or unable to stat
*  [client 80.211.230.161:33248] script '/var/www/html/w00tw00t.at.blackhats.romanian.anti-sec:).php' not found or unable to stat
*  [client 52.91.213.21:65025] script '/var/www/html/phpmyadmin.php' not found or unable to stat
*  [client 52.91.213.21:49590] script '/var/www/html/mysql.php' not found or unable to stat

The question is: Why would a normal user want to access some pages that would never be on a Hacker News clone? Well, it's obvious that there are not normal users, so an investigation began to see what exactly is "muieblackhat" and "w00tw00t.at.blackhats.romanian.anti-sec:)".
Most malicious web scanners are looking for known vulnerabilities in services that your website might have, like phpMyAdmin. Fortunatelly, the bots that we encountered were targeting those old vulnerabilities that were patched.
First of all, the IPs seem to be from all over the world. By using an IP location checker website we found out that we have this kind of requests to our website from Italy, Netherlands, USA and Ukraine. Our theory is that some of the attackers are using VPNs to go around websites that block countries like Ukraine, China, and Russia which are famous for hacking attacks.
Muieblackcat is a vulnerability scanning product. Remote attackers can use Muieblackcat to detect vulnerabilities on a target server. 
The other one, w00tw00t.at.blackhats.romanian.anti-sec:), is part of a pack called "ZmEu attacks" and it does the same things as Muieblackcat.
The way these bots work is that they are looking for vulnerabilities, in this case, Php vulnerabilities. They are trying to access PhpMyAdmin, which is a MySQL administrator page. And they are also looking for any SQL page they can find and get some access to. In the logs, we saw 20 requests for 20 different pages that don't exist coming from a single IP address and every request was either related to PhpMyAdmin, SQL pages or admin pages. Fortunately enough, the server returned 404 for all of them. 
## What are the dangers
If one of those bots gets access to your PhpMyAdmin page then the whole database is compromised. They will get all the information they can get their hands on: usernames and passwords, emails, phone numbers.
If you have, somewhere on your server, a page that will run SQL commands, these scrips are made to find that page and run it leading to dropped databases, admin username and password reset and much more.
The problem with these kind of scrips is that you can only detect them after they tried to do their job in the logs or if they already did it and you can see that the database is empty or that your website was taken over by the owner of the script and in some cases it might even go unnoticed if the attacker stole the content of the database without modifying anything.
If a website is compromised in any of these ways and news gets out, users will no longer trust the website and will start avoiding it and it can lead to its death.
## How to defend yourself
Fortunately, there are a few simple ways to defend yourself against muieblackcat, ZmEu attacks and the other web scanners/robots out there:
* If you use PhpMyAdmin, make sure you're using a strong password and, more importantly, make sure that you whitelist the access to it. That way, only trusted persons should have access to the database.
* Don't have online any kind of page that runs SQL scripts because they will be found and run. If you're doing testing and you need scripts for that, host them in a closed environment where they cannot be accessed from the outside.
* Check your logs and block every suspicious IP. Even though, the attacker might have failed to do any kind of damage, he might return later from the same IP and try something stronger.

If your website is not intended to run any php scripts, the best way to defend against the bots that we encountered is to stop php processing in your server altogether.
On our Hacker News clone, we implemented two major things to deal with these attacks.
Firstly, we made a few fake pages which we found to be frequently accessed by these attackers and we made them redirect back to their IP address on the same page. In the following case "[client 178.62.195.55:44528] script '/var/www/html/muieblackcat.php' not found or unable to stat" our webpage will redirect to 178.62.195.55:44528/muieblackcat.php. In the future, we will make these pages to blacklist the bot that tries to access them.
Secondly, we modified our .htaccess file to give a 403 Forbidden error, instead of the 404. The .htaccess is a file executed by the server and it can alter the server's configuration. In this case we used it to alter the error that the bots are receiving. We are not sure if this will have any impact on the bots, but hopefuly it will send a message that they are not welcome on our Hacker News clone.
We simulated a bot by going to one of the pages that the real bots tried to access and we encountered a 403 Forbidden error instead of the 404.

## Conclusion
In conclusion, everyone should keep in mind that if a website is accessible from the outside, it will be attacked. You can prevent some of those attacks, by making sure that there are no loose ends and closing down everything that normal users are not supposed to access.
## Sources
 [https://fortiguard.com/encyclopedia/ips/40582](https://fortiguard.com/encyclopedia/ips/40582)
 [https://www.checkpoint.com/defense/advisories/public/2015/cpai-2015-1063.html](https://www.checkpoint.com/defense/advisories/public/2015/cpai-2015-1063.html)
 [https://eromang.zataz.com/2011/08/14/suc027-muieblackcat-setup-php-web-scanner-robot/](https://eromang.zataz.com/2011/08/14/suc027-muieblackcat-setup-php-web-scanner-robot/)
[https://www.symantec.com/security_response/attacksignatures/detail.jsp?asid=30258](https://www.symantec.com/security_response/attacksignatures/detail.jsp?asid=30258)
[https://serverfault.com/questions/309309/what-is-muieblackcat](https://serverfault.com/questions/309309/what-is-muieblackcat)
