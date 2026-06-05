# 🚀 Quickstart: Adicione CI/CD em 5 Minutos

Este guia mostra como adicionar workflows reutilizáveis do `my_workflows` em qualquer repositório.

---

## ⏱️ 5 Passos

### 1️⃣ Escolha sua linguagem

```bash
# Node.js
LANGUAGE="node"

# Java
LANGUAGE="java"

# Python
LANGUAGE="python"
```

### 2️⃣ Clone ou baixe os callers

```bash
git clone https://github.com/D2nke/my_workflows.git tmp_workflows

# Copie os callers para seu repo
cp tmp_workflows/callers/$LANGUAGE/* seu-repo/.github/workflows/

# Limpe
rm -rf tmp_workflows
```

### 3️⃣ Configure os secrets no GitHub

Vá para **Settings → Secrets and variables → Actions** e adicione:

```
DOCKER_USERNAME     = seu_user_docker_hub
DOCKER_PASSWORD     = seu_token_docker_hub
SONAR_TOKEN         = (opcional) seu_token_sonarcloud
```

### 4️⃣ Commit e push

```bash
cd seu-repo
git add .github/workflows
git commit -m "ci: add workflows from D2nke/my_workflows"
git push origin feature-branch
```

### 5️⃣ Abra uma PR e veja a magia! ✨

```
Seu PR → GitHub Actions → pr-check.yml acionado automaticamente
                        ├─ pre-build ✓
                        ├─ security-scan ✓
                        └─ test-* ✓
                           └─ Resultado comentado no PR
```

---

## 📋 Checklist de Setup

- [ ] Escolheu linguagem (Node/Java/Python)
- [ ] Copiou callers para `.github/workflows/`
- [ ] Adicionou secrets (DOCKER_USERNAME, DOCKER_PASSWORD)
- [ ] Fez commit + push
- [ ] Abriu PR e viu workflows rodarem
- [ ] Fez push em `main` e viu `release.yml` rodar

---

## 🎯 O Que Cada Evento Faz

| Evento | Arquivo | O que faz |
|--------|---------|----------|
| Abrir/atualizar PR | `pr-check.yml` | Validação + testes + security |
| Push em `main` | `release.yml` | Full pipeline + Docker build + deploy + tag |
| Push em `develop` | `sandbox.yml` | Build Docker apenas (staging) |
| Workflow dispatch | `self-update.yml` | Sincroniza callers com fonte |

---

## 🔍 Verificando se Funcionou

### Seu PR foi comentado?
✅ Sucesso! Os workflows rodaram.

### Seu `main` fez deploy?
✅ Sucesso! A pipeline completa rodou.

### Viu erro de "Dockerfile not found"?
❌ Adicione um `Dockerfile` na raiz do repo.

### Viu erro de "package.json/pom.xml/setup.py not found"?
❌ Certifique-se de ter um arquivo de build na raiz.

---

## 🎨 Customize (Opcional)

Abra `.github/workflows/pr-check.yml` e ajuste:

```yaml
# Mudar versão do Node
uses: D2nke/my_workflows/.github/workflows/test-node.yml@latest
with:
  node-version: '21'  # ← Mude aqui

# Mudar versão do Java
with:
  java_version: '21'  # ← ou aqui

# Mudar versão do Python
with:
  python_version: '3.12'  # ← ou aqui
  coverage_fail_under: 80  # ← ou exigência de coverage
```

---

## 🆘 Troubleshooting

| Erro | Causa | Solução |
|------|-------|---------|
| `Dockerfile not found` | Arquivo não existe | Crie `Dockerfile` na raiz |
| `DOCKER_USERNAME secret not found` | Secret não configurado | Adicione em Settings → Secrets |
| `package.json not found` | Arquivo de build falta | Crie `package.json`, `pom.xml`, ou `setup.py` |
| `SonarQube connection failed` | SONAR_TOKEN não validado | Deixe `enable_mend: false` por enquanto |

---

## 📚 Próximos Passos

Depois que tudo funcionar:

1. Leia [Documentação Completa](./README.md)
2. Customize os workflows conforme necessário
3. Se atualizar callers, todos clientes sincronizam via `self-update.yml`

---

**Pronto!** Você agora tem CI/CD profissional. 🎉
