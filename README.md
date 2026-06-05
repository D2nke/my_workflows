# my_workflows

Workflows reutilizáveis de GitHub Actions para padronização de pipelines CI/CD com DevSecOps integrado.

---

## Problema / Solução / Impacto

**Problema:** Migrar centenas de pipelines do Azure DevOps para GitHub Actions individualmente levaria meses e resultaria em implementações inconsistentes. Cada equipe configuraria os gates de segurança de forma diferente — ou simplesmente ignoraria.

**Solução:** Um repositório centralizado com workflows reutilizáveis que as equipes referenciam. Os gates de segurança e qualidade são aplicados no nível do workflow — as equipes não conseguem contorná-los. Atualizações no padrão são aplicadas uma vez aqui e propagam automaticamente.

**Impacto:**
- CI/CD padronizado para todas as aplicações migradas
- Gates DevSecOps (SonarQube, Mend, Fortify) aplicados de forma consistente nos ambientes DEV, HOM e PRD
- Onboarding de novas aplicações reduzido de dias para horas
- Ponto único de manutenção dos padrões de pipeline

---

## Workflows disponíveis

| Workflow | Trigger | Descrição |
|----------|---------|-----------|
| `build-docker.yml` | `workflow_call` | Build e push de imagens Docker com layer caching |
| `security-scan.yml` | `workflow_call` | Gate DevSecOps: SonarQube (qualidade) + Mend (SCA) |
| `test-java.yml` | `workflow_call` | Execução de testes Maven/Gradle com publicação de relatório JUnit |
| `test-python.yml` | `workflow_call` | Execução de testes pytest com cobertura |
| `deploy.yml` | `workflow_call` | Deploy em servidor Linux, Databricks ou Azure App Service |

---

## Como usar

Em qualquer repositório, inclua os workflows via `uses:`:

```yaml
jobs:
  test:
    uses: D2nke/my_workflows/.github/workflows/test-java.yml@main
    with:
      java_version: '17'
      build_tool: 'maven'

  security:
    needs: test
    uses: D2nke/my_workflows/.github/workflows/security-scan.yml@main
    with:
      sonar_project_key: meu-projeto
      enable_mend: true
    secrets:
      sonar_token: ${{ secrets.SONAR_TOKEN }}
      mend_api_key: ${{ secrets.MEND_API_KEY }}

  build:
    needs: security
    uses: D2nke/my_workflows/.github/workflows/build-docker.yml@main
    with:
      image_name: ghcr.io/D2nke/minha-app
    secrets:
      registry_username: ${{ github.actor }}
      registry_password: ${{ secrets.GITHUB_TOKEN }}
```

Veja exemplos completos por stack em [my_app_demo](https://github.com/D2nke/my_app_demo).

---

## Pré-requisitos

Configure no repositório chamador:

| Secret / Variable | Descrição |
|-------------------|-----------|
| `SONAR_TOKEN` | Token de autenticação do SonarQube |
| `MEND_API_KEY` | API key do Mend (antigo WhiteSource) |
| `SSH_PRIVATE_KEY` | Chave SSH para deploy em servidor |
| `SONAR_HOST_URL` (variable) | URL do servidor SonarQube |
| `DEV_SERVER` (variable) | Endereço do servidor de DEV |

---

## Estrutura

```
.github/
  workflows/
    build-docker.yml    # Build e push de imagem Docker
    security-scan.yml   # SonarQube + Mend SCA
    test-java.yml       # Testes Java (Maven/Gradle)
    test-python.yml     # Testes Python (pytest)
    deploy.yml          # Deploy multi-target
```
