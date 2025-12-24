# NETWORKING


Perfect 👍
Below are **clean “from-scratch” notes**, written exactly like **step-1, step-2**, so you can keep them for reference or documentation.

---

# 📘 Transit Gateway Cross-Account Setup (FROM START)

## Step 1️⃣ Create Transit Gateway (Networking Account)

1. Login to **Networking Account**
2. Go to **VPC → Transit Gateways**
3. Click **Create Transit Gateway**
4. Configure:

   * Name: `tgw-central`
   * Amazon side ASN: default (or custom if required)
   * Auto accept shared attachments: **Disabled**
   * Default route table association: **Enabled**
   * Default route table propagation: **Enabled**
5. Click **Create Transit Gateway**

✔ Transit Gateway created

---

## Step 2️⃣ Create Transit Gateway Route Table

1. Go to **VPC → Transit Gateway Route Tables**
2. Click **Create Transit Gateway Route Table**
3. Name: `tgw-rt-main`
4. Associate with `tgw-central`

✔ Route table created

---

## Step 3️⃣ Share Transit Gateway with UAT & PROD Accounts

### Step 3.1 – Create Resource Share

1. In **Networking Account**
2. Go to **AWS RAM**
3. Click **Create resource share**
4. Name: `share-tgw`
5. Resource type → **Transit Gateway**
6. Select `tgw-central`
7. Add principals:

   * `<UAT_ACCOUNT_ID>`
   * `<PROD_ACCOUNT_ID>`
8. Create resource share

---

### Step 3.2 – Accept Share

In **UAT Account**:

1. Go to **AWS RAM**
2. Accept Transit Gateway share

Repeat same steps in **PROD Account**

✔ TGW visible in both accounts

---

## Step 4️⃣ Create VPC Attachments – UAT Account

### Step 4.1 Attach `isip-uat`

1. Login to **UAT Account**
2. Go to **VPC → Transit Gateway Attachments**
3. Click **Create attachment**
4. Select:

   * Transit Gateway: `tgw-central`
   * Attachment type: **VPC**
   * VPC: `isip-uat`
   * Subnets: **Private subnets (one per AZ)**
5. Create attachment

---

### Step 4.2 Attach `lime-uat`

Repeat same steps for:

* VPC: `lime-uat`

---

## Step 5️⃣ Create VPC Attachments – PROD Account

### Step 5.1 Attach `isip-prod`

1. Login to **PROD Account**
2. VPC → Transit Gateway Attachments → Create
3. Select:

   * TGW: `tgw-central`
   * VPC: `isip-prod`
   * Subnets: Private subnets
4. Create attachment

---

### Step 5.2 Attach `lime-prod`

Repeat same steps for:

* VPC: `lime-prod`

✔ Total attachments = 4

---

## Step 6️⃣ Accept VPC Attachments (Networking Account)

1. Login to **Networking Account**
2. Go to **VPC → Transit Gateway Attachments**
3. Select each attachment
4. Click **Actions → Accept**

✔ All attachments become **Available**

---

## Step 7️⃣ Associate Attachments with TGW Route Table

1. Go to **Transit Gateway Route Tables**
2. Select `tgw-rt-main`
3. Go to **Associations**
4. Associate all attachments:

   * isip-uat
   * lime-uat
   * isip-prod
   * lime-prod

---

## Step 8️⃣ Add Routes in TGW Route Table

In `tgw-rt-main` → Routes → Create route:

| Destination CIDR | Attachment |
| ---------------- | ---------- |
| isip-uat CIDR    | isip-uat   |
| lime-uat CIDR    | lime-uat   |
| isip-prod CIDR   | isip-prod  |
| lime-prod CIDR   | lime-prod  |

Example:

```
10.10.0.0/16 → isip-uat
10.11.0.0/16 → lime-uat
10.20.0.0/16 → isip-prod
10.21.0.0/16 → lime-prod
```

---

## Step 9️⃣ Update VPC Route Tables (ALL ACCOUNTS)

### Example – isip-uat VPC

Add routes in **private subnet route table**:

```
10.11.0.0/16 → TGW
10.20.0.0/16 → TGW
10.21.0.0/16 → TGW
```

Repeat similarly for:

* lime-uat
* isip-prod
* lime-prod

---

## Step 🔟 Security Group Configuration

Allow traffic from **other VPC CIDRs**

Example:

```
Inbound:
TCP 8080
Source: 10.0.0.0/8   (or specific CIDRs)
```

---

## Step 1️⃣1️⃣ Validate Connectivity

* Launch EC2 in each VPC
* Ping or curl across VPCs
* Verify:

  * TGW routes
  * VPC routes
  * SG rules

---

## ✅ Final Checklist

✔ TGW created
✔ TGW shared
✔ Attachments accepted
✔ TGW route table updated
✔ VPC routes updated
✔ SG allows traffic


