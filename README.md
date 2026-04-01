# 🛡️ Sonar Check

Analisa **todos os arquivos do seu delta** antes do push usando as regras corporativas do SonarQube — sem precisar abrir arquivo por arquivo no editor.

---

## Por que existe esta skill?

O plugin SonarQube for IDE analisa o arquivo aberto no editor, um de cada vez. Isso funciona bem para revisão pontual, mas não cobre o escopo completo de uma entrega.

Imagine que a IA gerou 50 arquivos para você, ou que sua feature tocou 20 componentes diferentes. Você teria que abrir cada um manualmente para o plugin detectar as issues.

**A skill resolve isso:** com um único comando, ela varre o delta inteiro do Git — todos os arquivos novos e modificados da sua branch — e entrega um diagnóstico consolidado antes do push. Se houver violações, gera um plano de remediação pronto para execução no modo Plan do editor.

|                                   | Plugin SonarQube for IDE    | Skill Sonar Check                             |
| --------------------------------- | --------------------------- | --------------------------------------------- |
| Escopo                            | Arquivo aberto no editor    | Todos os arquivos do delta Git                |
| Acionamento                       | Automático ao abrir arquivo | `/sonar-check` antes do push                  |
| Resultado                         | Highlights no editor        | Diagnóstico consolidado + plano de remediação |
| Bloqueio de surpresas na pipeline | Parcial                     | Completo para o delta                         |

---

## Início rápido

```
/sonar-check
```

A skill detecta automaticamente o modo de análise disponível no seu ambiente. Se algo não estiver configurado, ela informa o que fazer.

---

## Pré-requisitos

| Item                                      | Obrigatório   |
| ----------------------------------------- | ------------- |
| Docker rodando                            | ✅ Sempre     |
| User Token do SonarCloud                  | ✅ Sempre     |
| Plugin SonarQube for IDE (Connected Mode) | Modo 2 apenas |

> ⚠️ Use exclusivamente **User Token**. Project tokens e global tokens não funcionam com o MCP.

---

## Configuração do `mcp.json`

**Cursor:** `~/.cursor/mcp.json` · **VS Code:** `.vscode/mcp.json` · **Windows:** `%APPDATA%\Cursor\mcp.json`

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

**Como obter o User Token:** [sonarcloud.io](https://sonarcloud.io) → avatar → **My Account** → **Security** → **Generate Tokens**

**Como descobrir a porta do plugin:** nas configurações do SonarQube for IDE, procure pela porta do servidor embutido (geralmente `64120–64130`). Se não usar o Modo 2, deixe `SONARQUBE_IDE_PORT` em branco.

**Linux:** adicione `"--network=host"` nos `args` antes de `"mcp/sonarqube"`.

> ⚠️ **Não clique em "Configure MCP Server"** no painel do plugin se já configurou o `mcp.json` manualmente — isso cria um segundo container e causa falhas. Se acontecer:
>
> ```bash
> docker stop $(docker ps -q --filter ancestor=mcp/sonarqube)
> ```
>
> Reinicie o Cursor em seguida.

---

## Modos de análise

A skill testa na ordem abaixo e usa o primeiro disponível:

| Modo                                | Como funciona                           | Disponível em                               |
| ----------------------------------- | --------------------------------------- | ------------------------------------------- |
| **Modo 1** — `analyze_code_snippet` | Engine do Sonar via Docker, sem plugin  | Cursor, VS Code, IntelliJ, Claude Code      |
| **Modo 2** — `analyze_file_list`    | Engine do plugin IDE via Connected Mode | Cursor, VS Code, IntelliJ (não Claude Code) |

Para o Modo 2, configure o Connected Mode no plugin: painel **CONNECTED MODE** → **Connect to SonarQube Cloud** → autentique → selecione `sua-organizacao` → vincule o projeto.

---

## Solução de problemas

**"Configuração necessária"** — o MCP não foi encontrado. Configure o `mcp.json` acima.

**"MCP configurado mas não está respondendo"**

- Plugin não está em Connected Mode → abra o plugin e conecte à organização
- Containers duplicados → rode o comando acima e reinicie o Cursor
- `SONARQUBE_IDE_PORT` incorreta → verifique a porta nas configurações do plugin

**"Projeto não integrado ao Sonar"** — `sonar-project.properties` não encontrado na raiz do projeto.

**"Projeto não encontrado ou sem permissão"** — verifique o `sonar.projectKey` e se o token tem acesso ao projeto em `sua-organizacao`.
