# Installation
```bash
git clone https://github.com/Vlad-Misiukevich/m12_kafkastreams_python_azure.git
```
# Requirements
* Windows OS
* Python 3.8.6
* kubernetes-cli
* azure-cli
* terraform
# Description
1. Login to Azure:  
`az login`
![img.png](images/img.png)
2. Deploy infrastructure with terraform:  
`terraform init`  
`terraform plan -out terraform.plan`  
`terraform apply terraform.plan`
![img_1.png](images/img_1.png)
3. Connect to kubernetes cluster:  
```
az aks get-credentials \
    --resource-group rg-vmisiukevich-westeurope \
    --name aks-vmisiukevich-westeurope \
    --subscription 45a58420-2e1a-4ba1-b23b-c55612222ef5
```
![img_2.png](images/img_2.png)
4. Run the proxy for Kubernetes API server:  
`kubectl proxy`  
![img_3.png](images/img_3.png)
5. Build and push image to DockerHub:  
![img_4.png](images/img_4.png)
6. Create the namespace to use:  
`kubectl create namespace confluent` 
![img_5.png](images/img_5.png)
7. Set this namespace to default for your Kubernetes context:  
`kubectl config set-context --current --namespace confluent`  
![img_6.png](images/img_6.png)
8. Add the Confluent for Kubernetes Helm repository:  
`helm repo add confluentinc https://packages.confluent.io/helm`  
`helm repo update`
![img_7.png](images/img_7.png)
9. Install Confluent for Kubernetes:  
`helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes`
![img_8.png](images/img_8.png)
10. Install all Confluent Platform components:  
`kubectl apply -f ./confluent-platform.yaml`
![img_9.png](images/img_9.png)
11. Install a sample producer app and topic:  
`kubectl apply -f ./producer-app-data.yaml`
![img_10.png](images/img_10.png)
12. Check that everything is deployed:  
`kubectl get pods -o wide`
![img_11.png](images/img_11.png)
13. Set up port forwarding to Control Center web UI from local machine:  
`kubectl port-forward controlcenter-0 9021:9021`
![img_12.png](images/img_12.png)
14. Browse to Control Center:
![img_13.png](images/img_13.png)
15. Create a kafka topic in Control Center:
![img_14.png](images/img_14.png)
![img_15.png](images/img_15.png)
16. Set up port forwarding to connect cluster from local machine:  
`kubectl port-forward connect-0 8083:8083`
![img_16.png](images/img_16.png)
17. Create connector and read data from storage container into Kafka topic:  
`curl.exe -d "@C:\Users\Uladzislau_Misiukevi\PycharmProjects\m12_kafkastreams_python_azure\connectors\azure-
source-cc-expedia.json" -H "Content-Type: application/json" -X POST http://localhost:8083/connectors`
![img_17.png](images/img_17.png)
![img_18.png](images/img_18.png)
![img_19.png](images/img_19.png)
18. Build and push KStream docker image to DockerHub:  
`docker build -t kstream-app .`  
`docker tag kstream-app:latest vmisiukevich/kstream-app:latest`  
`docker push vmisiukevich/kstream-app:latest`
![img_20.png](images/img_20.png)  
![img_21.png](images/img_21.png)
19. Create a kafka topic in Control Center:  
![img_23.png](images/img_23.png)
20. Run KStream app container in the K8s kluster:  
`kubectl create -f kstream-app.yaml`
![img_22.png](images/img_22.png)  
![img_24.png](images/img_24.png)  
![img_25.png](images/img_25.png)
21. Set up port forwarding to connect ksqldb from local machine:  
`kubectl port-forward ksqldb-0 8088:8088`
![img_26.png](images/img_26.png)
22. Run KSQL:  
`kubectl run tmp-ksql-cli -i --tty --image confluentinc/cp-ksql-cli:5.4.7 http://ksqldb.confluent.svc.cluster.local:8088`  
![img_27.png](images/img_27.png)
23. Create stream in KSQL:  
`create stream hotelviews (hotel_id bigint, stay_category varchar) with (kafka_topic = 'expedia_ext', value_format='json');`
![img_28.png](images/img_28.png)
24. Determine initial offset:  
`SET 'auto.offset.reset'='earliest';`
![img_29.png](images/img_29.png)
25. Total amount of hotels for each category:  
`select stay_category, count(hotel_id) as hotel_count from hotelviews group by stay_category emit changes;`
![img_30.png](images/img_30.png)
26. Total amount of distinct hotels for each category:  
`select stay_category, count_distinct(hotel_id) as hotel_count from hotelviews group by stay_category emit changes;`
![img_31.png](images/img_31.png)
27. Create stream for common data visualization in kafka topic:  
`create stream visualization (id bigint, srch_ci varchar, srch_o varchar, hotel_id bigint, date_time varchar, stay_category varchar) with (kafka_topic = 'expedia_ext', value_format='json');`
![img_32.png](images/img_32.png)
28. Data visualization in kafka topic with KSQL:  
`select * from visualization emit changes limit 20;`
![img_33.png](images/img_33.png)