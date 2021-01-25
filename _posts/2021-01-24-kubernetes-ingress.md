---
title:  "Kubernetes Pi: Ingress"
date:   2020-01-24 00:00:00 +0000
categories: raspberrypi
tags:
    - cluster
    - raspberry pi
    - kubernetes
    - ingress
classes: wide
toc: true
header: 
    overlay_image: /assets/images/Kubernetes/traefik.jpg 
    overlay_filter: rgba(0, 0, 0, 0.4)
published: true

---

A brief explanation of Kubernetes Ingress.

This blog post is one in a series of posts about my experiences of deploying kubernetes to a Raspberry Pi cluster, if you haven't seen the others i'd advise checking them out [here](http://hfiorillo.github.io/). Today's blog, is more of a precursor to what is coming next.

## How do we expose services on a Kubernetes cluster?

There are a variety of different methods we can use to view the services running on our Kubernetes cluster, a [service](https://kubernetes.io/docs/concepts/services-networking/service/?utm_source=thenewstack&utm_medium=website&utm_campaign=platform) being a group of deployed pods that can then be viewed under a single cluster IP address. The service does the job of a typical [round robin load balancer](https://avinetworks.com/glossary/round-robin-load-balancing/) by hiding the ephemeral nature of the pods it is exposing which are scaled and replaced quickly at any time. However, these services are only accessible from within the cluster itself. This is was apparent at the dawn of k8s and so two mechanisms were directly integrated into service specification, NodePort and LoadBalancer, which can both be easily selected when writing a Kubernetes service manifest. Whilst both of these mechanisms work extremely well in specific circumstances they do have their caveats. NodePort's [random port selection](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) can mean its more difficult to configure other networking rules around it and [LoadBalancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/) only works when using a public cloud-hosted environment (public IP addresses can be expensive!). Defining many NodePort services creates a mess of random ports and defining LoadBalancer services can lead to paying for large amounts of cloud resources.

### Ingress

In order to circumnavigate these issues, Kubernetes birthed a new type of manifest, an Ingress. A mechanism, similar to the services mentioned above, that allows clients in an external network to invoke services running in a k8s cluster. [Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/) defines Ingress as an  '*API object that manages external access to the services in a cluster... Ingress exposes HTTP and HTTPS routes from outside the cluster to services within the cluster*'. To manage the access, an Ingress YAML manifest is written that declares the rules about how you want clients to be routed to a particular service. An example can be seen below:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
	namepsace: default
spec:
  rules:
    - http:
        paths:
          - path: /foo
            backend:
              serviceName: foo-service
              servicePort: 80
    - http:
        paths:
          - path: /bar
            backend:
              serviceName: bar-service
              servicePort: 80
```

### Ingress Controllers

The ingress manifest alone is useless, it requires an Ingress Controller which looks at the manifest and acts upon it by dynamically configuring itself based on the YAML objects defined in the resource. Ingress controllers are pods, so they're part of the cluster, and so they can see all the other pods. They are constructed using [reverse proxies](https://en.wikipedia.org/wiki/Reverse_proxy) which give it [Layer 7 routing](https://rancher.com/docs/rancher/v2.x/en/k8s-in-rancher/load-balancers-and-ingress/load-balancers/) and load balancing capabilities. Ingress controllers still require them to be exposed externally via a type of either NodePort or LoadBalancer but now there is a single entry point of traffic. As the Ingress Controller is implemented at level 7 it is able to terminate HTTPS, allowing it to distinguish each websites traffic as it understands url names and paths, therefore knowing which backend server to forward traffic to. In the example manifest above, you set up the services for your pods but don't assign them a type field - routing and load balancing is handled by the Ingress layer. The two paths of /foo and /bar are served by a common IP and domain name. Essentially an [API Gateway Pattern](https://learnk8s.io/kubernetes-ingress-api-gateway). Once installed, Ingress Controllers do all the work in the background, defining Ingress manifests for the controller will be extent of your troubles. 

A deep dive into ingress and ingress controllers can be found [here](https://oteemo.com/2019/10/28/ingress-101-what-is-kubernetes-ingress-why-does-it-exist/).

![Ingress](/assets/images/Kubernetes/ingress.png)

Reverse proxies have been around for years, as such there is a plethora of ingress controllers to choose from, here are a few worthy mentions:

- [NGINX Ingress Controller](https://github.com/kubernetes/ingress-nginx)
- [HAProxy Ingress Controller](https://www.haproxy.com/documentation/kubernetes/latest/installation/)
- [Traefik](https://traefik.io/)

The choice of ingress controller for my cluster is Traefik, this comes as default on Ranchers k3s - boring I know. It is easy to use, production ready and really popular. It's open source too. Some of its coolest features include:

- Service discovery to configure itself dynamically from the services themselves.
- All major protocols are supported and can be flexibly managed using configurable middle wares; load balancing, rate-limiting, circuit-breakers, mirroring and authentication.
- Supports SSL termination and can be used with an ACME provider such as [Lets Encrypt](https://letsencrypt.org/) for automatic certificate generation.

![https://traefik.io/traefik/#:~:text=Traefik is a leading modern,that makes deploying microservices easy.&text=It also comes with a,west service communication and more](/assets/images/Kubernetes/diagram.png).

Short and sweet today. I did attempt to expose the Traefik dashboard using ingress as an example but really struggled using the version of k3s I have deployed on my Pi's, perhaps I'll give it a whirl in the future and update this article then. I'll be back next week discussing all things monitoring and walk through how to deploy a monitoring stack of your own.