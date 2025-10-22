# Projeto AWS Step Functions — Workflow de Compras e Reembolsos  

Este projeto implementa um **workflow automatizado** no **AWS Step Functions**, capaz de **diferenciar solicitações de compra e reembolso**, executando funções distintas em cada caso e consolidando os resultados.

---

## Objetivos  

Demonstrar a orquestração de funções Lambda com o AWS Step Functions;  
Utilizar o estado **Choice** para tomada de decisão condicional; 
Automatizar fluxos para diferentes tipos de requisições (`PURCHASE` e `REFUND`);  
Consolidar respostas em uma função final (`Resp por Resultado`);  

---

## Arquitetura do Workflow  

O diagrama a seguir representa o fluxo criado:

<img width="477" height="344" alt="image" src="https://github.com/user-attachments/assets/8927febc-d2b2-4be4-9f73-1382650b5824" />


### Explicação do Fluxo  

1. **Start:** Início da execução.  
2. **Choice State:** Verifica o tipo de requisição (`$.type`).  
   - Se `$.type == "PURCHASE"` → executa `Lambda Resp por Compras`.  
   - Se `$.type == "REFUND"` → executa `Lambda Resp por Reembolso`.  
3. Após a execução, ambos os caminhos convergem para a função  
   **Lambda Resp por Resultado**, que retorna a resposta consolidada.  
4. **End:** Finaliza o workflow.

---

## Definição da State Machine (JSON)

```json
{
  "Comment": "Workflow para processamento de Compras e Reembolsos",
  "StartAt": "Choice",
  "States": {
    "Choice": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.type",
          "StringEquals": "PURCHASE",
          "Next": "Resp por Compras"
        },
        {
          "Variable": "$.type",
          "StringEquals": "REFUND",
          "Next": "Resp por Reembolso"
        }
      ]
    },
    "Resp por Compras": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGIAO:ID_DA_CONTA:function:purchase_handler",
      "Next": "Resp por Resultado"
    },
    "Resp por Reembolso": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGIAO:ID_DA_CONTA:function:refund_handler",
      "Next": "Resp por Resultado"
    },
    "Resp por Resultado": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGIAO:ID_DA_CONTA:function:result_handler",
      "End": true
    }
  }
}

## Funções Lambda

purchase_handler.py

def lambda_handler(event, context):
    print("Processando compra...")
    item = event.get("item", "desconhecido")
    value = event.get("value", 0)
    return {
        "status": "success",
        "operation": "PURCHASE",
        "item": item,
        "value": value,
        "message": f"Compra do item '{item}' no valor de R$ {value} processada com sucesso."
    }

refund_handler.py

def lambda_handler(event, context):
    print("Processando reembolso...")
    item = event.get("item", "desconhecido")
    value = event.get("value", 0)
    return {
        "status": "success",
        "operation": "REFUND",
        "item": item,
        "value": value,
        "message": f"Reembolso do item '{item}' no valor de R$ {value} processado com sucesso."
    }
result_handler.py

def lambda_handler(event, context):
    print("Consolidando resultado final...")
    return {
        "status": "OK",
        "finalResponse": event,
        "message": "Fluxo executado com sucesso via AWS Step Functions."
    }





