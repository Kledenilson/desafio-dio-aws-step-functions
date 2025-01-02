
# DESAFIO DIO: Projeto com AWS Step Functions - Fluxos de Processamento e Integração com Serviços AWS

Este repositório contém dois exemplos práticos de fluxos **AWS Step Functions** para diferentes cenários demonstrando integrações com serviços como  **S3**, **Bedrock** e **EC2**.

---

## 🛠️ Pré-requisitos

Antes de implementar os fluxos, certifique-se de:
- Ter uma conta AWS ativa.
- Configurar os serviços necessários (S3, Bedrock e EC2).
- Configurar as permissões apropriadas para o Step Functions acessar os recursos.

---

## 📂 Estrutura do Repositório

- **`desafio-dio-simples.json`**: Exemplo simples com Lambda e decisões baseadas em condições.
- **`desafio-dio-com-Bedrock-S3-e-EC2.json`**: Exemplo intermediário com Bedrock, S3 e EC2.

---

## 🚀 Exemplo 1: Fluxo Simples (desafio-dio-simples.json)

Este fluxo realiza as seguintes etapas:

1. **Valida o input inicial**.
2. **Decide se o valor é maior ou menor que 10**.
3. **Executa uma função Lambda** correspondente à decisão.
4. **Finaliza o fluxo**.

### 📄 JSON do Fluxo

O arquivo `desafio-dio-simples.json` contém o seguinte:

```json
{
  "Comment": "Fluxo simples para aprendizado sobre AWS Step Functions",
  "StartAt": "DESAFIO_DIO_ValidarInput",
  "States": {
    "DESAFIO_DIO_ValidarInput": {
      "Type": "Pass",
      "Parameters": {
        "Input.$": "$.valor"
      },
      "Next": "DESAFIO_DIO_VerificarValor"
    },
    "DESAFIO_DIO_VerificarValor": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.Input",
          "NumericGreaterThan": 10,
          "Next": "DESAFIO_DIO_ValorMaiorQue10"
        }
      ],
      "Default": "DESAFIO_DIO_ValorMenorOuIgualA10"
    },
    "DESAFIO_DIO_ValorMaiorQue10": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGIAO:ID_DA_CONTA:function:FuncaoMaiorQue10",
      "Next": "DESAFIO_DIO_Final"
    },
    "DESAFIO_DIO_ValorMenorOuIgualA10": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:REGIAO:ID_DA_CONTA:function:FuncaoMenorQue10",
      "Next": "DESAFIO_DIO_Final"
    },
    "DESAFIO_DIO_Final": {
      "Type": "Succeed"
    }
  }
}
```

### 📝 Configuração

- Substitua `REGIAO`, `ID_DA_CONTA` e `FuncaoMaiorQue10`/`FuncaoMenorQue10` com os valores correspondentes à sua conta e região.

---

## 🌟 Exemplo 2: Fluxo Avançado (desafio-dio-com-Bedrock-S3-e-EC2.json)

### 📜 Descrição

Este fluxo integra **Bedrock**, **S3** e **EC2** para realizar análise de sentimento em um arquivo e tomar ações com base no resultado.

### 📄 JSON do Fluxo

O arquivo `desafio-dio-com-Bedrock-S3-e-EC2.json` contém o seguinte:

```json
{
  "Comment": "Fluxo Step Functions usando Bedrock, S3 e EC2",
  "StartAt": "DESAFIO_DIO_ReceberArquivo",
  "States": {
    "DESAFIO_DIO_ReceberArquivo": {
      "Type": "Pass",
      "Parameters": {
        "BucketName": "meu-bucket",
        "Key": "texto-sentimento-usuario.txt"
      },
      "Next": "DESAFIO_DIO_RealizarAnaliseSentimento"
    },
    "DESAFIO_DIO_RealizarAnaliseSentimento": {
      "Type": "Task",
      "Resource": "arn:aws:bedrock:REGIAO:ID_DA_CONTA:model/bedrock-model-name",
      "Parameters": {
        "Input.$": "$.Key",
        "Configuration": {
          "TaskType": "sentiment-analysis"
        }
      },
      "Next": "DESAFIO_DIO_VerificarSentimento"
    },
    "DESAFIO_DIO_VerificarSentimento": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.Sentiment",
          "StringEquals": "Positivo",
          "Next": "DESAFIO_DIO_AcionarEC2"
        }
      ],
      "Default": "DESAFIO_DIO_SalvarResultadoNegativo"
    },
    "DESAFIO_DIO_AcionarEC2": {
      "Type": "Task",
      "Resource": "arn:aws:states:REGIAO:ID_DA_CONTA:action:ec2:runInstances",
      "Parameters": {
        "ImageId": "ami-EXEMPLO",
        "InstanceType": "t2.micro",
        "MinCount": 1,
        "MaxCount": 1,
        "KeyName": "minha-chave-ec2",
        "TagSpecifications": [
          {
            "ResourceType": "instance",
            "Tags": [
              {
                "Key": "Name",
                "Value": "DESAFIO_DIO_EC2_Instance"
              }
            ]
          }
        ]
      },
      "Next": "DESAFIO_DIO_FinalizarProcessamento"
    },
    "DESAFIO_DIO_SalvarResultadoNegativo": {
      "Type": "Task",
      "Resource": "arn:aws:s3:::meu-bucket",
      "Parameters": {
        "Bucket": "meu-bucket",
        "Key": "resultados/resultado-negativo.json",
        "Body.$": "$"
      },
      "Next": "DESAFIO_DIO_Final"
    },
    "DESAFIO_DIO_FinalizarProcessamento": {
      "Type": "Task",
      "Resource": "arn:aws:s3:::meu-bucket",
      "Parameters": {
        "Bucket": "meu-bucket",
        "Key": "resultados/resultado-positivo.json",
        "Body.$": "$"
      },
      "Next": "DESAFIO_DIO_Final"
    },
    "DESAFIO_DIO_Final": {
      "Type": "Succeed"
    }
  }
}
```

### 📝 Configuração

- **Substitua:**
  - `meu-bucket`: Nome do bucket S3 onde os arquivos serão armazenados.
  - `REGIAO` e `ID_DA_CONTA`: Informações da sua conta AWS.
  - `bedrock-model-name`: Nome do modelo Bedrock configurado.
  - `ami-EXEMPLO` e `minha-chave-ec2`: Configurações da sua instância EC2.

---

## 📖 Como Executar

1. **Configure os serviços AWS:**
   - **S3**: Crie o bucket `meu-bucket` e faça o upload do arquivo `texto-sentimento-usuario.txt`.
   - **Bedrock**: Configure um modelo para análise de sentimento.
   - **EC2**: Garanta que a instância EC2 está pronta para ser iniciada.

2. **Implante os Fluxos:**
   - Use o AWS Management Console para criar os Step Functions com os arquivos JSON fornecidos.

3. **Teste os Fluxos:**
   - Forneça os inputs apropriados para testar ambos os cenários.


