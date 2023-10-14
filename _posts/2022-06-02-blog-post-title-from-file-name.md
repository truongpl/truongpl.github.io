## Creating a hybrid cloud-on prem AI system (Part 2)

This post will cover the deployment section on a hybrid cloud-on prem cluster.

---

### Overall design
Let's take a look at the overall design again
![System Architecture](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Mainflow.png)


| Machine Number | Purpose                          | Type    | System specs                        |
|----------------|----------------------------------|---------|-------------------------------------|
| 1              | VPN master (Netmaker host)       | Cloud   | Ubuntu 20.04.5 LTS 1 core - 4GB           |
| 2              | Receipt service                  | Cloud   | Ubuntu 20.04.5 LTS 2 core - 8GB           |
| 3              | OCR Service/Layout   Service     | Cloud   | Ubuntu 22.04.2 LTS 2 cores - 8GB          |
| 4              | Workstation node                 | On Prem | Ubuntu 22.04.2 LTS 8 cores - 32GB Has GPU |
| 5              | Workstation node                 | On Prem | Ubuntu 20.04.5 LTS 8 cores - 32GB Has GPU |


We need to install Microk8s on 2,3,4,5. The details are provided in the next section.


#### Setup Microk8s and join cluster
First, I install microk8s on each node. Run below command

```
sudo snap install microk8s --classic
```
Next, on Machine No 2 (Cloud), execute the following command to obtain the join key for the other nodes:
```
microk8s.add-node
```
Once you have obtained the join key, use it to add each node to the cluster created by Machine No 2. Every time we execute the join command, I will need to re-run the add-node command.

After successfully joining the nodes, proceed with configuring network interfaces to address any connection issues, especially API server timeout problems. The guidance on network interface configuration is here (https://microk8s.io/docs/configure-host-interfaces)

To check the status of each node in the cluster, use the following command:

```
microk8s.kubectl get no -owide
```

My cluster is ready for deployment:
![Cluster](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Cluster.png)

#### Deployment
For each microservice, I wrote two files: deployment.yaml and service.yaml. For ease of use, I decided to put all of the deployment into a repository with structure below:

```bash
.
├── cert
│   ├── custom_tls.yaml
│   └── letsencrypt.yaml
├── document
│   ├── document_service.yaml
│   └── document.yaml
├── ocr
│   ├── ocr_service.yaml
│   └── ocr.yaml
├── layout
│   ├── layout_service.yaml
│   └── layout.yaml
├── dashboard
│   ├── dashboard_service.yaml
│   └── dashboard.yaml
├── demo
│   ├── demo_service.yaml
│   └── demo.yaml
├── ingress
│   ├── ingress_backup.yaml
│   └── ingress.yaml
├── serving
│   ├── serving_service.yaml
│   └── serving.yaml
├── redis
│   └── redis.yaml
├── secret
│   └── secret.yaml
├── time_slicing
│   └── time_slicing.yaml
└── website
    ├── website_service.yaml
    └── website.yaml
```

Each service, except the Tensorflow Serving, use the Flask + Gunicorn stacks. Design of the Document Service will look like 

![Component](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Component.png)

The main idea is that each endpoint will have a service that serves the endpoints business, each service will handle the call to AI Service (OCR Service and Layout Service) to get Model predictions, post process the data and insert the record to database.


A sample deployment.yamls
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: document-service
  labels:
    app: document-service
    version: v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: document-service
      version: v1
  template:
    metadata:
      labels:
        app: document-service
        version: v1
    spec:
      imagePullSecrets:
        - name: regcred
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nodetype
                operator: In
                values:
                - digitalocean 
      containers:
      - name: document-service
        image: 
        imagePullPolicy: Always
        ports:
        - containerPort: 8000
        command: ["gunicorn", "--bind","0:8000","main:app"]
        env:
        - name: OCR-IP
          value: ocr-service.default.svc.cluster.local
        - name: OCR_PORT
          value: "80"
        - name: LAYOUT_IP
          value: layout-service.default.svc.cluster.local
        - name: LAYOUT_PORT
          value: "80"
        - name: POSTGRES_USER # DB User
          valueFrom:
            secretKeyRef:
              name: system-secret
              key: POSTGRES_USER
        - name: POSTGRES_DB # DB Name
          valueFrom:
            secretKeyRef:
              name: system-secret
              key: POSTGRES_DB
        - name: POSTGRES_PASSWORD # DB Pwd
          valueFrom:
            secretKeyRef:
              name: system-secret
              key: POSTGRES_PASSWORD
        - name: POSTGRES_HOST # DB Host
          valueFrom:
            secretKeyRef:
              name: system-secret
              key: POSTGRES_HOST
        - name: POSTGRES_PORT # DB Port
          valueFrom:
            secretKeyRef:
              name: system-secret
              key: POSTGRES_PORT
        resources:
          limits:
            memory: 1Gi
            cpu: "1"
          requests:
            memory: 1Gi
            cpu: "1"
      
      imagePullSecrets:
        - name: regcred
```

I use NodeAffinity to let Kubernetes choose the right node for pods. The confidential keys is wrapped into system-secret secret. 
After finish all deployment.yaml, the next step is expose the api to Internet. This steps requires the ownership of domain. 

Microk8s has already support me to do this by enable the metallb.

```
microk8s enable ingress metallb
```
Set the two IPs of cloud nodes to the LoadBalancer's configuration when asked. After this steps, I configure the DNS record on Domain Administration to point to the IP of two nodes.

Finally, the system is live. I have implemented a gradio server for demonstration, located at (https://demo.ai4s.vn).
![Demo_Screen](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/App_screenshot.png)

The demo is fairly simple, it even works with iPhone (thanks to Gradio): Select a receipt or invoice -> press Submit button -> get the extraction.

#### Testing results

With a common receipt I achieve STP (Straight Through Processing), which means that receipts are seamlessly and automatically processed without the need for manual data entry. This efficient workflow ensures a streamlined and automated handling of receipt data.
![Result_1](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Result_1.png)

In the case of a lengthy receipt, the ability to detect and capture the majority of line items is a significant advantage. This capability translates into substantial time and cost savings in production. It ensures that a large portion of the receipt's content is accurately recognized and processed, reducing the need for manual intervention and enhancing overall operational efficiency.
![Result_2](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Result_2.png)

When dealing with Vietnamese receipts, it's important to acknowledge the need for ongoing improvement in your OCR (Optical Character Recognition) model, especially if the accuracy is currently suboptimal. Generating accurate Vietnamese words from a corpus can be challenging
![Result_3](https://github.com/truongpl/truongpl.github.io/raw/main/docs/assets/Result_3.png)


In overall, there are some room for improvements:
- Layout Information: Incorporating layout information by employing a hybrid model such as LayoutLM is an excellent suggestion. LayoutLM is designed to combine text and layout information, which can significantly improve the accuracy of text recognition on documents where the spatial arrangement of text matters, such as receipts and invoices.

- Character Recognition for Vietnamese: Considering character-level recognition for Vietnamese text, rather than word-level recognition, is a practical approach. Vietnamese script can be challenging for OCR due to its unique diacritics and accents, so recognizing individual characters can help improve accuracy, especially when dealing with various fonts and writing styles.

In the next post, I will cover the cost of building infrastructure and operation.
