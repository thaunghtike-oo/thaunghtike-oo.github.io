---
layout:     post
title:   "Argocd Image Updater"
date:       2023-06-08 15:13:18 +0200
image: argocdimageupdater.png
tags:
    - devops
    - argocd
categories: DevOps
---

<p>စာမရေးဖြစ်တာလည်း အတော်လေးကြာသွားပါပြီ။ ဒီနေ့တော့ ကျွန်တော် Argo CD အကြောင်းလေးနဲ့ပြန်လည်အစပြုလိုက်ပါမယ်။ DevOps Engineer တော်တော်များများ argocd ကတော့ရင်းနှီးပြီးသားဖြစ်မယ်ထင်ပါတယ်။ Topic တွေအများကြီးထဲကမှ Argo CD Image Updater အကြောင်းကို နားလည်သလောက် ရေးသွားပါမည်။ </p>

<h2>👉 Introduction</h2>

<p>တစ်နည်းအားဖြင့် image updater ဆိုတာ kubernetes workload တွေမှာအသုံးပြုထားတဲ့ container images တွေကို automatically update လုပ်ပေးသွားတဲ့ tool တစ်ခုလည်းဖြစ်ပါတယ်။ </p>

> A tool to automatically update the container images of Kubernetes workloads that are managed by Argo CD. The Argo CD Image Updater can check for new versions of the container images that are deployed with your Kubernetes workloads and automatically update them to their latest allowed version using Argo CD.


<p> ဆိုလိုရင်းကတော့ container registry ထဲမှာ image tag ဒါမှမဟုတ် digest အသစ်တစ်ခုရောက်လာတာနဲ့ GitOps repo ထဲမှာ image အသစ် နဲ့ update လုပ်ပေးသွားမှာဖြစ်ပါတယ်။ </p>

<h2>👉 Image Updater အကြောင်း</h2>

<p>image updater မပေါ်ခင်တုန်းကတော့ CICD မှာ script တွေရေးကြပြီး image အသစ်ကို gitops repo ထဲမှာ update ဖြစ်အောင်လုပ်ခဲ့ကြတာပေါ့။ ဒါဆို အနည်းငယ်တော့ သဘောပေါက်လောက်ပြီထင်တယ်။ အသေးစိတ်ကိုအောက်မှာ ဆက်လက်ဖတ်ရှုပေးကြပါ။

သူရဲ့ ကောင်းတဲ့အချက်က docker image တွေကို build and push လုပ်ပြီးတိုင်းမှာ ကျွန်တော်တို့ gitops repoမှာ image tag ကိုသွားပြောင်းပေးဖို့မလိုပါဘူး။ argocd image updater ကနေ အဲ့ဒါကို လုပ်ပေးသွားမှာဖြစ်ပါတယ်။ gitops repo ထဲမှာ မပြောင်းပဲ cluster ထဲကို တိုက်ရိုက် image update အောင်လည်း လုပ်လို့လည်းရပါတယ်။ ဒါကတော့ productionအတွက်မသင့်တော်ပါဘူး</p>

<h2>👉 Install လုပ်နည်း</h2>

Argocd ကို install ခဲ့တဲ့ namespace ထဲမှာပဲ image updater ကိုလည်း install ပေးဖို့လိုပါတယ်။ [ဒီမှာ](https://argocd-image-updater.readthedocs.io/en/stable/install/installation/) instruction တွေကြည့်ပြီး install နိုင်ပါတယ်။

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```
ပြီးသွားတဲ့အခါမှာတော့ အောက်ကလို image updater pod တစ်ခု run နေတာကိုတွေ့ရပါလိမ့်မည်။

```bash
$ kubectl get pods -n argocd
NAME                                                READY   STATUS    RESTARTS       AGE
argocd-application-controller-0                     1/1     Running   0              3d23h
argocd-applicationset-controller-59dcb74c66-8g2bn   1/1     Running   0              7d1h
argocd-image-updater-947df9d9b-7khm9                1/1     Running   0              43h
argocd-notifications-controller-56b4b9db6f-kprlx    1/1     Running   0              7d1h
argocd-redis-74d77964b-6dl5q                        1/1     Running   0              3d23h
argocd-repo-server-6dfd684ff-knj8z                  1/1     Running   0              7d1h
argocd-server-676c6b849b-r6lk5                      1/1     Running   0              7d1h
```
ဒုတိယအဆင့်အနေနဲ့ log level ကို debug ပြောင်းပေးရပါမည်။ argocd-image-updater-config ဆိုတဲ့ configmap ကို edit ပေးရပါမည်။

> kubectl edit configmaps --namespace argocd argocd-image-updater-config

အောက်ပါအတိုင်း log level ကို သတ်မှတ်ပေးလိုက်ပါ။

```bash
apiVersion: v1
data:
  log.level: debug
kind: ConfigMap
```
<p> နောက်ဆုံးအဆင့်အနေနဲ့ argocd image updater ကို ကျွန်တော်တို့ရဲ့ container registry နဲ့ authenticate လုပ်ပေးရပါမယ်။ ခုဏ က configmap ကိုပဲ ပြန် edit ပြီး မိမိရဲ့ registry credentails တွေကိုထည့်ပေးပါ။ အသေးစိတ်ကို [ဒီမှာ](https://argocd-image-updater.readthedocs.io/en/stable/configuration/registries/) ကြည့်နိုင်ပါတယ်။

အရင်ဆုံး docker-registry secret တစ်ခု create ပေးရပါမည်။ ကျွန်တော်ကတော့ azure container registry ကိုသုံးထားပါတယ်။</p>

```bash
kubectl -n argocd create secret docker-registry mycontainerregistry-secret --docker-server=mycontainerregistry.azurecr.io --docker-username=mycontainerregistry --docker-password=PASSWORD -o yaml --dry-run=client | kubectl -n argocd apply -f -
```
<p> credentials နေရာမှာ pull secret ကိုသုံးထားပါတယ် အောက်က argocd က namespace ဖြစ်ပြီး အနောက်ကတော့ registry ရဲ့ secret ဖြစ်ပါတယ်။ </p>

```bash
apiVersion: v1
data:
  log.level: debug
  registries.conf: |
    registries:
    - name: Azure Container Registry
      prefix: dinger.azurecr.io
      api_url: https://myreg.azurecr.io
      credentials: pullsecret:argocd/mycontainerregistry-secret
kind: ConfigMap
metadata:
```
> ဒီနေရာမှာ အရေးကြီးဆုံးတစ်ခုက argocd image updater pod ကို restart လုပ်ပေးရပါမယ်။

<p>pod ကို kubectl delete နဲ့ ဖျက်လိုက်ပါ။ pod အသစ်တစ်ခုပြန်ထွက်လာပါလိမ့်မည်။ </p>

<h2>👉 Let's see how it works </h2>

image ကို update လုပ်တဲ့ strategy တွေ ၄မျိုးလောက်ရှိတယ်။ latest, digest, name စသည်ဖြင့်ပေါ့။ အဲ့ထဲကမှ ကျွန်တော်က latest ကိုသုံးပါမည်။ creation time အရ updated အဖြစ်ဆုံး image နဲ့ workload တွေကို up-to-date ဖြစ်အောင်လုပ်ပေးသွားမှာပါ။

<p> အောက်က argocd application လေးကိုကြည့်ရအောင်။ အထူးသဖြင့် annotations တွေပေါ့။ ကျန်တာတွေကတော့ သာမန် argocd application တွေအတိုင်းပါပဲ </p>

```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: merchant-api-staging
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: myalias=dinger.azurecr.io/api-merchant-staging
    argocd-image-updater.argoproj.io/myalias.update-strategy: latest
    argocd-image-updater.argoproj.io/write-back-method: git
    argocd-image-updater.argoproj.io/git-branch: testing
    argocd-image-updater.argoproj.io/myalias.force-update: "true"
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  destination:
    namespace: staging
    name: in-cluster
  project: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: false
    syncOptions:
      - CreateNamespace=true
      - ApplyOutOfSyncOnly=true
  source:
    path: helm-charts/merchant-api-staging
    repoURL: git@github.com:dinger19/dinger-kubernetes.git
    targetRevision: staging
```
<p>api-merchant-staging ဆိုတဲ့ container image မှာ tag အသစ်တစ်ခုထွက်လာတိုင်း အောက်က git repo ( dinger19/dinger-kubernetes ) ရဲ့ staging branch ထဲက path (helm-charts/merchant-api-staging )ထဲမှာ image အသစ်ကို update ပေးမှာဖြစ်ပါတယ်။

write-back-method annotation မှာ git ဆိုတာက gitops repo ဖြစ်တဲ့ dinger-kubernetes မှာ new containe image ကိုသွား update လုပ်ပေးဖို့ပါ။

code changes ဖြစ်လို့ image အသစ်ရလာတိုင်း argocd image updater ကနေပြီးတော့ helm chart မှာ .argocd-soure ဆိုပြိးအောက်ကအတိုင်း file အသစ်တစ်ခုတွေ့ရမှာဖြစ်ပြီး မကြာခင်မှာ image ကလည်း update ဖြစ်သွားမှာဖြစ်ပါတယ်။</p>

```bash
helm:
  parameters:
  - name: image.name
    value: dinger.azurecr.io/api-merchant-staging
    forcestring: true
  - name: image.tag
    value: d93cc6d
    forcestring: true
```

<p> image update ဖြစ်တာကိုသိနိုင်ဖို့ argocd image updater ရဲ့ pod logs တွေကိုကြည့်ပြီးလည်းသိနိုင်ပါတယ်။

ဒီလောက်ဆို ကိုယ်တိုင်စမ်းသပ်လို့ရပြီထင်ပါတယ်။ အဆင်ပြေကြပါစေ။ အခက်အခဲရှိခဲ့လျှင်လည်း page messengerကဖြစ်စေ email ကဖြစ်စေ မေးနိုင်ပါတယ်ခင်ဗျ။</p>

<h2>👉 Reference</h2>

<ul>
    <li><a href="https://visionsincode.com/2023/02/11/argo-cd-image-updater-will-make-your-gitops-life-easier-for-your-sitecore-setup-in-kubernetes/">https://visionsincode.com/2023/02/11/argo-cd-image-updater-will-make-your-gitops-life-easier-for-your-sitecore-setup-in-kubernetes</a> </li>
</ul>

<p style="text-align:center">
    သင်ဆရာ မြင်ဆရာ ကြားဆရာများကိုလေးစားလျှက် 🙏🙏🙏
</p>
<p style="text-align:center">
   သောင်းထိုက်ဦး (UIT)
</p>
