[![Build Status](https://dev.azure.com/zupc88/project08/_apis/build/status/project08-CI?branchName=master)](https://dev.azure.com/zupc88/project08/_build/latest?definitionId=2&branchName=master)

# monolith
참고:  
Order 와 product 는 N:1 (다대일) 관계이다.  


-- 주문 하기  
http localhost:8088/orders productId=1 quantity=3 customerId="1@uengine.org" customerName="홍길동" customerAddr="서울시"

-- 주문 후 변경된 상품 수량 확인  
http http://localhost:8088/orders/1/product  

-- 주문 후 delivery 내역중 order  확인  
http http://localhost:8088/deliveries  
http http://localhost:8088/orders/1/delivery  

-- 배송 완료하기  
http PATCH localhost:8088/deliveries/1 deliveryState=DeliveryCompleted


-- 주문 취소 하기
http PATCH localhost:8088/orders/1 state=OrderCancelled





yaml 배포 준비
secret 추가
auto healing
auto scaling
deployment


******************************************************************
                           Configmap
******************************************************************


apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-dev
  namespace: default
data:
  DB_URL: skcc-bookstore-dev
  JAVA_OPTS: -client
---  
apiVersion: v1
kind: ConfigMap
metadata:
  name: spring-prod
  namespace: default
data:
  DB_URL: skcc-bookstore-prod
  JAVA_OPTS: -server










webwas@DESKTOP-JQ6ILBP:~$ k exec nginx-b448fb54d-h64dj env
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx-b448fb54d-h64dj
EMBED_TOMCAT_JAVA_OPTS=-client
DB_URL=skcc-bookstore-dev






******************************************************************
                           Deployment(기본 참조 자료)
******************************************************************
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${IMAGENAME}
  labels:
    app: ${IMAGENAME}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${IMAGENAME}
  template:
    metadata:
      labels:
        app: ${IMAGENAME}
    spec:
      containers:
        - name: ${IMAGENAME}
          image: ${ACR}/${IMAGENAME}:latest
          ports:
            - containerPort: 8080
          env:
            - name: VUE_APP_API_HOST
              value: http://${_GATEWAY_IP}:8080
              
              
              
******************************************************************
                           Deployment
******************************************************************
https://www.oops4u.com/2378
https://kubernetes.io/ko/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/






apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            limits:
              cpu: 500m
            requests:
              cpu: 200m
          livenessProbe:
            httpGet:
              path: /
              port: 80
              scheme: HTTP
            initialDelaySeconds: 5
            periodSeconds: 15
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /
              port: 80
              scheme: HTTP
            initialDelaySeconds: 5
            timeoutSeconds: 1
          env:
          - name: EMBED_TOMCAT_JAVA_OPTS
            valueFrom:
               configMapKeyRef:
                  name: spring-dev
                  key: JAVA_OPTS
          - name: DB_URL
            valueFrom:
               configMapKeyRef:
                  name: spring-dev
                  key: DB_URL
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  ports:
  - port: 80
  selector:
    app: nginx
---
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  maxReplicas: 4 # define max replica count
  minReplicas: 2  # define min replica count
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  targetCPUUtilizationPercentage: 50 # target CPU utilization
    
    
    
    
    
    
    
******************************************************************
                           Ingress
******************************************************************


apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
spec:
  rules:
  - host: nginx.skcc.co.kr
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx-svc
          servicePort: 80
