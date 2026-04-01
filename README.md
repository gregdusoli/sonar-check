# 🛡️ Sonar Check

Analisa **todos os arquivos do seu delta** antes do push usando as regras corporativas do SonarQube — sem precisar abrir arquivo por arquivo no editor.

## Por que existe esta skill?

O plugin SonarQube for IDE analisa o arquivo aberto no editor, um de cada vez. Isso funciona bem para revisão pontual, mas não cobre o escopo completo de uma entrega.

Imagine que a IA gerou 50 arquivos para você, ou que sua feature tocou 20 componentes diferentes. Você teria que abrir cada um manualmente para o plugin detectar as issues.

**A skill resolve isso:** com um único comando, ela varre o delta inteiro do Git — todos os arquivos novos e modificados da sua branch — e entrega um diagnóstico consolidado antes do push. Se houver violações, gera um plano de remediação pronto para execução no modo Plan do editor.

## Comparativo de funcionamento

|                                   | Plugin SonarQube for IDE    | Skill Sonar Check                             |
| --------------------------------- | --------------------------- | --------------------------------------------- |
| Escopo                            | Arquivo aberto no editor    | Todos os arquivos do delta Git                |
| Acionamento                       | Automático ao abrir arquivo | `/sonar-check` antes do push                  |
| Resultado                         | Highlights no editor        | Diagnóstico consolidado + plano de remediação |
| Bloqueio de surpresas na pipeline | Parcial                     | Completo para o delta                         |

## Modos de análise

A skill verifica seu ambiente na ordem abaixo e usa o primeiro resultado disponível para definir o modo de análise:

| Modo                                | Como funciona                           | Disponível em                               |
| ----------------------------------- | --------------------------------------- | ------------------------------------------- |
| **Modo 1** — `analyze_code_snippet` | Engine do Sonar via Docker, sem plugin  | Cursor, VS Code, IntelliJ, Claude Code      |
| **Modo 2** — `analyze_file_list`    | Engine do plugin IDE via Connected Mode | Cursor, VS Code, IntelliJ (não Claude Code) |

\* Para o Modo 2 é necessário utilizar o plugin [SonarQube for IDE](https://www.sonarsource.com/products/sonarqube/ide/) com [Connected Mode](https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/connected-mode) configurado.

## Configuração

Entenda os pré-requisitos e as 2 formas de realizar o setup:

#### Pré-requisitos

| Item                                      | Obrigatório                        |
| ----------------------------------------- | ---------------------------------- |
| Docker em execução                        | ✅ Sempre                          |
| User Token do SonarCloud                  | ✅ Sempre                          |
| Plugin SonarQube for IDE (Connected Mode) | [Modo 2](#modos-de-análise) apenas |

---

### 1. Usando o plugin SonarQube (recomendado)

- Instale o plugin [SonarQube for IDE](https://www.sonarsource.com/products/sonarqube/ide/) para o seu Editor de Código
- No plugin, faça login com a conta corporativa do Google para habilitar o [Connected Mode](https://docs.sonarsource.com/sonarqube-for-vs-code/connect-your-ide/connected-mode)
- Encontre a opção `AI Agents Configuration` e clique em `Configure SonarQube MCP Server`

> Confirme se o MCP do Sonar foi configurado corretamente através das configurações do Editor. Adicionalmente, verifique se o container Docker está em execução.

### 2. Configuração manual

> ⚠️ Use exclusivamente **User Token**. Project tokens e global tokens não funcionam com o MCP.
>
> ⚠️ **Não clique** em **Configure MCP Server** no painel do plugin se já existir configuração do MCP ativa — isso cria um segundo container e causa falhas na execução da skill.

- Visite o portal [sonarcloud.io](https://sonarcloud.io) para gera/obter seu token de usuário em **Avatar** → **My Account** → **Security** → **Generate Tokens**
- Modo 1: deixe `SONARQUBE_IDE_PORT` em branco - Modo 2: nas configurações do plugin, procure pela porta do servidor embutido (geralmente `64120–64130`).
- Acesse as configurações do Editor, navegue até a sessão de configurações de MCP e edite o arquivo `mcp.json`:

```json
{
  "sonarqube": {
    "command": "docker",
    "args": [
      "run",
      "-i",
      "--rm",
      "--init",
      "--pull=always",
      "-e",
      "SONARQUBE_TOKEN",
      "-e",
      "SONARQUBE_ORG",
      "-e",
      "SONARQUBE_IDE_PORT",
      "mcp/sonarqube"
    ],
    "env": {
      "SONARQUBE_TOKEN": "<seu-user-token>",
      "SONARQUBE_ORG": "<sua-organizacao>",
      "SONARQUBE_IDE_PORT": "<porta-do-plugin>"
    }
  }
}
```

\* **Linux:** adicione `"--network=host"` nos `args` antes de `"mcp/sonarqube"`.

## Usando a skill

```
/sonar-check
```

A skill detecta automaticamente o modo de análise disponível no seu ambiente. Se algo não estiver configurado, ela informa o que fazer.

## Solução de problemas

#### Configuração necessária

O MCP não foi encontrado. Siga os passos de configuração acima de acordo com seu ambiente.

#### MCP configurado mas não está respondendo

Causas possíveis:

- `SONARQUBE_IDE_PORT` incorreta → verifique a porta nas configurações do plugin
- Plugin não está em Connected Mode → abra o plugin e conecte à organização
- Containers duplicados → rode o comando acima e reinicie o Cursor.

#### Projeto não integrado ao Sonar

O arquivo `sonar-project.properties` não foi encontrado na raiz do projeto.

#### Projeto não encontrado ou sem permissão

Verifique o `sonar.projectKey` e se o token tem acesso ao projeto em `<sua-organizacao>`.

#### Container do MCP duplicado

```bash
$ docker stop $(docker ps -q --filter ancestor=mcp/sonarqube)
```

Reinicie o Editor e tente executar a skill novamente.
