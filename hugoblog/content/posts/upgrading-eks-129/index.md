---
title: Upgrading Kubernetes on EKS
date: "2025-02-24T15:00:01.123Z"
description: "Step by Step upgrading from 1.29 to 1.32."
---

Did this earlier today for a client.  You can't directly upgrade from 1.29 to 1.32, so instead have to repeat these steps for each version.


# Upgrading Amazon EKS from 1.29 to 1.32 in Stages

Upgrading Amazon Elastic Kubernetes Service (EKS) is a crucial maintenance task that ensures security, stability, and access to the latest Kubernetes features. Given AWS’s recommendation to upgrade one minor version at a time, this guide walks through upgrading **EKS from version 1.29 to 1.32 in stages**.

---

## **Pre-Upgrade Considerations**
Before proceeding, review the following:

- **Check AWS Release Notes**: Review [AWS Kubernetes version support](https://docs.aws.amazon.com/eks/latest/userguide/kubernetes-versions.html) to understand the changes in each version.
- **Ensure Add-on Compatibility**: Confirm that EKS add-ons (VPC CNI, CoreDNS, kube-proxy, and EBS CSI Driver) support the next Kubernetes version.
- **Backup Your Cluster**: Export Kubernetes manifests and take snapshots of persistent volumes if necessary.
- **Test in a Non-Production Environment**: Always validate upgrades in a staging cluster before applying them in production.
- **Verify Workload Compatibility**: Check for deprecated APIs and changes that may impact workloads.

---

## **Step-by-Step Upgrade Process**
We will upgrade incrementally from **1.29 → 1.30 → 1.31 → 1.32**.

### **1️⃣ Upgrade the EKS Control Plane**
The control plane should be upgraded before node groups and add-ons.

#### **Upgrade to 1.30**
Run the following command to upgrade:
```bash
aws eks update-cluster-version --region <your aws region> --name <your-cluster-name> --kubernetes-version 1.30
```
Monitor the progress:
```bash
aws eks describe-cluster --name <your-cluster-name> --query cluster.status
```
Wait until the status changes from `UPDATING` to `ACTIVE`.

---

### **2️⃣ Upgrade EKS Add-ons**
After upgrading the control plane, update EKS-managed add-ons to ensure compatibility.

#### **Check Installed Add-ons**
```bash
aws eks list-addons --cluster-name <your-cluster-name>
```

#### **Upgrade Amazon VPC CNI (or whichever)**
```bash
aws eks update-addon --cluster-name <your-cluster-name> --addon-name vpc-cni --resolve-conflicts OVERWRITE
```


Repeat the add-on upgrades **after each minor version upgrade** of the control plane.

---

### **3️⃣ Upgrade Worker Nodes**
If you use **managed node groups**, upgrade them one at a time:
```bash
aws eks update-nodegroup-version --cluster-name <your-cluster-name> --nodegroup-name <node-group-name>
```
Monitor node group updates:
```bash
aws eks describe-nodegroup --cluster-name <your-cluster-name> --nodegroup-name <node-group-name>
```

If using **self-managed nodes**, manually cordon, drain, and replace nodes:
```bash
kubectl cordon <node-name>
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
aws autoscaling terminate-instance-in-auto-scaling-group --instance-id <instance-id> --should-decrement-desired-capacity
```
Repeat for each Kubernetes version.

---

## **4️⃣ Post-Upgrade Validation**
After completing the upgrade to 1.32:

- Verify node health:
  ```bash
  kubectl get nodes
  ```
- Check if all workloads are running:
  ```bash
  kubectl get pods -A
  ```
- Validate cluster info:
  ```bash
  kubectl cluster-info
  ```
- Ensure application logs show no errors:
  ```bash
  kubectl logs -l app=<your-app-name> -n <namespace>
  ```

If issues arise, rollback to a previous version or adjust workloads to meet new Kubernetes requirements.

---

## **Summary**
* 1️⃣ Upgrade **control plane** (one version at a time).  
* 2️⃣ Upgrade **EKS add-ons** (VPC CNI, CoreDNS, kube-proxy).  
* 3️⃣ Upgrade **worker nodes** (managed or self-managed).  
* 4️⃣ Validate cluster health and workloads.  

By following this staged approach, you minimise downtime and ensure a smooth transition to **EKS 1.32**. 🚀

