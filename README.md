<<<<<<< HEAD
# codex-cli-runner
=======
# Codex CLI Runner

Skill para montar, executar, depurar e retomar comandos do Codex CLI com foco em baixo consumo de tokens, menor privilegio necessario e compatibilidade com as versoes atuais do Codex.

## O que esta skill faz

A `codex-cli-runner` padroniza como chamar o `codex exec` em tarefas locais ou de analise. Ela ajuda a escolher modelo, nivel de raciocinio, sandbox, diretorio de trabalho, fluxo de retomada de sessoes e tratamento de `stderr`.

O objetivo e evitar comandos caros, verbosos ou permissivos demais quando uma execucao menor e mais segura resolve o problema.

## Beneficios

- Reduz consumo de tokens usando `model_verbosity=low`, `model_reasoning_summary=none` e esforco de raciocinio adequado a tarefa.
- Usa o menor sandbox necessario: `read-only` para leitura, `workspace-write` para edicoes locais e `danger-full-access` apenas com permissao explicita.
- Evita o uso de `--full-auto`, que foi marcado como depreciado, e recomenda flags explicitas.
- Facilita a retomada de sessoes com `codex exec resume --last`, preservando modelo, esforco e sandbox originais.
- Melhora a compatibilidade entre instrucoes antigas e uso atual com modelos `gpt-5.5`, `gpt-5.4` e `gpt-5.4-mini`.
- Mantem a saida limpa ocultando progresso em `stderr` por padrao com `2>/dev/null`.
- Define uma ordem consistente para flags, reduzindo erros ao montar comandos longos.
- Ajuda a escolher custo versus profundidade: comandos simples podem usar `gpt-5.4-mini`, enquanto tarefas complexas usam `gpt-5.5`.

## Quando usar

Use esta skill quando precisar:

- montar um comando `codex exec`;
- executar uma analise rapida de repositorio;
- pedir uma revisao ou investigacao com baixo custo;
- fazer uma correcao local com `workspace-write`;
- continuar uma sessao anterior do Codex;
- migrar comandos antigos que usavam `--full-auto`;
- reduzir verbosidade e tokens em execucoes repetitivas.

## Padroes recomendados

| Tarefa | Modelo | Esforco | Sandbox |
|---|---|---|---|
| Gerar comando, documentar ou inspecionar algo pequeno | `gpt-5.4-mini` | `low` | `read-only` |
| Revisar ou analisar repositorio | `gpt-5.5` | `medium` | `read-only` |
| Corrigir bug ou editar arquivos locais | `gpt-5.5` | `medium` | `workspace-write` |
| Arquitetura, seguranca ou debugging profundo | `gpt-5.5` | `high` | menor necessario |
| Acesso amplo a maquina ou rede | `gpt-5.5` | `high` | somente com permissao |

## Exemplo de comando

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

Quando o usuario pedir para continuar uma execucao anterior, a skill recomenda reaproveitar a sessao existente:

```bash
printf '%s\n' "FOLLOW_UP_PROMPT" | codex exec --skip-git-repo-check resume --last - 2>/dev/null
```

Se o diretorio for relevante:

```bash
printf '%s\n' "FOLLOW_UP_PROMPT" | codex exec --skip-git-repo-check -C DIR resume --last - 2>/dev/null
```

## Tratamento de erros

Se `codex exec` falhar, a skill recomenda:

1. Informar a falha de forma breve.
2. Reexecutar sem `2>/dev/null` apenas quando o `stderr` for necessario para diagnostico.
3. Nao aumentar sandbox, rede ou esforco de raciocinio sem aprovacao.
4. Se o modelo for rejeitado, tentar a cadeia `gpt-5.5`, depois `gpt-5.4`, depois `gpt-5.4-mini`.
5. Se a sintaxe falhar, verificar `codex --version` e adaptar o comando para a CLI instalada.

## Estrutura da skill

```text
.
|-- SKILL.md
`-- agents/
    `-- openai.yaml
```

- `SKILL.md`: contem as regras principais de escolha de modelo, sandbox, esforco de raciocinio, retomada e formato de resposta.
- `agents/openai.yaml`: define o nome exibido e a descricao curta da skill na interface.

## Resultado esperado

Com esta skill, execucoes do Codex CLI ficam mais previsiveis, economicas e seguras. Ela e especialmente util para quem roda tarefas frequentes no terminal e quer evitar comandos excessivamente permissivos, saidas longas ou escolhas de modelo acima do necessario.
>>>>>>> f36c8c7 (Atualiza arquivos do projeto)
# Almerindop2p-codex-cli-runner
