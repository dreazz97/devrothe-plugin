# Execução segura e validação

Como aplicar o plano aprovado (restructure-plan.md) sem perder trabalho nem partir a aplicação.

## Contents
- Rede de segurança (gate)
- Executar fase a fase
- Preservar comportamento
- Validação final e relatório

## Rede de segurança (gate)

Antes da primeira edição, garantir um caminho de reversão:
- **Projeto com git**: confirmar a working tree limpa (ou commitar o estado atual) e criar um branch de
  trabalho (ex.: `fix/restructure`). Commitar ao fim de cada fase para um histórico revertível.
- **Projeto sem git**: sugerir `git init` + commit inicial; se o utilizador recusar, fazer uma cópia de
  segurança da pasta antes de mexer.

Não prosseguir para a execução sem um destes em vigor. Porquê: é a única forma de desfazer um refactor
que corra mal.

## Executar fase a fase

- Aplicar uma fase de cada vez, na ordem do plano.
- No fim de cada fase, **validar**: instalar deps, `lint`, `build`, testes e arranque conforme o stack.
- Só avançar para a fase seguinte com a atual verde. Commitar a fase (se git).
- Atualizar as tarefas (`TaskUpdate`) e o `README.md` à medida que o estado muda.
- Se uma fase introduzir um problema difícil de resolver, **parar e reportar** ao utilizador com o
  diagnóstico, em vez de forçar ou empilhar workarounds.

## Preservar comportamento

- Mover/renomear/reorganizar e trocar tecnologias mantendo o mesmo resultado funcional.
- A única mudança de comportamento permitida é a correção dos bugs listados no plano — e essa fica
  registada.
- Quando existir teste para uma área, usá-lo como rede antes e depois de a refatorar; se não existir e a
  área for de risco, criar primeiro um teste que fixe o comportamento atual, depois refatorar.

## Validação final e relatório

Ao terminar as fases:
- Correr a suíte de testes completa; **criar testes** para as vertentes afetadas que não os tinham
  (alinhar com `../create-application/references/testing.md`). A suíte tem de ficar **verde**.
- Confirmar lint, build e arranque sem erros.
- Apresentar um relatório final: desvios corrigidos (antes → depois), bugs resolvidos, ficheiros
  reorganizados, e o resultado dos testes (totais, falhas, cobertura se disponível). Indicar o branch
  usado para o utilizador rever/integrar.
