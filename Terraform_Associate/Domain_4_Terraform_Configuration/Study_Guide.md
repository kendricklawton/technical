# Domain 4: Terraform Configuration (HCL2, Custom Conditions & Ephemeral Secrets) — Study Guide (004)

> **Official 004 Blueprint Focus:** Objective 4 — Terraform Configuration (`4a` – `4h`).  
> **Tested Version:** Terraform 1.12+ (Includes **New 004 Objectives `4f`, `4g`, `4h`**!)

---

## 1. Differentiate `resource`, `data`, and `ephemeral` Blocks (`4a`, `4b`)

| Block Type | Purpose & Lifecycle | Reference Syntax in HCL | Persisted in `.tfstate`? |
| :--- | :--- | :--- | :--- |
| **`resource "<TYPE>" "<NAME>"`** | Provisions, updates, and destroys a managed infrastructure object | `<TYPE>.<NAME>.<ATTRIBUTE>` (e.g., `aws_vpc.main.id`) | **Yes** |
| **`data "<TYPE>" "<NAME>"`** | Queries read-only attributes from an existing external resource | `data.<TYPE>.<NAME>.<ATTRIBUTE>` (e.g., `data.aws_ami.ubuntu.id`) | **Yes** (cached in state) |
| **`ephemeral "<TYPE>" "<NAME>"`** *(New in 004 / TF 1.10+)* | Opens a temporary secret, token, or network connection during a run and closes it at run completion | `ephemeral.<TYPE>.<NAME>.<ATTRIBUTE>` | **NEVER** (excluded from plan & state!) |

---

## 2. Variables, Precedence, Outputs & Complex Types (`4c`, `4d`)

### 2.1 Variable Definition Precedence (Lowest $\rightarrow$ Highest)
1. **Environment variables:** `TF_VAR_<name>` (Lowest priority)
2. **`terraform.tfvars`** (if present)
3. **`terraform.tfvars.json`** (if present)
4. **`*.auto.tfvars` / `*.auto.tfvars.json`** (loaded in alphabetical order)
5. **CLI `-var` and `-var-file` flags** (Highest priority; last flag on CLI wins)

### 2.2 Primitive, Collection & Structural Types (`4d`)
- **Primitive Types:** `string`, `number`, `bool`
- **Collection Types (homogeneous—all elements share one type):**
  - `list(<TYPE>)`: Ordered, 0-indexed sequence (`["us-east-1a", "us-east-1b"]`).
  - `set(<TYPE>)`: Unordered collection of unique values (no duplicates, cannot index with `[0]`).
  - `map(<TYPE>)`: Key-value lookup table with string keys (`{ env = "prod", tier = "web" }`).
- **Structural Types (heterogeneous—elements can have different types):**
  - `object({ <ATTR> = <TYPE>, ... })`: Schema with named attributes; supports `optional(<TYPE>, <DEFAULT>)`.
  - `tuple([<TYPE_1>, <TYPE_2>, ...])`: Fixed-length sequence where each position has a strict type.
- **Local Values (`locals`):** Internal module constants/expressions referenced as `local.<NAME>` (note: declared as `locals { ... }` plural, referenced as `local.<NAME>` singular!).

---

## 3. Expressions, Functions, `count`, `for_each` & `dynamic` Blocks (`4e`)

### 3.1 `count` vs. `for_each`
- **`count = N`:** Creates an array of instances indexed `0` to `N - 1` (`aws_subnet.public[0]`). Removing an item from the middle of a list shifts subsequent indices and triggers unwanted recreations.
- **`for_each = <MAP_OR_SET_OF_STRINGS>`:** Creates instances keyed by `each.key` and `each.value` (`aws_subnet.public["us-east-1a"]`). Adding/removing keys never disturbs sibling instances. *(To pass a list of strings to `for_each`, convert it using `toset(var.subnet_list)`.)*

### 3.2 `dynamic` Blocks & Splat Expressions
```hcl
resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = var.ingress_ports # e.g., [80, 443]
    content {
      from_port   = ingress.value # Uses block label 'ingress.value' unless 'iterator' is set
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```
- **Splat Expression (`[*]`):** Extracts an attribute across all elements of a list or `count`-based resource: `aws_instance.web[*].id` (does **not** work on `for_each` maps; use `[for k, v in aws_instance.web : v.id]` or `values(aws_instance.web)[*].id`).

### 3.3 High-Yield Built-in Functions Reference (`4e`)

| Category | Function | Signature & Behavior | Exam Example |
| :--- | :--- | :--- | :--- |
| **Collection** | `lookup` | `lookup(map, key, default)` — Returns key value or fallback default if missing | `lookup(var.tags, "env", "dev")` |
| **Collection** | `element` | `element(list, index)` — Returns element at index; **wraps around modulo length** (never errors on out-of-bounds!) | `element(["a", "b"], 3)` $\rightarrow$ `"b"` |
| **Collection** | `merge` | `merge(map1, map2, ...)` — Combines multiple maps into one (later keys overwrite earlier) | `merge(local.common_tags, { Tier = "db" })` |
| **Collection** | `concat` | `concat(list1, list2, ...)` — Combines two or more lists into a single list | `concat(["10.0.1.0/24"], ["10.0.2.0/24"])` |
| **Collection** | `flatten` | `flatten(list_of_lists)` — Eliminates nested lists into a single flat list | `flatten([["a", "b"], ["c"]])` $\rightarrow$ `["a", "b", "c"]` |
| **Collection** | `distinct` | `distinct(list)` — Removes duplicate elements from a list preserving order | `distinct(["a", "b", "a"])` $\rightarrow$ `["a", "b"]` |
| **Collection** | `slice` | `slice(list, start, end)` — Returns half-open subset `[start, end)` | `slice(["a", "b", "c"], 0, 2)` $\rightarrow$ `["a", "b"]` |
| **Collection** | `zipmap` | `zipmap(keys_list, values_list)` — Constructs a map from a list of keys and values | `zipmap(["a", "b"], [1, 2])` $\rightarrow$ `{"a"=1, "b"=2}` |
| **Collection** | `toset` / `tolist` | Converts a list to an unordered set or vice versa | `for_each = toset(var.subnet_ids)` |
| **Filesystem** | `file` | `file(path)` — Reads raw file contents as literal string (no template interpolation) | `user_data = file("${path.module}/init.sh")` |
| **Filesystem** | `templatefile` | `templatefile(path, vars_map)` — Reads file and renders `${...}` variables inside it | `templatefile("user_data.tftpl", { port = 80 })` |
| **Encoding** | `jsonencode` | `jsonencode(value)` — Serializes HCL object/map to valid JSON string | `policy = jsonencode({ Version = "2012-10-17" })` |
| **Encoding** | `jsondecode` | `jsondecode(string)` — Parses a JSON string into an HCL object/map | `local.cfg = jsondecode(file("config.json"))` |
| **Error Handling** | `can` | `can(expression)` — Evaluates expression; returns `true` if valid, `false` on error | `can(regex("^ami-", var.ami_id))` |
| **Error Handling** | `try` | `try(expr1, expr2, ...)` — Returns the first expression that evaluates without error | `try(var.custom_port, var.default_port, 8080)` |
| **IP Network** | `cidrsubnet` | `cidrsubnet(prefix, newbits, netnum)` — Calculates a subnet address | `cidrsubnet("10.0.0.0/16", 8, 2)` $\rightarrow$ `"10.0.2.0/24"` |
| **String** | `join` / `split` | `join(separator, list)` and `split(separator, string)` | `join(",", ["a", "b"])` $\rightarrow$ `"a,b"` |

> [!WARNING]
> **004 Exam Rule:** Terraform HCL **does NOT support user-defined functions** in `.tf` configuration files. All functions used in Terraform configurations must be built-in functions supplied by the Terraform language engine.

---

## 4. Resource Dependencies & Lifecycle Rules (`4f` — Expanded in 004!)

- **Implicit Dependency (Preferred):** Created automatically when Resource B references an attribute of Resource A (`subnet_id = aws_subnet.app.id`).
- **Explicit Dependency (`depends_on`):** Used only when a hidden runtime dependency exists that is not visible in attribute references (e.g., an EC2 instance requires an `aws_iam_role_policy_attachment` or `aws_internet_gateway` to be active before boot).
- **`lifecycle` Block Meta-Arguments:**
  - `create_before_destroy = true`: Provisions the replacement resource **before** destroying the existing resource. Propagates upstream along dependencies!
  - `prevent_destroy = true`: Causes Terraform to reject any plan that would destroy the resource (so long as the block remains present in `.tf` code).
  - `ignore_changes = [tags, ami]` (or `all`): Instructs Terraform to ignore out-of-band modifications to the specified attributes.
  - `replace_triggered_by = [aws_security_group.app.id]`: Forces replacement of this resource whenever the referenced resource or attribute changes.

---

## 5. Validate Configuration Using Custom Conditions (`4g` — NEW IN 004!)

Terraform 1.12+ provides four distinct custom condition mechanisms tested heavily on the 004 exam:

```hcl
# 1. Input Variable Validation
variable "ami_id" {
  type = string
  validation {
    condition     = startswith(var.ami_id, "ami-")
    error_message = "The ami_id must start with 'ami-'."
  }
}

# 2. Resource Precondition & Postcondition (inside lifecycle block)
resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = "t3.micro"

  lifecycle {
    # Evaluated BEFORE creating/updating the resource
    precondition {
      condition     = data.aws_ami.selected.architecture == "x86_64"
      error_message = "Selected AMI must be x86_64 architecture."
    }

    # Evaluated AFTER provisioning; uses 'self' to inspect live resource attributes
    postcondition {
      condition     = self.public_dns != ""
      error_message = "EC2 instance must be assigned a public DNS hostname."
    }
  }
}

# 3. Continuous Validation 'check' Block (Produces WARNINGS only; never halts apply!)
check "health_check" {
  data "http" "app_status" {
    url = "https://${aws_instance.app.public_dns}/healthz"
  }

  assert {
    condition     = data.http.app_status.status_code == 200
    error_message = "Application health endpoint did not return HTTP 200."
  }
}
```

> [!IMPORTANT]
> **Crucial 004 Exam Distinction:**
> - `validation`, `precondition`, and `postcondition` **halt execution with a hard error** when `condition` evaluates to `false`.
> - A `check` block (`assert`) **never halts or blocks `plan` or `apply`**—it prints a **Warning** so engineers are alerted to real-world health/compliance issues without breaking deployment pipelines.

---

## 6. Sensitive Data, Vault, Ephemeral Values & Write-Only Arguments (`4h` — NEW IN 004!)

### 6.1 Why `sensitive = true` Is Not Enough
Marking a variable or output `sensitive = true` redacts its value in CLI terminal logs (`(sensitive value)`), **but the plaintext secret is still written into `terraform.tfstate` and saved binary plan files**.

### 6.2 Ephemeral Values (`ephemeral = true` & `ephemeral` Blocks — TF 1.10+)
To keep secrets completely out of `.tfstate` and plan files:
1. **Ephemeral Variables & Outputs (`ephemeral = true`):**
   ```hcl
   variable "session_token" {
     type      = string
     ephemeral = true # Value exists ONLY in memory during the run; never written to state or plan!
   }
   ```
   *Rule:* An `ephemeral = true` value can **only** be referenced in contexts that do not persist to state: `provider` blocks, `provisioner` connections, other `ephemeral` variables/locals/outputs, or resource **Write-Only arguments (`_wo`)**.
2. **Ephemeral Resources (`ephemeral "<TYPE>" "<NAME>"`):**
   Fetch short-lived credentials or generate temporary secrets (e.g., `ephemeral "random_password" "db"`, `ephemeral "aws_secretsmanager_secret_version" "creds"`, `ephemeral "vault_kv_secret_v2" "db"`) without ever recording them in `terraform.tfstate`.

### 6.3 Write-Only Arguments (`*_wo` and `*_wo_version` — TF 1.11+)
Managed resources normally persist all input arguments in `.tfstate` to detect drift. **Write-Only arguments** (suffixed with `_wo`, such as `password_wo` on `aws_db_instance`) accept ephemeral values, send them to the cloud API during create/update, and **store `null` in the state file**:
```hcl
ephemeral "random_password" "db_master" {
  length = 24
}

resource "aws_db_instance" "main" {
  identifier          = "prod-db"
  engine              = "postgres"
  instance_class      = "db.t3.micro"
  username            = "dbadmin"
  password_wo         = ephemeral.random_password.db_master.result
  password_wo_version = 1 # Because Terraform doesn't store password_wo in state, increment password_wo_version (e.g. 1 -> 2) whenever you want to rotate the password!
}
```

---

## 7. Provisioners and `terraform_data` (`4a`, `4f`)

### 7.1 Provisioner Types
Provisioners execute scripts on a local or remote machine to bootstrap or configure a resource:
1. **`local-exec`:** Runs a local command on the machine executing Terraform (e.g., triggering an Ansible playbook or writing an endpoint to a local file).
2. **`remote-exec`:** Runs commands on the remote instance over SSH or WinRM via a nested `connection` block.
3. **`file`:** Copies files or directories from the local machine executing Terraform to the newly provisioned remote machine.

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # Creation-time provisioner (default)
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
  }

  # Nested SSH connection block required for remote-exec and file
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }

  # Destroy-time provisioner: Runs BEFORE the instance is terminated
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Decommissioning instance ${self.id}' >> deregister.log"
  }
}
```

### 7.2 Provisioner Execution Timing & Failure Handling
- **Creation-Time (Default):** Runs immediately after the resource is created in the cloud.
  - If a creation-time provisioner fails and `on_failure = fail` (the default), Terraform raises an error, halts, and **marks the resource as tainted (degraded) in state**. On the next apply, Terraform destroys and recreates the resource!
  - If `on_failure = continue`, Terraform logs the error and continues managing the resource normally.
- **Destroy-Time (`when = destroy`):** Runs *before* the resource is destroyed. If it fails, Terraform halts and the resource **is not destroyed**, allowing operators to resolve issues safely.

### 7.3 Why Provisioners Are a Measure of Last Resort
HashiCorp explicitly warns that provisioners should be used **only as a last resort** because:
1. They break Terraform's declarative model by introducing imperative procedural shell scripts.
2. They are not idempotent and cannot detect or reconcile drift.
3. They require direct network access and credentials to the instance OS.
4. **Preferred Alternatives:** Cloud-init (`user_data`), pre-baked Golden Images (Packer), or container orchestrators (Kubernetes / ECS).

### 7.4 The `terraform_data` Resource (TF 1.4+)
- **Built-in Resource:** Requires no provider declarations. Replaces the legacy `null_resource`.
- **Primary Use Case:** Managing arbitrary values in state and triggering provisioners or replacements via `triggers_replace`:
  ```hcl
  resource "terraform_data" "bootstrap" {
    triggers_replace = [
      aws_instance.web.id,
      filemd5("${path.module}/bootstrap.sh")
    ]

    provisioner "local-exec" {
      command = "bash ${path.module}/bootstrap.sh ${aws_instance.web.public_ip}"
    }
  }
  ```
