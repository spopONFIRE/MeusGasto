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

Experimento: fazer o ViewModel criar diretamente `GastoRepositoryMemoria` e remover o contrato do campo.

Minha previsão: trocar para `GastoRepositoryDemo` passará a exigir edição do corpo do ViewModel.

O que observei: o ViewModel passou a depender da classe concreta, e para usar o Demo foi necessário alterar o código interno dele, não apenas o valor padrão do parâmetro.

O que explica o resultado: sem a interface como fronteira, o ViewModel conhece a implementação. O contrato `GastoRepository` é o que permite trocar a fonte de dados sem mexer no restante.

Como restaurei o código: voltei para `private val repositorio: GastoRepository = GastoRepositoryMemoria()`.

## Experimento 5: Regra na tela

Experimento: validar a descrição dentro do `onClick` do botão.

Minha previsão: a regra fica presa à tela e precisaria ser duplicada para ser reutilizada ou testada.

O que observei: a validação funcionou naquela tela, mas ficou misturada ao código visual, e outra tela precisaria repetir a mesma regra.

O que explica o resultado: regras pertencem ao ViewModel. Lá elas ficam em um único lugar, podem ser testadas sem interface e reutilizadas por qualquer View.

Como restaurei o código: removi a validação do `onClick` e mantive `aoAdicionar = vm::adicionar`.
