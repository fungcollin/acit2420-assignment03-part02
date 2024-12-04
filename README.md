# ACIT 2420 - Assignment 3 - Part Two: Nginx and Load Balancing

This repository contains an extension of part one instructions to create a load balancer: [Assignment Three - Part Two](#part-two)

## Introduction 

This repository will teach you how to setup an index.html file to run a designated script at a determined time using systemd service and timer. It will also feature the use of nginx and UFW firewall enabling.

## Table of Contents

- [Assignment Part One Instructions](#assignment-instructions)
    - [Task 1: System User Creation and Ownership](#task-1-system-user-creation-and-ownership)
    - [Task 2: Create .service and .timer Scripts](#task-2-create-service-and-timer-scripts)
    - [Task 3: Modify nginx.conf](#task-3-modify-nginxconf)
    - [Task 4: UFW](#task-4-setup-ufw)
    - [Task 5: Example Screenshot](#task-5-screenshot-of-droplet)    
- [Assignment Part Two Instructions](#Temp)
    - [Task 1: System User Creation and Ownership](#task-1-system-user-creation-and-ownership)
    - [Task 2: Create .service and .timer Scripts](#task-2-create-service-and-timer-scripts)
    - [Task 3: Modify nginx.conf](#task-3-modify-nginxconf)
    - [Task 4: UFW](#task-4-setup-ufw)
    - [Task 5: Example Screenshot](#task-5-screenshot-of-droplet)  
- [References](#references)

## Assignment Instructions

### Task 1: System User Creation and Ownership
---

**STEP 1**: Type the command to create a system user:

```bash
sudo useradd -r -d /var/lib/webgen -s /usr/sbin/nologin webgen
```

- ```-r```: Creates a system user.
- ```-d```: Sets the home directory.
- ```-s```: Sets login shell for user.
- ```-s /usr/sbin/nologin```: Creates a no login interaction. 

**STEP 2**: Create the Home Directory

Enter the below command to create the home directory ```/vib/lib/webgen/```:
```bash
sudo mkdir -p /var/lib/webgen
```

**STEP 3**: Create Subdirectory Files

Enter the below command to create the subdirectory ```bin``` and ```HTML```:
```bash
sudo mkdir -p /var/lib/webgen/bin /var/lib/webgen/HTML
```

**STEP 4**: Clone Required Files

Enter the below command to clone Nathan Serkin's repository containing the necessary ```generate_index``` script into the working directory:
```bash
git clone https://git.sr.ht/~nathan_climbs/2420-as2-start
```

**STEP 5**: Move File, ```generate_index``` and Grant Necessary Permission

Enter the below command to move file ```generate_index``` to ```/var/lib/webgen/bin```:
```bash
sudo mv 2420-as2-start/generate_index /var/lib/webgen/bin/
```

Enter the below command to grant executable permission to ```generate_index```:
```bash
sudo chmod +x /var/lib/webgen/bin_generate_index
```

**STEP 6**: Set Ownership

Enter the below command to set ownership to webgen:
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```
**STEP 7**: Create ```index.html``` 

Enter the below command to create file ```index.html```:
```bash
sudo nvim /var/lib/webgen/HTML/index.html
```

### Task 2: Create .service and .timer Scripts
---

**STEP 1**: Create ```generate-index.service```

Enter the following command to create and immediately prompt an edit of created file, ```generate-index.service```:
```bash
sudo nvim /etc/systemd/system/generate-index.service
```

Copy and paste the following script to ```generate-index.service```:
```ini
[Unit]
Description=Generate Index Service
Wants=network-online.target
After=network-online.target

[Service]
User=webgen
Group=webgen
ExecStart=/var/lib/webgen/bin/generate_index
```

> [!NOTE]
> A [Install] is not required as it is triggered by a `generate-index.timer` we will create. 


**STEP 2**: Create `generate-index.timer`

Enter the following command to create and immediately prompt an edit of created file, `generate-index.timer`:
```bash
sudo nvim /etc/systemd/system/generate-index.timer
```

Copy and paste the following script to `generate-index.timer` for it to run everyday at 05:00:
```ini
[Unit]
Description=Timer for Generate Index Service

[Timer]
OnCalendar=*-*-* 05:00:00
Unit=generate-index.service
Persistent=true

[Install]
WantedBy=timers.target
```
> [!IMPORTANT]
> Test Services Prior to Proceeding

**STEP 3**: Test `generate-index.service`

Enter the following command to start `generate-index.service`:
```bash
sudo systemctl start generate-index.service
```

Enter the following command to check status of `generate-index.service`:
```bash
sudo systemctl status generate-index.service
```

Enter the following command to view the logs of the service `generate-index.service`:
```bash
sudo journalctl -u generate-index.service
```

### Task 3: Modify nginx.conf
---

**STEP 1**: Install nginx

Enter the following code to install package, `nginx`:
```bash
sudo pacman -S nginx
```

**STEP 2**: Modify `user` in `nginx.conf` using the `sudo nvim`

Replace the user line with the following code:
```bash
user webgen;
```

> [!NOTE]
> If you only specify user, it will default the primary group of the user.

**STEP 3**: Prime for Separate Server Block

Use approach `sites-enabled` and `sites-available` and create directories:
```bash
sudo mkdir -p /etc/nginx/sites-enabled
sudo mkdir -p /etc/ngxin/sites-available
```
>[!NOTE]
> Use `-p` to create parent directory or errors will occur.

**STEP 4**: Create the Separate Server Block in `sites-available`:

Enter the following command to create a new server block to host index.html:
```bash
sudo nvim /etc/nginx/sites-available/webgen.conf
```
Enter the following script into `webgen.conf`:
```ini
server {
    listen 80;
    listen [::]:80;

    server_name localhost.webgen;

    root /var/lib/webgen/HTML;
    index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
}
```

**STEP 5**: Enable Server Block

Enter the following command to symlink and enable server block:
```bash
sudo ln -s /etc/nginx/sites-available/webgen.conf /etc/nginx/sites-enabled/ 
```

**STEP 6**: Update `nginx.conf` with `sites-enabled`

Enter the following code to access `nginx.conf`
```bash
sudo nvim /etc/nginx/sites-enabled/nginx.conf
```

Enter the below script to `nginx.conf`
```bash
http {
    ...
    include /etc/nginx/sites-enabled/*;
}
```

**STEP 7**: Test nginx Configuration

Enter the below command to test for success and errors:
```bash
sudo nginx -t
```

**STEP 8**: Restart and Start nginx

Enter the below command to restart, start, and check status of nginx:
```bash
sudo systemctl restart nginx
sudo systemctl start ngxinx
sudo systemctl status nginx
```

### Task 4: Setup UFW
---

**STEP 1**: Install UFW

Enter the following code to install package, `ufw`:
```bash
sudo pacman -S ufw
```

**STEP 2**: Allow SSH and HTTP

Enter the following command to allow SSH connection:
```bash
sudo ufw allow ssh
```

Enter the following command to allow HTTP traffic:
```bash
sudo ufw allow http
```
>[!IMPORTANT] 
> Potential Errors May Occur:

```WARN: initcaps
[Errno 2] iptables v1.8.10 (legacy): can't initialize iptables table `filter': Table does not exist (do you need to insmod?)
Perhaps iptables or your kernel needs to be upgraded.
```
Potential Steps to Resolving Iptables Error (These were steps I had to perform to resolve my errors):
- `sudo pacman -Syu`: To update your system
- `sudo pacman -S linux`: Update/Install latest linux package
- `sudo pacman -S linux-headers`: Update/Install latest linux-headers package
- `sudo pacman -S iptables`: Update/Install latest iptables package
- `sudo rystemctl restart iptables`: Restarts iptables
- `sudo reboot`: Reboot droplet

**STEP 3**: Enable SSH rate limiting:

Enter the below command to limit the rate of SSH by blocking any connection after 6 attempts within 30 seconds:
```bash
sudo ufw limit ssh
```

**STEP 4**: Enable UFW

Enter the below command to enable UFW:
```bash
sudo ufw enable
```

**STEP 5** Check UFW Status

Enter the command below to check UFW status:
```bash
sudo ufw status
```

Results should reflect:
```
Status: active

To                         Action      From
--                         ------      ----
22                         LIMIT       Anywhere
80                         ALLOW       Anywhere
22 (v6)                    LIMIT       Anywhere (v6)
80 (v6)                    ALLOW       Anywhere (v6)
```

### Task 5: Screenshot of Droplet 
---

This screenshot displays the ngxin details to the corresponding IP droplet:

![Screenshot](/assets/droplet-ip.png)

<br>
<br>
<br>

# Congratulations for reaching the end of part one tutorial !!!

<br>
<br>
<br>

---
---

<br>
<br>
<br>

# Part Two
---

## Introduction
This repository's goal is to guide you in setting up a load balancer on Digital Ocean and allow download of two files from two different IPs. 

### Task 1: Create Two Droplets and One Load Balancer
---

**STEP 1**: Create two droplets on Digital Ocean:

![Screenshot](/assets/create-droplet.png)

**STEP 2**: Add `web` tag to the two created droplets:

![Screenshot](/assets/droplet-overview-tag.png)

**STEP 3**: Confirm the `web` tag has been attached:

![Screenshot](/assets/droplet-tag.png)


**STEP 4**: Create load balancer:

![Screenshot](/assets/load-balancer-setup.png)

![IMPORTANT] Ensure the droplets and load balancer are on the same datacenter; Otherwise, they will not work together. 

**STEP 5**: Connect the load balancer to the droplet tagged `web`:

![Screenshot](/assets/load-balancer-droplet-tag.png)

### Task 2: Set Up `Documents` Directory
---

**STEP 1**: Clone new starter files

Enter the below command to git clone the required directory:
```bash
git clone https://git.sr.ht/~nathan_climbs/2420-as3-p2-start
```

**STEP 2**: Move `generate_file` to required directory

Enter the below command to move `generate_file` to `/var/lib/webgen/bin` directory:
```bash
sudo mv 2420-as3-p2-start/generate_index /var/lib/webgen/bin
```

**STEP 3**: Give required file privilges

Enter the below command to give `generate_index` executable permission:
```bash
sudo chmod +x /var/lib/webgen/bin/generate_index
```

**STEP 4**: Create `documents` directory in `/var/lib/webgen` directory

Enter the below command to create directory:
```bash
sudo mkdir -p /var/lib/webgen/documents
```

**STEP 5**: Create two files: `file-one` and `file-two` in `.../documents/` directory created in step 4
```bash
sudo touch /var/lib/webgen/documents/file /var/lib/webgen/documents/file-two
```

**STEP 6**: Create `index.html` file in `/var/lib/web/gen/HTML` directory
```bash
sudo touch /var/lib/webgen/HTML/index.html
```

**STEP 7**: Redefine `webgen` ownership if required
```bash
sudo chown -R webgen:webgen /var/lib/webgen
```

### Task 3: Modify `webgen.conf`
---

**STEP 1**: Setup `webgen.conf`

Enter the below code into `webgen.conf` as a simple method to access documents/files in the `documents` directory we previously created:

```bash
server {
   listen 80;
   listen [::]:80;

   server_name localhost.webgen;

    location / {
       root /var/lib/webgen/HTML;
       index index.html;
       try_files $uri $uri/ =404;
   }

   # Handle /documents/ requests
   location /documents {
       alias /var/lib/webgen/documents;
       autoindex on;
       autoindex_exact_size off;
       autoindex_localtime on;
   }
}
```
- `autoindex`: Enables the directory listing [^4] [^5]
- `autoindex_exact_size`: Displays file sizes in a human-readable way [^4] [^5]
- `autoindex_localtime`: Displays file timestamps [^4] [^5]


### References:
---
[^1]: “2420-notes/week-eleven.md · main · cit_2420 / 2420-notes-F24 · GitLab,” GitLab, Nov. 19, 2024. https://gitlab.com/cit2420/2420-notes-f24/-/blob/main/2420-notes/week-eleven.md‌

[^2]: “2420-notes/week-twelve.md · main · cit_2420 / 2420-notes-F24 · GitLab,” GitLab, Nov. 19, 2024. https://gitlab.com/cit2420/2420-notes-f24/-/blob/main/2420-notes/week-twelve.md

[^3]: “nginx - ArchWiki,” Archlinux.org, 2020. https://wiki.archlinux.org/title/Nginx

[^4]: “2420-notes/week-thirteen.md · main · cit_2420 / 2420-notes-F24 · GitLab,” GitLab, Nov. 30, 2024. https://gitlab.com/cit2420/2420-notes-f24/-/blob/main/2420-notes/week-thirteen.md

[^5]: “ngx_http_core_module - alias,” Nginx.org, 2024. https://nginx.org/en/docs/http/ngx_http_core_module.html#alias
