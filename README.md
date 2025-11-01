# dio-desafio-aws-step-functions
Projeto prático de orquestração de workflows com AWS Step Functions. O desafio é automatizar processos que integram serviços da AWS (como Lambda, SQS e SNS), aplicando conceitos de arquitetura serverless. O foco é documentar o processo de criação, execução e monitoramento do workflow e os insights adquiridos.
# Desafio de Projeto DIO: Orquestrando Workflows com AWS Step Functions

Este repositório documenta a jornada de aprendizado e a solução prática do desafio de projeto da [Digital Innovation One (DIO)](https://www.dio.me/) sobre o AWS Step Functions.

O objetivo principal foi explorar e consolidar o conhecimento sobre como orquestrar serviços da AWS (como Lambda, SQS, SNS, S3) de forma visual e automatizada, criando uma "Máquina de Estados" (State Machine).

---

## 🎯 Objetivo

O desafio consistiu em seguir as aulas práticas para construir um workflow automatizado, documentando os principais conceitos aprendidos, os serviços utilizados e os insights adquiridos durante o processo.

Este `README.md` serve como a entrega final do projeto, consolidando todo o conhecimento adquirido.

---

## 📖 O que é o AWS Step Functions?

Durante o estudo, estes foram meus principais aprendizados sobre o serviço:

* **O que é?** É um serviço de orquestração "serverless" (sem servidor). Ele permite coordenar múltiplos serviços da AWS em um fluxo de trabalho (workflow) visual.
* **Para que serve?** Ele resolve o problema de gerenciar processos complexos de múltiplos passos. Em vez de uma função Lambda "gigante" ou de "correntes" de Lambdas (Lambda-chamando-Lambda), o Step Functions centraliza a lógica do fluxo.
* **Máquina de Estados (State Machine):** Este é o conceito central. Todo workflow é uma máquina de estados. Ele é definido usando uma linguagem JSON chamada **Amazon States Language (ASL)**.
* **Tipos de "States" (Estados):**
    * `Task`: A unidade de trabalho (ex: "executar esta função Lambda").
    * `Choice`: Um "if/else" no workflow (ex: "o arquivo foi validado com sucesso?").
    * `Parallel`: Executa ramos do fluxo em paralelo.
    * `Wait`: Pausa o fluxo por um tempo determinado.
* **Tratamento de Erros:** Uma das maiores vantagens. É possível usar blocos `Try/Catch/Finally` (similares à programação) para capturar erros de um serviço (ex: uma falha na Lambda) e tomar uma ação (ex: "enviar uma notificação para o SNS") sem quebrar o fluxo inteiro.

---

## ⚙️ O Workflow Construído (Estudo de Caso)

Como este projeto é focado na documentação conceitual (baseado nas aulas da DIO), descrevo abaixo um workflow clássico de **Processamento de Pedidos**, que demonstra o poder do Step Functions.

Este fluxo de trabalho simula um sistema de e-commerce que precisa validar e processar novos pedidos à medida que chegam.

### Passos do Fluxo

O processo de negócio segue esta lógica:

1.  **Início (Trigger):** O fluxo é iniciado (neste exemplo, manualmente) recebendo um JSON com os dados do pedido.
2.  **Passo 1: Validar Pedido (Lambda):** Uma função Lambda (`ValidarPedidoLambda`) é acionada. Ela verifica se o pedido contém os campos obrigatórios (ex: `produtoId` e `quantidade > 0`).
3.  **Passo 2: Decisão (Choice State):** O fluxo analisa a saída da Lambda.
    * `if (pedido.valido == true)`: O pedido está correto e segue para processamento.
    * `else (pedido.valido == false)`: O pedido tem um erro e segue para notificação.
4.  **Passo 3 (Sucesso): Processar Pedido (Lambda):** Se a validação foi OK, uma segunda função Lambda (`ProcessarPedidoLambda`) é chamada para salvar os dados no **Amazon DynamoDB**.
5.  **Passo 4 (Falha): Notificar Erro (SNS):** Se a validação falhou, o fluxo publica uma mensagem em um Tópico do **Amazon SNS** para alertar a equipe de suporte sobre o pedido inválido.
6.  **Fim:** O workflow é concluído.

### Diagrama Conceitual do Workflow

Este diagrama em texto (Mermaid) ilustra o fluxo de estados descrito acima:

```mermaid
graph TD;
    style Inicio fill:#228B22,color:#fff
    style Fim fill:#228B22,color:#fff
    style Lambda1 fill:#FF9900,color:#fff
    style Lambda2 fill:#FF9900,color:#fff
    style Choice fill:#0073bb,color:#fff
    style SNS fill:#D82233,color:#fff

    Inicio[Início do Fluxo] --> Lambda1(Executa Lambda 'ValidarPedido');
    Lambda1 --> Choice{Pedido Válido?};
    Choice -- Sim --> Lambda2(Executa Lambda 'ProcessarPedido');
    Choice -- Não --> SNS(Envia Notificação via SNS);
    Lambda2 --> Fim[Fim];
    SNS --> Fim;
