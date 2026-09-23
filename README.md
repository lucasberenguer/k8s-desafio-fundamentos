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

## 💾 Níveis 2 e 3: Banco de Dados, Persistência e Configuração

### Objetivo da Camada de Dados
Implantar a base de dados PostgreSQL (`postgres:16`) garantindo a persistência física dos dados, comunicação interna estável via Service (`ClusterIP`) e o desacoplamento seguro das credenciais e parâmetros operacionais através de `Secret` e `ConfigMap`.

---

### Manifestos e Responsabilidades

* **`k8s/01-pvc.yaml` (Armazenamento Persistente):** Aloca 1Gi de espaço em disco via `PersistentVolumeClaim` (modo `ReadWriteOnce`), garantindo que o diretório de dados do banco sobreviva à recriação de Pods.
* **`k8s/02-secret.yaml` (Credenciais Sensíveis):** Armazena o usuário, senha do banco e a string de conexão (`PGRST_DB_URI`), evitando senhas fixas (*hardcoded*) nos manifests.
* **`k8s/03-configmap.yaml` (Parâmetros Operacionais):** Define configurações não sensíveis, como a porta padrão de rede e roles do schema.
* **`k8s/04-postgres.yaml` (Carga de Trabalho e Rede):** 
  * `Deployment`: executa a imagem oficial `postgres:16`, consome as credenciais e configurações via `valueFrom` (`secretKeyRef` e `configMapKeyRef`), e mapeia o PVC no caminho `/var/lib/postgresql/data`.
  * `Service`: do tipo `ClusterIP` com o nome `postgres-service`, expondo a porta `5432/TCP` para resolução de DNS interno dentro do cluster.

---

### Aplicação e Validação dos Recursos

```bash
# 1. Aplicar volumes, segredos e configurações
kubectl apply -f k8s/01-pvc.yaml
kubectl apply -f k8s/02-secret.yaml
kubectl apply -f k8s/03-configmap.yaml

# 2. Subir o Deployment e Service do PostgreSQL
kubectl apply -f k8s/04-postgres.yaml

# 3. Conferência do estado dos recursos
kubectl get pvc -n desafio-k8s
kubectl get secret,configmap -n desafio-k8s
kubectl get pods,svc -l app=postgres -n desafio-k8s

```

# Reflexão Nível 2:

emptyDir: O volume é criado diretamente no nó onde o Pod é instanciado e tem seu ciclo de vida estritamente acoplado ao Pod. Se o Pod for deletado, travar ou for recriado em outro nó, todo o conteúdo do diretório é destruído de forma irrecuperável.   

PersistentVolumeClaim (PVC): O volume é um recurso independente que abstrai o armazenamento persistente[cite: 5]. Caso o Pod seja deletado ou sofra um crash, os arquivos do banco de dados continuam gravados no disco; assim que o Kubernetes recria o Pod pelo Deployment, o volume é reanexado ao caminho /var/lib/postgresql/data, mantendo tabelas e registros íntegros[cite: 5].

# Reflexão Nível 3:

Trata-se apenas de codificação em Base64, e não de criptografia. Qualquer pessoa ou processo com permissão de leitura (get secret) no cluster consegue recuperar a senha original instantaneamente usando echo <valor> | base64 -d.

Significado prático para a segurança: O objeto Secret padrão serve para evitar senhas expostas em texto plano no Git ou em telas de terminal durante revisões. Para segurança real em ambientes produtivos, é obrigatório aplicar políticas rígidas de RBAC, habilitar a encriptação em repouso (Encryption at Rest) no etcd e integrar o cluster com cofres de chaves externos (ex.: AWS Secrets Manager, HashiCorp Vault ou External Secrets Operator).
