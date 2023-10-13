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

In the next sections, I will describe the steps to build OCR Engine and Layout Analytic Engine

#### Building OCR Engine
The conventional Optical Character Recognition (OCR) process, known as Text Spotting, involves two sequential steps: firstly, locating text within an image, and secondly, recognizing the text that has been located. While there are approaches that seek to streamline this entire procedure into an end-to-end solution, where a single network is trained to handle both tasks, I personally find this approach less appealing. This is due to a few key factors, including its practicality and its applicability in real-world scenarios. Here's why:

Text Spotting encompasses two distinct loss components: one related to text detection and the other tied to text recognition. Merging these two losses into a single unified loss function is not ideal from a technical standpoint.

In practice, adopting an end-to-end solution can restrict the adaptability of integrating new State-of-the-Art (SOTA) techniques for each of these tasks. Consider a scenario where a superior SOTA method emerges for Text Detection. To leverage this advancement, one would need to retrain the entire network. Even if the recognition layer is frozen, it would still be necessary to retrain both the Text Detection path and the common path, which could be quite cumbersome.

In summary, the traditional two-step Text Spotting approach offers certain advantages, and an end-to-end solution may not always be the most practical or efficient choice, given the complexities and specific requirements of the individual tasks involved.

Another reason that: I have some code for creating the synthetic text image for the Recognition, I can reuse that module.

##### Training Text Spotting 
There are many open source (EAST, PixelLink, CRAFT...), both on PyTorch and Tensorflow. I decided to use CRAFT and attention OCR for my text recognition

![Text Detection Compare](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/TD_Compare.png)

To enhance the accuracy of models on Vietnamese receipts, I collected about 200 receipts and label them, below are some samples that I have collected:

![Receipt_1](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/1.jpg)
![Receipt_2](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/2.jpg)
![Receipt_2](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/3.jpg)

I generated synthetic image for text recognition from a 6 million words corpus and trained them for two weeks. Some of my synthetic images:

![Gen_1](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/1.png)
![Gen_2](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/4.jpg)
![Gen_3](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/5.jpg)

#### Performance of Text Spotting
Training and test the Text Detection and Text Recognition give below accuracy

Text detection:

| Dataset    | Original model                            | Finetune model                               |
|------------|-------------------------------------------|----------------------------------------------|
| ICDAR 2013 | Precision: 96.5% Recall: 93.6% F1: 95.03% | Precision: 92.7% Recall: 89.3% F1: 90.96%    |
| ICDAR 2015 | Precision: 93.1% Recall: 88.5% F1: 90.7%  | Precision: 93.6% Recall: 86.2% Recall: 89.7% |


Text recognition:

| Dataset        | Case sensitive | Case insensitive |
|----------------|----------------|------------------|
| COCO Text 2017 | WER: 65.4%     | WER: 37.2%       |



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
