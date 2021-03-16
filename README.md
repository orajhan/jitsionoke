# Jitsi Meet app deployment in Kubernetes 

I referred to https://github.com/DushmanthaBandaranayake/jitsi-kubernetes-scalable-service and have made some changes below.\
Actually Dush's 10-config script is the most important one to make HPA work since JVB must have unique port to be scaled horizontally by design. 

### Prerequisite

 a. Build customized image with 10-config and push it into your own repository 

 b. Create your oen secret for jitsi app
```sh
 kubectl create secret generic jitsi-config -n jitsi --from-literal=JICOFO_COMPONENT_SECRET=... --from-literal=JICOFO_AUTH_PASSWORD=... --from-literal=JVB_AUTH_PASSWORD=...
```
 c. Used ingress controller to access Jitsi Meet app in web yaml. \
https://kubernetes.github.io/ingress-nginx/deploy/

### a few changes in Kubernetes

1. From JVB pod, I am accessing to XMPP Server via service in which 5222 so that I do not have to rely on IP address of Jitsi pod since pod ip will be changed if pod is re-created.
- XMPP_SERVER => web
- In Web yaml, add the following ports. 

```sh
 name: "xmpp"   #now JVB is accessing xmpp via service name using port 5222 \
    port: 5222\
    targetPort: 5222 
```

* I have seen many cases that Jitsi kicked the session as soon as the second participant joins the same meeting. This issue most likely related to port issue between JVB and XMPP server. 


2. added resource reuest in jvb yaml. If not, HPA will complain. 
```sh
  resources: \
   requests: \
     cpu: "0.5"  # any  
```

3. If you want to use TLS, please use tls-ingress instead of ingress in web-service.yaml
- tls-ingress.yaml  
- I am using CloudFlare TLS certification but it can be any. 

There may be a few more changes if I recall but these are what you need to know. 
Will update later on...
