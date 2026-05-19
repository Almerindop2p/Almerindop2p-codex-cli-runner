# Almerindop2p Codex CLI Runner

Skill para montar, executar, depurar e retomar comandos do Codex CLI com foco em baixo consumo de tokens, menor privilegio necessario e compatibilidade com as versoes atuais do Codex.

## O que esta skill faz

A `codex-cli-runner` padroniza como chamar o `codex exec` em tarefas locais, analises de repositorio, revisoes, correcoes e retomadas de sessoes anteriores.

Ela ajuda a escolher:

- modelo adequado para custo e complexidade;
- nivel de raciocinio necessario;
- sandbox mais restrito possivel;
- diretorio de trabalho com `-C`;
- fluxo correto para `resume --last`;
- tratamento de `stderr` para manter a saida limpa.

O objetivo e evitar comandos caros, verbosos ou permissivos demais quando uma execucao menor e mais segura resolve o problema.

## Beneficios

- Reduz consumo de tokens usando `model_verbosity=low`, `model_reasoning_summary=none` e esforco de raciocinio proporcional a tarefa.
- Usa o menor sandbox necessario: `read-only` para leitura, `workspace-write` para edicoes locais e `danger-full-access` apenas com permissao explicita.
- Evita `--full-auto`, que esta depreciado, e recomenda sandboxes explicitos.
- Facilita a retomada de sessoes com `codex exec resume --last`, preservando modelo, esforco e sandbox originais.
- Ajuda a migrar instrucoes antigas para um uso compativel com `gpt-5.5`, `gpt-5.4` e `gpt-5.4-mini`.
- Mantem a saida mais limpa ocultando progresso em `stderr` por padrao com `2>/dev/null`.
- Define uma ordem consistente para flags, reduzindo erro ao montar comandos longos.
- Evita escalar privilegio, rede ou raciocinio sem aprovacao do usuario.
- Trata `high` como o nivel maximo de raciocinio permitido pela skill.

## Instalacao passo a passo

A skill deve ficar dentro da pasta de skills do Codex com este formato:

```text
~/.codex/skills/codex-cli-runner/
|-- SKILL.md
`-- agents/
    `-- openai.yaml
```

Depois de instalar ou atualizar, reinicie o Codex para carregar a nova versao.

### Windows PowerShell

1. Feche o Codex, se ele estiver aberto.
2. Defina o caminho do ZIP baixado:

```powershell
$zipPath = "C:\Users\SEU_USUARIO\Downloads\skill Almerindo Junior - atualizado.zip"
```

3. Defina a pasta de destino da skill:

```powershell
$dest = "$env:USERPROFILE\.codex\skills\codex-cli-runner"
```

4. Crie uma pasta temporaria e extraia o ZIP:

```powershell
$temp = Join-Path $env:TEMP "codex-cli-runner-install"
Remove-Item -LiteralPath $temp -Recurse -Force -ErrorAction SilentlyContinue
Expand-Archive -LiteralPath $zipPath -DestinationPath $temp -Force
```

5. Crie a pasta de destino, caso ela ainda nao exista:

```powershell
New-Item -ItemType Directory -Path "$dest\agents" -Force | Out-Null
```

6. Copie os arquivos da skill para o destino:

```powershell
Copy-Item -LiteralPath "$temp\codex-cli-runner\SKILL.md" -Destination "$dest\SKILL.md" -Force
Copy-Item -LiteralPath "$temp\codex-cli-runner\agents\openai.yaml" -Destination "$dest\agents\openai.yaml" -Force
```

7. Confirme que os arquivos foram instalados:

```powershell
Get-ChildItem -Recurse -LiteralPath $dest
```

8. Abra o Codex novamente.

### macOS ou Linux

1. Feche o Codex.
2. Extraia o ZIP:

```bash
unzip "skill Almerindo Junior - atualizado.zip" -d /tmp/codex-cli-runner-install
```

3. Crie a pasta da skill:

```bash
mkdir -p "$HOME/.codex/skills/codex-cli-runner/agents"
```

4. Copie os arquivos:

```bash
cp /tmp/codex-cli-runner-install/codex-cli-runner/SKILL.md "$HOME/.codex/skills/codex-cli-runner/SKILL.md"
cp /tmp/codex-cli-runner-install/codex-cli-runner/agents/openai.yaml "$HOME/.codex/skills/codex-cli-runner/agents/openai.yaml"
```

5. Confira a instalacao:

```bash
find "$HOME/.codex/skills/codex-cli-runner" -maxdepth 3 -type f
```

6. Abra o Codex novamente.

### Atualizar uma versao antiga

Para atualizar, repita o processo de copia usando `-Force` no PowerShell ou sobrescrevendo com `cp` no macOS/Linux. Isso substitui `SKILL.md` e `agents/openai.yaml` pela versao nova.

Se a skill ja estiver carregada no Codex, a mudanca so sera aplicada depois de reiniciar o Codex.

## Quando usar

Use esta skill quando precisar:

- montar um comando `codex exec`;
- executar uma analise rapida de repositorio;
- pedir revisao, investigacao ou resumo com baixo custo;
- fazer uma correcao local com `workspace-write`;
- continuar uma sessao anterior do Codex;
- migrar comandos antigos que usavam `--full-auto`;
- reduzir verbosidade e tokens em execucoes repetitivas;
- adaptar comandos ao modelo e a versao instalada do Codex CLI.

## Padroes recomendados

| Tarefa | Modelo | Esforco | Sandbox |
|---|---|---|---|
| Gerar comando, documentar ou inspecionar algo pequeno | `gpt-5.4-mini` | `low` | `read-only` |
| Revisar ou analisar repositorio | `gpt-5.5` | `medium` | `read-only` |
| Corrigir bug ou editar arquivos locais | `gpt-5.5` | `medium` | `workspace-write` |
| Arquitetura, seguranca ou debugging profundo | `gpt-5.5` | `high` | menor necessario |
| Acesso amplo a maquina ou rede | `gpt-5.5` | `high` | `danger-full-access` apenas com permissao |

## Niveis de raciocinio

A skill recomenda escolher o menor nivel suficiente:

- `minimal`: formatacao, documentacao e pequenas inspecoes.
- `low`: correcoes simples, pequenas revisoes e geracao de comandos.
- `medium`: bugs comuns, edicoes de feature e refactors normais.
- `high`: arquitetura, seguranca, performance, refactors amplos e qualquer tarefa que exigiria o nivel maximo.

`high` e o teto recomendado. A skill nao recomenda escalar para um nivel acima disso.

## Exemplo de comando

Para leitura rapida:

```bash
codex exec --skip-git-repo-check -m gpt-5.4-mini -c model_reasoning_effort=low -c model_verbosity=low -c model_reasoning_summary=none --sandbox read-only "summarize this repository structure" 2>/dev/null
```

Para tarefas com edicao local:

```bash
codex exec --skip-git-repo-check -m gpt-5.5 -c model_reasoning_effort=medium -c model_verbosity=low -c model_reasoning_summary=none --sandbox workspace-write -C ./app "fix the failing unit tests with the smallest safe change" 2>/dev/null
```

Para prompts com aspas ou multiplas linhas:

```bash
printf '%s\n' "PROMPT" | codex exec --skip-git-repo-check -m gpt-5.5 -c model_reasoning_effort=medium -c model_verbosity=low -c model_reasoning_summary=none --sandbox workspace-write -C DIR - 2>/dev/null
```

## Retomar uma sessao

Quando o usuario pedir para continuar uma execucao anterior, a skill recomenda reaproveitar a sessao existente e herdar modelo, raciocinio e sandbox originais:

```bash
printf '%s\n' "FOLLOW_UP_PROMPT" | codex exec --skip-git-repo-check resume --last - 2>/dev/null
```

Se o diretorio for relevante para encontrar a sessao anterior, coloque `-C DIR` antes de `resume`:

```bash
printf '%s\n' "FOLLOW_UP_PROMPT" | codex exec --skip-git-repo-check -C DIR resume --last - 2>/dev/null
```

Depois de uma execucao bem-sucedida, a skill recomenda lembrar o usuario de que a sessao pode ser retomada depois com `codex resume`.

## Tratamento de erros

Se `codex exec` falhar, a skill recomenda:

1. Informar a falha de forma breve.
2. Reexecutar sem `2>/dev/null` apenas quando o `stderr` for necessario para diagnostico.
3. Nao aumentar sandbox, rede ou nivel de raciocinio sem aprovacao.
4. Lembrar que `high` e o nivel maximo permitido.
5. Se o modelo for rejeitado, tentar `gpt-5.5`, depois `gpt-5.4`, depois `gpt-5.4-mini`.
6. Se a sintaxe falhar, verificar `codex --version` e adaptar o comando para a CLI instalada.

## Estrutura da skill

```text
.
|-- SKILL.md
|-- README.md
`-- agents/
    `-- openai.yaml
```

- `SKILL.md`: contem as regras principais de modelo, sandbox, raciocinio, retomada, tratamento de erro e formato de resposta.
- `agents/openai.yaml`: define o nome exibido e a descricao curta da skill na interface.
- `README.md`: documenta o uso e os beneficios da skill para o repositorio Git.

## Resultado esperado

Com esta skill, execucoes do Codex CLI ficam mais previsiveis, economicas e seguras. Ela e especialmente util para quem roda tarefas frequentes no terminal e quer evitar comandos permissivos demais, saidas longas ou escolhas de modelo acima do necessario.
