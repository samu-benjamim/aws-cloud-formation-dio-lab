# 🏗️ Implementando minha Primeira Stack com AWS CloudFormation

> Repositório criado como entregável do desafio de laboratório da [DIO](https://www.dio.me/), com foco em consolidar conhecimentos sobre **Infrastructure as Code (IaC)** utilizando **AWS CloudFormation**.

---

## 📌 Sobre o Desafio

Este laboratório tem como objetivo a implementação prática de uma **Stack** utilizando o AWS CloudFormation, serviço nativo da AWS para provisionamento de infraestrutura através de código. O foco está em entender como declarar, versionar e gerenciar recursos de nuvem de forma automatizada e reproduzível.

---

## 🎯 Objetivos de Aprendizagem

- [x] Aplicar conceitos de Infrastructure as Code (IaC) em ambiente prático
- [x] Criar e gerenciar Stacks com AWS CloudFormation
- [x] Documentar processos técnicos de forma clara e estruturada
- [x] Utilizar o GitHub como ferramenta de compartilhamento de documentação técnica

---

## 📚 O que é AWS CloudFormation?

O **AWS CloudFormation** é o serviço de **Infrastructure as Code (IaC)** nativo da AWS. Em vez de criar recursos manualmente pelo console, você descreve toda a infraestrutura em um arquivo de template (YAML ou JSON) e o CloudFormation provisiona tudo automaticamente.

### Analogia prática

Pense no CloudFormation como uma **receita de bolo**: você escreve os ingredientes e o modo de preparo uma vez, e pode replicar o mesmo bolo (infraestrutura) quantas vezes quiser, em qualquer ambiente, sempre com o mesmo resultado.

### Por que usar CloudFormation?

| Sem CloudFormation | Com CloudFormation |
|---|---|
| Cliques manuais no console | Infraestrutura declarada em código |
| Difícil de reproduzir | Mesmo template = mesma infraestrutura |
| Sem histórico de mudanças | Versionável no Git |
| Risco de erro humano | Provisionamento automatizado e consistente |
| Rollback manual e complexo | Rollback automático em caso de falha |

---

## 🧩 Conceitos Fundamentais

### Template

Arquivo YAML ou JSON que descreve os recursos AWS que serão criados. É o "código" da sua infraestrutura.

**Estrutura básica de um template:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Minha primeira Stack com CloudFormation

Parameters:
  # Valores de entrada que podem ser customizados
  NomeAmbiente:
    Type: String
    Default: dev

Resources:
  # Recursos que serão criados (obrigatório)
  MinhaInstanciaEC2:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c55b159cbfafe1f0

Outputs:
  # Valores exportados após a criação
  EnderecoPublico:
    Value: !GetAtt MinhaInstanciaEC2.PublicDnsName
```

---

### Stack

Uma **Stack** é o conjunto de recursos AWS criados a partir de um template. Quando você executa um template, o CloudFormation cria uma Stack que agrupa e gerencia todos os recursos como uma unidade.

```
Template (código)  →  CloudFormation  →  Stack (recursos reais na AWS)
     📄                    ⚙️                  ☁️ EC2 + VPC + S3...
```

> Deletar uma Stack deleta **todos** os recursos criados por ela — muito útil para ambientes temporários de estudo.

---

### Seções do Template

| Seção | Obrigatória | Função |
|---|---|---|
| `AWSTemplateFormatVersion` | Não | Versão do formato (sempre `2010-09-09`) |
| `Description` | Não | Descrição legível do template |
| `Parameters` | Não | Valores de entrada customizáveis |
| `Mappings` | Não | Tabela de valores estáticos (ex: AMI por região) |
| `Conditions` | Não | Criação condicional de recursos |
| `Resources` | ✅ **Sim** | Recursos AWS a serem criados |
| `Outputs` | Não | Valores exportados após criação |

---

## 🛠️ Exemplo Prático — Stack com EC2 e Security Group

Template completo criando uma instância EC2 com Security Group configurado:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Stack de estudo - EC2 com Security Group

Parameters:
  TipoInstancia:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
    Description: Tipo da instancia EC2

Resources:

  MeuSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Permite acesso SSH e HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref TipoInstancia
      ImageId: ami-0c55b159cbfafe1f0
      SecurityGroups:
        - !Ref MeuSecurityGroup
      Tags:
        - Key: Name
          Value: instancia-cloudformation-lab

Outputs:
  IDInstancia:
    Description: ID da instancia criada
    Value: !Ref MinhaInstancia
  IPPublico:
    Description: IP publico da instancia
    Value: !GetAtt MinhaInstancia.PublicIp
```

---

## 🔧 Funções Intrínsecas Essenciais

O CloudFormation oferece funções nativas para tornar os templates dinâmicos:

| Função | Sintaxe curta | O que faz | Exemplo |
|---|---|---|---|
| `Ref` | `!Ref` | Referencia o valor de um parâmetro ou ID de recurso | `!Ref TipoInstancia` |
| `GetAtt` | `!GetAtt` | Obtém um atributo de um recurso criado | `!GetAtt Instancia.PublicIp` |
| `Sub` | `!Sub` | Substitui variáveis em uma string | `!Sub "Nome-${Ambiente}"` |
| `Join` | `!Join` | Concatena valores com um separador | `!Join ["-", [app, prod]]` |
| `Select` | `!Select` | Seleciona um item de uma lista | `!Select [0, !AZs ""]` |
| `If` | `!If` | Condicional baseado em Conditions | `!If [IsProd, m5.large, t2.micro]` |

---

## 🔄 Ciclo de Vida de uma Stack

```
            ┌──────────────────┐
            │  Template (.yaml)│
            └────────┬─────────┘
                     │ aws cloudformation create-stack
                     ▼
            ┌──────────────────┐
            │  CREATE_IN_PROGRESS │
            └────────┬─────────┘
           SUCESSO ──┤── FALHA
              │             │
              ▼             ▼
    ┌──────────────┐  ┌──────────────────┐
    │ CREATE_COMPLETE│  │ ROLLBACK_COMPLETE │
    └──────┬───────┘  └──────────────────┘
           │
     (atualizar template)
           │ aws cloudformation update-stack
           ▼
    ┌──────────────────────┐
    │  UPDATE_IN_PROGRESS  │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │  UPDATE_COMPLETE     │
    └──────────┬───────────┘
               │
     (remover tudo)
               │ aws cloudformation delete-stack
               ▼
    ┌──────────────────────┐
    │  DELETE_COMPLETE     │
    └──────────────────────┘
```

---

## ☁️ Criando a Stack pelo Console AWS

### Passo a passo

1. Acesse o **AWS CloudFormation** no console
2. Clique em **"Create Stack" → "With new resources"**
3. Em **Template source**, escolha:
   - Upload a template file (arquivo local)
   - Amazon S3 URL (template hospedado no S3)
4. Configure os **Parâmetros** (se o template tiver)
5. Defina **Tags** para organização e controle de custos
6. Configure **permissões IAM** se necessário
7. Revise e clique em **"Create Stack"**
8. Acompanhe o progresso na aba **"Events"**

---

## 🖥️ Criando a Stack pela AWS CLI

```bash
# Criar uma Stack
aws cloudformation create-stack \
  --stack-name minha-primeira-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=TipoInstancia,ParameterValue=t2.micro

# Verificar status
aws cloudformation describe-stacks \
  --stack-name minha-primeira-stack

# Atualizar a Stack
aws cloudformation update-stack \
  --stack-name minha-primeira-stack \
  --template-body file://template-atualizado.yaml

# Deletar a Stack (remove todos os recursos)
aws cloudformation delete-stack \
  --stack-name minha-primeira-stack
```

---

## 🔒 Boas Práticas

### ✅ O que fazer
- Sempre usar **YAML** (mais legível que JSON para templates)
- Usar **Parameters** para tornar templates reutilizáveis entre ambientes
- Adicionar **Tags** em todos os recursos (facilita controle de custos)
- Versionar templates no **Git** — infraestrutura como código precisa de histórico
- Usar **Outputs** para exportar valores importantes entre Stacks
- Habilitar **termination protection** em Stacks de produção
- Usar **Change Sets** para visualizar o impacto de uma atualização antes de aplicar

### ❌ O que evitar
- Colocar credenciais ou secrets diretamente no template — use **AWS Secrets Manager** ou **SSM Parameter Store**
- Criar recursos fora do CloudFormation em uma Stack já gerenciada (causa drift)
- Deletar recursos manualmente que fazem parte de uma Stack ativa
- Usar templates sem `Description` — dificulta manutenção futura

---

## 🆚 CloudFormation vs Outras Ferramentas de IaC

| | CloudFormation | Terraform | CDK |
|---|---|---|---|
| **Provedor** | AWS (nativo) | HashiCorp (multi-cloud) | AWS (nativo) |
| **Linguagem** | YAML / JSON | HCL | Python, TypeScript, Java... |
| **Multi-cloud** | ❌ Apenas AWS | ✅ Sim | ❌ Apenas AWS |
| **Curva de aprendizado** | Média | Média | Alta |
| **State management** | AWS gerencia | Arquivo `.tfstate` | AWS gerencia |
| **Melhor para** | Quem usa só AWS | Times multi-cloud | Devs que preferem código real |

---

## 🗂️ Estrutura do Repositório

```
📁 repo-cloudformation-dio/
├── 📄 README.md                        ← Este arquivo
├── 📁 templates/                       ← Templates CloudFormation
│   ├── ec2-basico.yaml                 ← Template do lab
│   └── ec2-com-security-group.yaml     ← Template com SG configurado
└── 📁 images/                          ← Capturas de tela
    ├── 01-upload-template.png
    ├── 02-stack-criando.png
    ├── 03-stack-completa.png
    └── 04-recursos-criados.png
```

---

## 🔗 Recursos Utilizados

- [AWS CloudFormation — Documentação Oficial](https://docs.aws.amazon.com/pt_br/AWSCloudFormation/latest/UserGuide/gettingstarted.walkthrough.html)
- [Referência de Tipos de Recursos AWS](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [Funções Intrínsecas do CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)
- [GitHub Quick Start — DIO](https://github.com/digitalinnovationone/github-quickstart)
- [GitBook: Formação GitHub Certification](https://aline-antunes.gitbook.io/formacao-fundamentos-github)

---

## 👨‍💻 Autor

Feito com 💙 durante os estudos na [DIO](https://www.dio.me/) — Plataforma de Educação em Tecnologia.

---

> *"Infraestrutura que não está no código não existe — existe apenas até alguém deletar."*
