# Anotações
--------------
## Diretivas
Diretivas permitem que se adicione comportamento a elementos no DOM.

Diretivas são classes que são instanciadas em resposta a uma estrutura particular DOM. Pelo controle da estrutura DOM, quais diretivas são importadas e seus seletores, o desenvolvedor pode usar o [padrão Composição](https://pt.wikipedia.org/wiki/Composi%C3%A7%C3%A3o_de_objetos) para ter o desejado comportamente na aplicação.

Diretivas são o pilar de uma aplicação em Angular. Nós usamos Diretivas para quebrar problemas complexos em menoes para componentes mais reutilizáveis. Diretivas permitem ao desenvolvedor torna HTML em uma DSL e então controlar o processo de montagem de aplicação.

Uma diretiva consiste de uma anotação de diretiva e uma classe controladora(controller). Quando o `seletor` da diretiva casa com elements no DOM, os seguintes passos ocorrem:

* Para cada diretiva, o *ElementInjector* (Injetor de Elementos) tenta resolver os argumentos do construtor da diretiva.
* Angular instancia diretivas a cada elemento encontrado usando o *ElementInjector* em uma busca de ordem em profundidade, como declarado no HTML

Aqui há um exemplo trivial de um decorador de um tooltip. A diretiva irá logar no console toda vez que um mouse passar na região de um tooltip:

``` javascript
@Directive({
  selector: '[tooltip]',     | O seletor CSS que dispara o decorador
  inputs: [                  | Lista quais propriedades precisam ser vinculados
    'text: tooltip'          |  - A propriedade do elemento DOM deve ser
  ],                         |    mapeado para propriedade de texto da diretiva.
  host: {                    | Lista quais eventos precisam ser mapeados
    '(mouseover)': 'show()'  |  - Invoca o método show() toda vez que
  }                          |    o evento mouseover (mouse sobre) é disparado.
})                           |
class Form {                 | Classe controladore de diretiva é instanciada,
                             | quando CSS é encontrado.
  text:string;               | propriedade de texto no Controller da Diretiva.
                             |
  show(event) {              | Método show() que implementa a ação de exibir no console.
    console.log(this.text);  |
  }
}
```

Exemplo de uso:

``` html
<span tooltip="Texto do vai aqui Tooltip.">Algum texto aqui.</span>
```
O desenvolvedor de uma aplicação agora pode usar livremente o atributo `tooltip` em qualquer lugar que este comportamento for necessário.
O código acima ensina o navegador um novo reutilizável e declarativo comportamento.

Note que  o vínculo dos dados irá funcionar com este decorador sem mais nenhum esforço com é mostrado abaixo.

``` html
<span tooltip="Olá {{usuario}}!">Algum texto aqui.</span>
```

**NOTA:** Aplicações em Angular não tem método main. Invés disso eles tem um Componente raiz. A Injeção de Dependência monta as dependências para formar uma aplicação angular funcional.

## Componentes
Um componente é uma diretiva o qual usa shadow DOM para criar comportamentos visuais encapsulados. Componentes são tipicamente usados para criar elementos visuais ou partir a aplicação em componentes menores.

* Only one component can be present per DOM element.
* Apenas um componente pode está presente por elemento DOM.
* Seletores de componentes CSS comumente são disparados pelo nome de seu elemento. (Boas práticas)
* Componentes tem seu próprio shadow view o qual é adicionado ao elemento como um Shadow DOM.
* Contextro de Shadow View é a inst?ância de um componente. (Ex.: expressões de templates são avaliados contra a instancia do componente.)

Cada componente Angular requer uma única anotação `@Component` ou pelo menos uma `@View`. A anotação `@Component` especifica quando um componente é instanciado, e quais propriedades e hostListeners ele está vinculado.

Quando um componente é instanciado, Angular

* Cria uma shadow DOM para o componente.
* Carrega o template selecionado na shadow DOM.
* Cria um Injetor filho o qual está configurado com o appInjector para o componente.

Exemplo de um Componente:

``` javascript
@Component({                      | Anotação de Componente
  selector: 'pane',               | Seletor CSS sobre o elemento <pane>
  inputs: [                       | Lista qual propriedade precisa ser vinculada
    'title',                      |  - atributo title mapeado para a propriedade title do componente
    'open'                        |  - atributo open mapeado para a proriedade open do componente
  ],                              |
})                                |
@View({                           | Annotation Anotação de View
  templateUrl: 'pane.html'        |  - URL do template em HTML
})                                |
class Pane {                      | Classe controladora de Componente
  title:string;                   |  - propriedade title
  open:boolean;

  constructor() {
    this.title = '';
    this.open = true;
  }

  // Public API
  toggle() => this.open = !this.open; 
  open() => this.open = true;
  close() => this.open = false;
}
```

`pane.html`:
``` html
<div class="outer">
  <h1>{{title}}</h1>
  <div class="inner" [hidden]="!open">
    <content></content>
  </div>
</div>
```

`pane.css`:
``` css
.outer, .inner { border: 1px solid blue;}
.h1 {background-color: blue;}
```

Example of usage:
``` html
<pane #pane title="Example Title" open="true">
  Some text to wrap.
</pane>
<button (click)="pane.toggle()">toggle</button>

```
