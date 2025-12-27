# Lab 02 — Lambda Greengrass (`lab02/lambda/index.js`)

## Contexto arquitetural
Este arquivo implementa uma função Node.js executada no **AWS IoT Greengrass** (edge) e também compatível com **AWS Lambda** (cloud). O padrão adotado é o de **função dual-mode**: a lógica de negócio é encapsulada em uma função simples (`greengrassHelloWorldRun`) que pode ser chamada tanto em ambiente edge quanto no handler cloud. Isso facilita reutilização, testes e portabilidade.

**Conceitos e boas práticas abordados:**
- **Separation of concerns:** a lógica principal está isolada da lógica de bootstrap (agendamento, handler).
- **Idempotência e simplicidade:** o trabalho executado é simples e determinístico.
- **Portabilidade:** uso de `process.env.IS_LAMBDA` para alternar comportamento entre edge e cloud.
- **Observabilidade básica:** saída via `console.log` para facilitar logs do Greengrass/Lambda.

## Código com anotações

```javascript
01 const os = require('os');
02 const util = require('util');
03 
04 const myPlatform = util.format('%s-%s', os.platform(), os.release());
05 
06 function greengrassHelloWorldRun() {
07     console.log("Hello World Greengrass on Lambda");
08     console.log(myPlatform);
09 }
10 
11 if (!process.env.IS_LAMBDA){
12     //execute only greengrass
13     setInterval(greengrassHelloWorldRun, 10000);
14 }
15 
16 exports.handler = function handler(event, context) {
17     //execute only "cloud" lambda
18     greengrassHelloWorldRun();
19 };
```

### Anotações técnicas
- **Linhas 1–2:** Importação de módulos nativos (`os`, `util`). Evita dependências externas e mantém o pacote leve, importante para funções edge/IoT.
- **Linha 4:** Gera string com plataforma e release. Padrão útil para **telemetria básica** e diagnóstico de ambiente.
- **Linhas 6–9:** Função única de negócio. **Boa prática:** manter o core de execução isolado, facilitando reutilização entre edge e cloud.
- **Linhas 11–14:** Quando **não** está em Lambda, executa no Greengrass com `setInterval`. Esse padrão é comum em componentes edge, onde a execução é contínua.
- **Linhas 16–19:** Handler padrão do AWS Lambda. Aqui a função é executada uma única vez, seguindo o ciclo de execução serverless.

## Observações de arquitetura
- **Edge vs Cloud:** Greengrass permite execução contínua, enquanto Lambda é orientado a eventos. O código ilustra essa diferença de ciclo de vida.
- **Padrão “single source of truth”:** a função `greengrassHelloWorldRun` é a fonte única da lógica de negócio.
- **Boas práticas para produção:** normalmente recomenda-se:
  - **Configuração via variáveis de ambiente** (intervalo, mensagens, flags de modo).
  - **Tratamento de erros** e **logs estruturados** para diagnósticos no edge.
  - **Graceful shutdown** em ambientes de longa duração.
