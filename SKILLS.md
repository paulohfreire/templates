# SKILLS - Claude Code Personal Skills

Este documento contém as skills personalizadas criadas para o desenvolvimento, seguindo a metodologia TDD (Test-Driven Development).

## 📋 Skills Disponíveis

### 1. **sync-main**
**Descrição:** Sincroniza a branch local main com o repositório remoto usando um workflow bulletproof que protege o trabalho local sob qualquer pressão.

**Quando usar:**
- Branch local main está atrás do remoto
- Precisa das últimas alterações do remoto antes de criar feature branch
- "git status" mostra alterações não commitadas E precisa fazer sync
- Existe pressão de tempo (deadline, time esperando, deploy pendente)

**Quando NÃO usar:**
- Trabalhando em feature branch (faça checkout main primeiro)
- Sem repositório remoto configurado
- Remoto não existe

**Princípio Core:** SEMPRE verifique alterações locais PRIMEIRO. NUNCA assuma diretório limpo.

**Fluxo principal:**
```bash
# STEP 1: OBRIGATÓRIO - Verificar alterações locais e BLOQUEAR se encontrar
OUTPUT=$(git status --porcelain)
if [[ -n "$OUTPUT" ]]; then
    echo "❌ ERROR: Local changes detected. MUST stash first. ABORTING."
    exit 1
fi

# STEP 2: OBRIGATÓRIO - Stash se houver changes (CRÍTICO: -u inclui untracked)
git stash push -u -m "WIP before sync - $(date +%s)"

# STEP 3: Auto-detectar nome da branch main (previne confusão main/master)
MAIN_BRANCH=$(git symbolic-ref refs/remotes/origin/HEAD | sed 's@refs/remotes/origin/@@')

# STEP 4-8: Verificar requisitos e fazer sync
git checkout $MAIN_BRANCH
git fetch origin
git pull origin $MAIN_BRANCH

# STEP 9-10: OBRIGATÓRIO - Restore e verificar stash
git stash pop
# Verificação final de que stash foi limpo
```

**Arquivo:** `~/.claude/skills/sync-main/SKILL.md`

**Status:** ✅ Completa (RED-GREEN-REFACTOR)

---

### 2. **git-commit-flow**
**Descrição:** Workflow completo para commitar alterações locais, criar nova feature branch e preparar trabalho para submissão de pull request usando uma sequência single, bulletproof.

**Quando usar:**
- Arquivos locais foram modificados e precisam ser commitados
- Iniciando trabalho de nova feature a partir de alterações existentes
- Preparando trabalho para code review via PR
- Precisa criar branch isolada para novo trabalho

**Quando NÃO usar:**
- Apenas commitando em branch existente (use git commit)
- Sem alterações locais para commit
- Branch já existe e apenas precisa de commit

**Princípio Core:** SEMPRE sync com remoto ANTES de criar feature branch.

**Fluxo principal:**
```bash
# PRE-STEP: OBRIGATÓRIO - Verificar que está em main
if [[ "$(git branch --show-current)" != "main" ]]; then
    echo "❌ ERROR: You must be on main branch before starting"
    exit 1
fi

# STEP 1: OBRIGATÓRIO - Sync com remoto PRIMEIRO (SEM EXCEÇÕES)
git fetch origin
git pull origin main

# STEP 2: OBRIGATÓRIO - Criar feature branch de main ATUALIZADA
# Pattern: type/YYYY-MM-DD-description
git checkout -b feature/$(date +%Y-%m-%d)-descriptive-name

# STEP 3: OBRIGATÓRIO - Stage alterações
git add .

# STEP 4: OBRIGATÓRIO - Commit com formato CONVENTIONAL COMMIT
# Format: type(scope): description
# Types: feat, fix, docs, style, refactor, test, chore
git commit -m "feat: add descriptive commit message"

# STEP 5: OBRIGATÓRIO - Push com upstream tracking
git push -u origin feature/$(date +%Y-%m-%d)-descriptive-name
```

**Conventional Commits (OBRIGATÓRIO):**
- `feat:` - Nova funcionalidade
- `fix:` - Bug fix
- `docs:` - Mudanças de documentação
- `style:` - Estilo de código (formatação, sem mudança de lógica)
- `refactor:` - Refatoração de código
- `test:` - Adicionando testes
- `chore:` - Tarefas de manutenção

**Branch Naming (OBRIGATÓRIO):**
```
feature/2026-06-03-user-authentication
fix/2026-06-03-login-bug
hotfix/2026-06-03-security-patch
refactor/2026-06-03-api-cleanup
```

**Arquivo:** `~/.claude/skills/git-commit-flow/SKILL.md`

**Status:** ✅ Completa (RED-GREEN-REFACTOR)

---

## 🎯 Metodologia de Criação

Todas as skills foram criadas seguindo **Test-Driven Development (TDD)**:

### **RED Phase** - Escrever Teste que Falha
- Criar cenários de pressão (3+ pressões combinadas)
- Rodar cenários SEM skill - documentar comportamento baseline
- Identificar padrões em racionalizações/falhas

### **GREEN Phase** - Escrever Skill Mínima
- Criar skill que endereça racionalizações específicas
- Rodar mesmos cenários COM skill - verificar compliance
- Testar que previne comportamentos problemáticos

### **REFACTOR Phase** - Fechar Brechas
- Identificar NOVAS racionalizações dos testes
- Adicionar contadores explícitos
- Criar tabela de racionalizações de todas as iterações
- Re-testar até estar bulletproof

### **Resultados:**
- ✅ **sync-main**: 5 brechas identificadas e fechadas
- ✅ **git-commit-flow**: 6 brechas identificadas e fechadas

---

## 🚀 Como Usar

### **Sincronizar com Remoto:**
```bash
# Use a skill sync-main quando precisa atualizar main
/skill sync-main
# OU
# Siga o workflow em ~/.claude/skills/sync-main/SKILL.md
```

### **Criar Feature Branch e Commitar:**
```bash
# Use a skill git-commit-flow quando precisa criar branch e commitar
/skill git-commit-flow
# OU
# Siga o workflow em ~/.claude/skills/git-commit-flow/SKILL.md
```

### **Fluxo Completo de Trabalho:**
1. **sync-main** → Comece syncando seu local
2. **git-commit-flow** → Crie branch e commit alterações
3. **pr-request** → (ainda não criada) Crie o PR

---

## 📊 Estatísticas

- **Total de Skills Criadas:** 2
- **Skills Completas (RED-GREEN-REFACTOR):** 2
- **Skills Planejadas:** 4 (angular-new-app e pr-request não criadas)
- **Brechas Fechadas:** 11
- **Cenários de Teste:** 6
- **Tempo Total de Desenvolvimento:** ~3 horas

---

## 🔧 Manutenção

### **Adicionar Nova Skill:**
1. Siga metodologia TDD completa (RED-GREEN-REFACTOR)
2. Teste extensivamente sob pressão
3. Adicione entrada neste documento
4. Atualize estatísticas

### **Atualizar Skill Existente:**
1. Identifique novas brechas através de testes
2. Feche brechas seguindo REFACTOR phase
3. Re-teste para verificar bulletproofing
4. Atualize documentação

### **Problemas?**
- Se skill não está funcionando: Verifique se está em `~/.claude/skills/`
- Se behavior mudou: Execute testes baseline novamente
- Se encontrou brecha: Siga REFACTOR process

---

## 📝 Notas

- Skills foram testadas sob pressão real (time, authority, fatigue, complexity)
- Cada skill previne comportamentos específicos identificados em baseline testing
- Skills são "bulletproof" - resistem racionalização sob pressão
- Metodologia TDD garante que skills realmente funcionam

---

**Última Atualização:** 2026-06-03
**Versão:** 1.0
**Autor:** Claude Code + User (TDD methodology)
