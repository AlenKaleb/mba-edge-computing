# Lab 03 — Recipe Greengrass Secret (`lab03/recipes/com.example.Secret-1.0.0.yaml`)

## Contexto arquitetural
Esta recipe define um componente que acessa o **AWS IoT Greengrass Secret Manager**, imprimindo o valor de um segredo. O arquivo demonstra o uso de **dependências declarativas**, **políticas de acesso** e **configuração via ARN**.

**Conceitos e boas práticas abordados:**
- **Dependências explícitas:** `ComponentDependencies` garante que o Secret Manager esteja disponível.
- **IAM/ACL local no Greengrass:** políticas de acesso por componente.
- **Configuração segura:** `SecretArn` parametriza o segredo sem hardcode.
- **Lifecycle multi-OS:** instalação e execução separadas para Linux/Windows.

## Código com anotações

```yaml
01 RecipeFormatVersion: '2020-01-25'
02 ComponentName: com.example.Secret
03 ComponentVersion: 1.0.0
04 ComponentDescription: Prints the value.
05 ComponentPublisher: Amazon
06 ComponentDependencies:
07   aws.greengrass.SecretManager:
08     VersionRequirement: "^2.0.0"
09     DependencyType: HARD
10 ComponentConfiguration:
11   DefaultConfiguration:
12     SecretArn: ''
13     accessControl:
14       aws.greengrass.SecretManager:
15         com.example.Secret:secrets:1:
16           policyDescription: Allows access to a secret.
17           operations:
18             - aws.greengrass#GetSecretValue
19           resources:
20             - "*"
21 Manifests:
22   - Platform:
23       os: linux
24     Lifecycle:
25       install: python3 -m pip install --user awsiotsdk
26       run: python3 -u {artifacts:path}/retrive_secret.py "{configuration:/SecretArn}"
27     Artifacts:
28       - URI: s3://gglab-2024-06-10-lab/com.example.Secret/1.0.0/retrive_secret.py
29   - Platform:
30       os: windows
31     Lifecycle:
32       install: py -3 -m pip install --user awsiotsdk
33       run: py -3 -u {artifacts:path}/retrive_secret.py "{configuration:/SecretArn}"
34     Artifacts:
35       - URI: s3://gglab-2024-06-10-lab/com.example.Secret/1.0.0/retrive_secret.py
```

### Anotações técnicas
- **Linhas 6–9:** Dependência rígida do Secret Manager. **Boa prática:** declarar serviços necessários para garantir ordem de instalação e compatibilidade.
- **Linhas 10–20:** Configuração default e política de acesso. O bloco `accessControl` permite que o componente chame `GetSecretValue`.
- **Linha 12:** `SecretArn` vazio por padrão — ideal para **injeção segura** em deployment.
- **Linhas 25–26 / 32–33:** Instala SDK e executa script passando o ARN configurado.
- **Linhas 27–28 / 34–35:** Artefato versionado em S3, mantendo o código sincronizado com a recipe.

## Observações de arquitetura
- **Princípio do menor privilégio:** use `resources` mais restritos do que `"*"` em produção, limitando ao ARN específico do segredo.
- **Segurança operacional:** evitar logs do valor do segredo, limitar permissões e usar rotação do segredo.
- **Gestão de dependências:** instalação dinâmica do SDK pode ser substituída por artefatos empacotados para ambientes offline.
