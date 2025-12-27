# Lab 01 — Recipe Greengrass HelloWorld 1.0.0 (`lab01/recipes/com.example.HelloWorld-1.0.0.yaml`)

## Contexto arquitetural
Este arquivo define uma **recipe de componente AWS IoT Greengrass**. A recipe descreve como o componente é configurado, executado e distribuído em diferentes plataformas. É um artefato central na arquitetura do Greengrass, permitindo **configuração declarativa** e **portabilidade multi-OS**.

**Conceitos e boas práticas abordados:**
- **Infraestrutura declarativa:** o comportamento do componente é descrito via YAML.
- **Configuração padrão:** `DefaultConfiguration` fornece valores iniciais que podem ser sobrescritos.
- **Multi-platform manifests:** separação de instruções por sistema operacional.
- **Lifecycle hooks:** definição de comandos de execução (`run`).

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
16   - Platform:
17       os: windows
18     Lifecycle:
19       run: |
20         py -3 -u {artifacts:path}/hello_world.py "{configuration:/Message}"
```

### Anotações técnicas
- **Linha 2:** Versão do formato da recipe. Importante para compatibilidade com o runtime Greengrass.
- **Linhas 3–6:** Metadados do componente (nome, versão, descrição, publisher) usados no catálogo e para versionamento.
- **Linhas 7–9:** Configuração padrão. `Message` pode ser sobrescrita via deployment. **Boa prática:** valores default seguros e claros.
- **Linhas 10–20:** Manifestos por plataforma. A separação garante que o comando correto seja usado em cada sistema operacional.
- **Linhas 14–15 / 19–20:** Comando `run` usa o caminho de artefato e uma configuração referenciada via **substituição de parâmetros** (`{configuration:/Message}`), padrão do Greengrass.

## Observações de arquitetura
- **Declarativo e versionado:** recipes são imutáveis por versão, permitindo rollbacks previsíveis.
- **Portabilidade:** suportar Linux e Windows é essencial para dispositivos IoT heterogêneos.
- **Boa prática recomendada:** adicionar também uma seção `Artifacts` para especificar a origem do script, garantindo reprodutibilidade do componente.
