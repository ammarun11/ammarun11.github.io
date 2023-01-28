---
title: "[Preview] DevOps With Gitlab CI/CD"
date: 2023-01-28
categories: [ngoprek, server, cloud, docker, cicd, gitlab, devops]
tags:
  - Jekyll
  - update
---

## Preview 


### DevOps pipeline

![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1gitlab.png)

### Stack Technology,
- Gitlab as CI/CD
- Harbor
- Kubernetes
- Trivy Scan image


### CI/CD Staging pipeline

![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2gitlab.png)
![Dashboard3](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3gitlab.png)
![Dashboard4](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/4gitlab.png)
![Dashboard5](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/5gitlab.png)

### Sample pipeline gitlab

```yaml
stages:
  - build
  - scan
  - update-chart-and-code
  - deploy-stg
  - create-mr
  - prod-approval
  - deploy-prod

variables:
  IMAGE: "cx-harbor.lab.id/ammar/odoo-gitlab"
  APP_NAME_STG: "odoo-gl-stg"
  REPO_BRANCH_STG: "staging"
  APP_NAME: "odoo-gl-prod"
  REPO_BRANCH: "production"
  REPO_URL: "https://go.lab.id/ammar/odoo-source.git"
  REPO_URL_2: "https://go.lab.id/ammar/odoo-source"
  HELM_REPO: https://cx-harbor.lab.id/chartrepo/ammar

before_script:
  - docker login $HARBOR_URL -u $HARBOR_USERNAME -p $HARBOR_PASSWORD
  - docker info
  - helm repo add ammar $HELM_REPO --force-update
  - helm repo list

Build and Push Image:
  stage: build
  only:
    - staging
  script:
    - export TAG=$(git tag --list | sort -V | tail -n1)
    - echo $TAG
    - cd odoo
    - echo "Build Container Image"
    - docker build -t $IMAGE:16-latest .
    - docker tag $IMAGE:16-latest $IMAGE:16-$TAG-rc
    - docker tag $IMAGE:16-latest $IMAGE:16-$TAG
    - docker push $IMAGE:16-$TAG-rc
    - docker push $IMAGE:16-$TAG
    - docker push $IMAGE:16-latest

Scan Image:
  stage: scan
  only:
    - staging
  before_script:
    - export TAG=$(git tag --list | sort -V | tail -n1)
    - echo $TAG
  script:
    - echo "$TAG image scanned using Harbor integrated Trivy Scanner"
    - trivy image --no-progress $IMAGE:16-$TAG-rc
    # - trivy image --no-progress --exit-code 0 --severity HIGH,CRITICAL $IMAGE:16-$TAG-rc
    - sleep 5

Push Chart & Code:
  allow_failure: true
  stage: update-chart-and-code
  only:
    - staging
  before_script:
    - export TAG=$(git tag --list | sort -V | tail -n1)
    - export IMAGE_STG=cx-harbor.lab.id/ammar/odoo-gitlab:16-$TAG-rc
    - export IMAGE_PROD=cx-harbor.lab.id/ammar/odoo-gitlab:16-$TAG
    - echo $TAG
  script:
    # Git Preparation
    - git config user.email "ammar@lab.id"
    - git config user.name "ammarun11"
    - git checkout staging
    - git pull origin staging
    # --- Update Helm Chart Version ---
    - yq e -i '.version=strenv(TAG)' helm/odoo-gl/Chart.yaml
    - yq e -i '.appVersion=strenv(TAG)' helm/odoo-gl/Chart.yaml
    - cat helm/odoo-gl/Chart.yaml
    # --- Update Staging Values ---
    - yq e -i '.container.image=strenv(IMAGE_STG)' helm/odoo-gl/values.yaml
    - cat helm/odoo-gl/values.yaml
    - yq e -i '.container.image=strenv(IMAGE_STG)' helm/odoo-gl/values-stg.yaml
    - cat helm/odoo-gl/values-stg.yaml
    # --- Update Production Values ---
    - yq e -i '.container.image=strenv(IMAGE_PROD)' helm/odoo-gl/values-prod.yaml
    - cat helm/odoo-gl/values-prod.yaml
    # --- Update Environment Variable ---
    - echo VERSION=$TAG > helm/version.env
    # --- Push Chart to Harbor ---
    - helm repo add ammar $HELM_REPO --force-update
    - helm cm-push --username $HARBOR_USERNAME --password $HARBOR_PASSWORD helm/odoo-gl/ ammar
    - helm repo update
    # --- Push Code Back to Staging Branch ---
    - git add .
    - git status
    - git commit -m "Update Helm Version to $TAG"
    - git push -o ci.skip "https://ammarun11:${ACCESS_TOKEN}@go.lab.id/ammar/odoo-source.git" $REPO_BRANCH_STG

Deploy Staging:
  stage: deploy-stg
  environment: staging
  only:
    - staging
  before_script:
    - export TAG=$(git tag --list | sort -V | tail -n1)
    - echo $TAG
    - export EXIST_STG=$(helm list -n odoo-gl-stg | tail -n +2 | wc -l)
    - echo $EXIST_STG
  script:
    - git checkout staging
    - git pull origin staging
    - echo "Deploying Staging Application..."
    - if [ "$EXIST_STG" == "0" ]; then helm install $APP_NAME_STG ammar/odoo-gl --version $TAG --values helm/odoo-gl/values-stg.yaml --wait --namespace $APP_NAME_STG; else helm upgrade $APP_NAME_STG ammar/odoo-gl --version $TAG --values helm/odoo-gl/values-stg.yaml --wait --namespace $APP_NAME_STG; fi
    - helm list -n $APP_NAME_STG
    - kubectl get all -n $APP_NAME_STG

Create MR:
  before_script: []
  needs: ["Deploy Staging"]
  variables:
    GIT_STRATEGY: none
  stage: create-mr
  only:
    - staging
  script:
    - ./merge-request.sh

Approval Release:
  stage: prod-approval
  only:
    - production
  environment:
    name: production
    url: $REPO_URL
  script:
    - curl -s -X POST https://api.telegram.org/bot$TELE_TOKEN/sendMessage -d chat_id=$TELE_CHAT_ID -d parse_mode=markdown -d text="Dear Team *ammar*, Please Approve Deployment of * $APP_NAME * on Branch *production*. Please goes to $REPO_URL_2/-/environments to *Accept.*"

Deploy Production:
  needs: ["Approval Release"]
  stage: deploy-prod
  only:
    - production
  environment: 
    name: production
  when: manual
  allow_failure: false
  before_script:
    - export TAG=$(git tag --list | sort -V | tail -n1)
    - echo $TAG
    - export EXIST_PROD=$(helm list -n odoo-gl-prod | tail -n +2 | wc -l)
    - echo $EXIST_PROD
  script:
    - echo "Deploying Production Application..."
    - if [ "$EXIST_PROD" == "0" ]; then helm install $APP_NAME ammar/odoo-gl --version $TAG --values helm/odoo-gl/values-prod.yaml --wait --namespace $APP_NAME; else helm upgrade $APP_NAME ammar/odoo-gl --version $TAG --values helm/odoo-gl/values-prod.yaml --wait --namespace $APP_NAME; fi
    - helm list -n $APP_NAME
    - kubectl get all -n $APP_NAME
```