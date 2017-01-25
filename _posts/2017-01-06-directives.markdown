## Diretivas
Diretivas permitem que se adicione comportamento a elementos no DOM.

Diretivas são classes que são instanciadas em resposta a uma estrutura particular DOM. Pelo controle da estrutura DOM, quais diretivas são importadas e seus seletores, o desenvolvedor pode usar o [padrão Composição](https://pt.wikipedia.org/wiki/Composi%C3%A7%C3%A3o_de_objetos) para ter o desejado comportamento na aplicação.

Diretivas são o pilar de uma aplicação em Angular. Nós usamos Diretivas para quebrar problemas complexos em menores para componentes mais reutilizáveis. Diretivas permitem ao desenvolvedor torna HTML em uma [DSL(Linguagens específicas de Domínio)](https://pt.wikipedia.org/wiki/Linguagem_de_dom%C3%ADnio_espec%C3%ADfico) e então controlar o processo de montagem de aplicação.

Uma diretiva consiste de uma anotação de diretiva e uma classe controladora(controller). Quando o `seletor` da diretiva casa com elementos no DOM, os seguintes passos ocorrem:

* Para cada diretiva, o *ElementInjector* (Injetor de Elementos) tenta resolver os argumentos do construtor da diretiva.
* Angular instancia diretivas a cada elemento encontrado usando o *ElementInjector* em uma busca de ordem em profundidade, como declarado no HTML

Aqui há um exemplo trivial de um decorador de um tooltip. A diretiva irá logar no console toda vez que um mouse passar na região de um tooltip:

```javascript
@Directive({
  selector: '[tooltip]',     | O seletor CSS que dispara o decorador
  inputs: [                  | Lista quais propriedades precisam ser vinculados
    'text: tooltip'          |  - A propriedade do elemento DOM deve ser
  ],                         |    mapeado para propriedade de texto da diretiva.
  host: {                    | Lista quais eventos precisam ser mapeados
    '(mouseover)': 'show()'  |  - Invoca o método show() toda vez que
  }                          |    o evento mouseover (mouse sobre) é disparado.
})                           |
class Form {                 | Classe controladora de diretiva é instanciada,
                             | quando CSS é encontrado.
  text:string;               | propriedade de texto no Controller da Diretiva.
                             |
  show(event) {              | Método show() que implementa a ação de exibir no console.
    console.log(this.text);  |
  }
}
```

#### Exemplo de uso:
```html
<span tooltip="Texto do Tooltip.">Algum texto aqui.</span>
```
O desenvolvedor de uma aplicação agora pode usar livremente o atributo `tooltip` em qualquer lugar que este comportamento for necessário.
O código acima ensina o navegador um novo comportamento reutilizável e declarativo.

Note que o vínculo dos dados irá funcionar com este decorador sem mais nenhum esforço com é mostrado abaixo.

```html
<span tooltip="Olá {% raw %}{{usuario}}{% endraw %}!">Algum texto aqui.</span>
```

**NOTA:** Aplicações em Angular não tem método main. Ao invés disso eles tem um Componente raiz. A Injeção de Dependência monta as dependências para formar uma aplicação angular funcional.