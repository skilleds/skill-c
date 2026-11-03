# Piso de C (schematize-c) — sempre-on

> **Componente novo de sistema nasce em Rust ou Zig.** C entra por **ADR de exceção**, não por
> fit. O que segue vale para o C que já existe ou que o ADR autorizou.

1. **A suíte roda sanitizada no CI:** ASan, UBSan **com `-fno-sanitize-recover=all`** (sem isso ele
   imprime e continua, e o CI fica verde), TSan onde há thread.
2. **`-Wall -Wextra -Werror -Wconversion`** — `-Werror` é o que faz o aviso ser lido.
3. **`_FORTIFY_SOURCE` exige `-O1`+** (em `-O0` não faz nada) + `-fstack-protector-strong`.
4. **Todo `malloc` tem dono escrito**; `free(p); p = NULL;`.
5. **Retorno conferido sempre** — `write` **parcial** e `snprintf` (que **trunca** e devolve o
   tamanho que teria) inclusive.
6. **VETADO:** `gets`, `strcpy`/`strcat`/`sprintf` sem limite, `scanf("%s")` sem largura, `alloca`
   com tamanho variável, `system()`.
7. **Aritmética de ponteiro só com limite conhecido**, e a soma checada contra overflow.
8. **Parser tem fuzzing no CI**, com corpus versionado e crash virando teste.
9. **`<stdatomic.h>`, não `volatile`**; ordem de aquisição de locks documentada.
10. **Teste que passa sem sanitizer não prova ausência de UB.**
11. <!-- herdado:engineering/orquestracao:curto -->**Orquestrador não desenvolve; subagent barato executa.** O agent principal só planeja, despacha e revisa; ação onerosa vira micro-tasks para o executor mais barato que resolve: `haiku` (mecânico) / `sonnet` (código; falhou → mesmo subagent corrige, até 2 rodadas) → Codex `gpt-5.6-terra` → Cursor → subagent `opus` (motivo) → `park`; **visual vai direto ao Codex**; **imagem nunca pelo Claude: detectar Codex/Gemini e perguntar ao usuário qual usar (Codex é a sugestão), nunca API de imagem paga**; papéis em `agentes.toml`. No overdev, cada item do checklist vai a um subagent e o principal revisa antes do `- [x]`. **Sem frota ociosa:** idle com pendência volta ao trabalho; dependente de outro agent → mata e enfileira com gatilho; terminou → mata (§9.6). Detalhe: `schematize-engineering` → `references/orquestracao.md` §9.<!-- /herdado -->

Gate: `bash .claude/skills/schematize-c/scripts/check-c.sh .`

- **Herda os pisos 22–29** da `schematize-engineering` (git como trava de conclusão, deploy só do git, ops é o gate da IA, sem agents inúteis, relatório 30 min/handoff 6 h, CI remoto só antes do hml, microserviço autônomo, build em dev/hml + higiene).

<!-- herdado:engineering/pisos-22-29:bloco -->**Herda os pisos 22–29 da `schematize-engineering`** (texto completo em `assets/CLAUDE.md` dela): 22 tudo no git — sem remote, push falho ou HEAD≠origin a demanda NÃO se conclui; 23 deploy só do git (VETADO scp/rsync/tar|ssh/docker save|load/git bundle); 24 o ops é o gate da IA — a IA nunca muda a VPS direto nem escreve SQL, só operação nomeada (`ssh vps "ops <op>"`); 25 sem agents inúteis (entregou, morreu); 26 relatório a cada 30 min e handoff a cada 6 h; 27 CI remoto só antes do hml, a pedido ou em tag (`workflow_dispatch` + `hml-*`/`v*`; VETADO `pull_request`, `push` em branch, `workflow_run`, cron sem justificativa); 28 microserviço autônomo (fila durável + 7 testes de resiliência); 29 build/teste na máquina dev/hml quando existe (ponte só no workspace de build, nunca prd) + higiene (`scripts/higiene.sh`).<!-- /herdado -->
