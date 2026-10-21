# Domain 6: Monitoring, Logging & Runtime Security — Quiz 1 (Audit Policies)

**Exam:** Certified Kubernetes Security Specialist (CKS)
**Format:** Performance Scenario & Multiple Choice Security Questions

---

### Question 1: AppArmor Profile Verification

A pod fails to start with error `AppArmor profile k8s-deny-write not loaded`. What command on the worker node verifies loaded AppArmor profiles?

- A. `systemctl status apparmor`
- B. `aa-status`
- C. `kubectl get apparmor`
- D. `crictl apparmor-list`

---

## Answer Key & Detailed Explanations

### Question 1: AppArmor Profile Verification
**Answer:** B
**Explanation:** `aa-status` displays the current AppArmor status and all profiles loaded in enforce or complain mode on that Linux host.

---

