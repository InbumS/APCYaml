# APCYaml

curl -o ncp-iam-authenticator https://kr.object.ncloudstorage.com/nks-download/ncp-iam-authenticator/v1.0.5/linux/amd64/ncp-iam-authenticator
chmod +x ./ncp-iam-authenticator
mkdir -p $HOME/bin && cp ./ncp-iam-authenticator $HOME/bin/ncp-iam-authenticator && export PATH=$PATH:$HOME/bin
echo 'export PATH=$PATH:$HOME/bin' >> ~/.bash_profile

export NCLOUD_ACCESS_KEY=NAVERCLOUD API
export NCLOUD_SECRET_KEY=NAVERCLOUD API
export NCLOUD_API_GW=https://ncloud.apigw.ntruss.com

ncp-iam-authenticator create-kubeconfig --region KR --clusterUuid 쿠버네티스 UUID --output kubeconfig.yaml
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client --output=yaml
kubectl get node --kubeconfig kubeconfig.yaml
cp kubeconfig.yaml .kube/config

source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

chmod o-r ~/.kube/config
chmod g-r ~/.kube/config

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
mkdir -p ~/install/mongodb
cd ~/install/mongodb
helm inspect values bitnami/mongodb > mongo.yaml

---

수정 후 설치
useStatefulSet: false -> true
rootPassword: "" -> "1234"
storageClass: "" -> "nks-block-storage"

helm install mongo -f mongo.yaml bitnami/mongodb

## cd ~

kubectl exec -it mongo-mongodb-0 /bin/bash

mongosh admin -u root -p 1234

use auth;
db.createCollection("users");
db.users.insert({"userid":"1","password":"$2b$10$OQlrdbdEeQHdIL0iQXNmqe6BfWBdbnYBmWRc/Vpy4iGbVTGQYtAa2","submitrole":"0","role":"0"});
db.createUser({"user":"kjw","pwd":"1234","roles":["readWrite","userAdmin"]});

---

exit 수동
exit 수동

---

git clone https://github.com/Korjw/APCYaml.git
cd APCYaml

---

wget https://github.com/strimzi/strimzi-kafka-operator/releases/download/0.24.0/strimzi-0.24.0.tar.gz
kubectl create ns kafka
tar -zxvf strimzi-0.24.0.tar.gz
cd strimzi-0.24.0/
sed -i 's/namespace: .*/namespace: kafka/' install/cluster-operator/*RoleBinding\*.yaml

---

vi install/cluster-operator/060-Deployment-strimzi-cluster-operator.yaml

        env:
        - name: STRIMZI_NAMESPACE
          value: default --> 추가
          #valueFrom:   --> 주석처리
          #  fieldRef:  --> 주석처리
          #    fieldPath: metadata.namespace  --> 주석처리

kubectl apply -f install/cluster-operator/ -n kafka
kubectl create -f install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml
kubectl create -f install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml
kubectl get pods -n kafka

---

cd ..

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
kubectl create ns ingress-nginx
helm repo update
helm search repo ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx

kubectl apply -f kafka-pv.yaml -f kafka-cluster.yaml -f kafka-topic.yaml

kubectl get all

---

kubectl apply -f auth_service.yaml -f auth_deploy.yaml -f dashboard_service.yaml -f dashboard_deploy.yaml -f kafka_deploy.yaml -f kafka_service.yaml

---

kubectl apply -f nginx-ingress.yaml
kubectl get ing

타겟그룹  30009 30920 30561 무조건 생성
리스너 4000 9200 5601

---

kubectl apply -f elasticsearch.yaml -f kibana.yaml

---

kubectl apply -f fluentd.yaml -f fluentd-config.yaml

---

프로메테우스

---

optional

---
kubectl exec my-cluster-kafka-0 -it -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic
kubectl exec my-cluster-kafka-0 -it -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning
