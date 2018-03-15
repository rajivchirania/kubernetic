# Secrets

Objects of type secret are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys

For more information about Secrets checkout the [Kubernetes User Guide](http://kubernetes.io/docs/user-guide/secrets/).

In this Tutorial we'll see how to build a dynamic Secret and read the secret as a variable from a Pod.

## Secret Sample

We don't want to store sensitive information under version control, and since Charts are stored under version control we want to avoid exposing them. So instead of creating a password and storing it inside the Chart, we'll generate it on-the-fly during deployment.

Go to the tutorial chart repo and run the `secret-sample` chart.

Here is the Secret definition:

```
apiVersion: v1
kind: Secret
metadata:
  name: "{{.Release.Name}}-secret"
  annotations:
    "helm.sh/hook": pre-install
type: Opaque
data:
  password: {{ b64enc (randAlphaNum 32) }}
  username: {{ b64enc "user1" }}
```


Charts are written in [Helm](https://github.com/kubernetes/helm/) format, and they support various [template functions](https://github.com/kubernetes/helm/blob/master/docs/charts.md#templates-and-values). Here is a [list](https://github.com/Masterminds/sprig) of some functions.

* The `.Release.Name` is resolved during the deployment of the release. This enables to have multiple Chart releases on the same namespace.
* In our scenario we'll use `randAlphaNum 32` to generate a random alpha-numeric string of 32 characters.
* Secret values are stored in base64 inside Kubernetes, so we wrap the command with `b64enc` function.
* We want the secret to be deployed before deploying the accompanying Pod, so we add a special Helm annotation `"helm.sh/hook": pre-install` to activate a [Hook](https://github.com/kubernetes/helm/blob/master/docs/charts.md#hooks). Note that the secret will not be deleted automatically when the release is deleted and it needs to be handled separately.

Here is the Pod definition inside the same Chart:

```
apiVersion: v1
kind: Pod
metadata:
  name: "{{.Release.Name}}-pod"
spec:
  terminationGracePeriodSeconds: 0
  restartPolicy: Never
  containers:
  - name: alpine
    image: alpine:3.4
    command: ["echo", "hello $(password)"]
    env:
    - name: password
      valueFrom:
        secretKeyRef:
          name: {{.Release.Name}}-secret
          key: password

```

Now go to the Secrets section. You'll see the generated Secret.

![](/assets/secret.png)

Go to the Pods section and click on the generated Pod you'll see the output of the Pod

![](/assets/secret-pod.png)

![](/assets/secret-pod-output.png)

# Cleanup

Go to the Releases section

![](/assets/cleanup-secret.png)

* Delete the release of the Chart `secret-sample-0.1.0`
* Go to the Secrets section and delete the generated secret.

