# Deploying My Portfolio Website to Azure Cloud

## Project Overview

I successfully migrated my portfolio website from a local Ubuntu server to Microsoft Azure cloud infrastructure. This project involved setting up a virtual machine, configuring Apache2 web server on Azure, and connecting a custom domain with proper DNS configuration. The entire process gave me hands-on experience with cloud computing, Linux server administration, and web hosting fundamentals.

## What I Built

A fully functional, publicly accessible portfolio website hosted on Azure infrastructure with a custom domain name, demonstrating end-to-end cloud deployment capabilities.

## Technologies Used

- **Cloud Platform**: Microsoft Azure
- **Operating System**: Ubuntu Server (20.04/22.04 LTS)
- **Web Server**: Apache2
- **File Transfer**: SSH, SCP
- **DNS Management**: Custom domain configuration
- **Security**: UFW Firewall, Azure Network Security Groups

---## The Journey: Step-by-Step

### Phase 1: Setting Up Azure Infrastructure

The first step was getting my cloud infrastructure ready. I logged into the Azure Portal and navigated to create a new resource. I chose Ubuntu Server as my operating system because I was already familiar with Ubuntu from running my local server.

#### Creating the Virtual Machine

When configuring the virtual machine, I had to make several important decisions:

**Choosing the VM Size**: I selected a size that balanced performance with cost-effectiveness, providing sufficient resources for hosting a portfolio website.

**Configuring Authentication**: I set up SSH key-based authentication instead of password authentication for better security. Azure generated an SSH key pair for me, and I downloaded the private key to my local machine, making sure to store it securely.

**Opening Network Ports**: I configured the Network Security Group to allow incoming traffic on essential ports:
- **Port 22** for SSH access (so I could manage the server remotely)
- **Port 80** for HTTP traffic (standard web traffic)
- **Port 443** for HTTPS traffic (for future SSL implementation)

After reviewing all my configurations, I hit the "Create" button and waited for Azure to provision my virtual machine. The deployment took about 3-4 minutes, and I received a notification once it was complete.

### Phase 2: Connecting to My Azure VM

Once the VM was deployed, I needed to connect to it. From the Azure Portal, I copied the public IP address that was assigned to my VM.

I opened my terminal and used SSH to connect:
```bash
ssh -i ~/.ssh/azure-vm-key.pem azureuser@<your-vm-public-ip>
```

The first thing I did after connecting was update all the system packages to ensure I had the latest security patches and software versions:
```bash
sudo apt update && sudo apt upgrade -y
```

This took a few minutes, but it was an important step to ensure my server was secure and up-to-date.

---### Phase 3: Installing and Configuring Apache2 on Azure VM

With the server updated, I was ready to install Apache2, which would serve my website files to visitors.

#### Installing Apache2
```bash
sudo apt install apache2 -y
```

After installation, I verified that Apache was running correctly:
```bash
sudo systemctl status apache2
```

The status showed "active (running)", which meant Apache had installed successfully and was already serving content.

I then configured Apache to start automatically whenever the VM rebooted:
```bash
sudo systemctl enable apache2
```

#### Setting Up the Firewall

I configured UFW (Uncomplicated Firewall) to allow web traffic:
```bash
sudo ufw allow 'Apache Full'
sudo ufw enable
sudo ufw status
```

This ensured that HTTP (port 80) and HTTPS (port 443) traffic could reach my web server while maintaining security.

#### Testing Apache Installation

To test that Apache was working, I ran a curl command locally on the VM:
```bash
curl http://localhost
```

This returned the default Apache2 Ubuntu welcome page HTML, confirming everything was configured correctly on the Azure VM.

---### Phase 4: Transferring My Website Files from Local Ubuntu Server

Now came the exciting part - getting my portfolio website from my local Ubuntu server to the Azure VM.

#### Preparing Files on Local Server

First, on my local Ubuntu server, I compressed all my website files into a tarball to make the transfer easier:
```bash
cd /path/to/my/portfolio
tar -czf portfolio.tar.gz *
```

#### Transferring Files to Azure

I used SCP (Secure Copy Protocol) to transfer the compressed file from my local server to the Azure VM:
```bash
scp -i ~/.ssh/azure-vm-key.pem portfolio.tar.gz azureuser@<azure-vm-ip>:~/
```

The transfer took a few minutes depending on the size of my files.

#### Extracting Files on Azure VM

Once the transfer completed, I SSH'd back into my Azure VM and extracted the files directly into Apache's web directory:
```bash
cd ~
sudo tar -xzf portfolio.tar.gz -C /var/www/html/
sudo rm /var/www/html/index.html  # Removed the default Apache page
```

#### Setting Proper Permissions

This was crucial - I had to make sure Apache had the right permissions to read and serve my files:
```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

The `www-data` user is what Apache runs as on Ubuntu, so giving this user ownership of the files ensured Apache could serve them properly.

---### Phase 5: Configuring My Custom Domain with Namecheap

I had purchased my domain **rilportfolio.study** from Namecheap. Now I needed to point it to my Azure VM so visitors could access my portfolio using my custom domain instead of just an IP address.

#### Accessing Namecheap DNS Management

1. I logged into my Namecheap account
2. Navigated to my domain list and selected **rilportfolio.study**
3. Clicked on "Manage" next to the domain
4. Selected "Advanced DNS" tab

#### Creating DNS A Records

I created two A records to point both the root domain and www subdomain to my Azure VM:

**Root Domain Record**:
- **Type**: A Record
- **Host**: @
- **Value**: `<my-azure-vm-public-ip>`
- **TTL**: Automatic

**WWW Subdomain Record**:
- **Type**: A Record  
- **Host**: www
- **Value**: `<my-azure-vm-public-ip>`
- **TTL**: Automatic

This configuration ensured that both `rilportfolio.study` and `www.rilportfolio.study` would point to my Azure-hosted website.

#### Waiting for DNS Propagation

After saving these records, I had to wait for DNS propagation. This is the process where DNS servers around the world update their records with my new configuration. Namecheap indicated it could take up to 48 hours, but in my case, it only took about 30-45 minutes.

I monitored the propagation using command-line tools:
```bash
nslookup rilportfolio.study
dig rilportfolio.study
```

Once these commands started returning my Azure VM's IP address, I knew the DNS had propagated successfully.

---### Phase 6: Testing Everything

Testing was crucial to ensure everything was working correctly at every level.

#### Local Testing on the Azure VM

First, I tested from within the VM itself to ensure Apache was serving my website correctly:
```bash
curl http://localhost
curl http://<azure-vm-public-ip>
```

Both commands returned my portfolio website's HTML, which was a great sign that Apache was configured properly and serving my files.

#### External Testing from My Local Machine

From my local Ubuntu server, I tested accessing the site via the Azure VM's public IP address:
```bash
curl http://<azure-vm-public-ip>
```

This also returned my website content, confirming that:
- The Azure Network Security Group rules were configured correctly
- Port 80 was open and accessible
- Traffic was flowing properly from the internet to my VM

#### Domain Name Testing

Once DNS had propagated (after about 30-45 minutes), I tested using my domain name:
```bash
curl http://rilportfolio.study
curl http://www.rilportfolio.study
```

Success! Both commands returned my website content, confirming that my Namecheap DNS records were working correctly.

#### Browser Testing

Finally, I opened my web browser and navigated to:
- `http://rilportfolio.study`
- `http://www.rilportfolio.study`
- `http://<azure-vm-ip>` (direct IP access)

All three URLs loaded my portfolio perfectly! I tested on multiple devices (desktop, mobile) and different browsers (Chrome, Firefox) to ensure compatibility and responsiveness.

The website was now live and accessible to anyone in the world with an internet connection.

---## Challenges I Faced

### Challenge 1: File Permissions Issues

Initially, after transferring my files to the Azure VM, my website wasn't displaying correctly - I was getting 403 Forbidden errors. After checking the Apache error logs:
```bash
sudo tail -f /var/log/apache2/error.log
```

I discovered it was a permissions issue. Apache couldn't read my files because they were owned by my user account (`azureuser`) instead of the `www-data` user that Apache runs as. 

**Solution**: Setting the correct ownership and permissions resolved this immediately:
```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

This taught me the importance of understanding Linux file permissions and user ownership in web server configurations.

### Challenge 2: DNS Propagation Patience

When I first set up my DNS records on Namecheap, I was eager to see results and kept testing my domain immediately. It kept showing "site can't be reached" errors, which made me think I had configured something wrong.

**Solution**: I learned that DNS propagation isn't instant - it can take anywhere from a few minutes to several hours. Once I waited about 30-45 minutes and cleared my local DNS cache, everything worked perfectly:
```bash
# On Linux, I cleared the DNS cache
sudo systemd-resolve --flush-caches
```

This taught me patience and the importance of understanding how DNS systems work globally.

### Challenge 3: Azure Network Security Group Configuration

At one point during testing, I could SSH into my VM but couldn't access the website from my browser. I realized I had forgotten to verify that the Azure Network Security Group had the proper inbound rules.

**Solution**: I went back to the Azure Portal, checked my VM's Network Security Group, and confirmed that inbound rules for ports 80 (HTTP) and 443 (HTTPS) were properly configured. Once verified, the website became accessible.

This reinforced the importance of understanding cloud networking and security configurations.

---## What I Learned

This project taught me several valuable skills that are directly applicable to real-world cloud and DevOps roles:

### 1. Cloud Infrastructure Management
I gained hands-on experience with Microsoft Azure, learning how to:
- Provision and configure virtual machines
- Manage cloud resources through the Azure Portal
- Understand cloud pricing and resource optimization
- Configure Network Security Groups for proper traffic control

### 2. Linux Server Administration
I became more comfortable with Linux system administration, including:
- Using SSH for secure remote server access
- Managing file permissions and ownership (`chmod`, `chown`)
- Understanding user and group management
- Working with system services using `systemctl`
- Reading and interpreting system logs for troubleshooting

### 3. Web Server Configuration
I learned how Apache2 works and how to:
- Install and configure Apache on Ubuntu
- Understand the `/var/www/html/` directory structure
- Configure the `www-data` user and its importance
- Troubleshoot web server issues using Apache logs
- Ensure Apache starts automatically on system boot

### 4. DNS and Domain Management
Understanding how DNS works was a major learning outcome:
- How A records map domain names to IP addresses
- The difference between root domain (@) and subdomains (www)
- DNS propagation and how it affects global accessibility
- Using tools like `nslookup` and `dig` for DNS troubleshooting
- Managing DNS records through a domain registrar (Namecheap)

### 5. Secure File Transfer
I learned practical skills in secure file management:
- Using SCP (Secure Copy Protocol) for file transfers
- Working with SSH keys for authentication
- Compressing and extracting files with `tar`
- Managing file transfers between local and remote systems

### 6. Network Security Fundamentals
I gained understanding of basic security principles:
- Configuring UFW (Uncomplicated Firewall) on Linux
- Understanding port management (22, 80, 443)
- Implementing SSH key-based authentication
- Configuring cloud-based security groups

### 7. Problem-Solving and Debugging
Perhaps most importantly, I developed critical troubleshooting skills:
- Reading and interpreting error logs
- Using command-line tools for testing (`curl`, `nslookup`, `dig`)
- Breaking down complex problems into smaller, manageable parts
- Researching solutions and applying documentation effectively

---## Key Commands Reference

Here's a quick reference of the essential commands I used throughout this project:

### System Updates
```bash
# Update package lists and upgrade installed packages
sudo apt update && sudo apt upgrade -y
```

### Apache2 Installation and Management
```bash
# Install Apache2
sudo apt install apache2 -y

# Check Apache status
sudo systemctl status apache2

# Enable Apache to start on boot
sudo systemctl enable apache2

# Restart Apache (if needed)
sudo systemctl restart apache2
```

### Firewall Configuration
```bash
# Allow Apache through firewall
sudo ufw allow 'Apache Full'

# Enable firewall
sudo ufw enable

# Check firewall status
sudo ufw status
```

### File Transfer and Permissions
```bash
# Create compressed archive (on local server)
tar -czf portfolio.tar.gz *

# Transfer files via SCP
scp -i ~/.ssh/azure-vm-key.pem portfolio.tar.gz azureuser@<vm-ip>:~/

# Extract files on Azure VM
sudo tar -xzf portfolio.tar.gz -C /var/www/html/

# Set proper ownership and permissions
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

### Testing and Troubleshooting
```bash
# Test website locally
curl http://localhost
curl http://<vm-ip>

# Check DNS propagation
nslookup rilportfolio.study
dig rilportfolio.study

# View Apache error logs
sudo tail -f /var/log/apache2/error.log

# View Apache access logs
sudo tail -f /var/log/apache2/access.log
```

---## Project Architecture
```
┌─────────────────────────────────────────────────────────┐
│                     Internet Users                       │
│                  (rilportfolio.study)                   │
└────────────────────┬────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │   Namecheap DNS       │
         │   A Records:          │
         │   @ → Azure VM IP     │
         │   www → Azure VM IP   │
         └───────────┬───────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │  Azure Cloud Platform │
         │                       │
         │  ┌─────────────────┐ │
         │  │ Network Security│ │
         │  │ Group (NSG)     │ │
         │  │ Port 22 (SSH)   │ │
         │  │ Port 80 (HTTP)  │ │
         │  │ Port 443 (HTTPS)│ │
         │  └────────┬────────┘ │
         │           │          │
         │  ┌────────▼────────┐ │
         │  │  Ubuntu VM      │ │
         │  │                 │ │
         │  │  ┌───────────┐  │ │
         │  │  │  Apache2  │  │ │
         │  │  │ Web Server│  │ │
         │  │  └─────┬─────┘  │ │
         │  │        │        │ │
         │  │  ┌─────▼─────┐  │ │
         │  │  │/var/www/  │  │ │
         │  │  │   html/   │  │ │
         │  │  │ Portfolio │  │ │
         │  │  │   Files   │  │ │
         │  │  └───────────┘  │ │
         │  └─────────────────┘ │
         └───────────────────────┘
                     ▲
                     │
         ┌───────────┴───────────┐
         │  Local Ubuntu Server  │
         │  (File Transfer via   │
         │       SCP/SSH)        │
         └───────────────────────┘
```

## Technology Stack Summary

| Component | Technology | Purpose |
|-----------|------------|---------|
| Cloud Provider | Microsoft Azure | Infrastructure hosting |
| Virtual Machine | Ubuntu Server 20.04/22.04 LTS | Operating system |
| Web Server | Apache2 | Serving web content |
| Domain Registrar | Namecheap | Domain management |
| DNS | Namecheap DNS | Domain name resolution |
| Security | Azure NSG + UFW | Network security & firewall |
| File Transfer | SCP/SSH | Secure file transfer protocol |
| Domain | rilportfolio.study | Custom domain name |

## Conclusion

This project was an incredible learning experience that took me from having a website on a local server to deploying it on enterprise-grade cloud infrastructure. I successfully:

✅ Provisioned and configured an Azure Virtual Machine  
✅ Installed and configured Apache2 web server on the cloud  
✅ Securely transferred website files from local server to Azure  
✅ Configured custom domain with proper DNS records  
✅ Implemented network security with firewalls and security groups  
✅ Tested and validated the deployment at multiple levels  

Every challenge I faced - from permission issues to DNS propagation - taught me something valuable about cloud infrastructure, Linux administration, and web hosting. The skills I gained are directly applicable to real-world DevOps and cloud engineering roles.

Most importantly, I now have a fully functional portfolio website hosted on professional cloud infrastructure, accessible to anyone in the world at **rilportfolio.study**.

## Live Project

🌐 **Visit the live website**: [rilportfolio.study](http://rilportfolio.study)

---

**Project Completed**: January 2026  
**Total Development Time**: ~4 hours (including DNS propagation)  
**Status**: ✅ Live and Operational

---

*This project demonstrates practical experience with cloud computing, Linux server administration, web server configuration, DNS management, and DevOps fundamentals.*
