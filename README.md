<h1 align="center">
   <img alt="Desafio aceito" title="#challengeAccepted" src="challenge_accepted.jpg" width="300px" />
</h1>

<h4 align="center">
  📱 Desafios Kubernetes |
</h4>

<p align="center">
  <a href="#introdução">Introdução</a>&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;
  <a href="#desafios">Desafios</a>&nbsp;&nbsp;&nbsp;|&nbsp;&nbsp;&nbsp;
</p>

## Introdução

Nesse repositório você encontrará uma série de desafios kubernetes.

## Desafios

➡️ Desafios | 

1. Crie um pod chamado "my-pod" usando uma imagem simples como "nginx" e verifique seu estado com os comandos de monitoramento do Kubernetes. - [PrimeiroDesafio](#primeirodesafio)
</br>

2. Implante um Deployment chamado "my-deployment" com três réplicas de uma aplicação baseada na imagem "httpd". Atualize a imagem do Deployment para uma versão mais recente.
</br>

3. Crie um ConfigMap chamado "app-config" com uma variável de configuração personalizada. Monte o ConfigMap em um pod e verifique se o valor foi aplicado corretamente.
</br>

4. Crie um Secret chamado "app-secret" contendo informações sensíveis. Injete o Secret como uma variável de ambiente em um pod e teste se está acessível.
</br>

5. Configure um PersistentVolume de 1Gi de armazenamento local e vincule-o a um PersistentVolumeClaim. Monte o volume em um pod e salve arquivos para verificar a persistência.
</br>

6. Crie um serviço do tipo ClusterIP para um Deployment chamado "backend" e teste a conectividade interna entre pods usando o nome do serviço.
</br>

7. Implante um Job chamado "batch-job" que execute um comando simples e termine. Verifique os logs do Job para confirmar sua execução.
</br>

8. Crie um Horizontal Pod Autoscaler para um Deployment chamado "hpa-deployment" e configure-o para escalar com base no uso de CPU. Aumente a carga e observe o escalonamento.
</br>

9. Crie um serviço do tipo NodePort para expor externamente um Deployment chamado "webapp". Acesse o serviço usando o endereço IP do Minikube e a porta atribuída.
</br>

10. Crie um pod chamado "restart-pod" com a política de reinício configurada como "OnFailure". Provoque uma falha no pod e observe seu comportamento.

## PrimeiroDesafio 
<P>Crie um pod chamado "my-pod”</P>
</br>
Comando para criar o pod

```
kubectl run my-pod --image=nginx
```
Comandos de monitoramento

```
kubectl get pods
kubectl describe pod my-pod
kubectl logs my-pod
```
</br>

<h2>2°- Desafio :</h2>
<p></p>Implante um Deployment chamado "my-deployment”</p>
</br>
Criar com 3 réplicas da imagem httpd:

```
kubectl create deployment my-deployment --image=httpd --replicas=3
```
Atualizar a imagem:

```
kubectl set image deployment/my-deployment httpd=httpd:latest
```
Verifique a atualização:

```
kubectl rollout status deployment/my-deployment
```

</br>

<h2>3°- Desafio :</h2>
<p>ConfigMap “APP-CONFIG”</p>
</br>
Criar o ConfigMap usando a linha de comando:

```
kubectl create configmap app-config --from-literal=APP_ENV=production --from-literal=APP_PORT=8080
```

Verificar o ConfigMap criado

```
kubectl get configmap app-config -o yaml
```

Montar o ConfigMap em um Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: APP_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_PORT
```

Aplique o arquivo corrigido

```
kubectl apply -f configmap-pod.yaml
```

Verifique se o Pod foi criado com sucesso

```
kubectl get pods
```

Entre no pod e veja as variáveis de ambiente

```
kubectl exec -it configmap-pod -- env
```

</br>

<h2>4°- Desafio :</h2>
<p>Secret “App-Secret”</p>
</br>
Criar o Secret com dados sensíveis

```
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=mysecretpassword \
  --from-literal=API_KEY=1234567890abcdef
```

Verificar o Secret criado

```
kubectl get secrets
kubectl describe secret app-secret
```

Usar o Secret em um Pod.
Crie um arquivo YAML para o Pod

```
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: nginx-container
    image: nginx
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: API_KEY
```

Aplique o arquivo YAML

```
kubectl apply -f secret-pod.yaml
```

 Verificar se os valores foram injetados

```
kubectl exec -it secret-pod -- env
```

</br>

<h2>5°- Desafio :</h2>
PersistentVolume e PersistentVolumeClaim
</br>
Criar um PersistentVolume (PV)
Crie um arquivo chamado persistent-volume.yaml com o seguinte conteúdo:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/dat
```

Aplique o PV com o comando:

```
kubectl apply -f persistent-volume.yaml
```

Criar um PersistentVolumeClaim (PVC)
Crie um arquivo chamado persistent-volume-claim.yaml com o seguinte conteúdo

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

Aplique o PVC com o comando:

```
kubectl apply -f persistent-volume-claim.yaml
```

Verifique se o PV e PVC foram vinculados

```
kubectl get pv
kubectlet pvc
```

</br>

<h2>6°- Desafio :</h2>
Serviço do tipo ClusterIP
</br>
Criar um Deployment
Crie um arquivo chamado backend-deployment.yaml:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: nginx
        ports:
        - containerPort: 80
```

Aplique o Deployment:

```
kubectl apply -f backend-deployment.yaml
```

Criar o Serviço ClusterIP
Crie um arquivo chamado backend-service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Aplique o Serviço:

```
kubectl apply -f backend-service.yaml
```

Verificar o Serviço

```
kubectl get services
```

Testar a Conectividade Interna

```
kubectl run curl-pod --image=curlimages/curl -it --restart=Never -- sh
```

Teste o acesso ao Serviço
Agora, no terminal do Pod curl-pod, use o comando curl para testar o Serviço

```
curl backend-service
```

</br>

<h2>7°- Desafio :</h2>
Job no Kubernetes
</br>
Criar o Job
Criar arquivo YAML chamado batch-job.yaml

```
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      containers:
      - name: batch-container
        image: busybox
        command: ["sh", "-c", "echo 'Hello from the Kubernetes Job!' && sleep 30"]
      restartPolicy: Never
  backoffLimit: 4
```

Aplicar o Job

```
kubectl apply -f batch-job.yaml
```

Verificar o Job

```
kubectl get jobs
```

Verificar os logs do Job

```
kubectl get pods --selector=job-name=batch-job
```

Após obter o nome do Pod, use o comando kubectl logs para ver os logs:

```
kubectl logs <nome-do-pod>
```

Você deve ver a saída do comando executado, que no caso será:

```
Hello from the Kubernetes Job!
```

</br>

<h2>8°- Desafio :</h2>
Horizontal Pod Autoscaler (HPA)
</br>
Criar um Deployment
Crie um arquivo chamado hpa-deployment.yaml com o seguinte conteúdo:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa
  template:
    metadata:
      labels:
        app: hpa
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

Aplique o Deployment:

```
kubectl apply -f hpa-deployment.ya
```

Criar o Horizontal Pod Autoscaler (HPA)
Crie um arquivo chamado hpa.yaml

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-deployment
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50
```

Aplique o HPA:

```
kubectl apply -f hpa.yaml
```

Verificar o HPA

```
kuectl get hpa
```

Testar o Autoescalonamento

```
kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh
```

Dentro do Pod load-generator, execute o seguinte comando

```
while true; do wget -q -O- http://hpa-deployment; done
```

Verificar o Número de Réplicas

```
kubectl get deployments
```

</br>

<h2>9°- Desafio :</h2>
Serviço NodePort
</br>
Crie um Deployment nginx, usando o comando:

```
kubectl create deployment webapp --image=nginx
```

Crie o Serviço NodePort

```
kubectl expose deployment webapp --type=NodePort --name=webapp-service --port=80 --target-port=80
```

Crie o arquivo service-nodeport.yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30007  # Ou qualquer número de porta disponível (entre 30000 e 32767)
  type: NodePort
```

Aplique o arquivo:

```
kubectl apply -f service-nodeport.yaml
```

Verifique o Serviço

```
kubectl get sv
```

Acessar o Serviço Externo

```
minikube service webapp-service
```

</br>

<h2>10°- Desafio :</h2>
Criar o Pod com Política de Reinício "OnFailure”
</br>
Crie um arquivo chamado pod-restart-onfailure.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: restart-pod
spec:
  restartPolicy: OnFailure
  containers:
  - name: nginx-container
    image: nginx
    command: ["sh", "-c", "echo 'Hello, world!' && exit 1"]  # Comando para forçar falha
```

Aplicar o YAML

```
kubectl apply -f pod-restart-onfailure.yaml
```

Verificar o Status do Pod

```
kubectl get pods
```

Verificar os Logs

```
kubectl logs restart-po
```

Você verá algo como

```
Hello, world!
```

Verificar a Reinicialização
Depois de alguns segundos ou minutos, o Pod será reiniciado automaticamente. Você pode verificar o número de reinicializações com:

```
kubectl describe pod restart-pod
```


</br>
</br>


<h2>Referências :</h2>


- [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [Secrets](https://kubernetes.io/pt-br/docs/concepts/configuration/secret/)
- [Persistent Volume](https://kubernetes.io/pt-br/docs/concepts/storage/persistent-volumes/)
- [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/)
- [Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Documentação oficial do Kubernetes](https://kubernetes.io/docs/home/) 
