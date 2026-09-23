# k8s-desafio-fundamentos

Repositório com os manifests declarativos e documentação do desafio prático de orquestração de contêineres com Kubernetes.

## 🛠 Ferramenta Utilizada
* **Docker Desktop** (com Kubernetes local / kind integrado)
* **kubectl** para gerenciamento de recursos

---

## 🚀 Nível 1: Namespace e Primeiro Contato

### 1. Criação do Namespace
Para isolar todos os recursos da aplicação, criação de um namespace dedicado:

```bash
kubectl apply -f k8s/00-namespace.yaml

#Subi um pod temporário para exercutar comandos básicos de inspeção e ciclo de vida:

# Execução do pod temporário
kubectl run pod-teste --image=busybox -n desafio-k8s -- sleep 3600

# Inspeção e verificação
kubectl get pods -n desafio-k8s
kubectl describe pod pod-teste -n desafio-k8s
kubectl logs pod-teste -n desafio-k8s

# Limpeza
kubectl delete pod pod-teste -n desafio-k8s

```

# Reflexão:

Não. Um Pod avulso criado diretamente pelo comando kubectl run não possui nenhum controlador de réplica monitorando o seu estado desejado. Uma vez deletado, seu ciclo de vida é encerrado definitivamente. Por essa razão, em ambientes de produção raramente criamos Pods avulsos, recorrendo a Deployments para garantir auto-recuperação e alta disponibilidade.


