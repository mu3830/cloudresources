# Azure Hands-On Demo: Personal Website Deployment

## **STEP 1: Create Resource Group**
**Location:** Azure Portal > Resource groups > Create
Basic Settings:

### **Basic Settings:**
- Name: `group[X]-rg` (e.g., `group1-rg`)
- Location: East US

**Create**

---


## **STEP 2: Create Virtual Network**
**Location:** Azure Portal > Virtual networks > Create

### **Basic Settings:**
- Select: Resource Group from Step 1
- Name: `group[X]-network` (e.g., `group1-network`)
- Region: East US

### **IP Addresses Tab:**
- Address space: `10.1.X.0/28`
  - Group 1: `10.1.X.0/28`
  - Group 2: `10.1.X.0/28`
  - Group 3: `10.1.X.0/28`
  - Group 4: `10.1.X.0/28`
  - Group 5: `10.1.X.0/28`

- **Add subnet:**
  - Name: `first-subnet`
  - Address range: Same as VNet address space `10.1.X.0/28`

### **Security/Tags:** Leave defaults
### **Create**

---

## **STEP 3: Create Virtual Machine**
**Location:** Azure Portal > Virtual machines > Create

### **Basics Tab:**
- Resource group: Same `group[X]-rg-[YOUR INITIAL]`
- VM name: `student[name]-vm-[YOUR INITIAL]`
- Region: East US
- Availability Option: No infrastructure redundancy required
- Security Type: Standard
- Image: Ubuntu Server 22.04 LTS x64 Gen2
- Size: B1s (1 vcpu, 1 GB RAM)
- **Authentication type:** Password
- **Username:** `azureuser`
- **Password:** Create a strong password (write it down!)

### **Networking Tab:**
- Virtual network: Select your `group[X]-network`
- Subnet: `web-subnet`
- Public IP: Create new
- **Select inbound ports:** SSH (22), HTTP (80), and HTTPS (443)

### **Create VM**

---

## **STEP 4: SSH Into Your VM**
**Location:** Azure Portal > Virtual machines > [your-vm] + Azure Cloud Shell

### **Get Connection Info:**
1. Go to your VM in Azure Portal
2. Note down your VM's **Public IP address**

### **Connect via SSH:**
1. Open **Cloud Shell** (>_) in top navigation
2. Connect using username/password:

```bash
# SSH into your VM (replace with your actual Public IP)
ssh azureuser@[YOUR-VM-PUBLIC-IP]

# When prompted, enter the password you created during VM setup
# Note: You won't see characters as you type the password (this is normal)
```

3. **You're now inside your VM!** The prompt will change to: `azureuser@student[name]-vm:~$`

---

## **STEP 5: Install Software and Deploy Website**
**Location:** Inside your VM via SSH

### **Update system and install nginx:**
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### **Test nginx installation:**
```bash
curl http://localhost
# Should see "Welcome to nginx!" page
```

### **Remove default nginx page:**
```bash
sudo rm /var/www/html/index.nginx-debian.html
```

### **Deploy your starter website:**
```bash
# Copy the provided starterpack.html file
sudo cp starterpack.html /var/www/html/index.html

# Set proper permissions
sudo chown www-data:www-data /var/www/html/index.html
sudo chmod 644 /var/www/html/index.html
```

### **Test your website locally:**
```bash
curl http://localhost & curl http://publicip
# Should show your starterpack.html content.
```

---

## **STEP 6: Network Testing (5 minutes)**
**Location:** Inside your VM via SSH

### **Check your network details:**
```bash
# Check your private IP address
ip addr show

# Note your private IP (will be something like 10.1.16.4)
```

### **Test group connectivity:**
```bash
# Test connectivity within your group (replace with classmate's IP)
ping -c 3 10.1.16.5

# Try to ping another group (should FAIL - demonstrates isolation)
ping -c 3 10.1.32.4
```

### **Exit SSH:**
```bash
exit
```

---

## **STEP 7: Share Network Information (5 minutes)**
**Location:** Azure Portal + Class Discussion

### **Get your VM's Public IP:**
1. Go to **Virtual machines** in Azure Portal
2. Click your VM name
3. Note down the **Public IP address**

### **Share with class:**
1. Write your **Public IP** on the board
2. Note your **Private IP** from Step 5
3. Compare private IPs within your group vs other groups

---

## **Learned So Far:**

**Compute:** VM runs nginx web server with direct file deployment  
**Networking:** VNet provides private network isolation between groups  
**Linux Skills:** File editing, permissions, and system administration  
**Network Understanding:** Private vs Public IP addressing

### **Key Demonstrations Completed:**
- **Direct VM Development:** Copying and editing files on cloud servers
- **Network Isolation:** /28 subnets provide group separation
- **Private IP Communication:** Can ping within group, not across groups
