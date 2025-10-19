# WordPress Enumeration

## Cheatsheet [](https://field-manual.brunorochamoura.com/manual/information-gathering/web-enumeration/common-web-app-enumeration/wordpress-enumeration/#cheatsheet)

```bash
wpscan --url http://alvida-eatery.local/  --enumerate vp,u,vt,tt --verbose -o target.log  --api-token   XmZXN8uqKwC7Mgk0PyisC6EUqAexnxteBjkxZssNCuY  
# OR 
wpscan --url http://<IP> --no-update -e vp,vt,tt,cb,dbe,u,m --plugins-detection aggressive --plugins-version-detection aggressive -f cli-no-color 2>&1 | tee "./scans/tcp80_wpscan.txt"
# # vp,vt (Vulnerable Plugins, Themes)	

# Enumeate Plugins
ffuf -c -w /usr/share/wordlists/wordpress-popular-plugins.txt -u http://<wordpress-root>/wp-content/plugins/FUZZ

#Accessing Wordpress shell
http://<IP>/retro/wp-admin/theme-editor.php?file=404.php&theme=90s-retro
http://<IP>/retro/wp-content/themes/90s-retro/404.php
```
The most effective way to gain administrative access to WordPress is by exploiting vulnerabilities in plugins and themes.  
Use [WPScan](https://github.com/wpscanteam/wpscan), as outlined in the enumeration notes, to identify vulnerable plugins and learn how to exploit them.
## Related Articles [](https://field-manual.brunorochamoura.com/manual/information-gathering/web-enumeration/common-web-app-enumeration/wordpress-enumeration/#related-articles)
- [WordPress Exploitation](https://field-manual.brunorochamoura.com/manual/exploitation/web-exploitation/common-web-app-exploitation/wordpress-exploitation/): After enumerating the plugins and themes, look into attack vectors.
## Overview [](https://field-manual.brunorochamoura.com/manual/information-gathering/web-enumeration/common-web-app-enumeration/wordpress-enumeration/#overview)

**[WordPress](https://wordpress.com/) is a popular open-source Content Management System (CMS) widely used for hosting blogs, forums, and even full-fledged company websites.**

However, its extensibility, particularly through third-party themes and plugins, often introduces vulnerabilities that attackers can exploit.

**The primary focus during enumeration should be on identifying vulnerable plugins**, as they represent the majority of vulnerabilities. WordPress stores plugins in the `wp-content/plugins` directory and themes in the `wp-content/themes` directory.

WordPress is most commonly found on ports 80 (HTTP) and 443 (HTTPS).

## User Roles [](https://field-manual.brunorochamoura.com/manual/information-gathering/web-enumeration/common-web-app-enumeration/wordpress-enumeration/#user-roles)

A standard WordPress installation includes five user roles:

1. **Administrator**: Has full administrative privileges, including adding/deleting users, posts, and editing the site’s source code. Access to this role is typically sufficient to gain full control of the server.
2. **Editor**: Can publish and manage posts, including those of other users.
3. **Author**: Can publish and manage their own posts.
4. **Contributor**: Can write and manage their own posts but cannot publish them.
5. **Subscriber**: Standard users with the ability to browse content and edit their profiles.


