**Comparison: On-Demand vs Spot Instances in AWS EKS with Karpenter**

---

### 1. **Overview**

AWS provides two main pricing models for EC2 instances used in EKS clusters:

* **On-Demand Instances**: Pay-as-you-go model
* **Spot Instances**: Unused EC2 capacity at a discounted rate (up to 90%)

Karpenter can dynamically provision both types based on availability and configuration.

---

### 2. **Pricing Comparison**

| Aspect              | On-Demand                 | Spot                                  |
| ------------------- | ------------------------- | ------------------------------------- |
| Cost                | Fixed (higher)            | Variable (up to 90% cheaper)          |
| Billing Granularity | Per second                | Per second                            |
| Budget Friendliness | Lower for small workloads | Highly cost-efficient for large scale |

---

### 3. **Availability and Reliability**

| Aspect            | On-Demand                       | Spot                              |
| ----------------- | ------------------------------- | --------------------------------- |
| Availability      | Always available (if quota met) | Depends on excess EC2 capacity    |
| Instance Lifespan | Not interrupted                 | Can be interrupted at any time    |
| AWS Notice        | N/A                             | 2-minute interruption warning     |
| AZ Sensitivity    | Low                             | High (availability varies per AZ) |

---

### 4. **Use Cases**

| Use Case                   | On-Demand   | Spot Instances                        |
| -------------------------- | ----------- | ------------------------------------- |
| Production workloads       | ✅ Preferred | ⚠️ Use cautiously (only if resilient) |
| Batch/Stateless jobs       | ✅ Supported | ✅ Highly suitable                     |
| CI/CD pipelines            | ✅           | ✅                                     |
| Real-time low-latency apps | ✅           | ⚠️ May risk interruptions             |
| Auto-scaling in Karpenter  | ✅           | ✅ With AZ failover logic              |

---

### 5. **Spot Instance Best Practices with Karpenter**

* **Use Multiple Availability Zones (AZs):**

  > Configure Karpenter across all AZs in the region to increase the chance of finding available Spot capacity.

* **Fallback Strategy:**

  > You can configure Karpenter to fallback to On-Demand or simply let pods stay pending if no Spot capacity is available.

* **Capacity Type in Provisioner:**

  ```yaml
  requirements:
    - key: "karpenter.k8s.aws/capacity-type"
      operator: In
      values: ["spot"]
  ```

* **Provisioner with AZ Spread:**

  ```yaml
  requirements:
    - key: "topology.kubernetes.io/zone"
      operator: In
      values: ["ap-south-1a", "ap-south-1b", "ap-south-1c"]
  ```

---

### 6. **Limitations of Spot Instances**

* **No SLA Guarantees:** Spot is not ideal for critical workloads.
* **Possible Pod Eviction:** Pods may be terminated with a 2-minute warning.
* **Unpredictable Pricing and Availability:** Highly volatile based on demand.

---

### 7. **Choosing Between Spot and On-Demand**

| Scenario                                | Recommendation   |
| --------------------------------------- | ---------------- |
| Mission-critical applications           | Use On-Demand    |
| Cost-optimized batch workloads          | Use Spot         |
| Auto-scaled environments with tolerance | Prefer Spot      |
| Mixed capacity strategy                 | Spot + On-Demand |

---

### 8. **Region and AZ Impact**

* **Spot capacity varies not only by AZ but by region**
* **You may experience no Spot capacity in one AZ but availability in another**
* **Choosing a region with lower historical interruptions (like us-east-1 or eu-west-1) may improve Spot success rate**

---

### 9. **Conclusion**

| Spot Instances | ✅ Cheaper, scalable, best for flexible apps   |
| -------------- | --------------------------------------------- |
| On-Demand      | ✅ Stable, reliable, for mission-critical apps |

Karpenter lets you combine both types intelligently. For best results:

* Use Spot across multiple AZs
* Configure tolerations for interruptions
* Use labels and taints properly

You can always monitor Spot interruptions via CloudWatch or Prometheus.
