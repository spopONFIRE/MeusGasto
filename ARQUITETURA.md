# ARQUITETURA

## 1. Se a fonte de dados mudar de memória para outra implementação, que partes podem permanecer iguais? Quais partes ainda podem precisar de adaptação?

Podem permanecer iguais a interface `GastoRepository`, toda a pasta `ui/` e as regras de validação do `GastosViewModel`, porque o ViewModel depende apenas do contrato (`listar`, `adicionar`, `remover`) e não de uma classe concreta. O experimento de trocar `GastoRepositoryMemoria` por `GastoRepositoryDemo` mostra isso: só o valor padrão do parâmetro do construtor muda.

Ainda podem precisar de adaptação os casos em que a nova fonte é diferente em natureza. Com Room, por exemplo, as operações passam a ser assíncronas (suspend functions ou Flow), então o contrato e a função `sincronizar` do ViewModel precisariam ser revistos, e seria preciso configurar o banco, as entidades e a criação do repositório (injeção de dependência ou fábrica de ViewModel).

## 2. Por que total e quantidade são calculados a partir da lista visível? Descreva um erro concreto que surgiria se fossem atualizados manualmente.

Porque são estado derivado: dependem apenas de `gastos` e `filtro`. Calculados em propriedades com `get()`, eles sempre refletem a lista que está na tela, sem que alguém precise lembrar de atualizá-los em cada ação (adicionar, remover, filtrar).

Se `total` fosse uma variável atualizada só ao adicionar, ao remover um gasto de R$ 32,00 a lista diminuiria mas o total continuaria o mesmo. Ao trocar o filtro para "Lazer", a lista mostraria só os gastos de lazer, mas o total continuaria somando todas as categorias. A tela exibiria informação inconsistente.

## 3. O que os componentes sem estado conseguem fazer nos previews que seria menos simples com dependência direta de um ViewModel?

Como recebem apenas valores e callbacks, eles podem ser desenhados isoladamente com dados de exemplo e lambdas vazias (`{}`). Dá para ver o formulário com mensagem de erro (`erro = "O valor precisa ser um número"`), o resumo vazio ou preenchido e a etiqueta ativa ou inativa sem executar o app, sem digitar nada e sem provocar a validação real. Com dependência direta de um ViewModel, seria preciso criar e preparar uma instância dele (e do repositório) para cada situação, e o preview ficaria acoplado à lógica.
