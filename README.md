##### K8s - Basics
    Four keys standards
        - apiVersion
        - kind
        - metadata
        - spec


### Create PODs

        apiVersion: v1
        kind: Pod
        metadata:
        name: nginx-prod
        labels:
            name: nginx-prod
            env: production
        spec:
        containers:
        - name: nginx-prod
            image: nginx
            resources:
            limits:
                memory: "240Mi"
                cpu: "500m"
            ports:
            - containerPort: 80

    Vinoth@Vinoths-MacBook-Pro httpd-deployment % kubectl get pods
    NAME                              READY   STATUS    RESTARTS   AGE
    http-deployment-6fbddf8d7-b5qcl   1/1     Running   0          6m23s
    http-deployment-6fbddf8d7-cldsq   1/1     Running   0          6m23s
    http-deployment-6fbddf8d7-nr97j   1/1     Running   0          6m23s
    http-deployment-6fbddf8d7-rhz8m   1/1     Running   0          6m23s
    http-deployment-6fbddf8d7-sfhvs   1/1     Running   0          6m23s

### Create Replicaset or ReplicationController 

    ReplicationController - doesn't have ability to automatically update existing replicas (like scale up and down) when they are already running
    ReplicaSet - has the ability to modifiy existing replication counts and scales up and download. 

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
        name: nginx-prod-replica
    spec:
        replicas: 3
        selector:
        matchLabels:
            env: production
        template:
        metadata:
            name: nginx-prod
            labels:
            env: production
        spec:
            containers:
            - name: myapp
                image: nginx
                resources:
                limits:
                    memory: "125mi"
                    cpu: "250m"
                ports:
                - containerPort: 80

    vinoth@Vinoths-MacBook-Pro httpd-deployment % kubectl get replicaset
    NAME                        DESIRED   CURRENT   READY   AGE
    http-deployment-6fbddf8d7   5         5         5       7m13s


### Create Deployment:
    To handle deployment strategies RollingUpdate, Recreate 

    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name:  http-deployment
    labels:
        tier:  http-deployment
    spec:
    replicas: 5
    template:
        metadata:
        labels:
            name:  httpd-frontend
        spec:
        containers:
            - image:  httpd:2.4
            name:  httpd-frontend
            resources:
                limits:
                cpu: "200m"
                memory: "50Mi"
            ports:
                - containerPort: 80
                protocol: TCP    
            env:
                - name:  env
                value:  production
            volumeMounts:
                - name: html
                mountPath: /usr/local/apache2/htdocs/
        volumes:
            - name: html
            configMap:
                name: httpd-config
    selector:
        matchLabels:
        name: httpd-frontend
    minReadySeconds: 5
    strategy:
        type: RollingUpdate
        rollingUpdate:
        #maxSurge: 1
        maxUnavailable: 1


    vinoth@Vinoths-MacBook-Pro httpd-deployment % kubectl get deployment
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    http-deployment   5/5     5            5           9m49s

    vinoth@Vinoths-MacBook-Pro httpd-deployment % kubectl describe deployment
    Name:                   http-deployment
    Namespace:              default
    CreationTimestamp:      Sat, 05 Nov 2022 18:19:35 +0530
    Labels:                 tier=http-deployment
    Annotations:            deployment.kubernetes.io/revision: 1
    Selector:               name=httpd-frontend
    Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        5
    RollingUpdateStrategy:  1 max unavailable, 25% max surge
    Pod Template:
    Labels:  name=httpd-frontend
    Containers:
    httpd-frontend:
        Image:      httpd:2.4
        Port:       80/TCP
        Host Port:  0/TCP
        Limits:
        cpu:     200m
        memory:  50Mi
        Environment:
        env:  production
        Mounts:
        /usr/local/apache2/htdocs/ from html (rw)
    Volumes:
    html:
        Type:      ConfigMap (a volume populated by a ConfigMap)
        Name:      httpd-config
        Optional:  false
    Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   http-deployment-6fbddf8d7 (5/5 replicas created)
    Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set http-deployment-6fbddf8d7 to 5

#### Create service for networking and Establish port to access outside node/pods

    apiVersion: v1
    kind: Service
    metadata:
    name:  http-service
    spec:
    type: NodePort
    ports:
    - port:  80
        targetPort:  80
        protocol: TCP
        nodePort: 30036
    selector:
        name: httpd-frontend

