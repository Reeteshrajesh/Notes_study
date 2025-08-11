# LifeCycle Management

## **1. cordon**

**Command:**

```bash
kubectl cordon <node-name>
```

**Meaning:**
Marks a node as **unschedulable** — the scheduler will not place **new** pods on this node.

**Details:**

* Does **not** affect running pods — they keep running.
* Useful before doing maintenance (patching, rebooting, upgrades).
* Scheduler sees the `spec.unschedulable: true` flag in the Node object.

**Example use case:**
You’re about to upgrade the kernel on a worker node — cordon it so no new pods land there during the process.

---

## **2. uncordon**

**Command:**

```bash
kubectl uncordon <node-name>
```

**Meaning:**
Makes a previously cordoned node **schedulable** again.

**Details:**

* Clears the `spec.unschedulable` flag.
* Scheduler will start assigning new pods to it.

**Example use case:**
After finishing maintenance on a node, you uncordon it so workloads can run there again.

---

## **3. drain**

**Command:**

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

**Meaning:**
Safely **evicts** all non-DaemonSet pods from a node.

**Details:**

* Used before node shutdown or removal.
* Ignores DaemonSet pods (since they must run on all nodes).
* Can delete pods using local storage (`emptyDir`) if `--delete-emptydir-data` is set.
* Marks the node as unschedulable **after** evicting pods.
* Uses the Kubernetes eviction API (honors PodDisruptionBudgets).

**Example use case:**
You’re about to remove a node from the cluster — drain it so workloads move elsewhere.

---

## **4. evict** (not a direct kubectl command for nodes, but a scheduler action)

**Meaning:**
The process of **removing pods** from a node in a controlled way.

**Details:**

* Can happen because of:

  * Node being drained.
  * Node becoming unhealthy (triggered by Node Controller).
  * Manual eviction request via the API.
* Respects PodDisruptionBudgets (limits on how many pods can be down at once).
* Scheduler will try to reschedule evicted pods elsewhere.

**Example use case:**
The Node Controller evicts pods from a node that’s been unreachable for more than 5 minutes.

---

## **5. describe**

**Command:**

```bash
kubectl describe node <node-name>
```

**Meaning:**
Shows **detailed information** about a node.

**Details:**

* Includes:

  * Labels, annotations, taints
  * Resource capacity & allocatable
  * Node conditions (Ready, DiskPressure, MemoryPressure)
  * Node addresses (internal/external IPs, hostname)
  * Running pods on that node
  * Events related to the node

**Example use case:**
Debugging why a node is marked `NotReady` — `describe` shows health conditions and events.

---

## **6. label**

**Command:**

```bash
kubectl label node <node-name> key=value
```

**Meaning:**
Adds or changes a **label** on a node.

**Details:**

* Labels are key-value pairs used for **node selection**.
* Commonly used for:

  * Environment tags (`env=prod`)
  * Role tags (`node-role.kubernetes.io/database=database`)
  * Hardware tags (`gpu=true`)
* Scheduler uses these labels with **nodeSelector** or **affinity** in pod specs.

**Example use case:**
You want certain pods to run only on GPU nodes:

```bash
kubectl label node gpu-node-1 gpu=true
```

---

## **7. taint**

**Command:**

```bash
kubectl taint nodes <node-name> key=value:effect
```

**Meaning:**
Marks a node with a **taint** — prevents pods from running unless they have a matching toleration.

**Details:**

* **Effects**:

  * `NoSchedule` → No new pods unless they tolerate the taint.
  * `PreferNoSchedule` → Try to avoid scheduling here.
  * `NoExecute` → Evict running pods unless they tolerate it.
* Used to dedicate nodes for special workloads.

**Example use case:**
Reserve a node for database workloads only:

```bash
kubectl taint nodes db-node dedicated=database:NoSchedule
```

---

## **8. annotate**

**Command:**

```bash
kubectl annotate node <node-name> key=value
```

**Meaning:**
Adds metadata to a node — **not used for scheduling**, but for informational or tooling purposes.

**Details:**

* Unlike labels, annotations are meant for:

  * Extra metadata (e.g., cluster-autoscaler info)
  * Storing config details
  * Keeping operational notes
* Can store larger values than labels.

**Example use case:**
Mark a node with maintenance info:

```bash
kubectl annotate node node-1 maintenance-window="2025-08-12T10:00Z"
```

---

## **9. delete**

**Command:**

```bash
kubectl delete node <node-name>
```

**Meaning:**
Removes the Node object from Kubernetes.

**Details:**

* Does **not** shut down the actual VM or server — just removes it from the cluster.
* If kubelet is still running, it will try to re-register unless `--register-node=false` is set.
* Common in cloud clusters — the cloud controller will remove a node object when the VM is deleted.

**Example use case:**
You decommission a worker VM — delete its Node object from Kubernetes.

---

## **10. edit**

**Command:**

```bash
kubectl edit node <node-name>
```

**Meaning:**
Opens the Node object in your default editor for manual changes.

**Details:**

* Lets you directly modify YAML (labels, taints, annotations, etc.).
* Not all fields are editable (some are managed by kubelet).

**Example use case:**
Manually add a label or change allocatable resources for a statically registered node.

---

## **11. patch**

**Command:**

```bash
kubectl patch node <node-name> -p '{"spec":{"unschedulable":true}}'
```

**Meaning:**
Updates specific fields in the Node object **without opening an editor**.

**Details:**

* Can do strategic merge, JSON merge, or JSON patch.
* Useful for automation and scripting.

**Example use case:**
Automate cordoning a node in a script:

```bash
kubectl patch node worker-1 -p '{"spec":{"unschedulable":true}}'
```

---

## Node Lifecycle & Operations Flow Diagram

```
                       ┌───────────────────────────┐
                       │     Node is Schedulable    │
                       │  (new pods can be placed)  │
                       └──────────────┬─────────────┘
                                      │
                           kubectl cordon
                                      │
                       ┌──────────────▼─────────────┐
                       │  Node is Unschedulable     │
                       │ (no new pods scheduled)    │
                       └──────────────┬─────────────┘
                                      │
                           kubectl drain (prep for maintenance)
                                      │
                        Eviction API removes non-DaemonSet pods
                                      │
                       ┌──────────────▼─────────────┐
                       │ Node Empty (only DS pods)  │
                       │ + Unschedulable            │
                       └──────────────┬─────────────┘
                                      │
                         ┌────────────┴────────────┐
                         │                         │
             kubectl uncordon             kubectl delete node
                         │                         │
           ┌─────────────▼─────────────┐   ┌───────▼───────────────────────┐
           │ Node back in scheduling    │   │ Node object removed from      │
           │ rotation                   │   │ cluster                       │
           └───────────────────────────┘   └───────────────────────────────┘
```

---

### **Where the other verbs fit in**

* **label** → Can be applied in any state, affects scheduling rules (`nodeSelector`, affinity).
* **taint** → Can be applied in any state, blocks pods unless tolerations match.
* **annotate** → Adds metadata for tools/scripts, no scheduling effect.
* **describe** → Can be used anytime to inspect status.
* **edit** / **patch** → Change node fields (cordon via patch, update labels, etc.).
* **evict** → Part of `drain` or triggered by Node Controller if node is unhealthy.

---

### **Quick mental model**

* **cordon** = Stop new pods
* **uncordon** = Allow new pods again
* **drain** = Move existing pods off (except DaemonSets)
* **delete** = Remove node from cluster entirely
* **taint** = Keep only certain pods
* **label** = Attract only certain pods
* **annotate** = Post-it note for automation/tools
* **describe** = Inspect details
* **patch/edit** = Make changes
