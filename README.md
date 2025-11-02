# Santander-code-girls-Step-Functions

# AWS Step Functions - Tudo que aprendi no Bootcamp (até agora)

> *Resumo pessoal do que entendi sobre Step Functions - não é perfeito, mas é o que funcionou pra mim*

---

## O que é Step Functions?

Pra mim, é tipo um **orquestrador de serviços da AWS**. Você cria um fluxo (um "workflow") onde cada passo chama um serviço diferente: Lambda, DynamoDB, SQS, SNS, até EC2 se precisar. Ele cuida da ordem, dos erros, dos retries, tudo automático.

É **serverless**, ou seja, não preciso gerenciar servidor nenhum. Pago só pelo que uso.

---

![Imagem do WhatsApp de 2025-11-02 à(s) 14 05 42_accd8ced](https://github.com/user-attachments/assets/0f4c6e18-9045-482c-b99d-16da045727dd)



## Como funciona na prática?

1. **Você define o fluxo em JSON (ASL - Amazon States Language)**
2. **Cada estado é uma ação** (Task, Choice, Wait, Parallel, etc.)
3. **Se der erro, ele para ou tenta de novo (com retry)**
4. **Você vê tudo no console: gráfico, histórico, logs**

---

## Exemplo que eu fiz (e funcionou!)

```json
{
  "Comment": "Fluxo simples: Lambda → DynamoDB → SNS",
  "StartAt": "ProcessarDados",
  "States": {
    "ProcessarDados": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:processar-csv",
      "Next": "SalvarNoDynamo"
    },
    "SalvarNoDynamo": {
      "Type": "Task",
      "Resource": "arn:aws:states:::dynamodb:putItem",
      "Parameters": {
        "TableName": "Pedidos",
        "Item.$": "$.dados"
      },
      "Next": "EnviarNotificacao"
    },
    "EnviarNotificacao": {
      "Type": "Task",
      "Resource": "arn:aws:states:::sns:publish",
      "Parameters": {
        "TopicArn": "arn:aws:sns:us-east-1:123456789012:pedidos-topic",
        "Message.$": "$.resultado"
      },
      "End": true
    }
  }
}
