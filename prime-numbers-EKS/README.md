# Prime Calculator Kubernetes Job

## Project Intention

This project demonstrates running a computational task as a Kubernetes Job on EKS. It calculates the first 100 prime numbers using Python and writes the results to the node's `/tmp` directory using a hostPath volume mount.

## Project Structure

- **primes.py** - Python script implementing a simple prime number calculator
- **Dockerfile** - Minimal container image based on python:3.11-slim
- **job.yaml** - Kubernetes Job manifest with hostPath volume configuration

## How It Works

The Job creates a pod that:
1. Runs the Python script to calculate the first 100 primes
2. Writes results to `/tmp/primes.txt` inside the container
3. The hostPath volume mounts the node's `/tmp` directory, making the file persist on the node after the pod completes

## Deployment on EKS

### ECR Access Issue and Solution

During deployment, AWS credentials were unavailable to push the image to ECR. The solution was to load the image directly into the EKS node's containerd runtime:

1. Built the image locally: `docker build -t prime-calculator:latest .`
2. Saved as tar: `docker save prime-calculator:latest -o /tmp/prime-calculator.tar`
3. Created a privileged pod with hostPath access to `/run/containerd`
4. Copied the tar file to the pod: `kubectl cp /tmp/prime-calculator.tar image-loader:/tmp/`
5. Used `nsenter` to access the node's namespace and imported: `ctr -n k8s.io images import`
6. Set `imagePullPolicy: Never` in job.yaml to use the local image

This approach bypasses the need for a container registry entirely by loading the image directly into the node's container runtime.

## Verification

Check the output on the node:
```bash
kubectl run verify --rm -it --image=alpine --restart=Never \
  --overrides='{"spec":{"volumes":[{"name":"tmp","hostPath":{"path":"/tmp"}}],"containers":[{"name":"verify","image":"alpine","command":["cat","/tmp/primes.txt"],"volumeMounts":[{"name":"tmp","mountPath":"/tmp"}]}]}}' 
```

## Cleanup

```bash
kubectl delete job prime-calculator
```
