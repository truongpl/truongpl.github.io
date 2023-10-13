## Creating a hybrid cloud-on prem AI system (Part 2)

This post will cover the deployment section on a hybrid cloud-on prem cluster

---

### Overall design
Let's take a look at the overall design again
![System Architecture](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Mainflow.png)


| Machine Number | Purpose                          | Type    | System specs                        |
|----------------|----------------------------------|---------|-------------------------------------|
| 1              | VPN master (Netmaker host)       | Cloud   | Ubuntu 20.04.5 LTS 1 core - 4GB           |
| 2              | Receipt service                  | Cloud   | Ubuntu 20.04.5 LTS 2 core - 8GB           |
| 3              | OCR Interface Analytic Interface | Cloud   | Ubuntu 22.04.2 LTS 2 cores - 8GB          |
| 4              | Workstation node                 | On Prem | Ubuntu 22.04.2 LTS 8 cores - 32GB Has GPU |
| 5              | Workstation node                 | On Prem | Ubuntu 20.04.5 LTS 8 cores - 32GB Has GPU |


We need to install Microk8s on 2,3,4,5. The details are provided in the next section


#### Setup Microk8s and join cluster
First, I install microk8s on each node. Run below command

```
sudo snap install microk8s --classic
```
On Machine No 2 (Cloud), we execute the join command to get join key for other node.
```
microk8s.add-node
```

Join each node to the cluster created by Machine No 2 (note that each time we execute the join command, we need to rerun the add-node).

After finish join, do some network interface configuration (https://microk8s.io/docs/configure-host-interfaces) to fix connection problems (mostly the API server timeout problem). 
Check the node status with this command
```
microk8s.kubectl get no -owide
```

My cluster is ready for deployment:


![Cluster](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Cluster.png)

#### Deployment
For each microservice, I wrote two files: deployment.yaml and service.yaml. For ease of use, I decided to put all of the deployment into a repository like below:

```bash
.
├── cert
│   ├── custom_tls.yaml
│   └── letsencrypt.yaml
├── cloud_platform
│   ├── cloud_platform_service.yaml
│   └── cloud_platform.yaml
├── core
├── dashboard
│   ├── dashboard_service.yaml
│   └── dashboard.yaml
├── demo
│   ├── demo_service.yaml
│   └── demo.yaml
├── gpu_platform
│   ├── gpu_platform_service.yaml
│   └── gpu_platform.yaml
├── ingress
│   ├── ingress_backup.yaml
│   └── ingress.yaml
├── LICENSE
├── line_item
│   ├── line_item_service.yaml
│   └── line_item.yaml
├── ocr
│   ├── ocr_platform_service.yaml
│   └── ocr_platform.yaml
├── redis
│   └── redis.yaml
├── secret
│   └── secret.yaml
├── timeout_imagepull
│   └── config_map.yaml
├── time_slicing
│   └── time_slicing.yaml
├── troubleshoot.guide
└── website
    ├── website_service.yaml
    └── website.yaml
```

The cloud_platform is the Receipt Microservice, gpu_platform is the OCR Engine and line_item is the Layout Analytic Engine. OCR service is my tensorflow serving deployment.

Each service, except the Tensorflow Serving, use the Flask + Gunicorn stacks. Design of each service will look like

![Component](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Component.png)