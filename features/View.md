# View (visões)
-----------------------------------
## Visão Geral

Uma View é uma primitiva base usada pelo angular para rederizar uma árvore DOM. 
Uma ViewContainer(Conteiner de Visões) é um local na View o qual aceita Views filhas.
Cada ViewContainer tem ViewContainerRef(Referência a um ViewContainer) associado que pode conter um infinidade de Views filhas.
Views formam uma estrutura de uma árvore que imita um ávore DOM.

* View é o construtor príncipal de renderização. Uma aplicação é apenas uma coleção de View que são aninhadas em uma estrutura que lembram árvores. Uma árvore de Views é uma versão simplificada de uma árvore DOM. Uma View pode ter um único Elemento DOM ou grandes estruturas DOM. A chave é que a árvore DOM em na View pode não sofrer mudanças estruturais (apenas mudanças de propriedades).
* Views represeintam uma instância sendo executada de uma View DOM. Isto implica que enquanto elementos em uma View pode mudar propriedades, eles não podem mudar estruturalmente, (Mudanças estruturais como, adição ou remoção de elementos, requer remoção de View filhas dos  ViewContainers). 
* Views podem ter zero ou mais ViewContainers. Uma ViewContainer é um marcador no DOM que permite a inserção de Views filhas.
* Views são criados de uma ProtoView (Protítipo de Visão). Uma ProtoView é compilado para uma View DOM o qual é bastante eficiente em criar Views.
* Views contém um objeto de contexto. Um contexto representa a instância de objeto contra o qual todas as expressõe são avaliadas.
* View contém um ChangeDetector (Detector de mudanças) para busca por mudanças detectadas no modelo.
* Views contém ElementInjector(Injetor de Elementos) para criação de Diretivas.

## View Simples

Vamos examinar uma View simples e todas suas partes em detalhes.

Assuma o seguinte Componente:

``` javascript
class Greeter {
  greeting:string;

  constructor() {
    this.greeting = 'Olá';
  }
}
```
E assuma a seguinte View HTML:

``` html
<div>
  Seu nome:
  <input var="name" type="Text">
  <br>
  {{greeting}} {{name.value}}!
</div>
```

O template acima é compilado pelo Compilador para criar uma ProtoView. A ProtoView é então usada para criar instâncias da View. O processo de instanciação envolve a clonagem do template acima e localizar todos os elementos que contenham vínculos e finalmente é instanciado as Diretivas associadas ao template. (Veja a seção de Compilação para mais detalhes.)

``` html
<div>                             | viewA(greeter)
  Your name:                      | viewA(greeter)
  <input var="name" type="Text">  | viewA(greeter): variável local 'name'
  <br>                            | viewA(greeter)
  {{greeting}} {{name.value}}!    | viewA(greeter): expressão de vínculo 'greeting' & 'name.value'
</div>                            | viewA(greeter)
```

A instância resultante da View é algo semelhante a isso (pseudo código simplificado):
``` javascript
viewA = new View({
  template: ...,
  context: new Greeter(),
  localVars: ['name'],
  watchExp: ['greeting', 'name.value']
});
```

Nota:
* Views usam instâncias de `Greeter` como contexto de avaliação.
* View reconhece variáveis locais como `name`.
* View sabe quais expressões precisam ser observadas.
* Views sabem o que é preciso para ser atualizado caso uma expressão observada mude.
* Todos os elementos DOM são pertencentes por uma única instância da View.
* A estrutura da DOM pode não mudar durante o tempo de execução. Para permitir mudanças estruturais no DOM necessitamos entender Views Compostas.

## View Compostas

Uma importante parte da aplicação é permitir mudanças na estrutura do DOM para renderizar os dados para o usuário. Em Angular isto é feito pela inserção de View filhas em um ViewContainer.

Vamos começas com uma View como a seguinte:

``` html
<ul>
  <li template="ng-for: #person of people">{{person}}</li>
</ul>
```

Durante o processo de compilação o Compilador quebra o template HTML em estas duas ProtoViews:

``` html
  <li>{{person}}</li>   | protoViewB(Locals)
```

and

``` html
<ul>                    | protoViewA(someContext)
  <template></template> | protoViewA(someContext): protoViewB
</ul>                   | protoViewA(someContext)
```

O próximo passo é compôr estas PropoViews em um View de fato oqual é rederizada para o usuário.

*Passo 1:* Instancia-se `viewA`

``` html
<ul>                    | viewA(someContext)
  <template></template> | viewA(someContext): new NgFor(new ViewContainer(protoViewB))
</ul>                   | viewA(someContext)
```

*Passo 2:* Intancia-se a diretiva `NgFor` o qual irá receber a `ViewContainerRef`. (A ViewContainerRef tem uma referência a `protoViewA`).

*Passo 3:* Como a diretiva irá executar irá pergunta ao `ViewContainerRef` para instanciar `protoViewB` e irá inserí-lo logo após a âncora do `ViewContainer`. Isto é repetivo a cada `person` in `people`.

Note que

``` html
<ul>                    | viewA(someContext)
  <template></template> | viewA(someContext): new NgFor(new ViewContainer(protoViewB))
  <li>{{person}}</li>   | viewB0(locals0(someContext))
  <li>{{person}}</li>   | viewB1(locals0(someContext))
</ul>                   | viewA(someContext)
```

*Step4:* All of the bindings in the child Views are updated. Notice that in the case of `NgFor`
the evaluation context for the `viewB0` and `viewB1` are `locals0` and `locals1` respectively.
Locals allow the introduction of new local variables visible only within the scope of the View, and
delegate any unknown references to the parent context.

``` html
<ul>                    | viewA
  <template></template> | viewA: new NgFor(new ViewContainer(protoViewB))
  <li>Alice</li>        | viewB0
  <li>Bob</li>          | viewB1
</ul>                   | viewA
```

Cada View pode ter entre zero ou mais ViewContainers. Pela inserção ou remoção de View filhas para ou de um ViewContainer, a aplicação pode  mudar a estrutura DOM em qualquer estado desejado. Uma View pode conter nós individuais ou complexas estruturas DOM. A inserção de pontos para View filhas, conhecido como ViewContainers, contém um elemento DOM o qual age como uma âncora. A âncora pode ser tanto um `template` quanto um elemento script dependendo do seu browser; Isto é usado para identificar onde as Views filhas irá ser inseridas. 

## Componente de Views

Uma View pode ter também conter Componentes. Componentes contém Shadow DOM para encapsulamento de seu estado interno de renderização. Ao contrário aos ViewContainers que podem ter zero ou mais Views, o Componente sempre terá exatamente um único Shadow View.

``` html
<div>                            | viewA
  <my-component>                 | viewA
    #SHADOW_ROOT                 | (fronteira de encapsulamento)
      <div>                      |   viewB
  renderização encapsulada       |   viewB
      </div>                     |   viewB
  </my-component>                | viewA
</div>                           | viewA
```

## Contexto de Avaliação

Cada View age como um contexto para avaliação de suas expressões. Há dois tipos de contextos: 

1. Uma instância de um componente Controller e
2. um contexto `Locals` para introdução de variáveis locais na View.

Vamos assumir o seguinte componente:

``` javascript
class Greeter {
  greeting:string;

  constructor() {
    this.greeting = 'Olá';
  }
}
```

E assumir a seguinte View HTML:

``` html
<div>                             | viewA(greeter)
  Your name:                      | viewA(greeter)
  <input var="name" type="Text">  | viewA(greeter)
  <br>                            | viewA(greeter)
  {{greeting}} {{name.value}}!    | viewA(greeter)
</div>                            | viewA(greeter)
```

A Interface do Usuário é contruida usando uma única View, e consequentemente um único contexto para `gretter`. Isto pode ser expressado em pseudo-código:

``` javascript
var greeter = new Greeter();
```

A View contém dois vínculos:

1. `gretting`: limita a propriedade `greeting` na instância `Greeter`.
2. `name.value`: isto põe um problema. Não há propriedade `name`na instância `Greeter`. Para resolver nós envolver a instância Greeter em uma instância `Local` como abaixo:

``` javascript
var greeter = new Locals(new Greeter(), {name: referência_ao_elemento_de_entrada })
```

Por envolvermos a instância de `Greeter` em `Locals`permitimos que a View intrtoduza variáveis o qual são adicionadas a instância de `Greeter`. Durante a resolução das expressões primeiramente checamos o Locals, e somente então a instância de `Greeter`.

## Ciclo de Vida da View (Hidratação e Desidatração)

Transição de View através de um particular conjunto de estados:

1. View podem ser criadas de um ProtoView.
2. Views pode ser anexadas em uma ViewContainerRef.
3. Sobre a anexação da View para uma ViewContainerRef, a View precisa ser hidratada. A hidratação envolve a instanciação de todas as Diretivas associadas com a View corrente. 
4. Até este ponto a View está preparada e renderizável. Multiplas mudanças pode ser entregues por Diretivas vindas da Detecção de Mudanças.
5. Em algum momento a View pode ser removida. Neste ponto todas as diretivas são destruidas durante o processo de Desidatração e a View torna-se inativa.
6. A View tem que esperar até que seja desanexada do DOM. O atraso em desanexar pode ser causado por alguma animação que esteja animando a saída da View.
7. Apóis a view ser desanexada do DOM está pronta para ser reutilizada. O reuso de Views permite a aplicação ser mais rápida in rederizações subsequentes.