## 🏗️ Architecture Overview

The deployed infrastructure mirrors the following structural layout:
* **1 VPC** with a `10.0.0.0/16` CIDR block.
* **2 Availability Zones** for high availability.
* **2 Private Subnets** (Isolated, no auto-assigned public IPs).
* **2 Public Subnets** (Internet-facing, auto-assigned public IPs).
* **1 Internet Gateway** attached to the custom VPC.
* **1 Public Route Table** managing external traffic routing (`0.0.0.0/0`).

---

## 🛠️ Step-by-Step Deployment Guide

### 1. Create the Custom VPC
1. Open the **AWS Management Console** and navigate to the **VPC Dashboard**.
2. Click on **Create VPC**.
3. Under **VPC settings**, choose **VPC only**.
4. Configure the following parameters:
   * **Name tag:** `my vpc`
   * **IPv4 CIDR block:** `10.0.0.0/16`
5. Click **Create VPC**.

---

### 2. Configure Subnets

#### 🔒 Private Subnets (Isolated backend layer)
Navigate to **Subnets** > **Create subnet** and select your newly created `my vpc`.

* **Private Subnet 1:**
  * **Subnet name:** `pvt subnet-1`
  * **Availability Zone:** Select your first preferred zone (e.g., `us-east-1a`).
  * **IPv4 CIDR block:** `10.0.0.0/24`
  * *Verification Note:* Click on `pvt subnet-1` > **Actions** > **Edit subnet settings**. Ensure **Enable auto-assign public IPv4 address** is **unchecked** to keep this subnet private.

* **Private Subnet 2:**
  * **Subnet name:** `pvt subnet-2`
  * **Availability Zone:** Select your second preferred zone (e.g., `us-east-1b`).
  * **IPv4 CIDR block:** `10.0.2.0/24`
  * *Verification Note:* Ensure auto-assign public IP remains **unchecked**.

#### 🌐 Public Subnets (Internet-facing layer)
Navigate to **Subnets** > **Create subnet** and select `my vpc`.

* **Public Subnet 1:**
  * **Subnet name:** `public subnet-1`
  * **Availability Zone:** Select your first preferred zone (e.g., `us-east-1a`).
  * **IPv4 CIDR block:** `10.0.3.0/24`
  * *Action Required:* Click on `public subnet-1` > **Actions** > **Edit subnet settings**. Check the box for **Enable auto-assign public IPv4 address** and click **Save**.

* **Public Subnet 2:**
  * **Subnet name:** `public subnet-2`
  * **Availability Zone:** Select your second preferred zone (e.g., `us-east-1b`).
  * **IPv4 CIDR block:** `10.0.4.0/24`
  * *Action Required:* Click on `public subnet-2` > **Actions** > **Edit subnet settings**. Check the box for **Enable auto-assign public IPv4 address** and click **Save**.

---

### 3. Establish the Internet Gateway (IGW)
To allow your public subnets to communicate with the outside world, create and attach an Internet Gateway:

1. Navigate to **Internet gateways** in the left sidebar and click **Create internet gateway**.
2. **Name tag:** `my int gtwy`
3. Click **Create internet gateway**.
4. Once created, click **Actions** > **Attach to VPC**.
5. Select `my vpc` from the drop-down menu and click **Attach internet gateway**.

---

### 4. Configure Routing Tables & Associations
By default, subnets are implicitly associated with the main route table (which keeps traffic local). To route public traffic out to the internet, follow these steps:

1. Navigate to **Route tables** and click **Create route table**.
2. Configure the settings:
   * **Name:** `public-route-table`
   * **VPC:** Select `my vpc`
3. Click **Create route table**.

#### Edit Routes (The Internet Gateway Target)
1. Select your new `public-route-table` and open the **Routes** tab at the bottom.
2. Click **Edit routes** > **Add route**.
3. Set **Destination** to `0.0.0.0/0` (all traffic).
4. Set **Target** to **Internet Gateway**, then select `my int gtwy`.
5. Click **Save changes**.

#### Subnet Association
1. Inside your `public-route-table`, click on the **Subnet associations** tab.
2. Click **Edit subnet associations**.
3. Check the boxes next to **public subnet-1** and **public subnet-2**.
4. Click **Save associations**.

---

## 🎯 Verification Matrix

| Subnet Name | CIDR Block | Availability Zone | Public IP Auto-Assign | Route Table Association | Target |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **pvt subnet-1** | `10.0.0.0/24` | AZ 1 | Disabled | Main (Default) | Local Only |
| **pvt subnet-2** | `10.0.2.0/24` | AZ 2 | Disabled | Main (Default) | Local Only |
| **public subnet-1**| `10.0.3.0/24` | AZ 1 | Enabled | `public-route-table` | `0.0.0.0/0` -> IGW |
| **public subnet-2**| `10.0.4.0/24` | AZ 2 | Enabled | `public-route-table` | `0.0.0.0/0` -> IGW |
