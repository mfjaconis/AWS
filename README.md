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

## 7. AWS Workflows com Step Functions

### Objetivo do laboratório
Este laboratório tem como objetivo consolidar seus **workflows automatizados com AWS Step Functions**. O entregável é um repositório organizado contendo anotações e insights adquiridos durante a prática, servindo como material de apoio para os seus estudos e futuras implementações.

### O que é o Step Functions
O **AWS Step Functions** orquestra etapas de um processo em um **workflow** (máquina de estados). Em vez de encadear serviços manualmente no código, você define o fluxo (ordem, condições, retries e erros) e a AWS executa cada passo.

- Cada execução do workflow tem um **estado** (estado atual da máquina).
- Os passos podem chamar serviços como **Lambda**, **SNS**, **SQS**, **DynamoDB**, **ECS/Fargate**, entre outros.
- O desenho do fluxo fica visual no console (Workflow Studio) e também em definição **ASL** (Amazon States Language — JSON).

### Conceitos principais
| Conceito | Descrição |
|----------|-----------|
| **State Machine** | A definição do workflow (os estados e as transições) |
| **State** | Uma etapa do fluxo (Task, Choice, Wait, Parallel, Map, Succeed, Fail, etc.) |
| **Task** | Estado que executa um trabalho (ex.: invocar uma Lambda) |
| **Choice** | Ramificação condicional (if/else do workflow) |
| **Wait** | Pausa por tempo ou até uma data |
| **Parallel / Map** | Execução em paralelo ou em loop sobre uma lista |
| **Execution** | Uma corrida concreta do workflow (com input/output e histórico) |
| **Retry / Catch** | Tratamento de falhas: tentar de novo ou seguir para um caminho de erro |

### Tipos de workflow
| Tipo | Quando usar |
|------|-------------|
| **Standard** | Processos longos, auditoria completa, até 1 ano de duração |
| **Express** | Alta vazão, curta duração, custo por execução/volume (event-driven) |

### Fluxo típico de prática
1. Defina o problema em etapas (ex.: validar → processar → notificar).
2. Crie (ou reutilize) funções **Lambda** / integrações para cada tarefa.
3. Monte a **state machine** no Step Functions (console ou ASL).
4. Configure permissões IAM (role da state machine + permissões das tasks).
5. Execute com um **input JSON** de teste e acompanhe o histórico da execução.
6. Ajuste retries, timeouts e caminhos de erro (`Catch` / `Fail`).
7. Documente no repositório o que funcionou, decisões e insights.

### Boas práticas
- Mantenha cada estado com **uma responsabilidade** clara.
- Prefira **integrações otimizadas** (SDK integrations) quando possível, em vez de Lambda só para “colar” APIs.
- Sempre trate falhas com **Retry** (erros transitórios) e **Catch** (erros esperados).
- Versionamento: trate a definição do workflow como código (IaC / repositório).
- Use nomes e comentários que facilitem leitura futura no material de estudos.

### Exemplo mental de workflow
`Receber pedido` → `Validar estoque (Choice)` → se ok: `Cobrar` → `Enviar notificação` → `Succeed`; se falha: `Registrar erro` → `Fail` / notificar suporte.

---

## 8. AWS CloudFormation

O **CloudFormation** é o serviço nativo de **Infrastructure as Code (IaC)** da AWS: você descreve a infraestrutura em um template (**YAML** ou **JSON**) e a AWS cria, atualiza ou remove os recursos de forma ordenada e repetível.

### Para que serve
- **Automatizar** a criação de recursos (EC2, VPC, S3, IAM, RDS, etc.) sem clicar manualmente no console.
- **Padronizar ambientes** (dev, staging, prod) com o mesmo template e parâmetros diferentes.
- **Versionar infraestrutura** no Git, como código de aplicação.
- **Reproduzir** stacks de forma idêntica em outra conta ou região.
- **Reduzir erros** humanos e acelerar deploys e rollbacks.
- **Documentar** a infraestrutura de forma declarativa (o template é a documentação).

**Casos de uso comuns:** subir um servidor web (EC2 + Security Group), criar buckets S3 com políticas, montar uma VPC completa, provisionar bancos RDS, pipelines CI/CD e ambientes de laboratório descartáveis.

### Como funciona
1. Você escreve um **template** com recursos (`AWS::EC2::Instance`, `AWS::EC2::SecurityGroup`, etc.).
2. Cria um **stack** no console, CLI ou SDK a partir desse template.
3. O CloudFormation **provisiona** os recursos na ordem das dependências.
4. Em um **update**, ele calcula o diff e aplica só o necessário.
5. Em um **delete**, remove (na maioria dos casos) os recursos gerenciados pelo stack.

### Conceitos principais
| Conceito | Descrição |
|----------|-----------|
| **Template** | Arquivo YAML/JSON que descreve a infraestrutura |
| **Stack** | Conjunto de recursos criados a partir de um template |
| **Resources** | Seção obrigatória com os recursos AWS |
| **Parameters** | Valores de entrada (ex.: AMI, tipo de instância) |
| **Outputs** | Valores de saída (ex.: IP público da EC2) |
| **Mappings / Conditions** | Mapas e condições lógicas no template |
| **Change set** | Pré-visualização do que um update vai alterar |

### Estrutura mínima de um template
- `AWSTemplateFormatVersion` (opcional, mas comum)
- `Description` (opcional)
- `Parameters` (opcional)
- `Resources` (**obrigatório**)
- `Outputs` (opcional)

### Exemplo: EC2 + Apache + firewall (Security Group)

O que o stack cria:
- **Security Group** (firewall): libera SSH (22) e HTTP (80)
- **EC2** com `UserData` que instala e inicia o **Apache**
- **Output** com o IP público / URL HTTP

> Ajuste `ImageId` (AMI) e `KeyName` para a região e o key pair da sua conta. Exemplo abaixo usa AMI Amazon Linux 2023 em `us-east-1` — valide o ID atual na sua região.

#### Versão YAML

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: EC2 com Apache e Security Group (HTTP + SSH)

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Nome do key pair para SSH
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Libera SSH e HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: ami-0c02fb55956c7d316  # troque pela AMI da sua região
      SecurityGroupIds:
        - !Ref WebServerSecurityGroup
      UserData:
        Fn::Base64: |
          #!/bin/bash
          dnf update -y
          dnf install -y httpd
          systemctl enable httpd
          systemctl start httpd
          echo "<h1>Apache via CloudFormation</h1>" > /var/www/html/index.html

Outputs:
  WebsiteURL:
    Description: URL HTTP da instancia
    Value: !Sub "http://${WebServerInstance.PublicIp}"
  PublicIP:
    Description: IP publico da EC2
    Value: !GetAtt WebServerInstance.PublicIp
```

#### Versão JSON (mesmo stack)

```json
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "EC2 com Apache e Security Group (HTTP + SSH)",
  "Parameters": {
    "KeyName": {
      "Type": "AWS::EC2::KeyPair::KeyName",
      "Description": "Nome do key pair para SSH"
    },
    "InstanceType": {
      "Type": "String",
      "Default": "t2.micro",
      "AllowedValues": ["t2.micro", "t3.micro"]
    }
  },
  "Resources": {
    "WebServerSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription": "Libera SSH e HTTP",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIp": "0.0.0.0/0"
          },
          {
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIp": "0.0.0.0/0"
          }
        ]
      }
    },
    "WebServerInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "InstanceType": { "Ref": "InstanceType" },
        "KeyName": { "Ref": "KeyName" },
        "ImageId": "ami-0c02fb55956c7d316",
        "SecurityGroupIds": [{ "Ref": "WebServerSecurityGroup" }],
        "UserData": {
          "Fn::Base64": {
            "Fn::Join": [
              "\n",
              [
                "#!/bin/bash",
                "dnf update -y",
                "dnf install -y httpd",
                "systemctl enable httpd",
                "systemctl start httpd",
                "echo \"<h1>Apache via CloudFormation</h1>\" > /var/www/html/index.html"
              ]
            ]
          }
        }
      }
    }
  },
  "Outputs": {
    "WebsiteURL": {
      "Description": "URL HTTP da instancia",
      "Value": {
        "Fn::Sub": "http://${WebServerInstance.PublicIp}"
      }
    },
    "PublicIP": {
      "Description": "IP publico da EC2",
      "Value": {
        "Fn::GetAtt": ["WebServerInstance", "PublicIp"]
      }
    }
  }
}
```

### Como subir o stack (console)
1. Console → **CloudFormation** → **Create stack** → **With new resources**.
2. **Upload a template file** (YAML ou JSON) ou informe um URL no S3.
3. Nomeie o stack (ex.: `ec2-apache-lab`).
4. Preencha **KeyName** (e opcionalmente `InstanceType`).
5. Avance, reconheça que o stack pode criar recursos IAM (se houver) e clique em **Create stack**.
6. Aguarde `CREATE_COMPLETE` e abra a aba **Outputs** para pegar o IP/URL.
7. No browser: `http://<PublicIP>` — deve aparecer a página do Apache.

### CLI (opcional)
```powershell
aws cloudformation create-stack `
  --stack-name ec2-apache-lab `
  --template-body file://ec2-apache.yaml `
  --parameters ParameterKey=KeyName,ParameterValue=meu-keypair
```

### YAML vs JSON
| | YAML | JSON |
|--|------|------|
| Legibilidade | Mais fácil de ler/escrever | Mais verboso |
| Comentários | Sim (`#`) | Não |
| Intrinsic functions | Atalhos (`!Ref`, `!GetAtt`, `!Sub`) | Forma longa (`Ref`, `Fn::GetAtt`) |
| Uso comum | Preferido em labs e produção | Útil quando gerado por ferramentas |

### Boas práticas
- Prefira **YAML** para templates escritos à mão.
- Não deixe SSH (`22`) aberto em `0.0.0.0/0` em produção — restrinja ao seu IP.
- Parametrize AMI, tipo de instância e CIDR.
- Use **Outputs** para IPs, IDs e URLs.
- Delete o stack ao terminar o lab para evitar cobrança (`Delete stack`).
- Valide o template: `aws cloudformation validate-template --template-body file://template.yaml`

### O que cada parte faz neste lab
| Recurso | Papel |
|---------|--------|
| `AWS::EC2::SecurityGroup` | Firewall: regras de entrada (ingress) |
| `AWS::EC2::Instance` | Máquina EC2 |
| `UserData` | Script na primeira inicialização (instala Apache) |
| `Parameters` | Entradas reutilizáveis (key pair, tipo) |
| `Outputs` | Expõe IP/URL após o create |

### CloudFormation vs Terraform

Ambos são ferramentas de **IaC** (Infrastructure as Code), mas com abordagens diferentes.

| Aspecto | CloudFormation | Terraform |
|---------|----------------|-----------|
| **Origem** | Serviço nativo da AWS | Open source (HashiCorp) |
| **Escopo** | Somente AWS | Multi-cloud (AWS, Azure, GCP, etc.) |
| **Linguagem** | YAML ou JSON (template CFN) | HCL (HashiCorp Configuration Language) |
| **Motor de execução** | Gerenciado pela AWS (sem instalar nada) | CLI local/CI (`terraform plan/apply`) |
| **Estado** | AWS mantém o estado do stack | Arquivo de estado local/remoto (ex.: S3 + DynamoDB) |
| **Integração AWS** | Suporte imediato a novos recursos AWS | Depende de providers da comunidade/HashiCorp |
| **Curva de aprendizado** | Mais simples se você usa só AWS | Mais flexível, mas exige aprender HCL e conceitos de state |
| **Custo** | Gratuito (paga só pelos recursos criados) | Gratuito (open source); Terraform Cloud tem planos pagos |
| **Rollback** | Automático em falhas de create/update | Depende da configuração; `terraform apply` reverte parcialmente |
| **Drift detection** | Stack drift detection no console | `terraform plan` compara estado desejado vs real |
| **Módulos** | Nested stacks, StackSets, módulos públicos (Registry) | Módulos reutilizáveis no Terraform Registry |

#### Quando usar CloudFormation
- Ambiente **100% AWS** e você quer a solução nativa.
- Times que preferem **zero instalação** (só console/CLI AWS).
- Integração direta com serviços AWS (SAM, CDK geram templates CFN).
- Contas corporativas que já padronizaram em CFN/StackSets.

#### Quando usar Terraform
- Infraestrutura **multi-cloud** ou híbrida.
- Equipe já usa HCL e ecossistema HashiCorp (Vault, Consul, etc.).
- Precisa de providers para serviços fora da AWS (Cloudflare, Datadog, GitHub, etc.).
- Quer um fluxo unificado de IaC para vários provedores.

#### Exemplo equivalente em Terraform (mesmo lab EC2 + Apache)

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

variable "key_name" {
  type = string
}

resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Libera SSH e HTTP"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web" {
  ami           = "ami-0c02fb55956c7d316" # troque pela AMI da sua região
  instance_type = "t2.micro"
  key_name      = var.key_name

  vpc_security_group_ids = [aws_security_group.web.id]

  user_data = <<-EOF
              #!/bin/bash
              dnf update -y
              dnf install -y httpd
              systemctl enable httpd
              systemctl start httpd
              echo "<h1>Apache via Terraform</h1>" > /var/www/html/index.html
              EOF
}

output "website_url" {
  value = "http://${aws_instance.web.public_ip}"
}
```

**Resumo:** CloudFormation é a ferramenta **nativa e integrada** da AWS; Terraform é **agnóstico de cloud** e mais flexível em ambientes heterogêneos. Para estudos focados em AWS, começar com CloudFormation ajuda a entender stacks, templates e recursos AWS; Terraform vale a pena quando o cenário envolve múltiplos provedores ou o time já adotou o ecossistema HashiCorp.

---

## Resumo rápido

1. Crie a conta → proteja o root com MFA.
2. Configure IAM (grupos + usuários + políticas).
3. Configure Budgets para não ser surpreendido pela fatura.
4. Use **EC2** para computação, **EBS** como disco da EC2, **Elastic IP** para IP público fixo e **S3** para armazenar objetos.
5. Use **Step Functions** para orquestrar workflows automatizados entre serviços AWS.
6. Use **CloudFormation** para provisionar infraestrutura como código na AWS (templates YAML/JSON → stacks); use **Terraform** quando precisar de IaC multi-cloud.