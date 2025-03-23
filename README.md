# **OPENSTACK-IN-EC2**

This README provides detailed steps on how to set up any **Openstack** in an **EC2 Instance**.

---

## **Prerequisites**

- **AMI (OS)**: Ubuntu 22.04 LTS
- **Instance Type**: `t2.large` (2 vCPUs, 8GB RAM)
- **Storage**: Increase to at least 30GB (default 8GB is too small)
- **Security Group Rules**:
  - **SSH (22)**: Your IP
  - **HTTP (80, 443)**: Open for Openstack Dashboard
- **Key Pair**: Create or use an existing SSH key

![1](https://github.com/user-attachments/assets/50e138ee-5e2f-4c40-ad13-5093e3e64b7c)

![1](https://github.com/user-attachments/assets/b5ff89c4-9e64-4e8e-b7b8-ddc5f283f969)

---

## **1. Connect to the EC2 Instance using PuTTY**

1. **Download PuTTY**:  
   If you don't have PuTTY installed on your machine, download it from [here](https://www.putty.org/) and install it.

2. **Convert PEM to PPK**:
   PuTTY does not support `.pem` key files directly, so you need to convert the `.pem` file to the **PPK** (PuTTY Private Key) format.

   a. Open **PuTTYgen** (which is installed along with PuTTY).

   b. In **PuTTYgen**, click **Load** and select your `.pem` file.

   c. Once loaded, click **Save private key** to save the key in the `.ppk` format.

3. **Find the Public IP Address** of your EC2 instance:  
   You can find the public IP address of your EC2 instance in the **EC2 Dashboard** under **Instances**.

4. **Launch PuTTY**:

   a. Open **PuTTY** on your machine.

   b. In the **Host Name (or IP address)** field, enter the **Public IP Address** of your EC2 instance.

   c. In the **Connection Type** section, ensure that **SSH** is selected (which is the default).

   d. Under **Category**, navigate to **Connection > SSH > Auth**.

   e. Click **Browse** and select the `.ppk` file you generated earlier.

5. **Connect to the EC2 Instance**:  
   Now, click **Open** to start the SSH session. The first time you connect, you’ll be asked to confirm the security fingerprint. Click **Yes** to continue.

6. **Login to the Instance**:  
   You should be prompted to login. For an Ubuntu instance, the default username is **`ubuntu`**. So, type `ubuntu` and hit **Enter**.

---

## **2. Update System and Install Dependencies**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git python3-dev python3-pip net-tools
```

---

## **3. Clone Devstack Repository**

```bash
git clone https://opendev.org/openstack/devstack.git
cd devstack
```

![2](https://github.com/user-attachments/assets/16e25228-b9a1-4e67-aed7-7eaf1b8bf760)

---

## **4. Create Devstack Configuration**

- Create a configuration file:
```bash
nano local.conf
```

- Paste the following:
```ini
[[local|localrc]]
ADMIN_PASSWORD=SuperSecret
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD

# Set Host IP (Detects Automatically)
HOST_IP=$(hostname -I | awk '{print $1}')

# Enable Only Essential Services
disable_service tempest
disable_service cinder
disable_service swift
disable_service heat

# Use Latest OpenStack Release
USE_PYTHON3=True
GIT_BASE=https://opendev.org
```

![3](https://github.com/user-attachments/assets/909e05e4-5b86-431d-bc5d-d5cd944f5cc3)

---

## **5. Install Openstack Using Devstack**

- Run the installation (takes ~30-45 mins):
```bash
./stack.sh
```
☑️ If successful, you’ll see `DevStack installed successfully!`.

![Screenshot 2025-03-05 095729](https://github.com/user-attachments/assets/aca4739a-679f-4c6b-a16a-f3e94c570d45)

---

## **6. Access Openstack Dashboard**

- Open Horizon Web UI:
```bash
http://your-ec2-public-ip/dashboard
```

![5](https://github.com/user-attachments/assets/9c52caf9-d219-4c64-93f7-fc337c989862)

- Login Details:
  - Username: `admin`
  - Password: `SuperSecret` (from `local.conf`)

![2](https://github.com/user-attachments/assets/fbc9613d-1b27-40e4-a796-7f73c8188c9f)

![3](https://github.com/user-attachments/assets/44e9abe2-b0f7-43d7-ac40-698ecdf9f346)

---

## **7. Verify Openstack is Running**

- Run:
```bash
source openrc admin admin
openstack service list
```
☑️ If services are listed, OpenStack is running! 🎉

![8](https://github.com/user-attachments/assets/eabc6c4f-6525-42d4-9688-649a3524704f)

---

## **8. Verification of Each Service**

To verify each service (Horizon, Neutron, Keystone, etc) is active and running we will run the following commands:

- **Keystone (Identity Service)**
```bash
openstack token issue
```
☑️ If successful, Keystone is working.

---

- **Horizon (Dashboard)**
  - Check if Apache is running:
  ```bash
  sudo systemctl status apache2
  ```
  - If it's inactive restart it:
  ```bash
  sudo systemctl restart apache2
  ```
  - Then access Horizon in a browser:
  ```bash
  http://your-ec2-public-ip/dashboard
  ```

---

- **Neutron (Networking)**
  - Check network agents:
  ```bash
  openstack network agent list
  ```
☑️ If agents are `:-) Alive`, Neutron is working.

---

- **Nova (Compute)**
  - Check compute services:
  ```bash
  openstack compute service list
  ```
☑️ If `nova-compute` is listed as `up`, Nova is running.

---

- **Glance (Image Service)**
  - Check available images:
  ```bash
  openstack image list
  ```
☑️ If images appear, Glance is working.

---

- **Cinder (Block Storage)**
  - Check storage services:
  ```bash
  openstack volume service list
  ```
☑️ If `cinder` services are `up`, Cinder is working.

---

## **9. Service Down**

- If a service is missing or down try restarting the specific service:
```bash
sudo systemctl restart devstack@keystone
```
Replace `keystone` with the failing service (e.g., `nova`, `neutron`, `glance`, etc.).

---

## **10. Deploy an Instance (VM) in Openstack Horizon**

1️⃣ **Open Openstack Dashboard**

  - Open your browser and go to:
  ```bash
  http://your-ec2-public-ip/dashboard
  ```
  - **Login Credentials**:
    - **Username**: `admin`
    - **Password**: `SuperSecret`

2️⃣ **Create a New Key Pair**

1. Go to: `Project` → `Compute` → `Key Pairs`
2. Click: `Create Key Pair`
3. Enter a Name: Example: `my-key`
4. Key Type: Select `SSH Key (RSA)`
5. Click: `Create Key Pair`
6. Download the Private Key (`.pem` file) and store it safely!
    - Example: `my-key.pem`
    - Set correct permissions:
    ```bash
    chmod 600 my-key.pem
    ```

3️⃣ **Upload an Image (OS for the VM)**

1. Go to: `Project` → `Compute` → `Images`
2. Click: `Create Image`
3. Enter Details:
    - Name: `Ubuntu-22.04`
    - Image Source: `Upload Image`
    - Format: `QCOW2`
    - URL (for Ubuntu 22.04):
    ```bash
    https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
    ```
4. Click: `Create Image`
5. Wait for the image to upload.

4️⃣ **Create a Flavor (VM Size)**

1. Go to: `Admin` → `Compute` → `Flavors`
2. Click: `Create Flavor`
3. Set the Following:
    - Name: `small`
    - vCPUs: `1`
    - RAM: `2048` MB (2GB)
    - Disk: `10` GB
4. Click: `Create Flavor`

5️⃣ **Set Up Networking**

1. Go to: `Project` → `Network` → `Networks`
2. Click: `Create Network`
3. Enter Details:
    - Network Name: `private-net`
    - Subnet Name: `private-subnet`
    - Network Address: `192.168.1.0/24`
4. Click: `Create`

Then, set up a router:
1. Go to: `Project` → `Network` → `Routers`
2. Click: `Create Router`
3. Set Name: `my-router`
4. Click: `Create Router`

Now `delete` the default `router` and go back to `Networks` and delete the `private network` as well.

6️⃣ **Launch an Instance (VM)**

1. Go to: `Project` → `Compute` → `Instances`
2. Click: `Launch Instance`
3. Enter Instance Details:
    - Instance Name: `my-instance`
    - Flavor: Select `small` (1 vCPU, 2GB RAM)
    - Image: Select your uploaded image (e.g., `Ubuntu-22.04`)
    - Network: Select `private-net`
    - Key Pair: Select `my-key`
4. Click: `Launch Instance`

7️⃣ **Assign a Floating IP (If Needed)**

If you want to access the instance over SSH:
1. Go to: `Project` → `Compute` → `Instances`
2. Click "Associate Floating IP"
3. Allocate a new Floating IP
4. Attach it to your instance

8️⃣ **Connect to the Instance via SSH**

1. Find the Floating IP of your instance:
```bash
openstack server list
```
2. SSH into the instance:
```bash
ssh -i my-key.pem ubuntu@<floating-ip>
```

---
