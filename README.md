<p align="center">
<img width="300" src="https://imgur.com/4meuXng.jpg" /><br>
A Web Dashbord for Nmap XML Report 
</p>

## Table Of Contents
- [Usage](#usage)
- [Features](#features)
- [PDF Report](#pdf-report)
- [XML Filenames](#xml-filenames)
- [CVE and Exploits](#cve-and-exploits)
- [Network View](#network-view)
- [RESTful API](#restful-api)
- [Third Parts](#third-parts)
- [Security Issues](#security-issues)
- [Contributors](#contributors)
- [Contacts](#contacts)


## Screenshot
<img src="https://i.imgur.com/ELZfqd0.png" /><br>
<img src="https://i.imgur.com/KsBv1S0.png" /><br>
<img src="https://i.imgur.com/g27mcc3.png" /><br>
<br>

## Usage
After an image has been created through the Dockerfile, the following steps need to be completed.

## Step 0
- Goto : 
- WebMap>docker
- Open Dockerfile
- Comment vimrc line, save and exit
- docker build .
- docker tag image_id webmap

## Step 1 
docker images

<img src="https://imgur.com/n3falhS.png" /><br>

## Step 2 
docker run -d -p 8000:8000 webmap --name webmap-api

<img src="https://imgur.com/Oz7UFl1.png" /><br>

## Step 3
docker ps -a

<img src="https://imgur.com/t0rvbmJ.png" /><br>

## Step 4
docker exec -ti image_name /bin/bash

<img src="https://imgur.com/8M96kzx.png" /><br>

## Step 5: 
- cd /opt/nmapdashboard/nmapreport/ 
- chmod 777 token.py
- exit  
- docker exec -ti image_name /opt/nmapdashboard/nmapreport/token.py

<img src="https://imgur.com/uG0gqfU.png" /><br>

## Step 6:
Browse to <ip>:8000 or localhost:8000 and paste the token
         
```bash
$ mkdir /tmp/webmap
$ docker run -d \
         --name webmap \
         -h webmap \
         -p 8000:8000 \
         -v /tmp/webmap:/opt/xml \
         n00bie/webmap

$ # now you can run Nmap and save the XML Report on /tmp/webmap
$ nmap -sT -A -T4 -oX /tmp/webmap/myscan.xml 192.168.1.0/24
```
Now point your browser to http://localhost:8000

### Generate new token
In order to access to the WebMap dashboard, you need a token. You can create a new token with:
```bash

$ docker exec -ti <webmap/container id> /root/token
```


## Features
- Import and parse Nmap XML files
- Run and Schedule Nmap Scan from dashboard
- Statistics and Charts on discovered services, ports, OS, etc...
- Inspect a single host by clicking on its IP address
- Attach labels on a host
- Insert notes for a specific host
- Create a PDF Report with charts, details, labels and notes
- Copy to clipboard as Nikto, Curl or Telnet commands
- Search for CVE and Exploits based on CPE collected by Nmap
- RESTful API

## Roadmap for v2.3x
You love WebMap and you know python? We need your help! This is what we want deploy for the v2.3:
- [todo] Improve template: try to define better the html template and charts
- [todo] Improve API: create a documentation/wiki about it
- [todo] Wiki: create WebMap User Guide on GitHub
- [working] Authentication or something that could blocks access to WebMap if != localhost
- [working] Scan diff: show difference between two scheduled nmap scan report
- [todo] Zaproxy: Perform web scan using the OWASP ZAP API

## Changes on v2.2
- fixed bug on missing services
- Run nmap from WebMap
- Schedule nmap run
- Add custom NSE scripts section

## Changes on v2.1
- Better usage of Django template
- Fixed some Nmap XML parse problems
- Fixed CVE and Exploit collecting problems
- Add new Network View
- Add RESTful API

## PDF Report
![WebMap](https://i.imgur.com/alWZix9.png)

## XML Filenames
When creating the PDF version of the Nmap XML Report, the XML filename is used as document title on the first page. 
WebMap will replace some parts of the filename as following:

- `_` will replaced by a space (` `)
- `.xml` will be removed

Example: `ACME_Ltd..xml`<br>
PDF title: `ACME Ltd.`

## CVE and Exploits
thanks to the amazing API services by circl.lu, WebMap is able to looking for CVE and Exploits for each CPE collected by Nmap. 
Not all CPE are checked over the circl.lu API, but only when a specific version is specified 
(for example: `cpe:/a:microsoft:iis:7.5` and not `cpe:/o:microsoft:windows`).

## Network View
![WebMap](https://i.imgur.com/j77jQz9.png)

## RESTful API
From `v2.1` WebMap has a RESTful API frontend that makes users able to query their scan files with something like:

```bash
curl -s 'http://localhost:8000/api/v1/scan?token=<token>'

    "webmap_version": "v2.1/master",
    "scans": {
        "scanme.nmap.org.xml": {
            "filename": "scanme.nmap.org.xml",
            "startstr": "Sun Nov  4 16:22:46 2018",
            "nhost": "1",
            "port_stats": {
                "open": 42,
                "closed": 0,
                "filtered": 0
            }
        },
        "hackthebox.xml": {
            "filename": "hackthebox.xml",
            "startstr": "Mon Oct  8 20:56:32 2018",
            "nhost": "256",
            "port_stats": {
                "open": 67,
                "closed": 0,
                "filtered": 2
            }
        }
    }
}
```

A user can get information about a single scan by append to the URL the XML filename:

```bash
curl -v 'http://localhost:8000/api/v1/scan/hackthebox.xml?token=<token>'

{
    "file": "hackthebox.xml",
    "hosts": {
        "10.10.10.2": {
            "hostname": {},
            "label": "",
            "notes": ""
        },
        "10.10.10.72": {
            "hostname": {
                "PTR": "streetfighterclub.htb"
            },
            "label": "",
            "notes": ""
        },
        "10.10.10.76": {
            "hostname": {},
            "label": "",
            "notes": ""
        },
        "10.10.10.77": {
            "hostname": {},
            "label": "Vulnerable",
            "notes": "PHNwYW4gY2xhc3M9ImxhYmVsIGdyZWVuIj5SRU1FRElBVElPTjwvc3Bhbj4gVXBncmFkZSB0byB0aGUgbGF0ZXN0IHZlcnNpb24g"
        },
...
```

and he can get all information about a single host by append the IP address to URL:

```bash
curl -v 'http://localhost:8000/api/v1/scan/hackthebox.xml/10.10.10.87?token=<token>'

    "file": "hackthebox.xml",
    "hosts": {
        "10.10.10.87": {
            "ports": [
                {
                    "port": "22",
                    "name": "ssh",
                    "state": "open",
                    "protocol": "tcp",
                    "reason": "syn-ack",
                    "product": "OpenSSH",
                    "version": "7.5",
                    "extrainfo": "protocol 2.0"
                },
                {
                    "port": "80",
                    "name": "http",
                    "state": "open",
                    "protocol": "tcp",
                    "reason": "syn-ack",
                    "product": "nginx",
                    "version": "1.12.2",
                    "extrainfo": ""
                },
                {
                    "port": "8888",
                    "name": "sun-answerbook",
                    "state": "filtered",
                    "protocol": "tcp",
                    "reason": "no-response",
                    "product": "",
                    "version": "",
                    "extrainfo": ""
                }
            ],
            "hostname": {},
            "label": "Checked",
            "notes": "",
            "CVE": [
                {
                    "Modified": "2018-08-17T15:29:00.253000",
                    "Published": "2018-08-17T15:29:00.223000",
                    "cvss": "5.0",
                    "cwe": "CWE-200",
                    "exploit-db": [
                        {
                            "description": "OpenSSH 7.7 - Username Enumeration. CVE-2018-15473. Remote exploit for Linux platform",
                            "file": "exploits/linux/remote/45233.py",
                            "id": "EDB-ID:45233",
                            "last seen": "2018-08-21",
                            "modified": "2018-08-21",
                            "platform": "linux",
                            "port": "",
                            "published": "2018-08-21",
                            "reporter": "Exploit-DB",
                            "source": "https://www.exploit-db.com/download/45233/",
                            "title": "OpenSSH 7.7 - Username Enumeration",
                            "type": "remote"
                        },
                        {
                            "id": "EDB-ID:45210"
                        }
                    ],
                    "id": "CVE-2018-15473",
                    "last-modified": "2018-11-02T06:29:06.993000",
                    "metasploit": [
...
```

## Third Parts
- [Django](https://www.djangoproject.com)
- [Materialize CSS](https://materializecss.com)
- [Clipboard.js](https://clipboardjs.com)
- [Chart.js](https://www.chartjs.org)
- [Wkhtmltopdf](https://wkhtmltopdf.org)
- [API cve.circl.lu](https://cve.circl.lu)
- [vis.js](http://visjs.org/)

## Security Issues
This app is not intended to be exposed to the internet, but to be used as localhost web application. Please, **DO NOT expose** this app to the internet, use your localhost or, 
in case you can't do it, take care to filter who and what can access to WebMap with a firewall rule or something like that. 
Exposing this app to the whole internet could lead not only to a stored XSS but also to a leakage of sensitive/critical/private 
informations about your port scan. Please, be smart.

