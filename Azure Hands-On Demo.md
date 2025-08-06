# Azure Hands-On Demo: Personal Website

## **Workshop Overview**
**Compute** (VM), **Storage** (File Share), and **Networking** (VNet).

---

## **STEP 1: Create Virtual Network**
**Location:** Azure Portal > Virtual networks > Create

### **Basic Settings:**
- Resource group: Create new `group[X]-rg` (e.g., `group1-rg`)
- Name: `group[X]-network` (e.g., `group1-network`)
- Region: East US

### **IP Addresses Tab:**
- Address space: `10.1.[X].0` (where X = group number)
  - Group 1: `10.1.1.0`
  - Group 2: `10.1.2.0`
  - Group 3: `10.1.3.0`
  - Group 4: `10.1.4.0`
  - Group 5: `10.1.5.0`

- **Add subnet:**
  - Name: `web-subnet`
  - Address range: `10.1.[X].0/28` 

### **Security/Tags:** Leave defaults
### **Create**

---

## **STEP 2: Create Storage Account & File Share**
**Location:** Azure Portal > Storage accounts > Create

### **Basic Settings:**
- Resource group: Use same `group[X]-rg`
- Name: `student[name]storage` (must be globally unique, lowercase only)
- Region: East US
- Performance: Standard
- Replication: LRS

### **Create Storage Account**

### **Create File Share:**
- Go to Storage account > File shares
- **Create share:**
  - Name: `website-files`
  - Quota: 5 GB

### **Upload HTML File:**
- Open the file share
- Click "Upload"
- Upload your completed `index.html`

---

## **STEP 3: Create Virtual Machine**
**Location:** Azure Portal > Virtual machines > Create

### **Basics Tab:**
- Resource group: Same `group[X]-rg`
- VM name: `student[name]-vm`
- Region: East US
- Image: Ubuntu Server 22.04 Gen2
- Size: B1s (1 vcpu, 1 GB RAM)
- **Authentication type:** Password
- **Username:** `USERNAME`
- **Password:** Create a strong password (write it down!)

### **Networking Tab:**
- Virtual network: Select your `group[X]-network`
- Subnet: `web-subnet`
- Public IP: Create new
- **Select inbound ports:** SSH (22) and HTTP (80)

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

1. **Inside your VM!** The prompt will change to: `azureuser@student[name]-vm:~$`

---

## **STEP 5: Install Software Directly on VM**
**Location:** Inside your VM via SSH

### **Update system:**
```bash
sudo apt update
sudo apt upgrade -y
```

### **Install nginx:**
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```

### **Test nginx:**
```bash
curl http://localhost
# Should see "Welcome to nginx!" page
```

---

## **STEP 6: Mount File Share to VM**
**Location:** Azure Cloud Shell + Inside VM via SSH

### **Part A: Get Storage Key (in Cloud Shell):**

First, **exit your SSH session**:
```bash
exit
```

Then get your storage key:
```bash
# Set variables (replace with your actual names)
RESOURCE_GROUP="group[X]-rg"
STORAGE_ACCOUNT="student[name]storage"

# Get storage account key
STORAGE_KEY=$(az storage account keys list \
  --resource-group $RESOURCE_GROUP \
  --account-name $STORAGE_ACCOUNT \
  --query '[0].value' \
  --output tsv)

echo "Storage key: $STORAGE_KEY"
# Copy this key - you'll need it in the next step
```

### **Part B: Mount File Share (inside VM):**

SSH back into your VM:
```bash
ssh azureuser@[YOUR-VM-PUBLIC-IP]
# Enter your password when prompted
```

Inside your VM, mount the file share:
```bash
# Install CIFS utilities
sudo apt install cifs-utils -y

# Create mount point
sudo mkdir -p /mnt/website

# Mount file share (replace STORAGE_ACCOUNT and STORAGE_KEY with your actual values)
sudo mount -t cifs //[STORAGE_ACCOUNT].file.core.windows.net/website-files /mnt/website \
  -o username=[STORAGE_ACCOUNT],password=[STORAGE_KEY],uid=www-data,gid=www-data,file_mode=0755,dir_mode=0755

# Verify mount worked
ls -la /mnt/website
# Should see your index.html file
```

---

## **STEP 7: Deploy Website**
**Location:** Inside your VM via SSH

### **Deploy the website:**
```bash
# Remove default nginx page
sudo rm -f /var/www/html/index.nginx-debian.html

# Copy HTML file from file share
sudo cp /mnt/website/index.html /var/www/html/

# Set proper permissions
sudo chown www-data:www-data /var/www/html/index.html
sudo chmod 644 /var/www/html/index.html

# Test deployment
curl http://localhost
# Should see your personal website HTML
```

### **Exit SSH:**
```bash
exit
```

---

## **STEP 8: Test & Share**
**Location:** Web Browser + SSH for testing

### **Test Your Website:**
1. Open browser
2. Visit: `http://[YOUR-VM-PUBLIC-IP]`
3. Verify your personal introduction page loads

### **Network Testing:**
SSH back into your VM for connectivity tests:
```bash
# SSH back into your VM
ssh azureuser@[YOUR-VM-PUBLIC-IP]
# Enter your password when prompted

# Check your private IP
ip addr show

# Test connectivity within your group (replace with classmate's IP)
ping -c 3 10.[X].0.5

# Try to ping another group (should fail - demonstrates isolation)
ping -c 3 10.[Y].0.4

# Exit SSH
exit
```

### **Share with others:**
1. Write your public IP on the board: `http://[YOUR-PUBLIC-IP]`
2. Visit other students' websites using their public IPs
3. Observe that private network pings work within groups but fail across groups

---

## **Takeaway:**

✅ **Compute:** VM runs nginx web server  
✅ **Storage:** File Share stores website files, mounted to VM  
✅ **Networking:** VNet provides private network, public IP allows internet access  

### **Key Demonstrations:**
- **Azure Portal:** GUI-based resource creation
- **Azure CLI:** Command-line management and automation
- **SSH:** Direct server management and troubleshooting
- **Private vs Public IPs:** Internal communication vs internet access
- **Network Isolation:** Groups can't reach each other's private networks
- **Cloud Storage Integration:** File shares mounted to compute resources