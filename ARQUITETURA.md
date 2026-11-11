# ARQUITETURA

## 1. Se a fonte de dados mudar de memória para outra implementação, que partes podem permanecer iguais? Quais partes ainda podem precisar de adaptação?

Podem continuar iguais a interface `GastoRepository`, toda a pasta `ui/` e as regras de validação do `GastosViewModel`. O ViewModel depende só do contrato (`listar`, `adicionar` e `remover`) e não de uma classe específica. Por isso, com o contrato no construtor, trocar `GastoRepositoryMemoria` por `GastoRepositoryDemo` exige mudar só o valor padrão do parâmetro, e a tela não precisa saber.

No experimento 4, quando tirei o contrato do construtor e criei o repositório dentro do ViewModel, usei o `GastoRepositoryDemo()` e os 4 gastos de exemplo apareceram, mas a troca exigiu editar o corpo da classe. Isso mostra que é a interface que permite trocar a fonte de dados em um único ponto.

Ainda podem precisar de adaptação os casos em que a nova fonte é diferente por natureza. Com Room, por exemplo, as operações são assíncronas (suspend functions ou Flow), então o contrato e a função `sincronizar` do ViewModel precisariam ser revistos. Também seria preciso configurar o banco e as entidades e decidir como o repositório é criado (injeção de dependência ou uma fábrica de ViewModel).

## 2. Por que total e quantidade são calculados a partir da lista visível? Descreva um erro concreto que surgiria se fossem atualizados manualmente.

Porque total e quantidade dependem só de `gastos` e de `filtro`. Quando são calculados com `get()`, eles sempre refletem o que está na tela, e ninguém precisa lembrar de atualizá-los a cada ação (adicionar, remover ou filtrar).

No experimento 3, guardei o total em uma variável atualizada só ao adicionar. Adicionei 10 e 20 e o total mostrou R$ 30,00. Ao remover o de 20, a lista ficou com um item, mas o total continuou em R$ 30,00. Ao filtrar por uma categoria sem gastos, como Transporte, o total também continuou em R$ 30,00. Com o repositório de exemplo, os 4 gastos iniciais apareceram na lista, mas o total não os somou. Em todos os casos, a tela mostrava um número que não batia com a lista.

## 3. O que os componentes sem estado conseguem fazer nos previews que seria menos simples com dependência direta de um ViewModel?

Esses componentes recebem apenas valores e callbacks, então podem ser desenhados sozinhos, com dados de exemplo e funções vazias (`{}`). Pelo código dos arquivos, dá para descrever o formulário com a mensagem de erro (`erro = "O valor precisa ser um número"`), o resumo vazio ou preenchido e a etiqueta ativa ou inativa, sem digitar nada e sem provocar a validação de verdade. Não consegui renderizar os previews no meu computador, então esta resposta é baseada no código dos previews e não em algo que eu tenha visto na tela.

Se o componente dependesse diretamente de um ViewModel, seria preciso criar e preparar uma instância dele, com repositório e estado, para cada situação que eu quisesse ver. O preview ficaria preso à lógica do app.