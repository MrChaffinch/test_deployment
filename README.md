# test_deployment
test_deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
 name: test_deploy
 labels:
   app: test_deploy
spec:
  selector:
    matchLabels:
      app: test_deploy
 replicas: 1
 template:
   metadata:
     labels:
       app: test_deploy
   spec:
     containers:
     - name: test_deploy
       image: some_image:version
       ports:
       - containerPort: 80
       livenessProbe:
         httpGet:
           path: /healthy
           port: 80
         periodSeconds: 10
       readinessProbe:
         httpGet:
           path: /healthy
           port: 80
         periodSeconds: 10
       startupProbe:
         httpGet:
           path: /healthy
           port: 80
         periodSeconds: 10
       resources:
         limits:
           cpu: "1"
           memory: 128m
         requests:
           cpu: 100m
           memory: 128m
     affinity:
       podAntiAffinity:
         requiredDuringSchedulingIgnoredDuringExecution:
           - labelSelector:
               matchExpressions:
                 - key: "app"
                   operator: In
                   values:
                     - test_deploy
             topologyKey: "kubernetes.io/hostname"
```
# Про трафик разговора не было, но добавил readinessProbe
# Указываем startup, т.к. приложенька долго стартует
# Affinity указываем, чтобы 2 пода не сидели на одной ноде 

```
apiVersion: v1
kind: Service
metadata:
 name: test_deploy
 labels:
   app: test_deploy
spec:
 ports:
 - port: 80
 selector:
   app: test_deploy
```
# Создаём сервис для доступа к приложеньке
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: test_deploy
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: test_deploy
  minReplicas: 1
  maxReplicas: 4
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 120
      policies:
        - type: Pods
          value: 4
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 240
      policies:
        - type: Pods
          value: 1
          periodSeconds: 120
    metrics:
      - type: Resource
        resource:
          name: cpu
          target:
            type: Utilization
            averageUtilization: 80
```
# Задаём политики для HorizontalPodAutoscaler, чтобы поды скейлились автоматически
# Соответственно задаём максимальное значение реплик это maxReplicas: 4
# Указываем, чтобы поды удалялись, когда нет нагрузки scaleDown
# В метрикс указываем, когда скейлить поды, значение averageUtilization по cpu, т.к. по раме у нас всё статично
