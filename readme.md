# Tarefas Automatizadas com AWS Lambda e Amazon S3

Este repositório documenta a prática do laboratório **Executando Tarefas Automatizadas com Lambda Function e S3** da [Digital Innovation One (DIO)](https://www.dio.me/). 

O objetivo deste projeto é explorar a arquitetura orientada a eventos (Event-Driven Architecture) na AWS, utilizando buckets do Amazon S3 para acionar funções Serverless no AWS Lambda. Essa prática é fundamental para consolidar conhecimentos em infraestrutura em nuvem, especialmente como base prática para exames de certificação AWS.

---

## 📌 Sumário
- [Visão Geral do Projeto](#-visão-geral-do-projeto)
- [Conceitos Chave](#-conceitos-chave)
- [Exemplo prático: Trigger de S3 para Lambda](#-exemplo-prático-trigger-de-s3-para-lambda)
- [S3 Object Lambda](#-s3-object-lambda)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Insights e Aprendizados](#-insights-e-aprendizados)

---

## 🚀 Visão Geral do Projeto

A integração nativa entre o Amazon S3 e o AWS Lambda permite que a execução de código seja acionada automaticamente sempre que um evento ocorrer em um bucket (como o upload, modificação ou exclusão de um arquivo). 

Neste laboratório, documentamos como criar essa ponte, automatizando o processamento de arquivos sem a necessidade de provisionar ou gerenciar servidores (Serverless).

---

## 🛠️ Conceitos Chave

* **Amazon S3 (Simple Storage Service):** Serviço de armazenamento de objetos altamente escalável. Atua como a origem dos eventos (Event Source).
* **AWS Lambda:** Serviço de computação Serverless que roda código em resposta a eventos. Pode ser escrito em Node.js, Python, Java, etc.
* **S3 Event Notifications:** Recurso que envia notificações (para SQS, SNS ou Lambda) quando eventos específicos ocorrem no bucket (ex: `s3:ObjectCreated:Put`).
* **IAM Roles (Políticas de Permissão):** A função Lambda precisa de uma *Execution Role* com permissões (`s3:GetObject` e `s3:PutObject`) para ler e gravar arquivos no S3 de forma segura.

---

## 💻 Exemplo prático: Trigger de S3 para Lambda

Abaixo está um exemplo de código em **Node.js** para a função Lambda. Sempre que um arquivo `.json` ou imagem for feito upload no bucket S3, a Lambda é acionada e lê os metadados do arquivo:

```javascript
import { S3Client, GetObjectCommand } from "@aws-sdk/client-s3";

const s3Client = new S3Client({ region: "us-east-1" });

export const handler = async (event) => {
    try {
        // O evento traz as informações do bucket e do arquivo que disparou a Lambda
        const bucketName = event.Records[0].s3.bucket.name;
        const objectKey = decodeURIComponent(event.Records[0].s3.object.key.replace(/\+/g, " "));

        console.log(`Iniciando processamento do arquivo: ${objectKey} no bucket: ${bucketName}`);

        const params = {
            Bucket: bucketName,
            Key: objectKey,
        };

        // Recupera o objeto do S3
        const command = new GetObjectCommand(params);
        const response = await s3Client.send(command);

        console.log("Arquivo recuperado com sucesso. Content-Type:", response.ContentType);
        
        // Aqui entraria a lógica de transformação do arquivo (redimensionar imagem, processar dados, etc.)
        
        return {
            statusCode: 200,
            body: JSON.stringify('Processamento concluído com sucesso!'),
        };
    } catch (error) {
        console.error("Erro ao processar o arquivo no S3:", error);
        throw error;
    }
};
