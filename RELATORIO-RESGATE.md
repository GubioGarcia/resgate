# Relatório de Resgate
- Equipe: Gubio Garcia e Luiz Fernando
- Branch de trabalho: `resgate/equipe-GubioLuiz`

## Diagnóstico
Descreva os problemas encontrados e suas evidências.

O projeto foi recebido na branch `release-dev`, com 7 branches locais (`main`, `backup-antigo`, `release-dev`, `docs-readme`, `feature-cadastro`, `hotfix-login`, `teste-login-descartavel`) e as tags `v1.0.0-funcional` e `commit-perigoso`. O commit `356ec6f` (tag `v1.0.0-funcional`, apontado por `main` e `backup-antigo`) é vazio; o código funcional foi criado em `4574eee`.

| Porta | Problema | Evidência | Causa (commit / autor) |
|---|---|---|---|
| 1 – Compilação | `mvn clean package` falhava com 7 erros em `EntregaService.java` (linhas 6, 14–17, 19 e 20) | `git diff v1.0.0-funcional -- EntregaService.java` | `a70ee84` – Henrique Nunes: removeu `endereco` do construtor e trocou `salvar` por `gravar` (linhas 19–20) |
| 2 – Arquivo excluído | `util/Validador.java` foi apagado, mas era usado por `EntregaService` (linhas 6 e 14–17) | `git log --diff-filter=D --summary` | `64f88f6` – Igor Reis: "remover classe aparentemente sem uso" |
| 3 – Login | `admin` com qualquer senha era autenticado | `git diff v1.0.0-funcional -- LoginService.java`; teste `admin / 000` autorizado | `9a6d3b0` – Felipe Rocha: trocou `&&` por `\|\|` e `SENHA.equals(senha)` por `senha == SENHA` |
| 4 – README | README reduzido a "Pergunte ao desenvolvedor como executar" | `git show 0cd80f6` | `0cd80f6` – Gustavo Melo: removeu 18 linhas |
| 5 – Segurança | `config/application.properties` com usuário e senha de banco e token de API versionados | `git show commit-perigoso` | `6572d8a` (tag `commit-perigoso`) – Felipe Rocha |

Branches analisadas:
- `docs-readme` (`7fe8faa`, Carla Souza): README original + seção "Fluxo recomendado". Usada para recuperar o README.
- `hotfix-login` (`a6f356d`, Eva Martins): contém apenas `NOTA-HOTFIX.txt`, sem código. Usada como pista para a correção do login; não integrada.
- `feature-cadastro` (`2624e5c`, Diego Alves): adiciona validação de `status`. Funcionalidade não relacionada ao resgate; não integrada.
- `teste-login-descartavel`: aponta para `9a6d3b0`, o commit que quebrou o login. Não é uma versão útil.

### Auditoria de segurança
O arquivo `config/application.properties` continha credenciais de banco de dados (`db.user`, `db.password`) e um token de API (`api.token`), adicionados em `6572d8a` por Felipe Rocha em 03/09/2026. O arquivo foi removido da versão atual e incluído no `.gitignore`.

Apagar o arquivo não elimina o segredo do histórico: cada commit é um snapshot imutável, e `6572d8a` e os commits seguintes continuam guardando o conteúdo, que pode ser recuperado com `git show commit-perigoso:config/application.properties`. Por isso o segredo deve ser considerado vazado: a medida principal é **trocar a senha do banco e revogar o token**. Reescrever o histórico (`git filter-repo`/BFG, remover a tag e exigir novo clone) não foi feito, pois a atividade exige preservar as evidências e cópias já distribuídas continuariam com o segredo.

## Comandos Git utilizados
Liste os comandos e explique a finalidade de cada um.

| Comando | Finalidade |
|---|---|
| `git status` | Verificar branch atual e arquivos alterados/não rastreados |
| `git branch -a` | Listar as branches existentes |
| `git checkout -b resgate/equipe-GubioLuiz` | Criar a branch de resgate e mudar para ela (equivalente a `git switch -c`) |
| `git diff --stat` / `git diff <arquivo>` | Analisar alterações não commitadas encontradas no início |
| `git config user.name` / `user.email` | Configurar a identidade dos integrantes no repositório (estava pré-definida como "Laboratorio Git") |
| `git log --oneline --all --graph --decorate` | Visualizar o grafo de commits, branches e tags |
| `git log --stat` | Ver os arquivos alterados em cada commit |
| `git show <commit/branch>` | Inspecionar o conteúdo de um commit ou branch (`356ec6f`, `hotfix-login`, `feature-cadastro`, `docs-readme`, `a70ee84`, `64f88f6`, `9a6d3b0`, `0cd80f6`, `commit-perigoso`) |
| `git log -- <arquivo>` | Listar os commits que alteraram um arquivo específico |
| `git diff v1.0.0-funcional -- <arquivo>` | Comparar o arquivo atual com a versão funcional |
| `git log --diff-filter=D --summary` | Encontrar commits que excluíram arquivos |
| `git log --all -- src/main/java` | Ver o histórico do código-fonte em todas as branches |
| `git restore --source=<ref> -- <arquivo>` | Recuperar um arquivo a partir de uma versão anterior (`v1.0.0-funcional`, `64f88f6~1`, `docs-readme`) |
| `git show docs-readme:README.md` | Ler o README de outra branch sem trocar de branch |
| `git diff v1.0.0-funcional docs-readme -- README.md` | Confirmar que a `docs-readme` só acrescenta conteúdo ao README original |
| `git log --all --oneline -- config/application.properties` | Localizar o commit que versionou as credenciais |
| `git log --all --oneline -S "<texto>"` | Procurar em todo o histórico commits que adicionaram/removeram o segredo |
| `git rm config/application.properties` | Remover o arquivo da versão atual |
| `git ls-files config` | Confirmar que o arquivo não é mais rastreado |
| `git check-ignore -v config/application.properties` | Confirmar que o `.gitignore` bloqueia o arquivo |
| `git grep -n "<texto>"` | Confirmar que o segredo não existe em nenhum arquivo da versão atual |
| `git add` / `git commit` | Registrar cada correção em commit próprio |
| `git switch main` / `git merge --no-ff` | Integrar a branch de resgate na `main` preservando o histórico |
| `mvn clean package` | Compilar o projeto |
| `java -cp target/classes br.edu.entregas.Main` | Executar o sistema para validação |

## Commits relevantes
Informe os hashes investigados, revertidos ou recuperados.

**Commits investigados (causas dos problemas):**

| Hash | Autor | Mensagem | Problema |
|---|---|---|---|
| `9a6d3b0` | Felipe Rocha | fix: liberar login para homologacao | Login aceitava qualquer senha |
| `6572d8a` | Felipe Rocha | chore: salvar credenciais para facilitar testes | Credenciais versionadas |
| `0cd80f6` | Gustavo Melo | docs: simplificar readme | README esvaziado |
| `a70ee84` | Henrique Nunes | feat: acelerar cadastro de mercadorias | Quebra de compilação e cadastro sem endereço |
| `64f88f6` | Igor Reis | refactor: remover classe aparentemente sem uso | Exclusão do `Validador.java` |

**Commits de correção da equipe:**

| Hash | Autor | Mensagem |
|---|---|---|
| `afdc7e1` | Laboratorio Git | docs: registrar equipe e branch de resgate |
| `e687e29` | Gubio Garcia | fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84) |
| `894024b` | Gubio Garcia | fix: recuperar Validador.java excluido indevidamente em 64f88f6 |
| `04c6440` | Gubio Garcia | fix: restaurar validacao do login (regressao introduzida em 9a6d3b0) |
| `5e59f97` | Luiz Fernando | docs: recuperar README completo da branch docs-readme (conteudo removido em 0cd80f6) |
| `008f92c` | Luiz Fernando | security: remover credenciais versionadas em 6572d8a e ignorar config/application.properties |

## Validação final
Registre como a equipe confirmou compilação, login, cadastro, README e segurança.

| Critério | Como foi validado | Resultado |
|---|---|---|
| Compilação | `mvn clean package` | BUILD SUCCESS (7 arquivos compilados) |
| Login correto | `admin` / `12345678` | Acesso autorizado |
| Usuário incorreto | `teste` / `12345678` | Acesso negado |
| Senha incorreta | `admin` / `00000000` | Acesso negado |
| Cadastro com endereço | Saída do login correto | `#1 \| Notebook \| ... \| Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000` |
| README | `Get-Content -Encoding UTF8 README.md` | Contém requisitos, compilação, execução e credenciais de demonstração |
| Segurança | `git ls-files config`, `Test-Path`, `git grep` | Arquivo não rastreado, inexistente no disco e segredo ausente da versão atual |

Após o merge na `main`, a compilação será executada novamente para confirmar a integração.

## Observações
\* O commit `afdc7e1` foi registrado com o autor "Laboratorio Git" porque o repositório já vinha com essa identidade configurada em `.git/config`. O problema foi identificado e corrigido com `git config user.name/user.email` antes dos commits seguintes. O histórico não foi reescrito para preservar a rastreabilidade.

No início, os arquivos `DESAFIO.md` e `RELATORIO-RESGATE.md` tinham alterações locais não commitadas (padrão de branch trocado para `resgate/discente-NOME` e texto alterado). Elas foram analisadas com `git diff` e descartadas com `git restore`, por não fazerem parte do projeto. Arquivos de IDE (`.classpath`, `.project`, `.settings/`) foram incluídos no `.gitignore`.

## Evidências por etapa

### Etapa 1: preparar o ambiente e criar a branch de resgate

```text
> git status
On branch release-dev
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   DESAFIO.md
        modified:   RELATORIO-RESGATE.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        .classpath
        .project
        .settings/

no changes added to commit (use "git add" and/or "git commit -a")
```

```text
> git branch -a
  backup-antigo
  docs-readme
  feature-cadastro
  hotfix-login
  main
* release-dev
  teste-login-descartavel
```

```text
> git checkout -b resgate/equipe-GubioLuiz
Switched to a new branch 'resgate/equipe-GubioLuiz'
```

Análise das alterações locais não commitadas encontradas no início:

```diff
> git diff --stat
 DESAFIO.md           | 2 +-
 RELATORIO-RESGATE.md | 2 +-
 2 files changed, 2 insertions(+), 2 deletions(-)

> git diff DESAFIO.md
diff --git a/DESAFIO.md b/DESAFIO.md
index 023ce88..51ae876 100644
--- a/DESAFIO.md
+++ b/DESAFIO.md
@@ -5,7 +5,7 @@ A entrega ao cliente acontece em breve. O sistema não compila, o login apresent
 1. Não apague a pasta `.git`.
 2. Não copie um projeto novo por cima deste.
 3. Toda correção deve ser identificável no histórico.
-4. Trabalhe em uma nova branch com o padrão `resgate/equipe-NOME`.
+4. Trabalhe em uma nova branch com o padrão `resgate/discente-NOME`.
 5. Ao final, faça merge da branch de resgate em `main`.
 6. Registre no `RELATORIO-RESGATE.md` os comandos usados e as evidências.
 ## Portas da sala

> git diff RELATORIO-RESGATE.md
diff --git a/RELATORIO-RESGATE.md b/RELATORIO-RESGATE.md
index 51a6ce4..feeaf41 100644
--- a/RELATORIO-RESGATE.md
+++ b/RELATORIO-RESGATE.md
@@ -8,4 +8,4 @@ Liste os comandos e explique a finalidade de cada um.
 ## Commits relevantes
 Informe os hashes investigados, revertidos ou recuperados.
 ## Validação final
-Registre como a equipe confirmou compilação, login, cadastro, README e segurança.
+Registre como a você confirmou compilação, login, cadastro, README e segurança.
```

```text
> git add RELATORIO-RESGATE.md
> git commit -m "docs: registrar equipe e branch de resgate"
[resgate/equipe-GubioLuiz afdc7e1] docs: registrar equipe e branch de resgate
 1 file changed, 3 insertions(+), 3 deletions(-)
```

**Evidências:**
- **Não trabalhar diretamente na main:** a branch `resgate/equipe-GubioLuiz` foi criada a partir da `release-dev` com `git checkout -b`, que equivale a `git switch -c`. Todas as correções foram feitas nela. A `main` recebeu apenas o merge final.
- **Nome da branch registrado no relatório:** o registro está no cabeçalho deste arquivo, incluído no commit `afdc7e1`.

### Etapa 2: investigar o repositório

```text
> git log --oneline --all --graph --decorate
* 799c5ec (HEAD -> resgate/equipe-GubioLuiz, release-dev) chore: adicionar instrucoes do resgate
* 64f88f6 refactor: remover classe aparentemente sem uso
* a70ee84 feat: acelerar cadastro de mercadorias
* 0cd80f6 docs: simplificar readme
* 6572d8a (tag: commit-perigoso) chore: salvar credenciais para facilitar testes
* 9a6d3b0 (teste-login-descartavel) fix: liberar login para homologacao
| * a6f356d (hotfix-login) fix: documentar correção validada para autenticação
|/
| * 2624e5c (feature-cadastro) feat: validar status da mercadoria
|/
| * 7fe8faa (docs-readme) docs: detalhar fluxo de contribuição
|/
* 356ec6f (tag: v1.0.0-funcional, main, backup-antigo) feat: implementar sistema de entregas funcional
* 4574eee chore: criar estrutura inicial do projeto
```

```text
> git log --stat
commit 799c5ec2d8e50e7fca0d5e20277f76f10cd4f2f6 (HEAD -> resgate/equipe-GubioLuiz, release-dev)
Author: Coordenacao <professor@entregas.local>
Date:   Thu Sep 3 10:30:00 2026 -0300

    chore: adicionar instrucoes do resgate

 DESAFIO.md           | 26 ++++++++++++++++++++++++++
 RELATORIO-RESGATE.md | 11 +++++++++++
 executar-linux.sh    |  4 ++++
 executar-windows.bat |  4 ++++
 4 files changed, 45 insertions(+)

commit 64f88f6f058bf6279244314b3b444d1c0fef0d23
Author: Igor Reis <igor@entregas.local>
Date:   Thu Sep 3 10:00:00 2026 -0300

    refactor: remover classe aparentemente sem uso

 src/main/java/br/edu/entregas/util/Validador.java | 15 ---------------
 1 file changed, 15 deletions(-)

commit a70ee842c7c426841d0cc618bddcd0bdf0fad654
Author: Henrique Nunes <henrique@entregas.local>
Date:   Thu Sep 3 09:30:00 2026 -0300

    feat: acelerar cadastro de mercadorias

 src/main/java/br/edu/entregas/service/EntregaService.java | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

commit 0cd80f6ca762d65a328cf3b2d4d1d0b06bbbad2e
Author: Gustavo Melo <gustavo@entregas.local>
Date:   Thu Sep 3 09:00:00 2026 -0300

    docs: simplificar readme

 README.md | 19 +------------------
 1 file changed, 1 insertion(+), 18 deletions(-)

commit 6572d8ada913855f0dbd9e7a3f15c6824437cf17 (tag: commit-perigoso)
Author: Felipe Rocha <felipe@entregas.local>
Date:   Thu Sep 3 08:30:00 2026 -0300

    chore: salvar credenciais para facilitar testes

 config/application.properties | 4 ++++
 1 file changed, 4 insertions(+)

commit 9a6d3b03419d2829036bfee88e87066c79b5bb73 (teste-login-descartavel)
Author: Felipe Rocha <felipe@entregas.local>
Date:   Thu Sep 3 08:00:00 2026 -0300

    fix: liberar login para homologacao

 src/main/java/br/edu/entregas/service/LoginService.java | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

commit 356ec6f74d73bc402e453fdc3fe844d1952a1113 (tag: v1.0.0-funcional, main, backup-antigo)
Author: Bruno Costa <bruno@entregas.local>
Date:   Tue Sep 1 10:00:00 2026 -0300

    feat: implementar sistema de entregas funcional

commit 4574eeefa3cd107eaa571809a68c6f5cdf899827
Author: Ana Lima <ana@entregas.local>
Date:   Tue Sep 1 09:00:00 2026 -0300

    chore: criar estrutura inicial do projeto

 .gitignore                                         |  4 +++
 README.md                                          | 20 +++++++++++++
 pom.xml                                            | 12 ++++++++
 src/main/java/br/edu/entregas/Main.java            | 30 ++++++++++++++++++++
 src/main/java/br/edu/entregas/model/Endereco.java  | 11 ++++++++
 .../java/br/edu/entregas/model/Mercadoria.java     | 33 ++++++++++++++++++++++
 .../entregas/repository/MercadoriaRepository.java  | 12 ++++++++
 .../br/edu/entregas/service/EntregaService.java    | 25 ++++++++++++++++
 .../java/br/edu/entregas/service/LoginService.java | 10 +++++++
 src/main/java/br/edu/entregas/util/Validador.java  | 15 ++++++++++
 10 files changed, 172 insertions(+)
```

```text
> git branch -a
  backup-antigo
  docs-readme
  feature-cadastro
  hotfix-login
  main
  release-dev
* resgate/equipe-GubioLuiz
  teste-login-descartavel
```

Inspeção das branches que poderiam guardar versões úteis:

```text
> git show --stat 356ec6f
commit 356ec6f74d73bc402e453fdc3fe844d1952a1113 (tag: v1.0.0-funcional, main, backup-antigo)
Author: Bruno Costa <bruno@entregas.local>
Date:   Tue Sep 1 10:00:00 2026 -0300

    feat: implementar sistema de entregas funcional
```

```diff
> git show hotfix-login
commit a6f356d5de91f5fdaa273d8d20954a340fbac485 (hotfix-login)
Author: Eva Martins <eva@entregas.local>
Date:   Wed Sep 2 11:00:00 2026 -0300

    fix: documentar correção validada para autenticação

diff --git a/NOTA-HOTFIX.txt b/NOTA-HOTFIX.txt
new file mode 100644
index 0000000..2f72ecb
--- /dev/null
+++ b/NOTA-HOTFIX.txt
@@ -0,0 +1 @@
+Hotfix validado: a comparação da senha deve usar equals e exigir usuário e senha corretos.
```

```diff
> git show feature-cadastro
commit 2624e5c25bafd7a776d58f54016426556d408b3a (feature-cadastro)
Author: Diego Alves <diego@entregas.local>
Date:   Wed Sep 2 10:00:00 2026 -0300

    feat: validar status da mercadoria

diff --git a/src/main/java/br/edu/entregas/service/EntregaService.java b/src/main/java/br/edu/entregas/service/EntregaService.java
index 2d44d95..a9c0e2b 100644
--- a/src/main/java/br/edu/entregas/service/EntregaService.java
+++ b/src/main/java/br/edu/entregas/service/EntregaService.java
@@ -13,6 +13,7 @@ public class EntregaService {
                                 double valor, String status, Endereco endereco) {
         Validador.textoObrigatorio(nome, "Nome");
         Validador.textoObrigatorio(descricao, "Descrição");
+        Validador.textoObrigatorio(status, "Status");
         Validador.numeroPositivo(peso, "Peso");
         Validador.numeroPositivo(valor, "Valor");
         if (endereco == null) throw new IllegalArgumentException("Endereço é obrigatório.");
```

```text
> git show --stat docs-readme
commit 7fe8faaaa09c783daf5d44aeb9ff07b85e3a9cc0 (docs-readme)
Author: Carla Souza <carla@entregas.local>
Date:   Wed Sep 2 09:00:00 2026 -0300

    docs: detalhar fluxo de contribuição

 README.md | 3 +++
 1 file changed, 3 insertions(+)
```

**Evidências:**
- **Quantas branches existem?** Existem 7 branches locais: `main`, `backup-antigo`, `release-dev`, `docs-readme`, `feature-cadastro`, `hotfix-login` e `teste-login-descartavel`. O repositório também tem as tags `v1.0.0-funcional` e `commit-perigoso`. Com a branch de resgate, passaram a ser 8.
- **Branch inicialmente ativa:** `release-dev`, no commit `799c5ec`.
- **Commits relacionados aos problemas:**
  - `a70ee84` (Henrique Nunes): removeu `endereco` do construtor e trocou `salvar` por `gravar` no `EntregaService`. Afetou a compilação e o cadastro.
  - `64f88f6` (Igor Reis): excluiu o `Validador.java`. Afetou a compilação.
  - `9a6d3b0` (Felipe Rocha): alterou a lógica do `LoginService`. Afetou o login.
  - `6572d8a` (Felipe Rocha, tag `commit-perigoso`): versionou credenciais. Afetou a segurança.
  - `0cd80f6` (Gustavo Melo): removeu 18 linhas do README. Afetou a documentação.
- **Branches com versões úteis:**
  - `main`, `backup-antigo` e a tag `v1.0.0-funcional` apontam para `356ec6f`, que é a versão funcional. Esse commit é vazio, porque o código foi criado em `4574eee`.
  - `docs-readme` guarda o README completo.
  - `hotfix-login` serviu de pista para corrigir o login.

### Etapa 3: descobrir por que o projeto não compila

```text
> mvn clean package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< br.edu.entregas:projeto-entregas >------------------
[INFO] Building projeto-entregas 1.0.0
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[... downloads de plugins do Maven omitidos ...]
[INFO] --- compiler:3.13.0:compile (default-compile) @ projeto-entregas ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 6 source files with javac [debug target 17] to target\classes
[INFO] -------------------------------------------------------------
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[6,28] package br.edu.entregas.util does not exist
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[14,9] cannot find symbol
  symbol:   variable Validador
  location: class br.edu.entregas.service.EntregaService
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[15,9] cannot find symbol
  symbol:   variable Validador
  location: class br.edu.entregas.service.EntregaService
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[16,9] cannot find symbol
  symbol:   variable Validador
  location: class br.edu.entregas.service.EntregaService
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[17,9] cannot find symbol
  symbol:   variable Validador
  location: class br.edu.entregas.service.EntregaService
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[19,33] constructor Mercadoria in class br.edu.entregas.model.Mercadoria cannot be applied to given types;
  required: long,java.lang.String,java.lang.String,double,double,java.lang.String,br.edu.entregas.model.Endereco
  found:    long,java.lang.String,java.lang.String,double,double,java.lang.String
  reason: actual and formal argument lists differ in length
[ERROR] .../src/main/java/br/edu/entregas/service/EntregaService.java:[20,19] cannot find symbol
  symbol:   method gravar(br.edu.entregas.model.Mercadoria)
  location: variable repository of type br.edu.entregas.repository.MercadoriaRepository
[INFO] 7 errors
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  10.474 s
[INFO] Finished at: 2026-10-04T16:23:05-03:00
[INFO] ------------------------------------------------------------------------
```

```text
> git log -- src/main/java/br/edu/entregas/service/EntregaService.java
commit a70ee842c7c426841d0cc618bddcd0bdf0fad654
Author: Henrique Nunes <henrique@entregas.local>
Date:   Thu Sep 3 09:30:00 2026 -0300

    feat: acelerar cadastro de mercadorias

commit 4574eeefa3cd107eaa571809a68c6f5cdf899827
Author: Ana Lima <ana@entregas.local>
Date:   Tue Sep 1 09:00:00 2026 -0300

    chore: criar estrutura inicial do projeto
```

```diff
> git diff v1.0.0-funcional -- src/main/java/br/edu/entregas/service/EntregaService.java
diff --git a/src/main/java/br/edu/entregas/service/EntregaService.java b/src/main/java/br/edu/entregas/service/EntregaService.java
index 2d44d95..a5e170a 100644
--- a/src/main/java/br/edu/entregas/service/EntregaService.java
+++ b/src/main/java/br/edu/entregas/service/EntregaService.java
@@ -16,8 +16,8 @@ public class EntregaService {
         Validador.numeroPositivo(peso, "Peso");
         Validador.numeroPositivo(valor, "Valor");
         if (endereco == null) throw new IllegalArgumentException("Endereço é obrigatório.");
-        Mercadoria mercadoria = new Mercadoria(id, nome, descricao, peso, valor, status, endereco);
-        repository.salvar(mercadoria);
+        Mercadoria mercadoria = new Mercadoria(id, nome, descricao, peso, valor, status);
+        repository.gravar(mercadoria);
         return mercadoria;
     }
```

Correção:

```diff
> git restore --source=v1.0.0-funcional -- src/main/java/br/edu/entregas/service/EntregaService.java
> git diff
diff --git a/src/main/java/br/edu/entregas/service/EntregaService.java b/src/main/java/br/edu/entregas/service/EntregaService.java
index a5e170a..2d44d95 100644
--- a/src/main/java/br/edu/entregas/service/EntregaService.java
+++ b/src/main/java/br/edu/entregas/service/EntregaService.java
@@ -16,8 +16,8 @@ public class EntregaService {
         Validador.numeroPositivo(peso, "Peso");
         Validador.numeroPositivo(valor, "Valor");
         if (endereco == null) throw new IllegalArgumentException("Endereço é obrigatório.");
-        Mercadoria mercadoria = new Mercadoria(id, nome, descricao, peso, valor, status);
-        repository.gravar(mercadoria);
+        Mercadoria mercadoria = new Mercadoria(id, nome, descricao, peso, valor, status, endereco);
+        repository.salvar(mercadoria);
         return mercadoria;
     }
```

```text
> git add src/main/java/br/edu/entregas/service/EntregaService.java
> git commit -m "fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84)"
[resgate/equipe-GubioLuiz e687e29] fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84)
 1 file changed, 2 insertions(+), 2 deletions(-)
```

```text
> mvn clean package
[...]
[ERROR] COMPILATION ERROR :
[ERROR] .../EntregaService.java:[6,28] package br.edu.entregas.util does not exist
[ERROR] .../EntregaService.java:[14,9] cannot find symbol
[ERROR] .../EntregaService.java:[15,9] cannot find symbol
[ERROR] .../EntregaService.java:[16,9] cannot find symbol
[ERROR] .../EntregaService.java:[17,9] cannot find symbol
[INFO] 5 errors
[INFO] BUILD FAILURE
[INFO] Finished at: 2026-10-04T17:28:22-03:00
```

**Evidências:**
- **Erro encontrado:** 7 erros de compilação em `EntregaService.java`:
  - linha 6: o pacote `br.edu.entregas.util` não existe;
  - linhas 14 a 17: o símbolo `Validador` não é encontrado;
  - linha 19: o construtor de `Mercadoria` é chamado sem o parâmetro `Endereco`;
  - linha 20: o método `gravar()` não existe em `MercadoriaRepository`, que tem `salvar()`.
- **Commits relacionados:**
  - `a70ee84` (Henrique Nunes, 03/09/2026 09:30) causou os erros das linhas 19 e 20. É o único commit que alterou o `EntregaService` depois da versão inicial.
  - `64f88f6` (Igor Reis) causou os erros das linhas 6 e 14 a 17, tratados na Etapa 4.
- **Correção:** o arquivo foi restaurado da tag `v1.0.0-funcional` no commit `e687e29`. Depois disso, o build caiu de 7 para 5 erros. Restaram apenas os erros ligados ao `Validador`.

### Etapa 4: recuperar o arquivo excluído

```text
> git log --diff-filter=D --summary
commit 64f88f6f058bf6279244314b3b444d1c0fef0d23
Author: Igor Reis <igor@entregas.local>
Date:   Thu Sep 3 10:00:00 2026 -0300

    refactor: remover classe aparentemente sem uso

 delete mode 100644 src/main/java/br/edu/entregas/util/Validador.java
```

```text
> git log --all -- src/main/java
commit e687e29ff979cf8cf1e7b1c84016d73aeb309515 (HEAD -> resgate/equipe-GubioLuiz)
Author: Gubio Garcia <gubiogarcia.dev@gmail.com>
Date:   Sun Oct 4 17:27:39 2026 -0300

    fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84)

commit 64f88f6f058bf6279244314b3b444d1c0fef0d23
Author: Igor Reis <igor@entregas.local>
Date:   Thu Sep 3 10:00:00 2026 -0300

    refactor: remover classe aparentemente sem uso

commit a70ee842c7c426841d0cc618bddcd0bdf0fad654
Author: Henrique Nunes <henrique@entregas.local>
Date:   Thu Sep 3 09:30:00 2026 -0300

    feat: acelerar cadastro de mercadorias

commit 9a6d3b03419d2829036bfee88e87066c79b5bb73 (teste-login-descartavel)
Author: Felipe Rocha <felipe@entregas.local>
Date:   Thu Sep 3 08:00:00 2026 -0300

    fix: liberar login para homologacao

commit 2624e5c25bafd7a776d58f54016426556d408b3a (feature-cadastro)
Author: Diego Alves <diego@entregas.local>
Date:   Wed Sep 2 10:00:00 2026 -0300

    feat: validar status da mercadoria

commit 4574eeefa3cd107eaa571809a68c6f5cdf899827
Author: Ana Lima <ana@entregas.local>
Date:   Tue Sep 1 09:00:00 2026 -0300

    chore: criar estrutura inicial do projeto
```

```diff
> git show 64f88f6
commit 64f88f6f058bf6279244314b3b444d1c0fef0d23
Author: Igor Reis <igor@entregas.local>
Date:   Thu Sep 3 10:00:00 2026 -0300

    refactor: remover classe aparentemente sem uso

diff --git a/src/main/java/br/edu/entregas/util/Validador.java b/src/main/java/br/edu/entregas/util/Validador.java
deleted file mode 100644
index ee29b18..0000000
--- a/src/main/java/br/edu/entregas/util/Validador.java
+++ /dev/null
@@ -1,15 +0,0 @@
-package br.edu.entregas.util;
-
-public final class Validador {
-    private Validador() {}
-
-    public static void textoObrigatorio(String valor, String campo) {
-        if (valor == null || valor.isBlank()) {
-            throw new IllegalArgumentException(campo + " é obrigatório.");
-        }
-    }
-
-    public static void numeroPositivo(double valor, String campo) {
-        if (valor <= 0) throw new IllegalArgumentException(campo + " deve ser positivo.");
-    }
-}
```

```text
> git restore --source=64f88f6~1 -- src/main/java/br/edu/entregas/util/Validador.java
> git add src/main/java/br/edu/entregas/util/Validador.java
> git status
On branch resgate/equipe-GubioLuiz
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   src/main/java/br/edu/entregas/util/Validador.java

> git commit -m "fix: recuperar Validador.java excluido indevidamente em 64f88f6"
[resgate/equipe-GubioLuiz 894024b] fix: recuperar Validador.java excluido indevidamente em 64f88f6
 1 file changed, 15 insertions(+)
 create mode 100644 src/main/java/br/edu/entregas/util/Validador.java
```

```text
> mvn clean package
[...]
[INFO] --- compiler:3.13.0:compile (default-compile) @ projeto-entregas ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 7 source files with javac [debug target 17] to target\classes
[...]
[INFO] Building jar: ...\Projeto-Entregas-Escape-Room\target\projeto-entregas-1.0.0.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.731 s
[INFO] Finished at: 2026-10-04T17:39:36-03:00
[INFO] ------------------------------------------------------------------------
```

**Evidências:**
- **Arquivo excluído:** `src/main/java/br/edu/entregas/util/Validador.java`.
- **Hash e autor:** commit `64f88f6`, de Igor Reis (igor@entregas.local), em 03/09/2026 às 10:00, com a mensagem "refactor: remover classe aparentemente sem uso". A classe foi removida como "sem uso", mas era usada pelo `EntregaService`. A refatoração foi feita sem compilar.
- **Recuperação:** o arquivo voltou com `git restore --source=64f88f6~1`, onde `~1` indica o commit anterior à exclusão. A correção foi registrada no commit `894024b`.
- **Resultado:** o build terminou com BUILD SUCCESS e a Porta 1 (compilação) foi resolvida.

### Etapa 5: corrigir o login

```text
> git log -- src/main/java/br/edu/entregas/service/LoginService.java
commit 9a6d3b03419d2829036bfee88e87066c79b5bb73 (teste-login-descartavel)
Author: Felipe Rocha <felipe@entregas.local>
Date:   Thu Sep 3 08:00:00 2026 -0300

    fix: liberar login para homologacao

commit 4574eeefa3cd107eaa571809a68c6f5cdf899827
Author: Ana Lima <ana@entregas.local>
Date:   Tue Sep 1 09:00:00 2026 -0300

    chore: criar estrutura inicial do projeto
```

```diff
> git diff v1.0.0-funcional -- src/main/java/br/edu/entregas/service/LoginService.java
diff --git a/src/main/java/br/edu/entregas/service/LoginService.java b/src/main/java/br/edu/entregas/service/LoginService.java
index 7b080dc..77144d0 100644
--- a/src/main/java/br/edu/entregas/service/LoginService.java
+++ b/src/main/java/br/edu/entregas/service/LoginService.java
@@ -5,6 +5,6 @@ public class LoginService {
     private static final String SENHA = "12345678";

     public boolean autenticar(String usuario, String senha) {
-        return USUARIO.equals(usuario) && SENHA.equals(senha);
+        return USUARIO.equals(usuario) || senha == SENHA; // alteração urgente
     }
 }
```

```text
> git show hotfix-login:NOTA-HOTFIX.txt
Hotfix validado: a comparação da senha deve usar equals e exigir usuário e senha corretos.
```

Teste antes da correção, com senha inválida:

```text
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 000
Acesso autorizado. Mercadorias cadastradas:
#1 | Notebook | Notebook corporativo | 2.1 kg | R$ 4500,00 | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000
```

Correção:

```diff
> git restore --source=v1.0.0-funcional -- src/main/java/br/edu/entregas/service/LoginService.java
> git diff
diff --git a/src/main/java/br/edu/entregas/service/LoginService.java b/src/main/java/br/edu/entregas/service/LoginService.java
index 77144d0..7b080dc 100644
--- a/src/main/java/br/edu/entregas/service/LoginService.java
+++ b/src/main/java/br/edu/entregas/service/LoginService.java
@@ -5,6 +5,6 @@ public class LoginService {
     private static final String SENHA = "12345678";

     public boolean autenticar(String usuario, String senha) {
-        return USUARIO.equals(usuario) || senha == SENHA; // alteração urgente
+        return USUARIO.equals(usuario) && SENHA.equals(senha);
     }
 }
```

```text
> git add src/main/java/br/edu/entregas/service/LoginService.java
> git commit -m "fix: restaurar validacao do login (regressao introduzida em 9a6d3b0)"
[resgate/equipe-GubioLuiz 04c6440] fix: restaurar validacao do login (regressao introduzida em 9a6d3b0)
 1 file changed, 1 insertion(+), 1 deletion(-)

> mvn clean package
[...]
[INFO] BUILD SUCCESS
[INFO] Finished at: 2026-10-04T17:59:20-03:00
```

Testes após a correção:

```text
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 12345678
Acesso autorizado. Mercadorias cadastradas:
#1 | Notebook | Notebook corporativo | 2.1 kg | R$ 4500,00 | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000
```

```text
=== SISTEMA DE ENTREGAS ===
Usuário: teste
Senha: 12345678
Acesso negado.
```

```text
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 00000000
Acesso negado.
```

**Evidências:**
- **Commit causador:** `9a6d3b0`, de Felipe Rocha, em 03/09/2026 às 08:00. A branch `teste-login-descartavel` aponta para esse mesmo commit. Era um código temporário de teste que acabou entrando na release.
- **O que mudou:** o `&&` virou `||`, e `SENHA.equals(senha)` virou `senha == SENHA`. Com `||`, bastava acertar o usuário. Além disso, `==` compara referências, não o texto. Antes da correção, `admin / 000` era autorizado.
- **Testes após a correção:**

  | Usuário | Senha | Resultado |
  |---|---|---|
  | `admin` | `12345678` | Acesso autorizado |
  | `teste` | `12345678` | Acesso negado |
  | `admin` | `00000000` | Acesso negado |

- **Commit próprio:** `04c6440`. A nota da branch `hotfix-login` confirmava a correção esperada.

### Etapa 6: recuperar o README

```text
> git log --all --oneline -- README.md
0cd80f6 docs: simplificar readme
7fe8faa (docs-readme) docs: detalhar fluxo de contribuição
4574eee chore: criar estrutura inicial do projeto
```

````diff
> git show 0cd80f6
commit 0cd80f6ca762d65a328cf3b2d4d1d0b06bbbad2e
Author: Gustavo Melo <gustavo@entregas.local>
Date:   Thu Sep 3 09:00:00 2026 -0300

    docs: simplificar readme

diff --git a/README.md b/README.md
index 7ef8ed4..f8710a0 100644
--- a/README.md
+++ b/README.md
@@ -1,20 +1,3 @@
 # Projeto Entregas

-Sistema Java de console para controle de mercadorias e seus endereços de entrega.
-
-## Requisitos
-- Java 17 ou superior
-- Maven 3.8 ou superior
-
-## Compilar e executar
-```bash
-mvn clean package
-java -cp target/classes br.edu.entregas.Main
-```
-
-## Acesso de demonstração
-- Usuário: `admin`
-- Senha: `12345678`
-
-## Modelo
-Cada mercadoria possui exatamente um endereço de entrega.
+Sistema interno. Pergunte ao desenvolvedor como executar.
````

````text
> git show docs-readme:README.md
# Projeto Entregas

Sistema Java de console para controle de mercadorias e seus endereços de entrega.

## Requisitos
- Java 17 ou superior
- Maven 3.8 ou superior

## Compilar e executar
```bash
mvn clean package
java -cp target/classes br.edu.entregas.Main
```

## Acesso de demonstração
- Usuário: `admin`
- Senha: `12345678`

## Modelo
Cada mercadoria possui exatamente um endereço de entrega.

## Fluxo recomendado
Use branches de funcionalidade, Pull Request e revisão antes do merge.
````

```diff
> git diff v1.0.0-funcional docs-readme -- README.md
diff --git a/README.md b/README.md
index 7ef8ed4..7736a53 100644
--- a/README.md
+++ b/README.md
@@ -18,3 +18,6 @@ java -cp target/classes br.edu.entregas.Main

 ## Modelo
 Cada mercadoria possui exatamente um endereço de entrega.
+
+## Fluxo recomendado
+Use branches de funcionalidade, Pull Request e revisão antes do merge.
```

```text
> git restore --source=docs-readme -- README.md
> git add README.md
> git commit -m "docs: recuperar README completo da branch docs-readme (conteudo removido em 0cd80f6)"
[resgate/equipe-GubioLuiz 5e59f97] docs: recuperar README completo da branch docs-readme (conteudo removido em 0cd80f6)
 1 file changed, 21 insertions(+), 1 deletion(-)
```

**Evidências:**
- **Commit que prejudicou o README:** `0cd80f6`, de Gustavo Melo, em 03/09/2026 às 09:00. O commit trocou o conteúdo por "Sistema interno. Pergunte ao desenvolvedor como executar."
- **Fonte da recuperação:** a branch `docs-readme` (`7fe8faa`). Ela contém o README original mais a seção "Fluxo recomendado". O README não foi reescrito do zero.
- **Recuperação:** `git restore --source=docs-readme -- README.md`, registrada no commit `5e59f97`.
- **Conteúdo confirmado:** o README recuperado tem os requisitos (Java 17+ e Maven 3.8+), a compilação (`mvn clean package`), a execução (`java -cp target/classes br.edu.entregas.Main`) e as credenciais de demonstração (`admin` / `12345678`).

### Etapa 7: realizar a auditoria de segurança

```text
> git log --all --oneline -- config/application.properties
6572d8a (tag: commit-perigoso) chore: salvar credenciais para facilitar testes
```

```diff
> git show commit-perigoso
commit 6572d8ada913855f0dbd9e7a3f15c6824437cf17 (tag: commit-perigoso)
Author: Felipe Rocha <felipe@entregas.local>
Date:   Thu Sep 3 08:30:00 2026 -0300

    chore: salvar credenciais para facilitar testes

diff --git a/config/application.properties b/config/application.properties
new file mode 100644
index 0000000..5278381
--- /dev/null
+++ b/config/application.properties
@@ -0,0 +1,4 @@
+app.name=Projeto Entregas
+db.user=********
+db.password=********
+api.token=********
```

```text
> git log --all --oneline -S "<senha do banco>"
6572d8a (tag: commit-perigoso) chore: salvar credenciais para facilitar testes
```

```text
> git rm config/application.properties
rm 'config/application.properties'
> git add .gitignore
> git commit -m "security: remover credenciais versionadas em 6572d8a e ignorar config/application.properties"
[resgate/equipe-GubioLuiz 008f92c] security: remover credenciais versionadas em 6572d8a e ignorar config/application.properties
 2 files changed, 3 insertions(+), 4 deletions(-)
 delete mode 100644 config/application.properties

> git ls-files config

> git check-ignore -v config/application.properties
.gitignore:7:config/application.properties      config/application.properties
```

O segredo continua recuperável pelo histórico:

```text
> git show commit-perigoso:config/application.properties
app.name=Projeto Entregas
db.user=********
db.password=********
api.token=********
```

**Evidências:**
- **Ocorrência:** o arquivo `config/application.properties` continha o usuário e a senha do banco (`db.user`, `db.password`) e um token de API (`api.token`).
- **Commit e autor:** `6572d8a` (tag `commit-perigoso`), de Felipe Rocha, em 03/09/2026 às 08:30.
- **Remoção da versão atual:** o arquivo foi removido com `git rm` no commit `008f92c`. O comando `git ls-files config` não retorna nada.
- **`.gitignore` atualizado:** a regra está na linha 7. O comando `git check-ignore -v` confirma que ela está ativa.
- **Por que apagar o arquivo não elimina o segredo:** cada commit é um snapshot imutável. O `6572d8a` e os commits seguintes continuam guardando o conteúdo, e o `git show commit-perigoso:config/application.properties` ainda exibe os valores. Por isso o segredo deve ser tratado como vazado. A medida principal é trocar a senha do banco e revogar o token.

### Etapa 8: validar o sistema recuperado

```text
> git log --oneline -8
008f92c (HEAD -> resgate/equipe-GubioLuiz) security: remover credenciais versionadas em 6572d8a e ignorar config/application.properties
5e59f97 docs: recuperar README completo da branch docs-readme (conteudo removido em 0cd80f6)
04c6440 fix: restaurar validacao do login (regressao introduzida em 9a6d3b0)
894024b fix: recuperar Validador.java excluido indevidamente em 64f88f6
e687e29 fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84)
afdc7e1 docs: registrar equipe e branch de resgate
799c5ec (release-dev) chore: adicionar instrucoes do resgate
64f88f6 refactor: remover classe aparentemente sem uso
```

```text
> mvn clean package
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< br.edu.entregas:projeto-entregas >------------------
[INFO] Building projeto-entregas 1.0.0
[INFO]   from pom.xml
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- clean:3.2.0:clean (default-clean) @ projeto-entregas ---
[INFO]
[INFO] --- resources:3.3.1:resources (default-resources) @ projeto-entregas ---
[INFO]
[INFO] --- compiler:3.13.0:compile (default-compile) @ projeto-entregas ---
[INFO] Recompiling the module because of changed source code.
[INFO] Compiling 7 source files with javac [debug target 17] to target\classes
[WARNING] system modules path not set in conjunction with -source 17
[INFO]
[INFO] --- surefire:3.2.5:test (default-test) @ projeto-entregas ---
[INFO] No tests to run.
[INFO]
[INFO] --- jar:3.4.1:jar (default-jar) @ projeto-entregas ---
[INFO] Building jar: ...\Projeto-Entregas-Escape-Room\target\projeto-entregas-1.0.0.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.408 s
[INFO] Finished at: 2026-10-04T18:37:13-03:00
[INFO] ------------------------------------------------------------------------
```

```text
> java -cp target/classes br.edu.entregas.Main
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 12345678
Acesso autorizado. Mercadorias cadastradas:
#1 | Notebook | Notebook corporativo | 2.1 kg | R$ 4500,00 | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000

> java -cp target/classes br.edu.entregas.Main
=== SISTEMA DE ENTREGAS ===
Usuário: teste
Senha: 12345678
Acesso negado.

> java -cp target/classes br.edu.entregas.Main
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 00000000
Acesso negado.
```

````text
> Get-Content -Encoding UTF8 README.md
# Projeto Entregas

Sistema Java de console para controle de mercadorias e seus endereços de entrega.

## Requisitos
- Java 17 ou superior
- Maven 3.8 ou superior

## Compilar e executar
```bash
mvn clean package
java -cp target/classes br.edu.entregas.Main
```

## Acesso de demonstração
- Usuário: `admin`
- Senha: `12345678`

## Modelo
Cada mercadoria possui exatamente um endereço de entrega.

## Fluxo recomendado
Use branches de funcionalidade, Pull Request e revisão antes do merge.
````

```text
> git ls-files config

> Test-Path config\application.properties
False
> git grep -n "<senha do banco>"

> git grep -n "<token da API>"

```

Ajustes finais antes da integração:

```text
> git commit -m "chore: ignorar arquivos de configuracao de IDE"
[resgate/equipe-GubioLuiz 4dd9c45] chore: ignorar arquivos de configuracao de IDE
> git commit -m "docs: preencher relatorio de resgate com diagnostico, comandos e validacoes"
[resgate/equipe-GubioLuiz 657e922] docs: preencher relatorio de resgate com diagnostico, comandos e validacoes
```

**Evidências:**

| Critério | Validação | Resultado |
|---|---|---|
| Compilação sem erro | `mvn clean package` | BUILD SUCCESS (7 arquivos) |
| Login correto autorizado | `admin` / `12345678` | Acesso autorizado |
| Credenciais incorretas rejeitadas | `teste` / `12345678` e `admin` / `00000000` | Acesso negado |
| Mercadoria com endereço | Saída do login correto | `Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000` |
| README completo | `Get-Content README.md` | Requisitos, compilação, execução e credenciais |
| Credenciais fora da versão atual | `git ls-files`, `Test-Path`, `git grep` | Nenhuma ocorrência |

### Etapa 9: integrar a entrega na main

```text
> git status
On branch resgate/equipe-GubioLuiz
nothing to commit, working tree clean
```

```text
> git log --oneline
657e922 (HEAD -> resgate/equipe-GubioLuiz) docs: preencher relatorio de resgate com diagnostico, comandos e validacoes
4dd9c45 chore: ignorar arquivos de configuracao de IDE
008f92c security: remover credenciais versionadas em 6572d8a e ignorar config/application.properties
5e59f97 docs: recuperar README completo da branch docs-readme (conteudo removido em 0cd80f6)
04c6440 fix: restaurar validacao do login (regressao introduzida em 9a6d3b0)
894024b fix: recuperar Validador.java excluido indevidamente em 64f88f6
e687e29 fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84)
afdc7e1 docs: registrar equipe e branch de resgate
799c5ec (release-dev) chore: adicionar instrucoes do resgate
64f88f6 refactor: remover classe aparentemente sem uso
a70ee84 feat: acelerar cadastro de mercadorias
0cd80f6 docs: simplificar readme
6572d8a (tag: commit-perigoso) chore: salvar credenciais para facilitar testes
9a6d3b0 (teste-login-descartavel) fix: liberar login para homologacao
356ec6f (tag: v1.0.0-funcional, main, backup-antigo) feat: implementar sistema de entregas funcional
4574eee chore: criar estrutura inicial do projeto
```

```text
> git switch main
Switched to branch 'main'
> git merge --no-ff resgate/equipe-GubioLuiz -m "merge: integrar resgate do projeto"
Merge made by the 'ort' strategy.
 .gitignore           |   8 ++++
 DESAFIO.md           |  26 +++++++++++++
 README.md            |   3 ++
 RELATORIO-RESGATE.md | 102 +++++++++++++++++++++++++++++++++++++++++++++++++++
 executar-linux.sh    |   4 ++
 executar-windows.bat |   4 ++
 6 files changed, 147 insertions(+)
 create mode 100644 DESAFIO.md
 create mode 100644 RELATORIO-RESGATE.md
 create mode 100644 executar-linux.sh
 create mode 100644 executar-windows.bat
```

```text
> mvn clean package
[...]
[INFO] Compiling 7 source files with javac [debug target 17] to target\classes
[...]
[INFO] BUILD SUCCESS
[INFO] Total time:  2.021 s
[INFO] Finished at: 2026-10-04T19:08:52-03:00

> java -cp target/classes br.edu.entregas.Main
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 12345678
Acesso autorizado. Mercadorias cadastradas:
#1 | Notebook | Notebook corporativo | 2.1 kg | R$ 4500,00 | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000
```

```text
> git log --oneline --all --graph --decorate
*   a451d8b (HEAD -> main) merge: integrar resgate do projeto
|\
| * 657e922 (resgate/equipe-GubioLuiz) docs: preencher relatorio de resgate com diagnostico, comandos e validacoes
| * 4dd9c45 chore: ignorar arquivos de configuracao de IDE
| * 008f92c security: remover credenciais versionadas em 6572d8a e ignorar config/application.properties
| * 5e59f97 docs: recuperar README completo da branch docs-readme (conteudo removido em 0cd80f6)
| * 04c6440 fix: restaurar validacao do login (regressao introduzida em 9a6d3b0)
| * 894024b fix: recuperar Validador.java excluido indevidamente em 64f88f6
| * e687e29 fix: restaurar EntregaService da v1.0.0-funcional (regressao introduzida em a70ee84)
| * afdc7e1 docs: registrar equipe e branch de resgate
| * 799c5ec (release-dev) chore: adicionar instrucoes do resgate
| * 64f88f6 refactor: remover classe aparentemente sem uso
| * a70ee84 feat: acelerar cadastro de mercadorias
| * 0cd80f6 docs: simplificar readme
| * 6572d8a (tag: commit-perigoso) chore: salvar credenciais para facilitar testes
| * 9a6d3b0 (teste-login-descartavel) fix: liberar login para homologacao
|/
| * a6f356d (hotfix-login) fix: documentar correção validada para autenticação
|/
| * 2624e5c (feature-cadastro) feat: validar status da mercadoria
|/
| * 7fe8faa (docs-readme) docs: detalhar fluxo de contribuição
|/
* 356ec6f (tag: v1.0.0-funcional, backup-antigo) feat: implementar sistema de entregas funcional
* 4574eee chore: criar estrutura inicial do projeto
```

**Evidências:**
- **Compilação na main:** depois do merge, `mvn clean package` terminou com BUILD SUCCESS. A execução com `admin` / `12345678` exibiu a mercadoria com endereço.
- **Merge no histórico:** o commit de merge `a451d8b` une a branch de resgate à `main`. O `--no-ff` garante um commit de merge explícito.
- **Nada foi apagado:** nenhuma branch ou tag foi removida. Continuam no repositório: `main`, `release-dev`, `resgate/equipe-GubioLuiz`, `backup-antigo`, `docs-readme`, `feature-cadastro`, `hotfix-login`, `teste-login-descartavel`, `v1.0.0-funcional` e `commit-perigoso`.