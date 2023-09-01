
# 3단계: 서비스 생성 또는 업데이트

aws iam create-policy --policy-name my-policy --policy-document file://proxy-auth.json

# request
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "appmesh:StreamAggregatedResources",
            "Resource": [
                "arn:aws:appmesh:ap-northeast-2:247553399539:mesh/color/virtualNode/my-service-a_my-apps"
            ]
        }
    ]
}

# result
```json
{
    "Policy": {
        "PolicyName": "my-policy",
        "PolicyId": "ANPATTI2VDLZU3EHZYJ3U",
        "Arn": "arn:aws:iam::247553399539:policy/my-policy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2023-08-30T02:47:26+00:00",
        "UpdateDate": "2023-08-30T02:47:26+00:00"
    }
}
```

--override-existing-serviceaccounts: 이미 존재하는 서비스 계정과 이름이 충돌할 경우 이 옵션을 사용하여 기존 계정을 덮어쓰거나 재정의할 수 있습니다.
--approve: 명령어를 실행하기 전에 변경 사항을 승인합니다. 이 옵션이 없으면 명령어가 실행되기 전에 변경 사항을 미리 확인하라는 메시지가 표시됩니다.

export CLUSTER_NAME=my-cluster
export IAMSERVICEACCOUNT_NAME=color-a

eksctl create iamserviceaccount \
    --cluster $CLUSTER_NAME \
    --namespace my-apps \
    --name my-service-a \
    --attach-policy-arn  arn:aws:iam::247553399539:policy/my-policy \
    --override-existing-serviceaccounts \
    --approve

---

## result
```sh
2023-08-30 02:53:52 [ℹ]  1 existing iamserviceaccount(s) (appmesh-system/appmesh-controller) will be excluded
2023-08-30 02:53:52 [ℹ]  1 iamserviceaccount (my-apps/my-service-a) was included (based on the include/exclude rules)
2023-08-30 02:53:52 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
2023-08-30 02:53:52 [ℹ]  1 task: { 
    2 sequential sub-tasks: { 
        create IAM role for serviceaccount "my-apps/my-service-a",
        create serviceaccount "my-apps/my-service-a",
    } }2023-08-30 02:53:52 [ℹ]  building iamserviceaccount stack "eksctl-my-cluster-addon-iamserviceaccount-my-apps-my-service-a"
2023-08-30 02:53:52 [ℹ]  deploying stack "eksctl-my-cluster-addon-iamserviceaccount-my-apps-my-service-a"
2023-08-30 02:53:52 [ℹ]  waiting for CloudFormation stack "eksctl-my-cluster-addon-iamserviceaccount-my-apps-my-service-a"
2023-08-30 02:54:22 [ℹ]  waiting for CloudFormation stack "eksctl-my-cluster-addon-iamserviceaccount-my-apps-my-service-a"
2023-08-30 02:54:23 [ℹ]  created serviceaccount "my-apps/my-service-a"
```