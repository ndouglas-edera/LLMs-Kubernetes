# LLMs-Kubernetes
Simple repository for benchmarking open-source LLM models on Kubernetes

```
kubectl apply -f https://raw.githubusercontent.com/ndouglas-edera/LLMs-Kubernetes/refs/heads/main/deployment.yaml
kubectl get all -n llm
```

It could take a minute or two for the LLM model to installed within the running pod, check the pod logs to track the progress
```
kubectl logs -f -n llm deployment/llm-ollama-deployment
```
Alternatively, see if the pull process is actually active:
```
ps aux | grep ollama
```
You should see your image locally:
```
docker images | awk '
  /REPOSITORY/ { print; next }
  /ollama\/ollama|ghcr.io\/open-webui\/open-webui/ { print "\033[31m" $0 "\033[0m"; next }
  { print }
'
```
List images for pods:
```
kubectl get pods -A -o=custom-columns='POD_NAME:.metadata.name,CONTAINER_IMAGES:.spec.containers[*].image'
```
It's also worth checking-out the file size of those newly-introduced images:
```
docker images ollama/ollama --format "{{.Size}}" | sed 's/.*/\x1b[31m&\x1b[0m/'
docker images ghcr.io/open-webui/open-webui --format "{{.Size}}" | sed 's/.*/\x1b[31m&\x1b[0m/'
```
You'll still need to port-forward both service to interact with them: <br/>
Make sure to do this in separate terminal tabs to avoid breaking connections.
```
kubectl port-forward svc/llm-ollama-service -n llm 8080:8080
kubectl port-forward svc/open-webui-service -n llm 3000:8080
```
Check labels associated with pods:
```
kubectl get pods -n llm --show-labels
```
Confirm the images associated with your pods:
```
kubectl get pods -n llm -o 'custom-columns=NAME:.metadata.name,READY:.status.containerStatuses[*].ready,STATUS:.status.phase,RESTARTS:.status.containerStatuses[*].restartCount,AGE:.metadata.creationTimestamp,IMAGE:.spec.containers[*].image'
```
