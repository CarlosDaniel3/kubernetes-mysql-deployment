# Kubernetes-mysql-Deployment

## Objetivo do desafio
Provisionar o deployment do MySQL em um cluster Kubernetes por meio do StatefulSet, 
configurando um persistent storage para os dados e expondo o serviço para acesso interno no cluster.

## Arquivos criados
Secret
```yaml
---
apiVersion: v1
kind: Secret
metadata: 
    name: mysql-password-secret
type: Opaque
data:
   ROOT_PASSWORD: cGFzc3dvcmQxMjM0NTY=
```

StatefulSet, PV e PVC
```yaml
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-sts
spec:
  selector:
    matchLabels:
      app: mysql
  serviceName: "mysql-service"
  replicas: 3
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql-container
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom: 
              secretKeyRef: 
                key: ROOT_PASSWORD
                name: mysql-password-secret
          - name: MYSQL_DATABASE
            value: "mysql_database"
          - name: MYSQL_USER
            value: "user"
          - name: MYSQL_PASSWORD
            value: "password"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: 
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
```

Service
```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  labels:
    app: mysql
spec:
  ports:
  - port: 3306
    name: database
  clusterIP: None
  selector:
    app: mysql
```

## Comandos Úteis
Comando para criar o cluster por meio do kind
```
kind create cluster --config=kind-config.yaml
```
Comando para aplicar os manifestos
```
kubectl apply -f ./manifests
```

## Validação do funcionamento do MySQL
Os seguintes comandos podem ser utilizados para validar o funcionamento do MySQL

Primeiro, listar os pods MySQL que foram criados
```
kubectl get po -l app=mysql
```
Depois, acessar um dos pods
```
kubectl exec -it <nome-do-pod> -- bash
```
Conectar-se ao MySQL
```
mysql -u user -p
```
Verificar se o banco de dados foi criado
```
SHOW DATABASES;
```
Por fim, executar alguns comandos básicos para finalizar os testes
```
USE mysql_database;
CREATE TABLE test_table (id INT PRIMARY KEY, name VARCHAR(50));
SHOW TABLES;
```

## Resultado esperado da validação
<div style="text-align: center"><br>
    <img align="center" alt="result" height="400px" width="500px" src="https://github.com/CarlosDaniel3/kubernetes-mysql-deployment/blob/main/assets/result.png">
</div>
