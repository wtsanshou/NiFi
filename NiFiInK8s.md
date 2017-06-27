# Run Nifi in Kubernetes

## Deploy Nifi in K8s

In k8s master, run nifi.yaml (in the folder k8s_yaml):
```bash
kubectl create -f nifi.yaml
```

**Note:** If nothing is running, the pod in k8s will be completed. So that, use the command `tail -f /dev/null` in the nifi.yaml

## Access k8s service

The following works are doing in NiFi GUI.

### ListenHttp

It is used to listen recived data from k8s micro service

**e.g:**

**Nifi Setting:**

**Base Path:** nifiListener
**Listening Port:** 8086

**k8s Setting:**

**Http Post to:** http://<nifi_ip>:8086/nifiListener

Here we can use kube-dns to resove nifi_ip. e.g: http://nifi-internal-server-svc.data-cleaning.svc.cluster.local:8086/nifiListener

### PostHttp

It is used to POST data to a target

**e.g:**

**Nifi Setting:**
```
URL: http://dcweb-internal-server-svc.data-cleaning.svc.cluster.local:8084/publish

SendAsFlowFile: false
Use Chunked Encoding: false
Content-type: application/json
```

**Note:** PostHttp doesn't have response function

### InvokeHttp

It could Http methods (GET, POST, PUT ...) and has a lot more routing options (Failure, No Retry, Original, Response, Retry)

We used this to get response from k8s micro service.


