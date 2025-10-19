---
type: notes
tags:
  - THM
  - Filtering
  - WWW
reference: https://tryhackme.com/r/room/uploadvulns
title: Upload Vulnerabilities/Filtering
difficulty: easy
date: 2024-11-30
author: Jim Heringer
---

# Task 6

Filtering is a defence mechanisms used to prevent malicious file uploads.  
Two types of filtering: _client_-side filtering and _server_-side filtering.  
## Filtering
### Client-Side
It's running in the user's browser as opposed to on the web server itself. This means that the filtering occurs before the file is even uploaded to the server.
* JavaScript is pretty much ubiquitous as the client-side scripting language, although alternatives do exist.
Because the filtering is happening on _our_ computer, it is trivially easy to bypass.  
### Server-Side 
A _server_-side script will be run on the server. 
* Traditionally PHP was the predominant server-side language
Server-side filtering tends to be more difficult to bypass, as you don't have the code in front of you. As the code is executed on the server, in most cases it will also be impossible to bypass the filter completely; instead we have to form a payload which conforms to the filters in place, but still allows us to execute our code.
## Filtering Types
### Extension Validation
File extensions used to validate contents of file.  They work in two ways. They either _blacklist_ extensions (i.e. have a list of extensions which are **not** allowed) or they _whitelist_ extensions (i.e. have a list of extensions which **are** allowed, and reject everything else).

### File Type 
File type filtering looks, once again, to verify that the contents of a file are acceptable to upload. Similar to Extension validation, but more intensive.  

#### MIME validation:
 MIME (**M**ultipurpose **I**nternet **M**ail **E**xtension) types are used as an identifier for files -- originally when transfered as attachments over email, but now also when files are being transferred over HTTP(S). The MIME type for a file upload is attached in the header of the request, and looks something like this:  
![[4 - Filtering-content.png]]
MIME types follow the format  type/subtype. In the request above, you can see that the image "spaniel.jpg" was uploaded to the server. As a legitimate JPEG image, the MIME type for this upload was "image/jpeg". The MIME type for a file can be checked client-side and/or server-side; however, as MIME is based on the extension of the file, this is extremely easy to bypass.    
#### Magic Number validation 
Magic numbers are the more accurate way of determining the contents of a file; although, they are by no means impossible to fake. The "magic number" of a file is a string of bytes at the very beginning of the file content which identify the content. For example,
![[4 - Filtering-magicnumber.png]]
Unlike Windows, Unix systems use magic numbers for identifying files; however, when dealing with file uploads, it is possible to check the magic number of the uploaded file to ensure that it is safe to accept. This is by no means a guaranteed solution, but it's more effective than checking the extension of a file.
### File Length 
File length filters are used to prevent huge files from being uploaded to the server via an upload form (as this can potentially starve the server of resources)As an example, our fully fledged PHP reverse shell from the previous task is 5.4Kb big -- relatively tiny, but if the form expects a maximum of 2Kb then we would need to find an alternative shell to upload.
### File Name
Files uploaded to a server should be unique. Additionally, file names should be sanitized on upload to ensure that they don't contain any "bad characters", which could potentially cause problems on the file system when uploaded (e.g. null bytes or forward slashes on Linux, as well as control characters such as `;` and potentially Unicode characters). What this means for us is that, on a well administered system, our uploaded files are unlikely to have the same name we gave them before uploading, so be aware that you may have to go hunting for your shell in the event that you manage to bypass the content filtering.

### File Content
More complicated filtering systems may scan the full contents of an uploaded file to ensure that it's not spoofing its extension, MIME type and Magic Number. This is a significantly more complex process than the majority of basic filtration systems employ, and thus will not be covered in this room.

___Any of these filters can all be applied client-side, server-side, or both.___

Similarly, different frameworks and languages come with their own inherent methods of filtering and validating uploaded files. As a result, it is possible for language specific exploits to appear; for example, until PHP major version five, it was possible to bypass an extension filter by appending a null byte, followed by a valid extension, to the malicious `.php` file. More recently it was also possible to inject PHP code into the exif data of an otherwise valid image file, then force the server to execute it.