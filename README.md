# APCYaml

쿠버네티스 배포 yaml 파일 및 nginx.conf (naver cloud 기준)

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

curl -L https://git.io/getLatestIstio | sh -
cd istio-1.17.1/
for i in install/kubernetes/helm/istio-init/files/crd\*yaml; do kubectl apply -f $i; done
kubectl create namespace istio-system
helm template \
--namespace istio-system \
--set tracing.enabled=true \
--set global.mtls.enabled=true \
--set grafana.enabled=true \
--set kiali.enabled=true \
--set servicegraph.enabled=true \
install/kubernetes/helm/istio \

> ./istioFex.yaml
> kubectl apply -f istioFex.yaml
> kubectl get pod --namespace=istio-system
> kubectl label namespace default istio-injection=enabled
> kubectl get ns --show-labels

---

nginx ingress & cert-manager

kubectl create namespace nginx-ingress
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
kubectl create
helm install nginx nginx-stable/nginx-ingress --namespace nginx-ingress --set controller.enableCustomResources=true --create-namespace --set controller.enableCertManager=true --set controller.enableExternalDNS=true

helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
 --namespace cert-manager --create-namespace \
 --version v1.9.1 --set installCRDs=true

---

vi samples/addons/kiali.yaml
nodeport 설정 및 prometheus url 설정

    external_services:
      prometheus:
        url: http://istio-syste-istio-ingres-8ead8-16689455-20d653ba2048.kr.lb.naverncp.com:9090

spec:
type: NodePort
ports:

- name: http
  port: 20001
  targetPort: 20001
  nodePort: 32001
- name: http-metrics
  appProtocol: http
  protocol: TCP
  port: 9090
  selector:
  app.kubernetes.io/name: kiali
  app.kubernetes.io/instance: kiali

kubectl apply -f samples/addons/kiali.yaml

타겟그룹 32001 생성
리스너 20001

---

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
mkdir -p ~/install/mongodb
cd ~/install/mongodb
helm inspect values bitnami/mongodb > mongo.yaml
vi mongo.yaml

---

수정 후 설치

---

useStatefulSet: false -> true
rootPassword: "" -> "1234"
storageClass: "" -> "nks-block-storage"

---

helm install mongo -f mongo.yaml bitnami/mongodb
cd ~

---

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

---

        env:
        - name: STRIMZI_NAMESPACE
          value: default --> 추가
          #valueFrom:   --> 주석처리
          #  fieldRef:  --> 주석처리
          #    fieldPath: metadata.namespace  --> 주석처리

---

kubectl apply -f install/cluster-operator/ -n kafka
kubectl create -f install/cluster-operator/020-RoleBinding-strimzi-cluster-operator.yaml
kubectl create -f install/cluster-operator/031-RoleBinding-strimzi-cluster-operator-entity-operator-delegation.yaml
kubectl get pods -n kafka

---

cd ..

kubectl apply -f kafka-pv.yaml -f kafka-cluster.yaml -f kafka-topic.yaml

kubectl get all

---

kubectl apply -f auth_service.yaml -f auth_deploy.yaml -f dashboard_service.yaml -f dashboard_deploy.yaml -f product_service.yaml -f product_deploy.yaml -f order_service.yaml -f order_deploy.yaml
kubectl apply -f kafka_deploy.yaml -f kafka_service.yaml

---

cloudflare에 record CNAME 기록
cd cert
kubectl apply -f secret.yaml -f issuer.yaml
cd ..
kubectl apply -f nginx-ingress.yaml
kubectl get ing

---

## kubectl apply -f elasticsearch.yaml -f kibana.yaml

타겟그룹 30012 30920 30561
리스너 4000 9200 5601

---

kubectl apply -f fluentd.yaml -f fluentd-config.yaml

---

프로메테우스

---

cd monitoring
kubectl create ns monitoring

kubectl apply -f prometheus-cluster-role.yaml -f prometheus-config-map.yaml
kubectl apply -f prometheus-deployment.yaml -f prometheus-node-exporter.yaml -f prometheus-svc.yaml
kubectl apply -f kube-state-cluster-role.yaml -f kube-state-svcaccount.yaml -f kube-state-deployment.yaml -f kube-state-svc.yaml
kubectl apply -f grafana.yaml

CRD 설정
kubectl create -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml

---

타겟그룹 30003 30004
리스너 9090 3000

---

git clone https://github.com/danielqsj/kafka_exporter.git
cd kafka_exporter/charts/kafka-exporter
vi values.yaml

---

수정
kafkaExporter:
kafka:
servers: - my-cluster-kafka-bootstrap.default.svc:9092

helm install kafka-exporter . -n monitoring

---

optional

---

17866 kubernetes
10122 카프카
7639 istio service
7645 istio control plane
14900 14314 nginx ingress controller

istioctl x uninstall --purge

kubectl exec my-cluster-kafka-0 -it -- bin/kafka-console-producer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic
kubectl exec my-cluster-kafka-0 -it -- bin/kafka-console-consumer.sh --bootstrap-server my-cluster-kafka-bootstrap:9092 --topic my-topic --from-beginning

---

인증서 (istio)

---

helm repo add jetstack https://charts.jetstack.io
helm repo update
helm3 install cert-manager --namespace cert-manager --version v0.16.1 jetstack/cert-manager --set installCRDs=true --set 'extraArgs={--dns01-recursive-nameservers=8.8.8.8:53,1.1.1.1:53}'
cd cert
kubectl apply -f secret.yaml -f issuer.yaml -f certificate.yaml
istio-gateway https 적용

가비아 domain : kim0418.shop
