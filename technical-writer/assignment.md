# Debug Commands in Kubernetes

Debug commands allow you to diagnose and resolve issues within your Kubernetes application. Whether your application is hosted by a single Pod or scales to multiple replicas, basic debugging helps you:

- Check which Pods are currently active
- Get detailed information about each Pod
- Issue commands on containers within a Pod
- Troubleshoot issues with your application

To debug in Kubernetes, you can use `kubectl`, the Kubernetes Command Line Interface (CLI). `kubectl` communicates with the Kubernetes API and returns useful information about your application.

> **Tip**
> 
> This page describes `kubectl` commands for common use cases. For a full list of `kubectl` commands and flags, consult the [Kubectl documentation](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands). For detailed introductions on how to install and get started with `kubectl`, check out the [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/) page.

To get started with the commands on this page, open a new terminal window.

> **Note**
>
> In all examples on this page, replace placeholders like `<pod-name>`, `<namespace>` and `<container-name>` with actual values.

## Check Pod Status

The first step to debugging your application is to check if it is working properly. Use the `kubectl get` command to retrieve information about the status of your application.

For example, the `kubectl get pods` subcommand displays the status of each Pod:

```shell
kubectl get pods
```

**Example Output:**
```shell
NAME         READY   STATUS    RESTARTS   AGE  
pod-1-name   1/1     Running   0          5m  
pod-2-name   1/1     Running   0          5m  
```



### Get Pods in a Namespace

By default, `kubectl get` displays results from your application's default namespace. To display Pod information for a specific namespace, add the `--namespace` flag followed by the namespace itself:

```shell
kubectl get pods --namespace <namespace>
```

To display results from all namespaces, add the `--all-namespaces` flag:

```shell
kubectl get pods --all-namespaces
```

## Review Logs for a Pod

Normally, your application outputs print statements or error messages directly to the console (`stdout`). In Kubernetes, the Pod's container captures and logs these messages. This helps with debugging, since you can review specific containers to identify which one is raising an error.

To display a container's logs, use the `kubectl logs` command. If your Pod only has one container, simply add the Pod's name:

```shell
kubectl logs <pod-name>
```
If the Pod has multiple containers, add the `-c` flag followed by container's name:

```shell
kubectl logs <pod-name> -c <container-name>
```
**Example Output**:

```
2025-02-20T08:32:04.123Z | INFO | Synchronizing data with remote server
2025-02-20T08:32:05.456Z | INFO | Data synchronization in progress
2025-02-20T08:32:06.789Z | ERROR | Failed to connect to remote server: Timeout error
2025-02-20T08:32:07.012Z | INFO | Data synchronization completed successfully
```

To display logs for every container in a Pod, set the `--all-containers` flag to `true`:

```shell
kubectl logs <pod-name> --all-containers=true
```

## Debug Pod Containers

When you identify an issue within a container, you can use the `kubectl exec` command to explore its environment and debug it from the inside.

If the Pod has only one container, simply specify the Pod name and the command you want to issue:

```shell
kubectl exec <pod-name> -- <command>
```

If the Pod has multiple containers, `kubectl exec` issues your command on the first container in the Pod by default.

To specify a container to debug, add the `-c` flag followed by the name of the container:

```shell
kubectl exec <pod-name> -c <container-name> -- <command>
```


### Open an Interactive Shell

If you need to debug a container extensively, you can open an interactive shell inside the container:

```shell
kubectl exec <pod-name> -c <container-name> -- bash
```

After you issue this command:

- Your terminal sends input to the `bash` shell within the container.
- The `bash` shell displays output in your terminal.

This allows you to debug the container without any extra syntax. Simply enter a command:

```shell
<command>
```

## Debug a Crashing Pod

The `kubectl exec` command only works with functional containers. To debug a container that is crashing, you can use `kubectl debug` to create a clone of its Pod:

```shell
kubectl debug <pod-name> -it --image=busybox --copy-to=<clone-name>
```

**Example Output:**

```
Defaulting debug container name to debugger-abcde.
If you don't see a command prompt, try pressing enter.
root@<clone-name>:/#
```

This clone is mostly identical to the original Pod, but is optimized for debugging. It contains a single `busybox` container that will not crash, even if it encounters an error.

The `-it` flag  opens a terminal within the `busybox` container. You can use `kubectl exec` commands in this terminal to debug the clone. Then, make any necessary changes to the actual Pod.

When you finish debugging, make sure to delete the clone so it does not consume cluster resources:

```shell
kubectl delete pod <pod-name> <clone-name>
```



# Resources

- [List of Kubernetes commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-)

- [What is Kubernetes](https://kubernetes.io/docs/concepts/overview/)


