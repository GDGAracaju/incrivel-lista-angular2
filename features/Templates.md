# Templates
-----------------------------------
Templates são marcações os quais são adicionados ao HTML para descrever de forma declarativa como o modelo de aplicação deve ser projetado para o DOM como também como eventos DOM podem invocar quais métodos no controller. Templates contém sintaxes as quisl são o núcleo para o Angular e permite vínculo-de-dados (data-binding), vínculo-de-eventos (event-binding) e, instanciação de templates. 

O design da sintaxe de templates tem estas propriedades:

* Todas as expressões de vínculo-de-dados são facilmente identificáveis. (Ex.: nunca há ambiguidades seja se o valor possa ser interpretado como uma string literal ou como uma expressão).
* Todos os eventos e suas declarações são fácilmente identificáveis.
* Todos os locais de instanciação de DOM são facilmente identificáveis.
* Todas os lugares de declaração de variáveis são facilmente identificáveis.

As propriedades acima garantem que os templates são facilmente analisados por ferramentas (como IDEs) e raciocinado pelas pessoas.
Em nenhum ponto é necessário entender quais diretivas estão ativas ou quais semânticas estão em ordem para raciocinar sobre as características de tempo de execução do template. 

## Vínculos de Propriedade

Vincular dados de modelo de aplicação para a Interface de Usuário é o tipo mais comum de vínculos em uma aplicação Angular. Os vínculos são sempre na forma de `nome-da-propriedade` o qual é atribuídauma `expressão`. A forma genérica é: 

**Forma curta:** `<algum-elemento [alguma-propriedade]="expressao">`

**Forma canônica:** `<algum-elemento bind-alguma-propriedade="expressao">`

Onde:
* `algum-elemento` pode ser um elemento DOM existente.
* `alguma-propriedade` (escapado por `[]` ou `bind-`) é o nome da propridade em `algum-elemento`. Neste caso o caixa de traço (dash-case) é convetido em camel-case `algumaPropriedade`.
* `expressao` é uma expressão válida (como vai ser definido na seção abaixo). 

Examplo:
``` html
<div [title]="user.firstName">
```

No exemplo acima a propriedade `title` do elemento `div` será atualizado toda vez que o `user.firstName`mudar seu valor. 

Pontos chave:

* O vínculo é a propriedade do elemento e não o atributo do elemento.
* Para prevenir que elementos customizados possam ser lidos como uma `expressão`literal sobre o elemento title, o nome do atributo é escapado. Neste caso o `title` é escapado para `[title]`através da adição de colchetes `[]`.
* Um valor de um vínculo (neste caso `user.firstName`) irá sempre ser uma expressão, nunca uma string literal.

**NOTA:** Angular 2 vincula propriedades de elementos ao invés de atributos de elementos. Isto é feito para melhor dar suporte a elementos customizados. e permitir vínculos para outros valores além de string.

**NOTA:** Alguns editores/pre-processadores do lado de servidores podem ter problemas para gerar `[]` em torno dos nome de atributos. Por esta razão Angular também suporta a versão canônica que é prefixando `bind-`. 

## Vinculos de Eventos

Vincular eventos permite ligar eventos vindo do DOM (ou outros componentes) para o controller Angular.

**Forma curta:** `algum-elemento (algum-evento)="declaração">`

**Forma Canônica:** `algum-elemento on-algum-evento="decaração">`

Onde: 

* `algum-elemento` Algum elemento o qual pode gerar eventos  DOM (ou tem uma diretiva Angular que gera o evento).
* `algum-evento` (escapado com `()` ou `on-`) é o nome do evento `algum-evento`. Neste caso o dash-case é convertido para `algumEvento` em camelcase.  
* `declação`é uma declaração válida (como definida na seção abaixo).

Se a execução de uma declaração retornar `false`, então `preventDefault` é aplicado ao evento DOM. 

Angular escuta por eventos DOM disparados (como no caso de clicar em qualquer filho), como mostrado abaixo:

**Forma curta:** `<algum-elemento (algum-evento)="declaração">`

**Forma canônica:** `<algum-elemento on-algum-evento="declaração">`

Exemplos:
``` js
@Component(...)
class Example {
  submit() {
    // Fazer algo quando o botão é clicado
  }
}

<button (click)="submit()">Submeter</button>
```

No exemplo acima, quando clica-se no botão de submeter angular irá chamar o método `submit` do controller em torno do componente.

**NOTA:** Ao contrário do Angular v1, Angular 2 trata vínculos de evento como contrutores principais e não como diretivas. Isto significa que não há necessidade de criar uma diretiva de evento para cada tipo de evento. Isto permite ao Angular 2 de facilmente vincular eventos customizados de Elementos Customizados, cujo nomes de eventos não são conhecidos antecipadamente

## Interpolação de Strings

Vínculo de propriedades sãp apenas vínculos de dados o qual o angular dá suporte, mas, por conveniencia Angular suporta uma sintaxe de interpolação o qual é apenas um atalho para sintaxe de vínculo de dados.

``` html
<span>Olá {{name}}!</span>
```

é um atalho para:

``` html
<span [text|0]=" 'Olá ' + stringify(name) + '!'">_</span>
```

O que quer dizer acima é vincular a expressão `'Olá ' + stringify(name) + '!'` para o zerésimo filho da propriedade `text` de `span`. O índice é necessário neste caso no caso de haver mais de um nó de text, ou ser o nó text que desejamos vincular não é o primeiro.

Similarmente as mesmas regras se aplicam a interpolação dentro de atributos.

``` html
<span title="Olá {{name}}!"></span>
```

é atalho para:

``` html
<span [title]=" 'Olá ' + stringify(name) + '!'"></span>
```

**NOTA:** `stringify()` é uma função implicita o qual converte seus argumentos em sua representação em string, enquanto mantém `null` e `undefined` como string vazias.

## Templates em linha

Vínculo de dados permitem atualizar propriedades do DOM, mas não permite para mudar a estrutura do DOM. Para mudar estruturas do DOM precisamos da habilidade de definir templates filhos em Views (Visões). As Views podem ser inseridas e removidas de acordo com a necessidade para mudar estruturas do DOM. 

**Forma curta:**
``` html
Olá {{user}}!
<div template="ng-if: isAdministrator">
  ...menu do adminstrador aqui...
</div>
```

**Forma canônica**
``` html
Olá {{user}}!
<template [ng-if]="isAdministrator">
  <div>
    ...menu do administrador aqui...
  </div>
</template>
```

Onde:
* `template`define um teplate filho e designa a âncora onde Views(instâncias do template) irão ser inseridas. O template pode ser definido implicitamente com o atributo `template`, que torna o elemento corrente em um template, ou explicitamente com o elemento `<template>`. Declaração explicita é longa, mas permite templates com mais de uma raiz em um nó DOM. 
* `viewport` é requerido para templates. A diretiva é responsável por decidir quando e em que ordem as views filhas serão inseridas em sua localização. Tal diretiva usualmente pode ter mais de um vínculo e pode ser representado tanto como `viewport-directive-bindinds` ou `viewport-directive-microsyntax` no elemento ou atributo do `template`. Veja microsintaxe de templates para mais detalhes.

## Microsintaxe de Template

Frequentemente é necessário codificar uma gama de diferentes vínculos em um template para controlar como a instanciação do template ocorre. Um exemplo é utilizando `ng-for`.

``` html
<form #foo=form>
</form>
<ul>
  <template [ng-for] #person [ng-for-of]="people" #i="index">
    <li>{{i}}. {{person}}<li>
  </template>
</ul>
```

Onde:
* `[ng-for]` dispara a diretiva for.
* `#person` exporta o item implícito `ng-for`.
* `[ng-for-of]="people"` vincula um objeto iterável para o controller `ng-for`.
* `#i=index` exporta o índice de item como `i`.

O exemplo acima é explicito mas um tanto prolixo. Por esta razão na maioria dos casos a versão curta é a sintaxe mais preferida.

``` html
<ul>
  <li template="ng-for; #person; of=people; #i=index;">{{i}}. {{person}}<li>
</ul>
```

Note que cada par chave-valor é traduzido para uma declaração `key=value;` no atributo do `template`. Isto faz que a sintaxe repetitiva seja muito enxuta, mas podemos fazer melhor. Acaba que a maioria da pontuação é opcional na versão curta que permitem-nos encutar ainda mais o texto. 

``` html
<ul>
  <li template="ng-for #person of people #i=index">{{i}}. {{person}}<li>
</ul>
```
Nós podemos usar `var` ao invés de `#` e adicionar `:` para `for` o qual cria o seguinte recomendável microsintaxe para `ng-for`.

``` html
<ul>
  <li template="ng-for: var person of people; var i=index">{{i}}. {{person}}<li>
</ul>
```

Finalmente, podemos mover a palavra-chave `ng-for` para o lado esquerdo e prefixá-lo com `*`com abaixo:

``` html
<ul>
  <li *ng-for="var person of people; var i=index">{{i}}. {{person}}<li>
</ul>
```

/o formato foi intencionalmente definido mais livremente, então os desenvolvedores de diretivas podem criar expressivas microsintaxes, para suas diretivas. O código seguinte descreve uma definição mais formal.

```
expressão: ...                     // como definido na seção Expressões
local: [a-zA-Z][a-zA-Z0-9]*         // nome variável exportada disponível para vínculos
intena: [a-zA-Z][a-zA-Z0-9]*      // nome de variável interna o qual exporta diretivas.
chave: [a-z][-|_|a-z0-9]]*            // chave o qual mapeia nomes de atributos
expressões-chave: key[:|=]?expressão  // vínculos o qual mapeia expressões a propriedades
exportarVar: [#|var]local(=internal)? // vínculo que exporta uma variável local de uma diretiva
microssintaxe: ([[key|keyExpression|varExport][;|,]?)*
```

Onde
* `expressão` é expressão Angular definido na seção: Expressões.
* `local` é um identificador local para variáveis locais.
* `interna` é uma variável interna o qual a diretiva é exportada para vínculos.
* `chave` é um nome de um atributo usualmente usado para disparar uma diretiva específica.
* `exressõesChave` é um nome de propriedade o qual a espeção será ligada.
* `exportarVar` permite exportar o estada interno da diretiva como variáveis para vínculos mais a diante. Se não há nome especificado `interno`, a exportação é uma variável implícita.
* `microsintaxe` permite a você construir microsintaxes simples o qual pode claramente identificar quais expreções está vonculadas a quais variáveis estão exportadas para vínculos.

**NOTA:** O `template`deve estar presente para fazer claro ao usuário que um sub-template está sendo criado. Isso vai contra a filosofia de que o desenvolvedor deve ser apto a pensar sobre o template sem entender as semânticas de um instanciador de diretivas.