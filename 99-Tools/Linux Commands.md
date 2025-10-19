# Linux Commands
## Delete files with special characters
```bash
ls -ali  # Find Inode on far left column
find . -inum <inode ID of your file> -exec rm -i {} \;
find . -inum 266880 -exec rm -i {} \;
```

## wget Cclone Website
```bash
# from PEN200, Phishing basics - 11.3 listing 4
 wget -E -k -K -p -e robots=off -H -Dzoom.us -nd "https://zoom.us/signin#/login"

```
- if cloning, you need to clean it up, and possibly and response pages
- check what happens in form submittal
- Possibly aslo remove security measures
- remove the OWASP CSRFGuard code, which presented an alert box when we first loaded the page.
- `grep "csrf_js" *`