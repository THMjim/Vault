# NOTES

- https://www.brunorochamoura.com/posts/cpts-report/
- https://field-manual.brunorochamoura.com/manual/information-gathering/active-directory-enumeration/users-enumeration/

## KB Note structures
- https://www.youtube.com/watch?v=7LU6m_CF3cQ
- examples : https://www.brunorochamoura.com/posts/cpts-tips/field-manual-structure.png

[Knowledge Base]
├── 01_Initial_Access
│   └── SQL_Injection_Techniques.md
├── 02_Enumeration_Services
│   ├── 21_FTP.md 
│   ├── 22_SSH.md
│   ├── 80_HTTP.md
│   └── 445_SMB.md
└── 03_Privilege_Escalation
    └── Linux_Sudo_Exploits.md

## Images
[Knowledge Base]
├── _assets_ 
│   ├── sql_injection_flow.png
│   ├── ftp_auth_diagram.jpg
│   └── linux_sudo_example.gif
├── 01_Initial_Access
│   └── SQL_Injection_Techniques.md
... (other folders)


![SQL Injection Flowchart](..\_assets_\sql_injection_flow.png)


[Knowledge Base]
├── 01_Initial_Access
│   ├── images
│   │   └── sql_injection_flow.png
│   └── SQL_Injection_Techniques.md
├── 02_Enumeration_Services
│   ├── images
│   │   └── ftp_auth_diagram.jpg
│   ├── 21_FTP.md 
...
![SQL Injection Flowchart](images/sql_injection_flow.png)
---

## CTF BOX folder
This is the most common and robust approach, often used with Obsidian or Joplin as the editor:
Create a folder on your Host OS (e.g., C:\Users\You\Pentest_Notes).
Share this folder with the Kali VM using your VM software's Shared Folders feature (e.g., VMware Shared Folders or VirtualBox Guest Additions).
Use a Markdown editor (Obsidian, VS Code, Joplin) on the Host OS to write your notes, but set the notes location to the shared folder.
Access the shared folder from inside your Kali VM (often mounted under /mnt/hgfs/ or /media/sf_) to quickly copy/paste commands and large tool output.
Bonus: Use a Git client on the host to sync this shared folder to a private GitLab/GitHub repository for cloud backup and version control.



    [OSCP-A]
├── 01_Work_Log / Scratchpad/
│   ├── MS01_192.168.1.141.md (or WorkLog.txt - detailed notes of everything you run)
│   ├── MS02_10.10.10.142.md (or WorkLog.txt - detailed notes of everything you run)
│   └── Todo.txt (or Checklist.md - to track pending tasks)
├── 02_Tools_Output / Raw_Data/
│   ├── nmap_full.txt
│   ├── nmap_fast.txt
│   ├── gobuster_output.txt
│   └── /winpeas_raw/ (folder for large tool dumps)
├── 03_Loot / Findings/
│   ├── Credentials.txt (usernames, hashes, passwords)
│   ├── Flags.txt (user.txt and root.txt flags)
│   └── Hashes.txt
│   └── MS01-SAM-Dump.txt
│   └── MS02-SAM-Dump.txt
│   └── Evidence_Screenshots/ (Final screenshots of fl
└── 04_Scripts / Payloads/
    ├── exploit.py
    ├── reverse_shell_one_liners.txt
    └── winPEAS.exe



