# my_workflows

Workflows reutilizáveis de GitHub Actions para padronização de pipelines CI/CD com DevSecOps integrado + mecanismo de self-update automático.

[![GitHub](https://img.shields.io/badge/GitHub-D2nke%2Fmy__workflows-blue)](https://github.com/D2nke/my_workflows)

---

## 🎯 Problema / Solução / Impacto

**Problema:** Centenas de repositórios precisam de CI/CD padronizado. Manter pipelines atualizadas em cada repo individualmente é insustentável — mudanças de segurança não se propagam, e há inconsistência entre ambientes.

**Solução:** Um repositório centralizado com:
- **8 Workflows reutilizáveis** (pre-build, test-*, security, build-docker, deploy, files-update)
- **12 Callers** organizados por linguagem (Node, Java, Python) com 4 tipos cada (pr-check, release, sandbox, self-update)
- **Sistema de self-update** que sincroniza automaticamente callers para clientes via `repository_dispatch`

**Impacto:**
- ✅ CI/CD padronizado para 100+ repositórios (não apenas migrados)
- ✅ Gates DevSecOps (SonarQube, Mend) aplicados em TODO push
- ✅ Atualizar 1 arquivo aqui = todas as pipelines dos clientes atualizam automaticamente
- ✅ Onboarding de novas apps: 5 minutos (copiar caller para `.github/workflows`)
- ✅ Logs claros e profissionais em cada step

---

## 📚 Workflows Reutilizáveis

| Workflow | Tipo | Descrição |
|----------|------|-----------|
| `pre-build.yml` | Validação | Verifica ENVs obrigatórios e arquivos essenciais |
| `test-java.yml` | Testes | Maven/Gradle + JUnit + Jacoco coverage |
| `test-python.yml` | Testes | pytest + coverage (fail under 70%) |
| `test-node.yml` | Testes | npm test + Jest + coverage |
| `security-scan.yml` | Security | SonarQube quality gate + Mend SCA |
| `build-docker.yml` | Build | Multi-stage Docker build com layer caching |
| `deploy.yml` | Deploy | Linux server (SSH) / Databricks / Azure App Service |
| `files-update.yml` | Sync | Sincroniza callers de fonte para clients (self-update) |

---

## 📂 Callers por Linguagem

### Node
```
callers/node/
├── pr-check.yml     → Roda em abertura/atualização de PR
├── release.yml      → Full pipeline: build → security → tests → docker → deploy → tag
├── sandbox.yml      → Staging em develop
└── self-update.yml  → Sincroniza este caller com clientes
```

### Java
```
callers/java/
├── pr-check.yml     → Testes unitários (Maven 17)
├── release.yml      → Full pipeline com JUnit + Jacoco
├── sandbox.yml      → Staging
└── self-update.yml  → Self-update
```

### Python
```
callers/python/
├── pr-check.yml     → pytest com coverage >= 70%
├── release.yml      → Full pipeline com pytest + coverage
├── sandbox.yml      → Staging
└── self-update.yml  → Self-update
```

---

## 🚀 Como Usar em um Novo Repositório

### Opção 1: Node (Recomendado para começar)
```bash
# 1. Clone/copie o caller
cp D2nke/my_workflows/callers/node/* seu-repo/.github/workflows/

# 2. Configure secrets no seu repo
#    - DOCKER_USERNAME
#    - DOCKER_PASSWORD (ou token)
#    - SONAR_TOKEN (opcional, para security-scan)

# 3. Commit e push
git add .github/workflows
git commit -m "ci: add workflows from my_workflows"
git push origin feature-branch

# 4. Abra PR → pr-check.yml acionado automaticamente ✨
```

### Opção 2: Java
```bash
cp D2nke/my_workflows/callers/java/* seu-repo/.github/workflows/
```

### Opção 3: Python
```bash
cp D2nke/my_workflows/callers/python/* seu-repo/.github/workflows/
```

---

## 🔄 Fluxo de Execução

### PR Check (ao abrir PR)
```
pull_request [main/develop]
  ↓
pr-check.yml (caller)
  ├─ pre-build.yml        ✓ Validação
  ├─ security-scan.yml    ✓ Snyk/SonarQube
  └─ test-*.yml           ✓ Testes + coverage
     └─ Resultado comentado no PR
```

### Release (ao fazer push em main)
```
push [main]
  ↓
release.yml (caller)
  ├─ pre-build.yml        ✓
  ├─ security-scan.yml    ✓ Snyk
  ├─ test-*.yml           ✓
  ├─ build-docker.yml     ✓ Build + push Docker Hub
  ├─ deploy.yml           ✓ Deploy em produção
  └─ release-tag          ✓ v0.0.N
     └─ Deploy em produção! 🎉
```

### Self-Update (Sincronização Automática)
```
Admin modifica callers/node/release.yml
  ↓
Executa self-update.yml (workflow_dispatch)
  ↓
repository_dispatch enviado para TODOS os clientes Node
  ↓
Cada cliente: files-update.yml copia callers/node/* → .github/workflows/
  ↓
PR auto-criada → auto-merge (opcional)
  ↓
Todos os clientes sincronizados! ✨
```

---

## 🔐 Secrets & Variables Necessárias

### Secrets (Configure em cada repo cliente)
```yaml
DOCKER_USERNAME       # Docker Hub user
DOCKER_PASSWORD       # Docker Hub token/password
SONAR_TOKEN          # SonarCloud token (opcional)
MEND_API_KEY         # Mend SCA token (opcional)
SSH_PRIVATE_KEY      # SSH key para deploy em servidor (opcional)
AZURE_CREDENTIALS    # Azure credentials (opcional)
DATABRICKS_TOKEN     # Databricks token (opcional)
```

### Organization Variables (Compartilhado, opcional)
```yaml
DOCKER_REGISTRY      # docker.io (padrão)
SONAR_HOST_URL       # URL do SonarQube/SonarCloud
DEV_SERVER          # Endereço servidor dev
```

---

## 📝 Exemplos de Uso

### Em um repo Node (my_app_demo)
```yaml
# .github/workflows/pr-check.yml
name: PR Check (Node)

on:
  pull_request:
    branches: [main, develop]

jobs:
  run:
    uses: D2nke/my_workflows/callers/node/pr-check.yml@latest
```

### Em um repo Java
```yaml
# .github/workflows/release.yml
name: Release (Java)

on:
  push:
    branches: [main]

jobs:
  run:
    uses: D2nke/my_workflows/callers/java/release.yml@latest
```

### Em um repo Python
```yaml
# .github/workflows/sandbox.yml
name: Sandbox (Python)

on:
  push:
    branches: [develop]

jobs:
  run:
    uses: D2nke/my_workflows/callers/python/sandbox.yml@latest
```

---

## 📊 Estrutura Completa

```
D2nke/my_workflows/
├── .github/workflows/          # 8 Workflows reutilizáveis
│   ├── pre-build.yml
│   ├── test-java.yml
│   ├── test-python.yml
│   ├── test-node.yml
│   ├── security-scan.yml
│   ├── build-docker.yml
│   ├── deploy.yml
│   └── files-update.yml
│
├── callers/                    # 12 Callers (3 langs × 4 tipos)
│   ├── node/
│   │   ├── pr-check.yml
│   │   ├── release.yml
│   │   ├── sandbox.yml
│   │   └── self-update.yml
│   ├── java/
│   │   ├── pr-check.yml
│   │   ├── release.yml
│   │   ├── sandbox.yml
│   │   └── self-update.yml
│   └── python/
│       ├── pr-check.yml
│       ├── release.yml
│       ├── sandbox.yml
│       └── self-update.yml
│
└── reference/
    └── reusable/              # Documentação de exemplo (Bradesco)
```

---

## 💡 Boas Práticas

- ✅ Use `@latest` nas referências (`uses: D2nke/my_workflows/...@latest`)
- ✅ Sempre passe `secrets: inherit`
- ✅ Customize via inputs (java_version, node_version, python_version)
- ✅ Logs claros: `echo "→ Executando..." && ... && echo "✓ Pronto"`
- ❌ Não copie código entre workflows
- ❌ Não commite secrets
- ❌ Não use branches específicas (sempre `@latest`)

---

## 🔗 Links Úteis

- [Documentação Completa](./specs/README.md)
- [Quick Reference](./specs/QUICK_REFERENCE.md)
- [Arquitetura Detalhada](./specs/02_arquitetura_detalhada.md)
- [Diagramas de Fluxo](./specs/03_diagrama_fluxo.md)
- [Implementação Prática](./specs/04_implementacao_pratica.md)

---

## 📞 Suporte

Para questões ou contribuições, abra uma issue ou PR neste repositório.

---

**Implementado com** ❤️ **para** ✨ **Platform Engineering**
