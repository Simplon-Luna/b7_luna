·

Last week at work, I worked on OaaS (Orchestrator as a Service) on the bash script, as well as making the bash script and pipelines for automating the validation and merging of branches on GitLab's merge requests from working branches to main branch.

——

According to me, Docker is a platform that handles/host container images while K8s is a platform that handles a whole virtual infrastructure in a set of nodes controlled by its controller. In those nodes, K8s can host a lot of different complements, like pods (to host container images for example), services, etc...

——

A projected volume is a set of volumes to be accessible by a pod but not dedicated to it and potentially regroupping multiple sources under the same folder.

A concrete use case would be the usage of the same projected volume to mulitple pods, using the same user (identical RunAsUser), insuring that way that all of them have the proper rights to the files projected. But it could also be used to project token·s (like API tokens) to multiple mods while handling the generation safekeeping outside of the pods.

——
I'll consider that the AKS cluster is already set properly and that the GPU is indeed dedicated to a single node, that I'll name "node 1".

A way to do that would be to taint a node (let's say Node 1) with a "GPU_required" taint and add a matching toleration in the AI Engine pod, that would "repel" the WebUI pod and wouldn't repel the AI engine pod. Then, using a "nodeSelector" to target Node 1's label to the AI Engine pod would ensure that this pod would be on the proper node with the GPU.

That way, the AI Engine would have access to the resources it needs without competition with other pods that aren't granted the tolerance and would be easily adaptable.

——

1 - The issue that the author is facing is that his subprocess doesn't find a file while it is supposedly present in the folder.

2 - The root cause of the issue is a lack of "RTFM"... Or a misunderstanding of said manual. It is said that Python subprocesses can't find executables in paths with quotes and that arguments are required as a string or sequence of program arguments. Here, the args are "none"... - Not a bug.

3 - The easiest way to solve the issue TEMPORARILY, would be to change the default "shell = False" to "shell = True". But it will lead to other issues. The proper way to handle it would be to check that all paths are without quotations, and the manual also include that arguments should be used while they're using "none". I quote : "args is required for all calls and should be a string, or a sequence of program arguments."
——
While monitoring the network traffic an attacker could find custom HTTP request headers and change the object ID to gain access rights that wasn't intended.

As the example quoted from the link :

While monitoring the network traffic of a wearable device, the following HTTP PATCH request gets the attention of an attacker due to the presence of a custom HTTP request header X-User-Id: 54796. Replacing the X-User-Id value with 54795, the attacker receives a successful HTTP response, and is able to modify other users' account data.

——
```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp
    image: my-app-image
    command: [required commands]
  initContainers:
  - name: init-register
    image: ubuntu:latest
    command: ["./register-app.sh"]
  restartPolicy: OnFailure
```

The manifest is at its core a basic pod deployment manifest but we must include the "initContainers:" field to specify to run that initiating container before any other containers of the pod and at it's start, right after getting network and volumes.

In that field, we specify a basic image to run the bash script, provided by the "state-of-the-art" IT department and run the bash script with the command.

I specified a restart policy so that, for whatever reason it fails, it'll try again.

——

The problem that the Conventional Commits specification tries to solve is non-standard commit messages in aim of making the commit history more readable and easier to be treated with automation.

If following the shorter type of conventional commits, I think that it is a good behavior while working with multiple persons on the same project. Especially notifying the breaking changes with the addition of a "!" after the type would allow automations to ignore the commit and not merge it.

But humans being humans, it's definitely not fool-proof and an automated commit message formats that devs can just fill.

——

"A total of 82% said a lack of true flexibility and customization is the top reason so much time is spent on building and managing internal tools and workflows."

"The only way to effectively address those conflicting goals is to adopt a platform that automates much of the workflow associated with building an application development environment" said Amano.

——

