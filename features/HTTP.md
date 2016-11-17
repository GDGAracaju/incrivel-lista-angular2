# HTTP
-----------------------------------

HTTP é disponibilizado como uma classe injetável, com métodos para executar requisições HTTP. Requisições chamadas retornam um *EventEmitter (Emissor de Evento)* o qual emitirar uma única *Response* quando uma resposta é recebida. 

O método `toRx()`do *EventEmitter* precisa ser chamado para que se obtenha um objeto RxJS *Subject (Assunto)*. *EventEmitter* não provê combinadores como map, e tem difetentes semânticas para subscrição/observação(subscribing/observing).

## Uso

``` TypeScript
import {Http, HTTP_BINDINGS} from 'angular2/http';

@Component({selector: 'http-app', viewBindings: [HTTP_BINDINGS]})
@View({templateUrl: 'people.html'})
class PeopleComponent {
  constructor(http: Http) {
    http.get('people.json')
      // Obtém o objeto RxJS Subject
      .toRx()
      // Chama o método map (mapeamento) sobre a resposta observável para obter o objeto people (pessoas) mapeamento
      .map(res => res.json())
      // Subescreve-se para ser observável para obter o objeto people mapeado e anexá-lo ao
      // componente
      .subscribe(people => this.people = people);
  }
}
```

To use the *EventEmitter* returned by Http, simply pass a generator (See "interface Generator" in the Async Generator spec: https://github.com/jhusain/asyncgenerator) to the observer method of the returned emitter, with optional methods of next, throw, and return.

``` js
http.get('people.json').observer({next: (value) => this.people = people});
```

O construção padrão para realizar requisições, *XMLHttpRequest*, é abstraído como "Backend" (XHRBackend no caso), o qual pode ser simulado (mocked) através de injeção de dependências pela substituição do vínculo XHRBackend, como segue o seguinte exemplo:

``` TypeScript
import {MockBackend, BaseRequestOptions, Http} from 'angular2/http';

var injector = Injector.resolveAndCreate([
  BaseRequestOptions,
  MockBackend,
  bind(Http).toFactory(
      function(backend, defaultOptions) {
        return new Http(backend, defaultOptions);
      },
      [MockBackend, BaseRequestOptions])
]);
var http = injector.get(Http);
http.get('request-from-mock-backend.json').toRx().subscribe((res:Response) => doSomething(res));
```
