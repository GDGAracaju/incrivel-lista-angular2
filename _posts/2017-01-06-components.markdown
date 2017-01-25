## Componentes
Um componente é uma diretiva o qual usa shadow DOM para criar comportamentos visuais encapsulados. Componentes são tipicamente usados para criar elementos visuais ou partir a aplicação em componentes menores.

* Apenas um componente pode está presente por elemento DOM.
* Seletores de componentes CSS comumente são disparados pelo nome de seu elemento. (Boas práticas)
* Componentes tem seu próprio shadow view o qual é adicionado ao elemento como um Shadow DOM.
* Contexto de Shadow View é a instância de um componente. (Ex.: expressões de templates são avaliados contra a instância do componente.)

Cada componente Angular requer uma única anotação `@Component` ou pelo menos uma `@View`. A anotação `@Component` especifica quando um componente é instanciado e, quais propriedades e hostListeners ele está vinculado.

Quando um componente é instanciado, Angular

* Cria uma shadow DOM para o componente.
* Carrega o template selecionado na shadow DOM.
* Cria um Injetor filho o qual está configurado com o appInjector para o componente.

#### Exemplo de um Componente:
```javascript
@Component({                      | Anotação de Componente
  selector: 'pane',               | Seletor CSS sobre o elemento <pane>
  inputs: [                       | Lista qual propriedade precisa ser vinculada
    'title',                      |  - atributo title mapeado para a propriedade title do componente
    'open'                        |  - atributo open mapeado para a propriedade open do componente
  ],                              |
})                                |
@View({                           | Anotação de View
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

#### pane.html:
```html
<div class="outer">
  <h1>{% raw %}{{title}}{% endraw %}</h1>
  <div class="inner" [hidden]="!open">
    <content></content>
  </div>
</div>
```

#### pane.css:
```css
.outer, .inner { border: 1px solid blue;}
.h1 {background-color: blue;}
```

#### Exemplo de uso:
```html
<pane #pane title="Example Title" open="true">
  Some text to wrap.
</pane>
<button (click)="pane.toggle()">toggle</button>
```