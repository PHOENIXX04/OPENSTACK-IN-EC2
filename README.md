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

---

## **5. Install Openstack Using Devstack**

- Run the installation (takes ~30-45 mins):
```bash
./stack.sh
```
☑️ If successful, you’ll see `DevStack installed successfully!`.

---

## **6. Access Openstack Dashboard**

- Open Horizon Web UI:
```bash
http://your-ec2-public-ip/dashboard
```
- Login Details:
  - Username: `admin`
  - Password: `SuperSecret` (from `local.conf`)

---

## **7. Verify Openstack is Running**

- Run:
```bash
source openrc admin admin
openstack service list
```
☑️ If services are listed, OpenStack is running! 🎉

---

## **8. Conclusion**

Openstack is now setup and running on your EC2 instance. You can use Openstack to create and deploy Cloud Computing Environments.

For more advanced configurations, refer to the [Devstack Documentation](https://github.com/openstack/devstack)
