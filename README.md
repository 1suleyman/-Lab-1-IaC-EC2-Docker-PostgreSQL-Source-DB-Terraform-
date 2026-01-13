# 🧩 EC2 → Aurora PostgreSQL Migration

## 🧪 Lab 1 (IaC) — EC2 + Docker PostgreSQL (Source DB, Terraform)

In this lab, I **re-implemented Lab 1 using Terraform** to make the entire setup **repeatable, idempotent, and disposable**.
Instead of manually configuring EC2, Docker, and PostgreSQL, everything is now provisioned and configured **automatically via Infrastructure as Code (IaC)**.

This lab proves that the **same result can be recreated reliably every time** — a critical requirement before performing any database migration.

---

## 📋 Lab Overview

**Goal:**

* Recreate the EC2 + Docker PostgreSQL source database using **Terraform**
* Automate OS bootstrapping with `user_data.sh`
* Enforce **safe SSH access** using CIDR restrictions
* Automatically **seed the database** with schema, data, and roles
* Validate idempotency and clean teardown with `terraform destroy`

**Learning Outcomes:**

* Understand **idempotent infrastructure**
* Use Terraform modules and variables effectively
* Bootstrap EC2 instances with `user_data`
* Automate Docker + PostgreSQL setup
* Seed databases safely during instance initialization
* Validate infrastructure end-to-end

---

## 🛠 Step-by-Step Journey

---

### Step 1: Why Terraform (Idempotency)

After completing the manual version of Lab 1, the next step was to make it:

* **Repeatable**
* **Predictable**
* **Version-controlled**
* **Destroyable**

This is where **Terraform** comes in.

> *Idempotent* means:
> “I can run this again and again and always get the same result.”

---

### Step 2: Terraform Project Structure

The configuration was split into **clear, reusable files**:

* `providers.tf`
* `variables.tf`
* `terraform.tfvars`
* `main.tf`
* `outputs.tf`
* `modules/ec2/*`
* `user_data.sh`

This mirrors **real-world Terraform project layouts**.

---

### Step 3: Package Manager Decision (DNF vs YUM)

While reviewing the bootstrap script, an important question came up:

> **Should Docker be installed using `yum` or `dnf`?**

**Answer:**

* Amazon Linux 2023 uses **DNF**
* `yum` still works as a **wrapper**
* Best practice → **use `dnf` explicitly**

This is reflected in the final `user_data.sh`.

---

### Step 4: EC2 Bootstrap via `user_data.sh`

The EC2 instance is fully configured at launch using a single script:

**Key actions performed automatically:**

* Install Docker
* Enable Docker on boot
* Install Docker Compose v2 plugin
* Create `docker-compose.yml`
* Start PostgreSQL 13 container
* Wait for DB readiness
* Seed schema, data, and roles

The full script used for this lab is documented in `user_data.sh` .

---

### Step 5: Secure SSH Access (CIDR-Based)

Instead of allowing SSH from anywhere:

* A variable `allowed_ssh_cidr` was created
* Set to **my public IP `/32`**
* Applied to the security group via Terraform

This ensures:

* SSH access is **restricted**
* Security matches **real-world practices**

---

### Step 6: Terraform Apply (End-to-End)

```bash
terraform init
terraform plan
terraform apply
```

**Observed order:**

1. Security group created
2. EC2 instance launched
3. User data executed
4. Docker installed
5. PostgreSQL container started
6. Database seeded automatically

---

### Step 7: Validation (Critical)

After provisioning, everything was verified manually:

```bash
ssh -i labec2.pem ec2-user@<public-ip>
```

Checks performed:

```bash
docker version
docker compose version
docker ps
```

✔️ Docker installed
✔️ Docker Compose installed
✔️ PostgreSQL container running

---

### Step 8: Database Validation

Connect to PostgreSQL:

```bash
docker exec -it postgres_db psql -U admin -d appdb
```

Test data:

```sql
SELECT * FROM customers;
```

<img width="336" height="106" alt="appdb=# SELECT  FROM customers;" src="https://github.com/user-attachments/assets/f339da50-684e-49eb-b832-af10ee646575" />

✔️ Table exists
✔️ Data seeded
✔️ Roles created
✔️ Permissions applied

The database is now **fully migration-ready**.

<img width="541" height="116" alt="lab-ec2-docker-postgres" src="https://github.com/user-attachments/assets/f2465330-aee5-4f50-a2c9-16332bd31130" />

---

### Step 9: Clean Teardown (Confidence Check)

```bash
terraform destroy
```

✔️ All resources deleted cleanly
✔️ No leftover infrastructure
✔️ Cost-safe and reproducible

---

## ✅ Key Terraform Concepts Used

| Concept          | Purpose                |
| ---------------- | ---------------------- |
| `user_data`      | OS + app bootstrapping |
| Variables        | Config flexibility     |
| CIDR restriction | SSH security           |
| Modules          | Code reuse             |
| Outputs          | Visibility             |
| Destroy          | Cost control           |

---

## 💡 Notes / Engineering Insights

* `user_data` is powerful but must be **idempotent**
* Database seeding should be **safe to re-run**
* Docker Compose v2 requires **plugin installation**
* Waiting for DB readiness avoids race conditions
* Terraform + Docker = strong migration foundation

---

## 📌 Lab Summary

| Area              | Status | Key Takeaway              |
| ----------------- | ------ | ------------------------- |
| Manual setup      | ✅      | Baseline understanding    |
| Terraform IaC     | ✅      | Repeatable infrastructure |
| Docker automation | ✅      | No manual steps           |
| DB seeding        | ✅      | Migration-ready           |
| Security          | ✅      | SSH locked to IP          |
| Teardown          | ✅      | Zero cost residue         |

---

## ✅ References

* Terraform EC2 User Data
  [https://developer.hashicorp.com/terraform/language/resources/provisioners/syntax](https://developer.hashicorp.com/terraform/language/resources/provisioners/syntax)
* Amazon Linux 2023 Docs
  [https://docs.aws.amazon.com/linux/al2023/](https://docs.aws.amazon.com/linux/al2023/)
* Docker Compose v2
  [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
* PostgreSQL 13
  [https://www.postgresql.org/docs/13/](https://www.postgresql.org/docs/13/)
