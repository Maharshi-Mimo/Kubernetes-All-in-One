# Limit Range   

- CPU specific limit range

```
apiVersion: v1
kind: LimitRange
metadata:
    name: cpu-resource-limit
spec:
    limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```
- Memory specific limit range

```
apiVersion: v1
kind: LimitRange
metadata:
    name: memory-resource-limit
spec:
    limits:
    - default:
        memory: 500m
      defaultRequest:
        memory: 500m
      max:
        memory: "1"
      min:
        memory: 100m
      type: Container
```

# ResourceQuota 

- Resource Quota per namespace 

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ResourceQuota
spec:
  hard:
    requests.cpu: 2
    requests.memory: 1Gi
    limits.cpu: 4
    limits.memory: 2Gi
```