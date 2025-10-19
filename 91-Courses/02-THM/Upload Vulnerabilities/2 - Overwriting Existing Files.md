---
type: notes
tags:
  - THM
  - www
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabiliites/Overwriting Existing Files
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---
# Task 4
When files are uploaded to the server, a range of checks should be carried out to ensure that the file will not overwrite anything which already exists on the server.  
If, however, no such precautions are taken, then we might potentially be able to overwrite existing files on the server. Realistically speaking, the chances are that file permissions on the server will prevent this from being a serious vulnerability.

Viewing source code for web page, you might see : 
![[2 - Overwriting Existing Files-dog-image.png]]
Inside the red box, we see the code that's responsible for displaying the image that we saw on the page. It's being sourced from a file called "spaniel.jpg", inside a directory called "images".

Now we know where the image is being pulled from -- can we overwrite it?
## Task Questions 1
Q: What is the name of the image file which can be overwritten?    
A: mountains.jpg  
![[2 - Overwriting Existing Files-question 1.png]]

## Task Questions 2
Overwrite the image. What is the flag you receive?
Well done -- you overwrote the file!
Your flag is: THM{OTBiODQ3YmNjYWZhM2UyMmYzZDNiZjI5} 

