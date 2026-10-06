# Projeto Entregas

Sistema Java de console para controle de mercadorias e seus endereços de entrega.

- **Versão atual:** 1.0.0
- **Tecnologias:** Java 17, Maven
- **Armazenamento:** em memória (os dados não são persistidos entre execuções)

## Sumário
1. [Requisitos e instalação](#1-requisitos-e-instalação)
2. [Configuração](#2-configuração)
3. [Como executar](#3-como-executar)
4. [Como usar](#4-como-usar)
5. [Arquitetura](#5-arquitetura)
6. [Interface interna (API)](#6-interface-interna-api)
7. [Manutenção e contribuição](#7-manutenção-e-contribuição)
8. [Problemas comuns](#8-problemas-comuns)
9. [Histórico de versões](#9-histórico-de-versões)

---

## 1. Requisitos e instalação

### Requisitos
| Ferramenta | Versão mínima | Como verificar |
|---|---|---|
| Java (JDK) | 17 | `java -version` |
| Maven | 3.8 | `mvn -v` |
| Git | qualquer versão recente | `git --version` |

O projeto não possui dependências externas: o `pom.xml` usa apenas a biblioteca padrão do Java.

### Instalação
1. Clone o repositório:
   ```bash
   git clone <https://github.com/GubioGarcia/resgate>
   cd Projeto-Entregas-Escape-Room
   ```
2. Confirme as versões do Java e do Maven com os comandos da tabela acima.
3. Compile o projeto:
   ```bash
   mvn clean package
   ```
   Ao final deve aparecer `BUILD SUCCESS` e as classes compiladas ficam em `target/classes`.

## 2. Configuração

| Item | Onde fica | Valor atual |
|---|---|---|
| Versão do Java (source/target) | `pom.xml` → `maven.compiler.source` / `target` | `17` |
| Codificação dos fontes | `pom.xml` → `project.build.sourceEncoding` | `UTF-8` |
| Versão do sistema | `pom.xml` → `version` | `1.0.0` |
| Usuário e senha de acesso | `LoginService.java` (constantes `USUARIO` e `SENHA`) | `admin` / `12345678` |

### Variáveis de ambiente
O sistema não lê variáveis de ambiente. Só é preciso que `java` e `mvn` estejam no `PATH`
(e, no caso do Maven, que `JAVA_HOME` aponte para um JDK 17 ou superior).

### Arquivos de configuração
| Arquivo | Versionado? | Conteúdo |
|---|---|---|
| `config/application.properties.example` | Sim | Modelo com as chaves, **sem valores** |
| `config/application.properties` | Não (`.gitignore`) | Valores reais, apenas na máquina local |

Para criar a configuração local, copie o modelo e preencha os valores:

```bash
# Linux/macOS
cp config/application.properties.example config/application.properties
```
```bat
:: Windows
copy config\application.properties.example config\application.properties
```

| Chave | Descrição |
|---|---|
| `db.user` | Usuário do banco de dados |
| `db.password` | Senha do banco de dados |
| `api.token` | Token de acesso à API externa |

> A versão atual do sistema ainda não lê esse arquivo; ele está preparado para futuras integrações.
> **Nunca** coloque valores reais no `.example`. Se uma nova chave for necessária, adicione-a
> nos dois arquivos (no `.example`, sem valor).

## 3. Como executar

**Com Maven (qualquer sistema):**
```bash
mvn clean package
java -cp target/classes br.edu.entregas.Main
```

**Com os scripts prontos** (compilam e executam em sequência):
- Windows: `executar-windows.bat`
- Linux/macOS: `./executar-linux.sh` (se necessário, antes: `chmod +x executar-linux.sh`)

## 4. Como usar

Ao iniciar, o sistema pede usuário e senha.

**Acesso de demonstração**
- Usuário: `admin`
- Senha: `12345678`

**Login válido:** o sistema cadastra uma mercadoria de exemplo e lista as mercadorias cadastradas.
```
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 12345678
Acesso autorizado. Mercadorias cadastradas:
#1 | Notebook | Notebook corporativo | 2.1 kg | R$ 4500,00 | AGUARDANDO ENVIO | Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000
```

**Login inválido** (usuário ou senha incorretos): o sistema encerra.
```
=== SISTEMA DE ENTREGAS ===
Usuário: admin
Senha: 000
Acesso negado.
```

### Regras de cadastro
- Cada mercadoria possui **exatamente um** endereço de entrega (obrigatório).
- Nome e descrição são obrigatórios.
- Peso e valor devem ser positivos.

## 5. Arquitetura

### Estrutura de pacotes
```
src/main/java/br/edu/entregas/
├── Main.java                         # Ponto de entrada (console)
├── model/
│   ├── Mercadoria.java               # Entidade mercadoria
│   └── Endereco.java                 # Endereço de entrega (record)
├── service/
│   ├── LoginService.java             # Autenticação
│   └── EntregaService.java           # Regras de cadastro e listagem
├── repository/
│   └── MercadoriaRepository.java     # Armazenamento em memória (ArrayList)
└── util/
    └── Validador.java                # Validações reutilizáveis
```

### Responsabilidades
| Camada | Classe | Responsabilidade |
|---|---|---|
| Apresentação | `Main` | Lê usuário e senha, chama os serviços e exibe o resultado |
| Serviço | `LoginService` | Valida usuário **e** senha |
| Serviço | `EntregaService` | Valida os dados e cadastra/lista mercadorias |
| Repositório | `MercadoriaRepository` | Guarda as mercadorias em memória |
| Modelo | `Mercadoria`, `Endereco` | Representam os dados do domínio |
| Utilitário | `Validador` | Valida texto obrigatório e número positivo |

### Fluxo
```
Main ──► LoginService.autenticar()
  │
  └──► EntregaService.cadastrar() ──► Validador ──► MercadoriaRepository.salvar()
  └──► EntregaService.listar()    ──► MercadoriaRepository.listar()
```

## 6. Interface interna (API)

O sistema é uma aplicação de console e **não expõe API HTTP** (não há endpoints).
Os métodos públicos usados entre as camadas são:

| Método | Parâmetros | Retorno | Erros |
|---|---|---|---|
| `LoginService.autenticar` | `usuario`, `senha` | `true` se ambos corretos | — |
| `EntregaService.cadastrar` | `id`, `nome`, `descricao`, `peso`, `valor`, `status`, `endereco` | `Mercadoria` criada | `IllegalArgumentException` se algum dado for inválido |
| `EntregaService.listar` | — | Lista (somente leitura) de `Mercadoria` | — |

Exemplo:
```java
EntregaService service = new EntregaService();
Endereco endereco = new Endereco("Av. Goiás", "Sala 8", "1000", "74000-000", "Goiânia", "GO");
service.cadastrar(1, "Notebook", "Notebook corporativo", 2.1, 4500.00, "AGUARDANDO ENVIO", endereco);
service.listar().forEach(System.out::println);
```

## 7. Manutenção e contribuição

### Fluxo recomendado
Use branches de funcionalidade, Pull Request e revisão antes do merge.

1. Crie uma branch a partir da `main`:
   - `feature/<descricao>` para novas funcionalidades
   - `hotfix/<descricao>` para correções urgentes
   - `docs/<descricao>` para documentação
2. Faça commits pequenos, um por correção, seguindo o padrão:
   `tipo: descrição` — tipos usados: `feat`, `fix`, `docs`, `refactor`, `chore`, `security`.
3. Antes de abrir o Pull Request, confirme que `mvn clean package` executa sem erro
   e que o login recusa credenciais inválidas.
4. Abra o Pull Request para a `main` e aguarde revisão antes do merge.

### Segurança
- **Nunca** versione senhas, tokens ou arquivos de configuração com credenciais.
- Se um segredo for commitado, removê-lo do arquivo não basta: ele continua no histórico.
  Considere-o vazado e troque a senha/revogue o token.

### Histórico de manutenção
O diagnóstico e as correções do resgate do projeto estão em [`RELATORIO-RESGATE.md`](RELATORIO-RESGATE.md).

## 8. Problemas comuns

| Problema | Causa provável | Solução |
|---|---|---|
| `mvn` não é reconhecido como comando | Maven não instalado ou fora do `PATH` | Instale o Maven e adicione a pasta `bin` ao `PATH` |
| `invalid target release: 17` | Maven usando um JDK antigo | Aponte `JAVA_HOME` para um JDK 17+ |
| `Could not find or load main class br.edu.entregas.Main` | Projeto não compilado | Execute `mvn clean package` antes |
| Acentos aparecem como `�` no Windows | Console fora de UTF-8 | Execute `chcp 65001` antes de rodar o sistema |
| `Acesso negado.` com a senha correta | Espaços extras ou maiúsculas | Digite exatamente `admin` e `12345678` |

## 9. Histórico de versões

A documentação é versionada junto com o código: toda alteração de comportamento deve
atualizar este README no mesmo Pull Request.

| Versão | Data | Alterações |
|---|---|---|
| 1.0.0 | 2026-09-01 | Versão funcional inicial (tag `v1.0.0-funcional`): login, cadastro e listagem de mercadorias |
| 1.0.0 (resgate) | 2026-10-04 | Correção da compilação, recuperação do `Validador`, correção do login, recuperação do README e remoção de credenciais versionadas |
| 1.0.0 (docs) | 2026-10-05 | README reestruturado: instalação, configuração, uso, arquitetura, API, manutenção, problemas comuns e histórico. Adicionado o modelo `config/application.properties.example` |
