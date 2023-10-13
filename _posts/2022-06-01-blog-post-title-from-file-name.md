## Creating a hybrid cloud-on prem AI system

Over the last four months, a childhood friend of mine, who holds a PhD in Mass Communications and Markting and is presently employed at RMIT University, has made the bold choice to establish an AI company. Their primary mission is to develop solutions aimed at securing government funding. Given the high costs associated with deploying a full-scale AI infrastructure on the cloud, I've opted for a hybrid system as a cost-effective alternative.

The system must meet the following criteria:
- Ensuring high uptime
- Maintaining low operational costs
- Delivering fast response times
- Supporting easy integration and removal of models.


---

### Overall system design
To ensure high uptime, I decide to deploy a part of system on cloud, it acts as an AI interface: serves the request from frontend and execute API call to AI Workstation. Overall design will look like:

![System Architecture](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Mainflow.png)

#### Bridging the network
To establish a Kubernetes cluster, it is essential that each node in our network is visible to one another. Typically, our workstation is situated behind a router network, and we are presented with two options:

- Port forwarding: This involves forwarding all ports from the WAN to the corresponding local node IP ports.
- Utilizing a VPN: This method brings all nodes into a shared VPN.
I have chosen to go with option 2, using Netmaker as a VPN, primarily due to its ease of use. Below is my settings:

| Machine Number | Purpose                          | Type    | System specs                        |
|----------------|----------------------------------|---------|-------------------------------------|
| 1              | VPN master                       | Cloud   | Ubuntu 20.04.5 LTS 1 core - 4GB           |
| 2              | Receipt service                  | Cloud   | Ubuntu 20.04.5 LTS 2 core - 8GB           |
| 3              | OCR Interface Analytic Interface | Cloud   | Ubuntu 20.04.5 LTS 2 cores - 8GB          |
| 4              | Workstation node                 | On Prem | Ubuntu 20.04.5 LTS 8 cores - 32GB Has GPU |
| 5              | Workstation node                 | On Prem | Ubuntu 20.04.5 LTS 8 cores - 32GB Has GPU |

Using Netmaker as a VPN, our installation is simpler:
- Rent VPS with Ubuntu 20.04 LTS
- Setup Netmaker on VPN master
- Join the mesh network with Netclient
- Create cluster on Machine 2,3,4,5 with Microk8s

A sample visualization with Netmaker is below:
![Netmaker Overview](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Netmaker.png)



#### Some T-SQL Code

```tsql
SELECT This, [Is], A, Code, Block -- Using SSMS style syntax highlighting
    , REVERSE('abc')
FROM dbo.SomeTable s
    CROSS JOIN dbo.OtherTable o;
```

#### Some PowerShell Code

```powershell
Write-Host "This is a powershell Code block";

# There are many other languages you can use, but the style has to be loaded first

ForEach ($thing in $things) {
    Write-Output "It highlights it using the GitHub style"
}
```
