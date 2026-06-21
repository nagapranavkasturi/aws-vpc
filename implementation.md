
## Network Design

| Component          | CIDR           |
| ------------------ | -------------- |
| VPC                | `10.0.0.0/16`  |
| Public Subnet AZ-1 | `10.0.1.0/24`  |
| Public Subnet AZ-2 | `10.0.2.0/24`  |
| Private App AZ-1   | `10.0.11.0/24` |
| Private App AZ-2   | `10.0.12.0/24` |
| Private DB AZ-1    | `10.0.21.0/24` |
| Private DB AZ-2    | `10.0.22.0/24` |

---

# Step 1: Create the VPC

AWS Console → **VPC** → **Your VPCs** → **Create VPC**

Choose:

* Resources to create: **VPC only**
* Name: `prod-vpc`
* IPv4 CIDR: `10.0.0.0/16`
* IPv6: No IPv6
* Tenancy: Default

Click **Create VPC**.

📸 Screenshot: `01-vpc.png`

---

# Step 2: Create Internet Gateway

VPC → **Internet Gateways** → **Create Internet Gateway**

Name:

```
prod-igw
```

Click **Create**.

Then:

Actions → **Attach to VPC** → Select `prod-vpc`.

📸 Screenshot: `02-igw.png`

---

# Step 3: Create Public Subnet 1

VPC → **Subnets** → Create subnet

* VPC: `prod-vpc`
* Name: `prod-public-az1`
* Availability Zone: `ap-south-1a`
* CIDR: `10.0.1.0/24`

Create.

Enable Public IP:

Actions → Edit subnet settings

Enable:

```
Auto-assign public IPv4
```

---

# Step 4: Create Public Subnet 2

Create another subnet:

* Name: `prod-public-az2`
* AZ: `ap-south-1b`
* CIDR: `10.0.2.0/24`

Enable Auto-assign Public IPv4.

📸 Screenshot: `03-public-subnets.png`

---

# Step 5: Create Private App Subnets

Create:

### App Subnet 1

* Name: `prod-app-az1`
* AZ: `ap-south-1a`
* CIDR: `10.0.11.0/24`

### App Subnet 2

* Name: `prod-app-az2`
* AZ: `ap-south-1b`
* CIDR: `10.0.12.0/24`

---

# Step 6: Create Private DB Subnets

### DB Subnet 1

* Name: `prod-db-az1`
* AZ: `ap-south-1a`
* CIDR: `10.0.21.0/24`

### DB Subnet 2

* Name: `prod-db-az2`
* AZ: `ap-south-1b`
* CIDR: `10.0.22.0/24`

📸 Screenshot: `04-private-subnets.png`

---

# Step 7: Allocate Elastic IPs

Go to:

EC2 → **Elastic IPs**

Allocate:

1. `EIP-NAT-1`
2. `EIP-NAT-2`

📸 Screenshot: `05-elastic-ips.png`

⚠️ **Cost Warning:** Elastic IPs attached to NAT Gateways incur charges.

---

# Step 8: Create NAT Gateway 1

VPC → NAT Gateways → Create

* Name: `prod-nat-az1`
* Public Subnet: `prod-public-az1`
* Elastic IP: `EIP-NAT-1`

Create.

---

# Step 9: Create NAT Gateway 2

* Name: `prod-nat-az2`
* Public Subnet: `prod-public-az2`
* Elastic IP: `EIP-NAT-2`

Create.

Wait until both show:

```
Available
```

📸 Screenshot: `06-nat-gateways.png`

⚠️ **NAT Gateways are NOT Free Tier.** Delete them after practice if needed.

---

# Step 10: Create Public Route Table

VPC → Route Tables → Create

Name:

```
prod-public-rt
```

Add Route:

| Destination | Target                        |
| ----------- | ----------------------------- |
| `0.0.0.0/0` | Internet Gateway (`prod-igw`) |

Associate with:

* prod-public-az1
* prod-public-az2

📸 Screenshot: `07-public-route-table.png`

---

# Step 11: Create Private Route Table AZ-1

Create Route Table:

```
prod-private-rt-az1
```

Add:

| Destination | Target       |
| ----------- | ------------ |
| `0.0.0.0/0` | prod-nat-az1 |

Associate:

* prod-app-az1
* prod-db-az1

---

# Step 12: Create Private Route Table AZ-2

Create:

```
prod-private-rt-az2
```

Add:

| Destination | Target       |
| ----------- | ------------ |
| `0.0.0.0/0` | prod-nat-az2 |

Associate:

* prod-app-az2
* prod-db-az2

📸 Screenshot: `08-private-route-tables.png`

---

# Step 13: Create Security Groups

## ALB Security Group

Inbound:

| Type        | Source      |
| ----------- | ----------- |
| HTTP (80)   | `0.0.0.0/0` |
| HTTPS (443) | `0.0.0.0/0` |

Name:

```
prod-alb-sg
```

---

## App Security Group

Inbound:

| Type      | Source      |
| --------- | ----------- |
| HTTP (80) | prod-alb-sg |
| SSH (22)  | Bastion SG  |

Name:

```
prod-app-sg
```

---

## DB Security Group

Inbound:

| Type         | Source      |
| ------------ | ----------- |
| MySQL (3306) | prod-app-sg |

Name:

```
prod-db-sg
```

📸 Screenshot: `09-security-groups.png`

---

# Step 14: Configure Network ACLs

Create:

### Public NACL

Allow:

* 80
* 443
* 1024–65535

Associate with Public Subnets.

---

### Private NACL

Allow:

* 80
* 443
* 3306
* 1024–65535

Associate with Private Subnets.

📸 Screenshot: `10-nacls.png`

---

# Step 15: Enable VPC Flow Logs

VPC → Your VPCs → Select `prod-vpc`

Actions → Create Flow Log

Settings:

* Filter: All
* Destination: CloudWatch Logs
* Traffic Type: All
* Name:

```
prod-vpc-flowlogs
```

Create.

📸 Screenshot: `11-flowlogs.png`

---

# Step 16: Verify Connectivity

You can optionally launch:

* A Bastion EC2 instance in a Public Subnet.
* An App EC2 instance in a Private App Subnet.

Verify:

* Bastion has internet access.
* Private EC2 reaches the internet through the NAT Gateway (for updates).
* Private EC2 cannot be accessed directly from the internet.

---

# Final Architecture Checklist

✅ VPC Created

✅ Internet Gateway Attached

✅ 2 Public Subnets

✅ 2 Private App Subnets

✅ 2 Private DB Subnets

✅ 2 NAT Gateways

✅ 3 Route Tables

✅ Security Groups Configured

✅ Network ACLs Configured

✅ VPC Flow Logs Enabled

---

