# ESTRAGOS

## Experimento 1: Construir ViewModel na tela

Experimento: trocar `viewModel()` por `GastosViewModel()` em `TelaGastos`.

Minha previsão: a instância deixa de ser gerenciada pelo ViewModelStore e pode ser recriada, perdendo o estado (por exemplo, ao girar a tela).

O que observei: ao girar a tela, os gastos cadastrados e os campos digitados foram perdidos, voltando ao estado inicial. Com `viewModel()` eles permanecem.

O que explica o resultado: `viewModel()` obtém uma instância associada ao ciclo de vida da tela, que sobrevive a mudanças de configuração. Criar o ViewModel diretamente não passa pelo ViewModelStore, então uma nova instância (com repositório novo e vazio) é criada.

Como restaurei o código: voltei a chamada para `vm: GastosViewModel = viewModel()`.

## Experimento 2: Lista não observável

Experimento: trocar `mutableStateListOf` por `mutableListOf` em `_gastos`.

Minha previsão: a lista não vai redesenhar após adicionar ou remover.

O que observei: ao adicionar ou remover, a lista não atualizou de imediato; só mudou quando outro estado observável (como um campo de texto) provocou recomposição.

O que explica o resultado: `mutableListOf` não notifica o Compose sobre alterações. Só `mutableStateListOf` registra as mudanças e dispara a recomposição dos Composables que a leem.

Como restaurei o código: voltei para `mutableStateListOf<Gasto>()`.

## Experimento 3: Total manual

Experimento: guardar `total` em uma variável e atualizá-la somente ao adicionar.

Minha previsão: o total ficará incorreto ao remover e ao filtrar.

O que observei: ao remover um item o total não diminuiu, e ao filtrar por uma categoria o total continuou somando todos os gastos, divergindo da lista exibida.

O que explica o resultado: um valor armazenado precisa ser mantido em todos os caminhos que alteram os dados. Um caminho esquecido gera inconsistência. O valor derivado com `get()` é recalculado sempre que lido.

Como restaurei o código: voltei a `val total: Double get() = gastosVisiveis.sumOf { it.valor }`.

## Experimento 4: Acoplamento à classe

Experimento: troquei o construtor do ViewModel para criar `GastoRepositoryMemoria()` dentro da própria classe, sem receber o contrato `GastoRepository`.

Minha previsão: trocar a fonte de dados para `GastoRepositoryDemo` passaria a exigir editar o corpo do ViewModel.

O que observei: com o repositório criado dentro da classe, o app funcionou normalmente. Ao trocar para `GastoRepositoryDemo()`, os 4 gastos de exemplo (Almoço no bandejão, Passagem de ônibus, Conta de luz e Cinema) apareceram na lista, mas foi preciso editar o corpo do ViewModel, e não só o valor padrão do parâmetro, como seria com o contrato `GastoRepository`.

O que explica o resultado: sem a interface `GastoRepository` como fronteira, o ViewModel conhece a classe concreta e fica preso a ela. É o contrato que permite trocar a fonte de dados mexendo em um único ponto e sem alterar a lógica do ViewModel nem a interface.

Como restaurei o código: voltei ao construtor `class GastosViewModel(private val repositorio: GastoRepository = GastoRepositoryMemoria()) : ViewModel()`.

## Experimento 5: Regra na tela

Experimento: troquei `onClick = aoAdicionar` por `onClick = { if (descricao.isNotBlank()) aoAdicionar() }` no botão de `FormularioGasto`, colocando a validação da descrição dentro da tela.

Minha previsão: a regra ficaria presa à tela e precisaria ser duplicada para ser reutilizada em outra tela ou testada.

O que observei: ao tocar em Adicionar com a descrição vazia, nenhuma mensagem de erro apareceu e o gasto também não foi adicionado. Antes da mudança, aparecia "Informe a descrição do gasto". A validação do valor (por exemplo `abc`) continuou mostrando a mensagem normalmente, porque essa regra continua no ViewModel.

O que explica o resultado: com a verificação no `onClick`, a função `adicionar()` do ViewModel nem é chamada quando a descrição está vazia, então o ViewModel não tem como definir a mensagem de `erro`. A regra ficou dividida entre dois lugares: a descrição na View e o valor no ViewModel. Isso gera comportamentos inconsistentes e obriga outra tela a repetir a mesma regra. No ViewModel, ela fica em um único lugar, pode ser testada sem interface e vale para qualquer View.

Como restaurei o código: voltei para `onClick = aoAdicionar`