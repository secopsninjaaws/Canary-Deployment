# 📌 Canary Deployment Manual no Kubernetes

Este repositório demonstra um **Canary Deployment manual** no Kubernetes usando dois deployments distintos (V1 e V2) para rotear o tráfego de forma gradual entre as versões do serviço.

## 📂 Estrutura do Repositório

- `namescape.yaml` → Criação do namespace `nginx-deployment`
- `depolymenV1.yaml` → Primeira versão (`V1`) do deployment usando a imagem `nginx`
- `deploymentV2.yaml` → Segunda versão (`V2`) do deployment usando a imagem `httpd`
- `Serv-Ingress.yaml` → Configuração do **Service** e do **Ingress** para rotear o tráfego

## 🏗️ Passo a Passo da Implantação

### 1️⃣ Criar o Namespace  
Execute o comando abaixo para criar o namespace onde os recursos serão implantados:
```sh
kubectl apply -f namescape.yaml
```

### 2️⃣ Criar a Primeira Versão do Deployment (`V1`)
O primeiro deployment (`V1`) utiliza a imagem `nginx` e terá 7 réplicas:
```sh
kubectl apply -f depolymenV1.yaml
```

### 3️⃣ Criar o Service e o Ingress
O Service expõe os pods dentro do cluster, e o Ingress configura o roteamento externo:
```sh
kubectl apply -f Serv-Ingress.yaml
```

### 4️⃣ Criar a Segunda Versão do Deployment (`V2`)
O deployment `V2` utiliza a imagem `httpd` e possui apenas 3 réplicas. Isso permite um controle manual da porcentagem do tráfego que recebe a nova versão:
```sh
kubectl apply -f deploymentV2.yaml
```

## 🎯 Como funciona o Canary Deployment Manual?
Este setup permite uma estratégia **canary** manual, onde:
1. O tráfego será balanceado entre os pods `nginx` e `httpd` com base no número de réplicas.
2. Inicialmente, `nginx` tem 7 pods e `httpd` tem 3 pods, então ~70% do tráfego ainda será servido pelo `nginx`.
3. Para aumentar o tráfego para `httpd`, aumente as réplicas de `V2` e reduza as de `V1` gradativamente:
   ```sh
   kubectl scale deployment nginx-deploymentv2 --replicas=5
   kubectl scale deployment nginx-deploymentv1 --replicas=5
   ```

## 🚀 Testando o Deployment
Para verificar os pods em execução:
```sh
kubectl get pods -n nginx-deployment
```
Para inspecionar o tráfego sendo roteado:
```sh
kubectl get ingress -n nginx-deployment
```
Para acessar o serviço dentro do cluster:
```sh
kubectl port-forward service/nginx-service 8080:80 -n nginx-deployment
```
Agora, acesse `http://localhost:8080` e verifique qual versão está respondendo!

## 🛠️ Rollback (Reverter para Versão Anterior)
Caso precise reverter para `V1`, escale `V2` para 0 réplicas e aumente `V1`:
```sh
kubectl scale deployment nginx-deploymentv2 --replicas=0
kubectl scale deployment nginx-deploymentv1 --replicas=7
```

---

💡 **Dica:** Para um Canary Deployment automatizado, considere usar ferramentas como Argo Rollouts ou Flagger!
