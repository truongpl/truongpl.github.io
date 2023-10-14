## Creating a hybrid cloud-on prem AI system (Part 3)

This post will cover the cost of building Infrastructure and operation.

### Building workstation
I have a mining rig and borrowed some NVIDIA GeForce RTX 3060 GPUs for the workstations. I assembled two workstations with costs of $1,500 and $2,000 each.

| Number        | Specs                                                                | Cost  |
|---------------|----------------------------------------------------------------------|-------|
| Workstation 1 | PC: Intel Core I7-7700 Ram: 32Gb GPU: 2 x RTX 3060 SSD: 1TB HDD: 4TB | 1500$ |
| Workstation 2 | PC: Intel Xeon E5-2680 Ram: 32Gb GPU: 2 x RTX 3060 SSD: 1TB HDD: 4TB | 2000$ |

Each workstation is placed on different places and cost an additional 50$ for a static IP.

### Operation cost
The cloud machines cost hosting fee, two workstations cost electricity and internet fee, db instance cost and storage cost

| Machine Number | Purpose              | Specs                           | Cost | Remark                  |
|----------------|----------------------|---------------------------------|------|-------------------------|
| 1              | VPN Master           | Ubuntu 20.04.5 LTS 1 core - 4GB | 14$  | Cloud fee               |
| 2              | Receipt service      | Ubuntu 20.04.5 LTS 2 core - 8GB | 84$  | Cloud fee               |
| 3              | OCR/Layout Service   | Ubuntu 20.04.5 LTS 2 core - 8GB | 84$  | Cloud fee               |
| 4              | Workstation 1        | Above                           | 50$  | Electricity + internet fee |
| 5              | Workstation 1        | Above                           | 50$  | Electricity + internet fee |
| 6              | DB Instance          |                                 | 15$  | Cloud fee               |
| 7              | Container Registry   | N/A                             | 5$   | Cloud fee               |
| 8              | Storage              | N/A                             | 0    |                         |


With above specifications, the cloud node can be scale into 3 pods each. With the workstation, I made a time slicing (https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html) to optimize the GPU usage. I have 4 tensorflow serving pods for each workstation.

My operation cost is: 
```
(14$ + 84$*2 + 50$*2 + 15$ + 5$) + bandwidth cost = 302$ + bandwidth 
```
### Performance
I've created a multi-threaded code to stress test the server. The server can handle a maximum of 25 requests, and a successful handle is indicated by a HTTP 200 status code.

| Image                   | Processing speed | Size of batch images | Average processing per image |
|-------------------------|------------------|----------------------|------------------------------|
| Small (480x640)         | 3-5s             | 25                   | 0.2s                         |
| Medium (640x960)        | 5-6s             | 25                   | 0.24s                        |
| Normal (1280x960)       | 8-10s            | 25                   | 0.4s                         |
| Large (1920x1080)       | 10-15s           | 25                   | 0.6s                         |
| Ultra Large (4032x3024) | 25-45s           | 25                   | 1.8s                         |

In summary, my system can process:

```
Total = (60s / 1.8s) * 60 * 24 * 30 = 1.440.000 images per month
```

Each ultra-large image has an average size of 4MB, resulting in a total monthly bandwidth consumption of 5625GB. This bandwidth usage remains within the Cloud's bandwidth limit, as my Cloud VPS provides a free bandwidth allowance of 9000GB for each VPS. Therefore, I won't incur any additional charges.

In overall, my cost is approximately `302$/month`, which is roughly half the price when compared to a single instance `g5.xlarge` of AWS, which cost `734.38$/month`. This concludes the post on building a hybrid cluster. Thank you for taking the time to read.
