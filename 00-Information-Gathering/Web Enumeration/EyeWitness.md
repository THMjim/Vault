# EyeWitness

## Cheatsheet 

| **Action**                                                                        | **Description**                                                                                                                      |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `EyeWitness -f urls.txt -d <directory-to-be-created>`                             | Takes screenshots from a file containing a list of URLs and saves the results in the specified directory.                            |
| `EyeWitness -x nmap.xml -d <directory-to-be-created>`                             | Processes the XML output from an Nmap scan to take screenshots of discovered hosts and saves the results in the specified directory. |
| `EyeWitness --web --proxy-ip 127.0.0.1 --proxy-port 8080 --proxy-type socks5 ...` | Captures screenshots of websites by routing traffic through a proxy, supporting various proxy types (e.g., SOCKS5).                  |
| `EyeWitness ... --delay <seconds>`                                                | Adds a delay before capturing screenshots, useful for loading slow or heavy web pages.                                               |
## Overview

**EyeWitness is a tool designed to automate the process of capturing screenshots of websites, collecting server header information, and identifying default credentials when applicable.**

This tool is particularly useful when handling large lists of targets, such as after an [Nmap](https://field-manual.brunorochamoura.com/manual/information-gathering/service-enumeration/tools/nmap/) scan or when working with bulk domain data.

The generated HTML report, complete with screenshots, provides a visual representation of the targets, making it easier to identify interesting services, misconfigurations, or vulnerabilities worth further investigation.