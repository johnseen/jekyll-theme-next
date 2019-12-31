---
type: technology
title: Kubernets安装部署confluence
date: 2019-12-30
category: K8S
description: k8s-confluence
---

# Kubernets安装部署confluence


## 安装部署MySQL

1. helm安装mysql  `helm install -n mysql-release -f ~/.helm/cache/archive/mysql/values.yaml`
2.  mysql 配置文件如下:

```
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.28"

strategy:
  type: Recreate

busybox:
  image: "busybox"
  tag: "1.29.3"

testFramework:
  enabled: true
  image: "dduportal/bats"
  tag: "0.4.0"

## Specify password for root user
##
## Default: random 10 character string
mysqlRootPassword: scmadmin

## Create a database user
##
mysqlUser: confluence
## Default: random 10 character string
mysqlPassword: scmadmin

## Allow unauthenticated access, uncomment to enable
##
# mysqlAllowEmptyPassword: true

## Create a database
##
# mysqlDatabase:

## Specify an imagePullPolicy (Required)
## It's recommended to change this to 'Always' if the image tag is 'latest'
## ref: http://kubernetes.io/docs/user-guide/images/#updating-images
##
imagePullPolicy: IfNotPresent

## Additionnal arguments that are passed to the MySQL container.
## For example use --default-authentication-plugin=mysql_native_password if older clients need to
## connect to a MySQL 8 instance.
args: []

extraVolumes: |
  # - name: extras
  #   emptyDir: {}

extraVolumeMounts: |
  # - name: extras
  #   mountPath: /usr/share/extras
  #   readOnly: true

extraInitContainers: |
  # - name: do-something
  #   image: busybox
  #   command: ['do', 'something']

# Optionally specify an array of imagePullSecrets.
# Secrets must be manually created in the namespace.
# ref: https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod
# imagePullSecrets:
  # - name: myRegistryKeySecretName

## Node selector
## ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector
nodeSelector: {}

## Tolerations for pod assignment
## Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
##
tolerations: []

livenessProbe:
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 3

## Persist data to a persistent volume
persistence:
  enabled: true
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 8Gi
  annotations: {}

## Use an alternate scheduler, e.g. "stork".
## ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
##
# schedulerName:

## Security context
securityContext:
  enabled: false
  runAsUser: 999
  fsGroup: 999

## Configure resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  requests:
    memory: 256Mi
    cpu: 100m

# Custom mysql configuration files path
configurationFilesPath: /etc/mysql/conf.d/

# Custom mysql configuration files used to override default mysql settings
configurationFiles: 
  mysql.cnf: |-
    [mysqld]
    character-set-server=utf8
    transaction-isolation=READ-COMMITTED
    collation-server=utf8_bin
    default-storage-engine=INNODB
    max_allowed_packet=40M
    innodb_log_file_size=256M

    
# Custom mysql init SQL files used to initialize the database
initializationFiles: {}
#  first-db.sql: |-
#    CREATE DATABASE IF NOT EXISTS first DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
#  second-db.sql: |-
#    CREATE DATABASE IF NOT EXISTS second DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;

metrics:
  enabled: false
  image: prom/mysqld-exporter
  imageTag: v0.10.0
  imagePullPolicy: IfNotPresent
  resources: {}
  annotations: {}
    # prometheus.io/scrape: "true"
    # prometheus.io/port: "9104"
  livenessProbe:
    initialDelaySeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    initialDelaySeconds: 5
    timeoutSeconds: 1
  flags: []
  serviceMonitor:
    enabled: false
    additionalLabels: {}

## Configure the service
## ref: http://kubernetes.io/docs/user-guide/services/
service:
  annotations: {}
  ## Specify a service type
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types
  type: ClusterIP
  port: 3306
  # nodePort: 32000
  # loadBalancerIP:

## Pods Service Account
## ref: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
serviceAccount:
  ## Specifies whether a ServiceAccount should be created
  ##
  create: false
  ## The name of the ServiceAccount to use.
  ## If not set and create is true, a name is generated using the mariadb.fullname template
  # name:

ssl:
  enabled: false
  secret: mysql-ssl-certs
  certificates:
#  - name: mysql-ssl-certs
#    ca: |-
#      -----BEGIN CERTIFICATE-----
#      ...
#      -----END CERTIFICATE-----
#    cert: |-
#      -----BEGIN CERTIFICATE-----
#      ...
#      -----END CERTIFICATE-----
#    key: |-
#      -----BEGIN RSA PRIVATE KEY-----
#      ...
#      -----END RSA PRIVATE KEY-----

## Populates the 'TZ' system timezone environment variable
## ref: https://dev.mysql.com/doc/refman/5.7/en/time-zone-support.html
##
## Default: nil (mysql will use image's default timezone, normally UTC)
## Example: 'Australia/Sydney'
# timezone:

# Deployment Annotations
deploymentAnnotations: {}

# To be added to the database server pod(s)
podAnnotations: {}
podLabels: {}

## Set pod priorityClassName
# priorityClassName: {}


## Init container resources defaults
initContainer:
  resources:
    requests:
      memory: 10Mi
      cpu: 10m

```
    

## 创建Confluence镜像

1. docker run -v /data/:/var/atlassian/application-data/confluence --name="confluence" -d -p 8090:8090 -p 8091:8091 atlassian/confluence-server
2. docker cp mysql-connector-java-5.1.47.jar confluence:/opt/atlassian/confluence/lib
3. docker tag 50bd4772f66b  registry.apps.clue.kube.com/confluence:v1
4. docker push registry.apps.clu.kube.com/confluence:v1

## 集群中部署Confluence

### 创建k8s拉取私有仓库镜像的秘钥

   ``` kubectl create secret docker-registry regsecret --docker-server=registry.apps.clu.kube.com --docker-username=admin --docker-password=admin123 ```

### 编写confluence安装yaml文件

```
apiVersion: v1
kind: Service
metadata:
  name: confluence-service
spec:
  ports:
  - name: http
    port: 8090
    targetPort: 8090
  - name: sync
    port: 8091
    targetPort: 8091
  selector:
    app: confluence-deployment
  type: NodePort
---

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: confluence-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: confluence-deployment
    matchExpressions:
      - {key: tier, operator: In, values: [confluence-deployment]}
  template:
    metadata:
      labels:
        app: confluence-deployment
        tier: confluence-deployment
    spec:
      containers:
      - name: confluence-deployment
        image: registry.apps.clu.kube.com/confluence:v1
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8090
          name: http
          protocol: TCP
        - containerPort: 8091
          name: http2
          protocol: TCP
      imagePullSecrets:
      - name: regsecret

```
## 使用Traefik为confluence创建Ingress
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-nginx-traefik
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: nginx.apps.clu.kube.com
    http:
      paths:
      - path: /  
        backend:
          serviceName: nginx
          servicePort: 80
  - host: confluence.apps.clu.kube.com
    http:
      paths:
      - path: /  
        backend:
          serviceName: confluence-service
          servicePort: 8090
      - path: /synchrony
        backend:
          serviceName: confluence-service
          servicePort: 8091
```

## 使用kubetail监控pod的实时日志

1. 下载kubetail ``
2. 将kubetail移动到`/usr/bin`下 `kubetail /usr/bin `
3. 使用kubetail查看pod日志 `kubetail conflunence-deployment-*`
4. 具体使用方法,请参考[kubetail](https://github.com/johanhaleby/kubetail)项目地址

## 参考资料
1. [k8s pod详解](https://www.cnblogs.com/kevingrace/p/11309409.html)
2. [confluence官方Docker镜像](https://hub.docker.com/r/atlassian/confluence-server/)
3. [kubetail项目](https://github.com/johanhaleby/kubetail)
