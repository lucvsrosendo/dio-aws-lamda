# Portfólio de Laboratórios AWS - DIO

Este repositório consolida as documentações, templates e códigos desenvolvidos durante os desafios práticos de AWS na [Digital Innovation One (DIO)](https://www.dio.me/). O objetivo é demonstrar a aplicação de conceitos de arquitetura em nuvem, desde a orquestração de microsserviços até o provisionamento automatizado de infraestrutura e arquiteturas orientadas a eventos.

---

## 📌 Sumário
1. [Orquestração Serverless com AWS Step Functions](#1-orquestração-serverless-com-aws-step-functions)
2. [Implementando a Primeira Stack com CloudFormation](#2-implementando-a-primeira-stack-com-cloudformation)
3. [Automação Avançada com CloudFormation](#3-automação-avançada-com-cloudformation)
4. [Tarefas Automatizadas com AWS Lambda e Amazon S3](#4-tarefas-automatizadas-com-aws-lambda-e-amazon-s3)
5. [Estrutura do Repositório](#-estrutura-do-repositório)

---

## 1. Orquestração Serverless com AWS Step Functions

O AWS Step Functions é utilizado para orquestrar múltiplos serviços da AWS (como Lambda, SQS e DynamoDB) em fluxos de trabalho automatizados e auditáveis, desacoplando a lógica de negócios da execução das funções individuais.

### Conceitos e Estrutura ASL
Durante a prática, foram explorados os estados fundamentais da Amazon States Language (ASL): Task, Choice, Parallel e o tratamento nativo de erros com Retry e Catch.

Exemplo de Fluxo (JSON):

    {
      "Comment": "Workflow com Tratamento de Erros Serverless",
      "StartAt": "ProcessarDados",
      "States": {
        "ProcessarDados": {
          "Type": "Task",
          "Resource": "arn:aws:lambda:us-east-1:123456789012:function:MinhaLambda",
          "Retry": [{ "ErrorEquals": ["States.ALL"], "IntervalSeconds": 2, "MaxAttempts": 3 }],
          "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "Falha" }],
          "Next": "Sucesso"
        },
        "Sucesso": { "Type": "Succeed" },
        "Falha": { "Type": "Fail", "Cause": "Erro no processamento da Lambda." }
      }
    }

---

## 2. Implementando a Primeira Stack com CloudFormation

O AWS CloudFormation permite provisionar recursos de infraestrutura de forma declarativa (Infraestrutura como Código - IaC) usando templates em YAML ou JSON, garantindo rastreabilidade e versionamento da infraestrutura.

### Conceitos Abordados
* Parameters: Injeção de variáveis customizadas no momento do deploy.
* Resources: Declaração dos recursos físicos da nuvem (EC2, S3, Security Groups).
* Outputs: Exportação de dados, como IPs públicos e ARNs, para uso futuro.
* Ciclo de Vida: Processos de criação, atualização segura (Change Sets) e exclusão limpa da Stack sem deixar recursos órfãos.

---

## 3. Automação Avançada com CloudFormation

Dando um passo além da criação básica, este laboratório focou na automação real de ambientes, garantindo que servidores sejam iniciados, configurados e atualizados sem nenhuma intervenção humana.

### Recursos Avançados
* User Data: Scripts Bash injetados no provisionamento da EC2 para instalar pacotes (ex: servidor Apache) na inicialização do sistema operacional.
* Funções Intrínsecas: Uso extensivo de !Sub para substituição de variáveis e !Base64 para codificar os scripts de inicialização.

Exemplo de Automação de Servidor (YAML):

    Resources:
      ServidorWeb:
        Type: 'AWS::EC2::Instance'
        Properties:
          InstanceType: t2.micro
          ImageId: ami-0c55b159cbfafe1f0
          UserData:
            Fn::Base64: !Sub |
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "<h1>Servidor Automatizado via CloudFormation</h1>" > /var/www/html/index.html

---

## 4. Tarefas Automatizadas com AWS Lambda e Amazon S3

Este projeto explora arquiteturas orientadas a eventos (Event-Driven Architecture), utilizando o Amazon S3 para engatilhar funções AWS Lambda, além de explorar o S3 Object Lambda para transformar dados em tempo real.

### Arquitetura S3 Object Lambda
Permite adicionar código personalizado (Lambda) às requisições de leitura (GET) do S3. É ideal para casos de uso como redimensionar imagens on-the-fly ou mascarar dados sensíveis (PII) sem precisar duplicar os arquivos no storage.

Template CloudFormation para S3 Object Lambda (YAML):

    Resources:
      LambdaProcessamento:
        Type: 'AWS::Lambda::Function'
        Properties:
          Handler: 'index.handler'
          Runtime: 'nodejs18.x'
          Code:
            ZipFile: |
              exports.handler = async (event) => { 
                  // Lógica de transformação/mascaramento de dados ao ler do S3
                  return { statusCode: 200 }; 
              };

      S3ObjectLambdaAccessPoint:
        Type: 'AWS::S3ObjectLambda::AccessPoint'
        Properties:
          ObjectLambdaConfiguration:
            TransformationConfigurations:
              - Actions: ['GetObject']
                ContentTransformation:
                  AwsLambda:
                    FunctionArn: !GetAtt LambdaProcessamento.Arn

---

## 📁 Estrutura do Repositório

Sugestão de organização dos arquivos para o seu GitHub:

    ├── step-functions/
    │   └── state-machine.json       # Definição do Workflow ASL
    ├── cloudformation-basico/
    │   └── primeira-stack.yaml      # Template IaC simples (Bucket S3 e EC2)
    ├── cloudformation-automacao/
    │   └── web-server-auto.yaml     # Template IaC com automação de UserData
    ├── lambda-s3/
    │   ├── index.js                 # Código-fonte da função AWS Lambda
    │   └── s3-object-lambda.yaml    # Template da infraestrutura S3 + Lambda
    ├── images/                      # Capturas de tela dos painéis da AWS
    └── README.md                    # Esta documentação unificada
