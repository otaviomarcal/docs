# Guia GitFlow customizado

Este guia define um fluxo de trabalho **GitFlow otimizado** para garantir que **todas as features estejam disponíveis no ambiente de desenvolvimento** (`dev`), mas **somente as aprovadas subam para produção**, de forma organizada e rastreável.

## 🚀 Fluxo Resumido
1️⃣ **Todas as features vão para a `dev` para testes**  
2️⃣ **Quando uma feature for aprovada para produção, criamos um PR dela para uma `release/xxxx`**  
3️⃣ **A release sobe para `main` via PR, garantindo controle e rastreabilidade**  
4️⃣ **A `dev` sempre recebe as mudanças que foram para produção, garantindo sincronização**  

---

## 📌 1. Criando e Trabalhando em uma Feature
Cada dev cria sua **feature branch** (`GI-1414`, `SCR-2060`, etc.) e a mergeia na `dev`:  
```bash
git checkout dev
git pull origin dev
git checkout -b GI-1414
```

Após desenvolver e testar localmente:
```bash
git add .
git commit -m "GI-1414: Ajusta validação de login"
git push origin GI-1414
```

Agora a feature está pronta para ser mergeada na `dev`.

---

## 📌 2. Merge da Feature na `dev`
Todas as features devem estar disponíveis para testes em `dev`. Antes de mergear:  
```bash
git checkout dev
git pull origin dev
git checkout GI-1414
git rebase dev  # Para garantir que está atualizada
```

Agora, mergeamos para `dev`:
```bash
git checkout dev
git merge --no-ff GI-1414 -m "Merge GI-1414 para dev"
git push origin dev
```

Agora a `dev` tem **todas as features disponíveis para teste**.

---

## 📌 3. Criando uma Release com Apenas as Features Aprovadas
Quando uma feature for aprovada para produção, criamos uma release **somente com as mudanças aprovadas**.

1. Criamos a release a partir da `main`:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b release/stable
   git push origin release/stable
   ```

2. Criamos um Pull Request da feature para a `release/stable` no GitHub.  
   - Exemplo: **PR de `GI-1414` → `release/stable`**.  
   - **Apenas features aprovadas são adicionadas à release**.

Agora a release **só contém as mudanças que irão para produção**.

---

## 📌 4. PR da Release para a `main`
Agora que a release está pronta, fazemos um **PR da `release/stable` para a `main`**.

1. Criamos um Pull Request no GitHub:  
   **`release/stable` → `main`**  
2. O Tech Lead revisa e aprova.  
3. Mergeamos via CLI:
   ```bash
   git checkout main
   git pull origin main
   git merge --no-ff release/stable -m "Merge release para main"
   git push origin main
   ```

Agora a **versão está oficialmente na `main`** e pronta para deploy.

---

## 📌 5. Garantindo que a `dev` receba tudo que subiu para `main`
Após o deploy, sincronizamos a `main` com a `dev`, garantindo que nada fique perdido:
```bash
git checkout dev
git pull origin dev
git merge --no-ff main -m "Sincronizando dev com main após release"
git push origin dev
```

Agora **a `dev` tem tudo que já subiu para produção**, garantindo que nenhuma feature seja perdida.

---
---

## Helper
## ❌ 📌 Removendo Features da `dev` e `uat` que Não Serão Aprovadas
Se uma feature foi mergeada na `dev` e/ou `uat`, mas **não será aprovada e nunca subirá para produção**, siga este fluxo para removê-la **sem quebrar a aplicação**.

### ✅ **Passo a Passo para Remover a Feature da `dev`**
1. **Identifique o commit do merge da feature na `dev`**  
   ```bash
   git log --oneline --graph --decorate --all
   ```
   - Localize o commit onde `GI-1010` foi mergeada.  
   - Anote o **hash do commit** (exemplo: `abc1234`).  

2. **Reverta a feature, criando um commit que remove as alterações**  
   ```bash
   git revert -m 1 abc1234
   ```
   - O parâmetro `-m 1` é necessário porque esta revertendo um **merge commit**.  

3. **Resolva possíveis conflitos e finalize a reversão**  
   ```bash
   git add .
   git commit -m "Revertendo GI-1010 da dev"
   ```

4. **Envie a reversão para o repositório remoto**  
   ```bash
   git push origin dev
   ```

Agora a `dev` **não contém mais a `GI-1010`**, mas o histórico está preservado.  

---

### ✅ **Removendo a Feature da `uat`**
Se a `GI-1010` também foi mergeada na `uat`, o processo é o mesmo:

1. **Vá para a `uat` e puxe as últimas mudanças**  
   ```bash
   git checkout uat
   git pull origin uat
   ```
2. **Reverta o merge da `GI-1010` na `uat`**  
   ```bash
   git revert -m 1 abc1234
   ```
3. **Resolva conflitos (se houver), confirme e envie para o repositório**  
   ```bash
   git add .
   git commit -m "Revertendo GI-1010 da uat"
   git push origin uat
   ```

Agora a `GI-1010` foi completamente removida da `uat` **sem afetar outras features**.

---

## 🚀 Benefícios desse Fluxo
✔ **A `dev` contém todas as features para testes simultâneos**.  
✔ **As releases são criadas apenas com as features aprovadas**.  
✔ **PRs garantem rastreabilidade e organização**.  
✔ **A `main` só recebe código revisado e aprovado**.  
✔ **Nenhuma feature será perdida ou atrasada**.  
✔ **Features que não forem aprovadas podem ser removidas sem quebrar o código**.  

📌 **Esse fluxo é seguro e organizado para equipes que precisam de controle total no deploy!** 🔥  
