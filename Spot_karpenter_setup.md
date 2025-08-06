# Spot-Only Node Provisioning with Karpenter (EKS)

## Overview

This document explains how to configure **Karpenter** for **Spot-only instance provisioning** in **Amazon EKS**, enabling automatic fallback across multiple **Availability Zones (AZs)** when Spot capacity is unavailable. The goal is to reduce cost while ensuring high availability within the region, without using On-Demand instances.

---

## Goals

* Provision EKS nodes using **only Spot instances**
* Enable **automatic fallback** across AZs if Spot capacity is unavailable
* Do **not** fallback to On-Demand under any condition
* Ensure **Pending** state is acceptable if Spot is unavailable in all AZs

---

## Why Spot-Only?

| Benefit               | Description                                        |
| --------------------- | -------------------------------------------------- |
| **Cost savings**      | Up to 90% cheaper than On-Demand                   |
| **Stateless apps**    | Ideal for retry-tolerant or scalable workloads     |
| **Multi-AZ fallback** | Reduces risk of Spot unavailability in a single AZ |
| **No overprovision**  | Avoid unnecessary On-Demand fallback               |

---

## Prerequisites

* Subnets created in **at least 2-3 AZs** (e.g., `ap-south-1a`, `1b`, `1c`)
* Subnets and security groups **tagged correctly**:

  ```
  karpenter.sh/discovery: <cluster-name>
  ```
* IAM Role and Instance Profile for Karpenter
* Sufficient Spot quotas and limits in selected AZs

---

## Sample Karpenter Provisioner YAML

```yaml
apiVersion: karpenter.sh/v1beta1
kind: Provisioner
metadata:
  name: spot-only
spec:
  requirements:
    - key: karpenter.sh/capacity-type
      operator: In
      values: ["spot"]
    - key: topology.kubernetes.io/zone
      operator: In
      values: ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
    - key: node.kubernetes.io/instance-type
      operator: In
      values: ["m5.large", "c5.large", "t3.large", "t3a.large"]
  provider:
    subnetSelector:
      karpenter.sh/discovery: <cluster-name>
    securityGroupSelector:
      karpenter.sh/discovery: <cluster-name>
  consolidation:
    enabled: true
  ttlSecondsAfterEmpty: 30
```

---

## How It Works

### Allocation Strategy

* Karpenter will **only provision Spot nodes**
* Karpenter will **try all listed AZs** in order of availability
* If none of the AZs have Spot capacity, the pod will remain **Pending**

### Auto Fallback Example

| AZ          | Spot Availability | Action                  |
| ----------- | ----------------- | ----------------------- |
| ap-south-1a | ❌ Not available   | Skip                    |
| ap-south-1b | ✅ Available       | Launch Spot node        |
| ap-south-1c | ✅ (fallback)      | Used if `1b` also fails |

---

## Recommendations

* **Add multiple instance types** to improve Spot capacity chances
* **Use Spot interruption handling** in workloads if needed
* Enable **consolidation** to reduce cost further

---

## Monitoring

Use the following tools for visibility:

* **kubectl**:

  ```bash
  kubectl get pods --field-selector=status.phase=Pending -A
  ```
* **Karpenter Controller Logs**:
  Check reasons for unavailability, AZ failure, or limits
* **CloudWatch Metrics & Alarms**
* **Prometheus + Grafana Dashboards** (for Karpenter metrics)

---

## Final Notes

* **On-Demand is NOT used**, even if no Spot is available
* Ensure that your workloads can handle temporary unavailability
* Ideal for: **CI jobs**, **auto-scalers**, **stateless services**, **batch jobs**, **Jitsu ingestion workers**, etc.

---

## FAQs

**Q: What if Spot is not available in any AZ?**

> The pods remain Pending until capacity becomes available.

**Q: Does Karpenter switch AZs automatically?**

> Yes, it evaluates all listed AZs dynamically based on availability.

**Q: Should I enable On-Demand fallback?**

> Not in this configuration. This is Spot-only by design.

---

## References

* [Karpenter Official Docs](https://karpenter.sh/)
* [EKS Best Practices](https://aws.github.io/aws-eks-best-practices/)
* [AWS Spot Instance Advisor](https://aws.amazon.com/ec2/spot/instance-advisor/)
