# Generated-YAML

+ 建立 Deployment 物件
+ 選擇建立 service 物件
+ 透過nginx 將服務部屬起來
+ index.html 為主要的檔案, 使用javascript 編寫
+ constraints-LimitRanges 為每個NS 都打上預設的範圍防止有人沒有打Limit



## 建立在k8s 上

### 建立configmap

```
kubectl create configmap html --from-file=index.html \
                              --from-file=bootstrap.min.css \
                              --from-file=bootstrap.min.js \
                              --from-file=jquery-3.1.1.min.js \
                              --from-file=js-yaml.min.js \
                              --from-file=vue.js
```



### 佈署服務

```
## 部屬
kubectl apply -f deployment-service.yaml

## Ingress
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: generated-yaml-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: "yaml.35.201.159.50.nip.io"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          serviceName: generated-yaml-service
          servicePort: 80
EOF
```



## 為每個ns 都打上constraints-LimitRanges

```
$ns=xxxx

kubectl -n $ns  apply -f constraints-LimitRanges/constraints-LimitRanges.yaml
```



```
# constraints-LimitRanges/constraints-LimitRanges.yaml 內容

apiVersion: v1
kind: LimitRange
metadata:
  name: mem-min-max-demo-lr
spec:
  limits:
  - max:
      memory: 1Gi
      cpu: 1000m
    min:
      memory: 100Mi
      cpu: 100m
    type: Container
```

