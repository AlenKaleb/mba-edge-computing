# Lab 01 — Recipe Greengrass HelloWorld 1.1.0 (`lab01/recipes/com.example.HelloWorld-1.1.0.yaml`)

## Contexto arquitetural
Esta recipe é uma evolução da versão anterior, adicionando **Artifacts** para distribuir o script a partir de um bucket S3. Isso é essencial para **reprodutibilidade**, **controle de versões** e **distribuição escalável** em ambientes edge.

**Conceitos e boas práticas abordados:**
- **Distribuição de artefatos:** referência explícita ao script via URI S3.
- **Lifecycle hooks multi-OS:** execução separada por plataforma.
- **Configuração declarativa:** parâmetros por configuração para customização do comportamento.

## Código com anotações

```yaml
01 ---
02 RecipeFormatVersion: '2020-01-25'
03 ComponentName: com.example.HelloWorld
04 ComponentVersion: '1.0.0'
05 ComponentDescription: My first AWS IoT Greengrass component.
06 ComponentPublisher: Amazon
07 ComponentConfiguration:
08   DefaultConfiguration:
09     Message: world
10 Manifests:
11   - Platform:
12       os: linux
13     Lifecycle:
14       run: |
15         python3 -u {artifacts:path}/hello_world.py "{configuration:/Message}"
16     Artifacts:
17       - URI: s3://greengrass-lab-2024-06-10/artifacts/com.example.HelloWorld/1.0.0/hello_world.py
18   - Platform:
19       os: windows
20     Lifecycle:
21       run: |
22         py -3 -u {artifacts:path}/hello_world.py "{configuration:/Message}"
23     Artifacts:
24       - URI: s3://greengrass-lab-2024-06-10/artifacts/com.example.HelloWorld/1.0.0/hello_world.py
```

### Anotações técnicas
- **Linhas 1–9:** Mesmos metadados e configuração default da recipe anterior.
- **Linhas 11–15 / 19–22:** `Lifecycle.run` executa o script e injeta o valor `Message` de configuração.
- **Linhas 16–17 / 23–24:** `Artifacts` aponta para o script no S3. Isso permite:
  - **Distribuição centralizada** do binário/script.
  - **Controle de versão** por path (`/1.0.0/`).
  - **Reproducibilidade** em múltiplos dispositivos.

## Observações de arquitetura
- **Supply chain de software:** usar S3 como origem de artefatos é padrão no Greengrass para manter rastreabilidade.
- **Imutabilidade:** cada versão de componente deve apontar para artefatos versionados, evitando “drift”.
- **Boa prática adicional:** alinhar `ComponentVersion` com o nome do arquivo para evitar confusão operacional.
