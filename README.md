Context:
I’ve never used GCP so I’m a bit unfamiliar with some of the terminology. Thankfully, knowing AWS and Terraform helps with having a general understanding.
I’m also very new to Kubernetes and am taking various courses on learning it.


Step 1:
Fork repo.

I ran into permission issues so I created my own repo: git@github.com:gzholder/appdat_exam.git

Step 2:
Create GKE cluster.

I looked at the [README](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine) and used that as a template.

Step 3:
Write a kustomization that deploys Podinfo to a cluster.

I was stuck on figuring out how to take the code for Podinfo and use it as an artifact to deploy. How would this Kustomization know to look at the Podinfo repo?
I copied the [DOCKERFILE](https://github.com/stefanprodan/podinfo/blob/master/Dockerfile) as this looked like this was all needed. Specifically
```bash
RUN CGO_ENABLED=0 go build -ldflags "-s -w \
    -X github.com/stefanprodan/podinfo/pkg/version.REVISION=${REVISION}" \
    -a -o bin/podinfo cmd/podinfo/*

RUN CGO_ENABLED=0 go build -ldflags "-s -w \
    -X github.com/stefanprodan/podinfo/pkg/version.REVISION=${REVISION}" \
    -a -o bin/podcli cmd/podcli/*
```

I wasn’t sure how to automate the actual install. I think adding the following to the Dockerfile may help:
```bash
RUN kubectl apply -k github.com/stefanprodan/podinfo//kustomize
```
If kubectl isn't installed on the clusters then adding the following prior to the previous command  may help:
```bash
RUN  curl -L https://dl.k8s.io/v1.10.6/bin/linux/amd64/kubectl -o /usr/local/bin/kubectl
```

Step 4:
Setting up ingress. I created two files, one for ingress and one for internal services. The ingress will take all incoming requests from the backend function and redirect them to the service and port specified in services.yaml.

