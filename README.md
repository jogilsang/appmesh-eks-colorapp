# Reference
https://awskrug.github.io/eks-workshop/servicemesh_with_appmesh/
https://aws.amazon.com/ko/blogs/containers/cross-amazon-eks-cluster-app-mesh-using-aws-cloud-map/

# Prerequisites
export MESH_NAME=<your-choice-of-mesh-name, e.g. default>
export ENVIRONMENT_NAME=<friendlyname-for-stack e.g. AppMeshSample>
export KEY_PAIR_NAME=<key-pair to access ec2 instances where apps are running>
export ENVOY_IMAGE=<the latest recommended envoy image, see https://docs.aws.amazon.com/app-mesh/latest/userguide/envoy.html>
export CLUSTER_SIZE=<number of ec2 instances to spin up to join cluster
export SERVICES_DOMAIN=<domain under which services will be discovered, e.g. "default.svc.cluster.local">
export COLOR_GATEWAY_IMAGE=<image location for colorapp's gateway, e.g. "<youraccountnumber>.dkr.ecr.amazonaws.com/gateway:latest" - you need to build this image and use your own ECR repository, see below>
export COLOR_TELLER_IMAGE=<image location for colorapp's teller, e.g. "<youraccountnumber>.dkr.ecr.amazonaws.com/colorteller:latest" - you need to build this image and use your own ECR repository, see below>

# STEP1. Create the App Mesh

aws appmesh create-mesh --mesh-name APP_MESH_DEMO

## result
```json
{
    "mesh": {
        "meshName": "APP_MESH_DEMO",
        "metadata": {
            "arn": "arn:aws:appmesh:ap-northeast-2:247553399539:mesh/APP_MESH_DEMO",
            "createdAt": "2023-08-31T00:59:26.499000+00:00",
            "lastUpdatedAt": "2023-08-31T00:59:26.499000+00:00",
            "meshOwner": "247553399539",
            "resourceOwner": "247553399539",
            "uid": "617b5439-4e45-4a22-9a9a-dedf8f21eefe",
            "version": 1
        },
        "spec": {},
        "status": {
            "status": "ACTIVE"
        }
    }
}
```

# STEP2. Create Virtual Nodes
A virtual node acts as a logical pointer to a k8s service.
inbound traffic -> virtual node (listener)
outbound traffic -> virtual node (backends)

- colorgateway-vn
- colorteller-vn
- colorteller-black-vn
- colorteller-blue-vn
- colorteller-red-vn

---

- colorgateway-vn

aws appmesh create-virtual-node --mesh-name APP_MESH_DEMO --cli-input-json file://1_create_virtual_node_colorgateway.json

```json
{
    "virtualNode": {
        "meshName": "APP_MESH_DEMO",
        "metadata": {
            "arn": "arn:aws:appmesh:ap-northeast-2:247553399539:mesh/APP_MESH_DEMO/virtualNode/col
orgateway-vn",
            "createdAt": "2023-08-31T01:15:21.569000+00:00",
            "lastUpdatedAt": "2023-08-31T01:15:21.569000+00:00",
            "meshOwner": "247553399539",
            "resourceOwner": "247553399539",
            "uid": "b90873a7-0124-491d-b3a7-015d6b193b67",
            "version": 1
        },
        "spec": {
            "backends": [
                {
                    "virtualService": {
                        "virtualServiceName": "colorteller.default.svc.cluster.local"
                    }
                }
            ],
            "listeners": [
                {
                    "portMapping": {
                        "port": 9080,
                        "protocol": "http"
                    }
                }
            ],
            "serviceDiscovery": {
                "dns": {
                    "hostname": "colorgateway.default.svc.cluster.local"
 
```

- colorteller-vn

aws appmesh create-virtual-node --mesh-name APP_MESH_DEMO --cli-input-json file://2_create_virtual_node_colorteller.json

## result
```json
{
    "virtualNode": {
        "meshName": "APP_MESH_DEMO",
        "metadata": {
            "arn": "arn:aws:appmesh:ap-northeast-2:247553399539:mesh/APP_MESH_DEMO/virtualNode/colorteller-vn",
            "createdAt": "2023-08-31T01:25:40.196000+00:00",
            "lastUpdatedAt": "2023-08-31T01:25:40.196000+00:00",
            "meshOwner": "247553399539",
            "resourceOwner": "247553399539",
            "uid": "ca1b708f-24db-4073-88e7-066746765485",
            "version": 1
        },
        "spec": {
            "listeners": [
                {
                    "portMapping": {
                        "port": 9080,
                        "protocol": "http"
                    }
                }
            ],
            "serviceDiscovery": {
                "dns": {
                    "hostname": "colorteller.default.svc.cluster.local"
                }
            }
        },
        "status": {
            "status": "ACTIVE"
        },
        "virtualNodeName": "colorteller-vn"
    }
```

aws appmesh create-virtual-node --mesh-name APP_MESH_DEMO --cli-input-json file://3_create_virtual_node_colorteller-blue.json

aws appmesh create-virtual-node --mesh-name APP_MESH_DEMO --cli-input-json file://4_create_virtual_node_colorteller-blue.json

aws appmesh create-virtual-node --mesh-name APP_MESH_DEMO --cli-input-json file://5_create_virtual_node_colorteller-red.json

# STEP3. Create Virtual Routers and Routes
Virtual routers handle traffic for one or more service names within your mesh




aws appmesh create-route --mesh-name APP_MESH_DEMO --cli-input-json file://1_create_route_colorgateway.json

aws appmesh create-route --mesh-name APP_MESH_DEMO --cli-input-json file://2_create_route_colorteller.json



# STEP4. Create k8s app

## 4_1_이미지 생성
- Deploy the gateway image

```sh
cd examples/apps/colorapp/src/gateway
aws ecr create-repository --repository-name=gateway
export COLOR_GATEWAY_IMAGE=$(aws ecr describe-repositories --repository-names=gateway --query 'repositories[0].repositoryUri' --output text)
export AWS_ACCOUNT_ID=247553399539
export AWS_DEFAULT_REGION=ap-northeast-2
export GO_PROXY=direct
export DOCKER_DEFAULT_PLATFORM=linux/amd64
./deploy.sh

# (END)가 보일경우 q 로 넘어가기
```

- Deploy the colorteller image

```sh
cd (...)
aws ecr create-repository --repository-name=colorteller
export COLOR_GATEWAY_IMAGE=$(aws ecr describe-repositories --repository-names=gateway --query 'repositories[1].repositoryUri' --output text)
export AWS_ACCOUNT_ID=247553399539
export AWS_DEFAULT_REGION=ap-northeast-2
export GO_PROXY=direct
export DOCKER_DEFAULT_PLATFORM=linux/amd64
./deploy.sh
```

# (END)가 보일경우 q 로 넘어가기
```sh
export ECR_REPO_IMAGE_COLORGATEWAY=247553399539.dkr.ecr.ap-northeast-2.amazonaws.com/gateway:latest
export ECR_REPO_IMAGE_COLORTELLER=247553399539.dkr.ecr.ap-northeast-2.amazonaws.com/colorteller:latest

sed -i -e "s|ECR_REPO_IMAGE_COLORGATEWAY|${ECR_REPO_IMAGE_COLORGATEWAY}|" ./colorapp.yaml
sed -i -e "s|ECR_REPO_IMAGE_COLORTELLER|${ECR_REPO_IMAGE_COLORTELLER}|" ./colorapp.yaml
```

# Reference
kubectl apply -f https://raw.githubusercontent.com/geremyCohen/colorapp/master/colorapp.yaml

## 4_2_colorapp.yaml에 대한 kubectl apply

kubectl apply -f ./colorapp.yaml

- service/colorgateway
- service/colorteller
- service/colorteller-black
- service/colorteller-blue
- service/colorteller-red
- deployment.apps/colorgateway
- deployment.apps/colorteller-white
- deployment.apps/colorteller-blue
- deployment.apps/colorteller-black
- deployment.apps/colorteller-red



## curl 실행가능한 pod
kubectl run curler --image=curlimages/curl -i --tty -- sh -n test
kubectl run test --image=busybox -i --tty -n test

# STEP5. Test the App on App Mesh


각각의 pod에서 아래 명령어 실행 시
curl localhost:9080/color;echo
white
red
blue
black 확인


curl -s --connect-timeout 2 http://colorgateway.default.svc.cluster.local:9080/color

curl -s --connect-timeout 2 http://colorteller-blue.default.svc.cluster.local:9080/color

