# ESTRAGOS

## Experimento 1: Construir ViewModel na tela

Experimento: troquei `viewModel()` por `GastosViewModel()` na assinatura de `TelaGastos`.

Minha previsão: a instância criada diretamente não seria gerenciada pelo ViewModelStore e, ao girar a tela, o estado seria perdido.

O que observei: ao girar o celular para a horizontal, a tela não rolava (a lista ficava fora da área visível), então não consegui ver os itens e não consegui confirmar se os gastos tinham sido perdidos ou só estavam escondidos. Por isso não afirmo o resultado observado deste experimento.

O que explica o resultado: pela documentação do Android, uma rotação recria a Activity. Um ViewModel obtido com `viewModel()` fica guardado no ViewModelStore e sobrevive à recriação. Um ViewModel criado diretamente com `GastosViewModel()` não passa pelo ViewModelStore, então a expectativa é que uma nova instância, com repositório vazio, seja criada quando a tela é recriada. Isso é a explicação conceitual, e eu não consegui comprová-la no celular por causa do problema de rolagem.

Como restaurei o código: usei `git restore` e voltei a assinatura para `vm: GastosViewModel = viewModel()`.

## Experimento 2: Lista não observável

Experimento: troquei `mutableStateListOf<Gasto>()` por `mutableListOf<Gasto>()` em `_gastos`.

Minha previsão: a lista não seria redesenhada depois de adicionar ou remover um gasto.

O que observei: ao remover um gasto, ele saiu da lista, mas o valor do total continuou o mesmo. Não consegui separar esse efeito do experimento 3 (total manual), então não posso afirmar que a lista deixou de atualizar por causa do `mutableListOf`.

O que explica o resultado: pela documentação do Compose, só estados observáveis, como `mutableStateOf` e `mutableStateListOf`, avisam o Compose quando mudam. Uma `mutableListOf` comum não avisa, então a tela só seria redesenhada se outro estado observável mudasse, como um campo de texto. Isso é a explicação conceitual, e não um resultado que eu tenha isolado no celular.

Como restaurei o código: usei `git restore` e voltei para `mutableStateListOf<Gasto>()`.

## Experimento 3: Total manual

Experimento: troquei o total calculado (`val total get() = gastosVisiveis.sumOf { it.valor }`) por uma variável `var total` que só era atualizada dentro de `adicionar()`.

Minha previsão: o total ficaria errado ao remover um gasto e ao filtrar por categoria.

O que observei: adicionei um gasto de 10 e outro de 20 e o total mostrou R$ 30,00. Ao remover o de 20, a lista ficou com um item, mas o total continuou em R$ 30,00. Ao filtrar por uma categoria sem nenhum gasto, como Transporte, o total continuou em R$ 30,00. Depois, com o repositório de exemplo, os 4 gastos iniciais apareceram na lista, mas o total não os somou.

O que explica o resultado: o total passou a ser um valor guardado, e só o caminho de adicionar o atualizava. Remover, filtrar e carregar os dados iniciais não passavam por ele, então ele ficou desatualizado. Calculado a partir de `gastosVisiveis`, o total acompanha a lista e o filtro automaticamente.

Como restaurei o código: usei `git restore` e voltei à propriedade `val total: Double get() = gastosVisiveis.sumOf { it.valor }`, sem a linha `total += valor`.

## Experimento 4: Acoplamento à classe

Experimento: troquei o construtor do ViewModel para criar `GastoRepositoryMemoria()` dentro da própria classe, sem receber o contrato `GastoRepository`.

Minha previsão: trocar a fonte de dados para `GastoRepositoryDemo` passaria a exigir editar o corpo do ViewModel.

O que observei: com o repositório criado dentro da classe, o app funcionou. Ao trocar para `GastoRepositoryDemo()`, os 4 gastos de exemplo (Almoço no bandejão, Passagem de ônibus, Conta de luz e Cinema) apareceram na lista, mas foi preciso editar o corpo do ViewModel, e não o valor padrão de um parâmetro.

O que explica o resultado: sem a interface `GastoRepository` como fronteira, o ViewModel conhece a classe concreta e fica preso a ela. Com o contrato no construtor, a troca seria feita em um único ponto, no valor padrão do parâmetro.

Como restaurei o código: usei `git restore` e voltei ao construtor `class GastosViewModel(private val repositorio: GastoRepository = GastoRepositoryMemoria()) : ViewModel()`.

## Experimento 5: Regra na tela

Experimento: troquei `onClick = aoAdicionar` por `onClick = { if (descricao.isNotBlank()) aoAdicionar() }` no botão de `FormularioGasto`, colocando a validação da descrição dentro da tela.

Minha previsão: a regra ficaria presa à tela e precisaria ser duplicada para ser reutilizada em outra tela ou testada.

O que observei: ao tocar em Adicionar com a descrição vazia, nenhuma mensagem de erro apareceu e o gasto também não foi adicionado. Antes da mudança, aparecia "Informe a descrição do gasto".

O que explica o resultado: com a verificação no `onClick`, a função `adicionar()` do ViewModel nem é chamada quando a descrição está vazia, então o ViewModel não tem como definir a mensagem de `erro`. A regra fica dividida entre a View e o ViewModel, e outra tela teria de repetir a mesma verificação. No ViewModel, ela fica em um único lugar.

Como restaurei o código: usei `git restore` e voltei para `onClick = aoAdicionar`.

## Observação extra: rolagem na horizontal

Ao girar o celular para a horizontal, a tela original não rolava, e os itens adicionados ficavam fora da área visível. A causa está no código: a `LazyColumn` estava dentro de uma `Column` fixa, que não rola. Troquei por uma única `LazyColumn` com título, formulário, filtro, resumo e itens, e depois disso testei no celular e a tela passou a funcionar.