# Kubernetes Enterprise Cluster Guides

This repository contains a collection of real-world Kubernetes cluster operations, setup procedures, and troubleshooting steps performed in enterprise production environments.

The guides are based on hands-on experience and cover a wide range of topics such as installation, scaling, networking, ETCD management, control plane modifications, and cluster upgrade/downgrade.

---

##  Directory Structure

- **`setup/`**  
  Step-by-step instructions for installing Kubernetes, configuring Calico, HAProxy, kubeadm, and more.

- **`node-ops/`**  
  Guides for joining new nodes, changing IPs, handling power cycles, and control plane IP migrations.

- **`etcd/`**  
  Instructions for deploying ETCD, adding/removing nodes, resetting, and managing ETCD certificates and IP changes.

- **`upgrade/`**  
  Procedures for safely upgrading and downgrading Kubernetes across control planes and worker nodes.

---

##  Highlights

- Written from real operational experience
- Suitable for production-level Kubernetes clusters
- Includes HA setup, Calico with custom registry, secure ETCD configuration
- YAML samples and CLI commands included

---

##  Use Cases

- Demonstrate your hands-on Kubernetes knowledge in interviews
- Use as a base for building your own cluster documentation
- Customize for your team's internal docs

---

##  Contact

Feel free to reach out via GitHub if you found this helpful or want to contribute.
