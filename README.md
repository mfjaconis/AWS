# AWS — Guia básico

## 1. Criação de conta na AWS

1. Acesse [https://aws.amazon.com](https://aws.amazon.com) e clique em **Criar uma conta da AWS**.
2. Informe e-mail, senha e nome da conta.
3. Escolha o tipo de conta (**Pessoal** ou **Empresarial**).
4. Preencha dados de contato e aceite o contrato.
5. Cadastre um cartão de crédito (a AWS usa para verificação; o Free Tier cobre muitos recursos no início).
6. Confirme o telefone (SMS ou chamada).
7. Escolha um plano de suporte (o **Basic** é gratuito).
8. Após o login, você entra no **root user** — use-o só para configuração inicial; no dia a dia, use usuários IAM.

### Boas práticas iniciais
- Ative MFA no usuário root.
- Crie um usuário administrador IAM e evite usar o root.
- Defina alertas de billing/budget o quanto antes.

---

## 2. Configuração de IAM

O **IAM (Identity and Access Management)** controla quem pode acessar a conta e o que cada identidade pode fazer.

### Conceitos principais
| Conceito | Descrição |
|----------|-----------|
| **Usuário** | Identidade de uma pessoa ou aplicação |
| **Grupo** | Conjunto de usuários que compartilham as mesmas permissões |
| **Política (Policy)** | Documento JSON que define permissões (`Allow` / `Deny`) |
| **Role** | Identidade assumida temporariamente (ex.: por uma EC2) |
| **MFA** | Autenticação em dois fatores |

### Passos recomendados
1. No Console AWS, abra **IAM**.
2. Em **Account settings**, ative MFA no root (se ainda não fez).
3. Crie políticas ou use políticas gerenciadas pela AWS (ex.: `AdministratorAccess`, `AmazonS3ReadOnlyAccess`).
4. Prefira o princípio do **menor privilégio**: conceda só o necessário.

---

## 3. Criação de um grupo IAM

1. Console AWS → **IAM** → **User groups** → **Create group**.
2. Defina um nome (ex.: `Developers`, `Admins`, `BillingViewers`).
3. Em **Attach permissions policies**, selecione as políticas do grupo.
   - Exemplo Admin: `AdministratorAccess`
   - Exemplo leitura S3: `AmazonS3ReadOnlyAccess`
4. Confirme em **Create group**.

Todos os usuários do grupo herdam as políticas anexadas ao grupo.

---

## 4. Adicionar um usuário ao grupo

### Criar o usuário
1. **IAM** → **Users** → **Create user**.
2. Informe o nome do usuário.
3. Marque **Provide user access to the AWS Management Console** (se for acesso pelo console) e/ou gere **Access key** (se for CLI/SDK).
4. Em **Set permissions**, escolha **Add user to group** e selecione o grupo criado.
5. Revise e conclua. Guarde a senha/access keys em local seguro.

### Adicionar um usuário já existente
1. **IAM** → **Users** → selecione o usuário.
2. Aba **Groups** → **Add user to groups**.
3. Marque o grupo → **Add to groups**.

---

    ## 5. Configuração de alerta de budget (orçamento)

O **AWS Budgets** avisa quando o custo ou o uso se aproxima de um limite.

### Passo a passo
1. Console → **Billing and Cost Management** → **Budgets**.
2. Clique em **Create budget**.
3. Escolha **Use a template** (mais simples) ou **Customize**.
4. Tipo comum: **Cost budget** (orçamento de custo).
5. Defina:
   - Nome do budget (ex.: `alerta-mensal-50`)
   - Período: **Monthly**
   - Valor do orçamento (ex.: US$ 50)
6. Em **Budget alerts**, configure limiares, por exemplo:
   - 50% do valor previsto
   - 80% do valor previsto
   - 100% do valor real/previsto
7. Informe o e-mail que receberá os alertas.
8. Revise e crie o budget.

### Dica
- Ative também **Billing preferences** → receber alertas de fatura por e-mail.
- Combine Budgets + **Cost Explorer** para entender onde o dinheiro está sendo gasto.

---

## 6. Como funcionam EC2, EBS, Elastic IP e S3

### EC2 (Elastic Compute Cloud)
Serviço de **máquinas virtuais** na nuvem.

- Você escolhe AMI (sistema operacional/imagem), tipo de instância (CPU/RAM), rede (VPC/subnet) e armazenamento.
- A instância roda enquanto estiver **running**; você paga pelo tempo ligada (e por outros recursos associados).
- Estados comuns: `pending` → `running` → `stopping`/`stopped` → `terminated`.
- Acesso típico: SSH (Linux) ou RDP (Windows), com key pair / Security Groups.
- Security Group = firewall virtual (portas e IPs permitidos).

**Uso típico:** servidores web, APIs, aplicações, bastion hosts, workloads que precisam de SO completo.

---

### EBS (Elastic Block Store)
**Disco de bloco** conectado a uma instância EC2 (como um HD/SSD).

- Volumes EBS são **persistentes**: sobrevivem a stop/start da instância (diferente do storage efêmero instance store).
- Tipos comuns: `gp3`/`gp2` (geral), `io2` (IOPS altas), `st1`/`sc1` (throughput).
- Pode criar **snapshots** (backup pontual) e restaurar em novos volumes.
- Em geral, um volume EBS fica em uma **Availability Zone** e se anexa a uma EC2 na mesma AZ.
- Você paga pelo tamanho provisionado e, em alguns tipos, por IOPS/throughput.

**Uso típico:** disco do sistema operacional, banco de dados, arquivos da aplicação na EC2.

---

### Elastic IP
Endereço **IPv4 público fixo** que você pode associar a recursos na conta (em geral, a uma EC2).

- IP público padrão da EC2 muda se a instância for parada e iniciada de novo.
- Elastic IP permanece o mesmo ao reiniciar/associar a outra instância.
- Pode **associar** e **desassociar** entre instâncias.
- Atenção: Elastic IP **alocado e não associado** (ou associado a instância parada, conforme regras atuais) pode gerar cobrança — evite deixar IPs ociosos.

**Uso típico:** servidores que precisam de IP estável (DNS apontando para o IP, allowlists de clientes, etc.).

---

### S3 (Simple Storage Service)
Armazenamento de **objetos** (arquivos) altamente durável e escalável.

- Estrutura: **Bucket** (container) → **Objects** (arquivos) com chave (caminho/nome).
- Não é um disco montável como EBS; é acesso via API/HTTP (upload/download).
- Classes de armazenamento: Standard, Intelligent-Tiering, Standard-IA, Glacier, etc. (custo × frequência de acesso).
- Controles: políticas de bucket, IAM, ACLs (evitar), versionamento, criptografia, lifecycle rules.
- Casos de uso: backups, sites estáticos, data lakes, mídia, logs, artefatos de CI/CD.

**Diferença rápida vs EBS:**
| | EBS | S3 |
|--|-----|----|
| Tipo | Bloco (disco) | Objeto (arquivo) |
| Ligado a | Instância EC2 | Independente |
| Acesso | Sistema de arquivos na VM | API / URL |
| Ideal para | SO, DB na EC2 | Arquivos, backups, dados compartilhados |

---

## Resumo rápido

1. Crie a conta → proteja o root com MFA.
2. Configure IAM (grupos + usuários + políticas).
3. Configure Budgets para não ser surpreendido pela fatura.
4. Use **EC2** para computação, **EBS** como disco da EC2, **Elastic IP** para IP público fixo e **S3** para armazenar objetos.