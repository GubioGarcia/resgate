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