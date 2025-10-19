---
type: notes
tags:
  - THM
  - Filtering
  - WWW
  - js
  - javascript
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabilities/Magic Numbers
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---

# Task 9
As mentioned previously, ***magic numbers*** are used as a more accurate identifier of files. The magic number of a file is a string of hex digits, and is always the very first thing in a file. Knowing this, it's possible to use magic numbers to validate file uploads, simply by reading those first few bytes and comparing them against either a whitelist or a blacklist
* List of magic numbers : https://en.wikipedia.org/wiki/List_of_file_signatures  

## Hexedit
NOTE: in the `hexedit` tool, use ***CTRL+X*** to save and exit after making changes.  
Use the Linux `file`command, to check the file type of our reverse shell php file.  
Open the file in a text editor, and add 4 As "AAAA", to the top of the file.  
Now open this file in `hexeditor`   or `hexedit`
![[7 - Magic Numbers-4aaaas.png]]
Change this to the magic number we found earlier for JPEG files: `FF D8 FF DB`  
Use the Linux `file` command on the php file, it should show JPEG image data type.

# Challenge
Directory indexing has been turned off, so you will need to access the shell directly using its URI.  

Bypass the magic number filter to upload a shell. Find the location of the uploaded shell and activate it. Your flag is in `/var/www/`.


Website is : magic.uploadvulns.thm
Flag : Grab the flag from /var/www/

Navigated to the website, and attempted to upload 1.php.
Got an error saying only GIFS.
* Found the GIF magic # string from the WIKI   .`47 49 46 38 37 61`  
* used `hexedit` to add that to the front of our 1.php file,
* Validated that it lists as a gif now with the file command![[7 - Magic Numbers-GIF really!.png]]
* Now file successfully uploaded
* file is located at `http://magic.uploadvulns.thm/graphics/1.php`
* Trying to execute the file outright didn't work, just got  garbage back.. 
* *OOOOPS*, when I edited the FRONT of the file, it REMOVED the php html flag that was supposed to be there.. 
* Adding that back and everything worked fine.. 

FLAG : THM{MWY5ZGU4NzE0ZDlhNjE1NGM4ZThjZDJh}
