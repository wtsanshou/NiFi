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

## Demo

### Sensor

POST to:  http://nifimagr-internal-server-svc.data-cleaning:8086/nifiMagrListener

### NiFi Manager

* ListenHTTP: 
```
Base Path: nifiMagrListener

Listening Port: 8086
```

* PostHTTP:
```
URL: http://nificleaner-internal-server-svc.data-cleaning.svc.cluster.local:8086/nifiCleanerListener

SendAsFlowFile: false
Use Chunked Encoding: false
Content-type: application/json
```

### NiFi Cleaner

* ListenHTTP: 

```
Base Path: nifiCleanerListener

Listening Port: 8086
```

* InvokeHTTP:
```
HTTP Method: POST
Remote URL: http://cleanmicro-internal-server-svc.data-cleaning.svc.cluster.local:8080/v0.1/clean

SendAsFlowFile: false
Use Chunked Encoding: false
Content-type: application/json
```

* PostHTTP:

Relationship with *InvokeHTTP*: `Response`

```
URL: http://nifitoweb-internal-server-svc.data-cleaning.svc.cluster.local:8086/nifiWebListener

SendAsFlowFile: false
Use Chunked Encoding: false
Content-type: application/json
```

### Cleaner

* ListenHTTP and Response

```
Base Path: v0.1/clean

Listening Port: 8084
```

### NiFi To Web

* ListenHTTP: 

```
Base Path: nifiWebListener

Listening Port: 8086
```

* PostHTTP:
```
URL: http://dcweb-internal-server-svc.data-cleaning.svc.cluster.local:8084/publish

SendAsFlowFile: false
Use Chunked Encoding: false
Content-type: application/json
```

### Web Server

* ListenHTTP: 

```
Base Path: publish

Listening Port: 8084
```
