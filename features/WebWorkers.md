# Web Workers (Trabalhadores Web)

Angular 2 inclui suporte nativo para escrita de aplicações que vivem em um WebWorker. Este documento descreve como escrever aplicações que tomam vantagem deste recurso. 
Também provê uma descrição detalhada da infraestrutura de mensagens subjacente que o Angular usa para comunicação entre o processo principal e os Workers. Esta infraestrutura que o angular usa pode ser modificado por um desenvolvedor de aplicações para executar uma aplicação Angular 2 de um iFrame, janelas / abas diferentes, servidores, etc...


## Introdução 

O suporte a WebWorker em Angular2 foi desenvolvido para que fosse fácil aumentar a paralelização em sua aplicação Web. 
Quanto você escolhe executar em um WebWorker Angular executa ambas lógica de aplicação e a maioria do núcleo do framework Angular em um WebWorker.
Por descarregar o tanto de código quanto possível em um WebWorker mantemos a thread de Interface de Usuário livre para manipular eventos, manipular o DOM, e executar animações. Isto provê uma melhor taxa de quadros e Experiência de Usuário para aplicações.

## Inicialização (Bootstrapping) de uma  Aplicação WebWorker
Inicialização de uma aplicação WebWorker não é muito diferente de uma Inicialização de uma aplicação comum.
A diferença primária é que você não passa o componente raiz diretamente em ```bootstrap```.
Ao invés disso, você passa o nome de um script de background que chama ```bootstrapWebWorker```com seu componente raiz. 

### Examplo
Para inicializar Hello World em WebWorker precisamos fazer o seguinte em TypeScript:
```HTML
<html>
  <head>
     <script src="https://github.jspm.io/jmcriffey/bower-traceur-runtime@0.0.87/traceur-runtime.js"></script>
     <script src="https://jspm.io/system@0.16.js"></script>
     <script src="angular2/web_worker/ui.js"></script>
  </head>
  <body>
    <hello-world></hello-world>
    <script>System.import("index")</script>
  </body>
</html>
```
```TypeScript
// index.js
import {bootstrap} from "angular2/web_worker/ui";
bootstrap("loader.js");
```
```JavaScript
// loader.js
importScripts("https://github.jspm.io/jmcriffey/bower-traceur-runtime@0.0.87/traceur-runtime.js", "https://jspm.io/system@0.16.js", "angular2/web_worker/worker.js");
System.import("app");
```
```TypeScript
// app.ts
import {Component, View, bootstrapWebWorker} from "angular2/web_worker/worker";
@Component({
  selector: "hello-world"
})
@View({
  template: "<h1>Olá {{name}}</h1>
})
export class HelloWorld {
  name: string = "Jane";
}

bootstrapWebWorker(HelloWorld);
```
Há algumas coisas importantes para tomar nota aqui:
* Do lado da Interface nós importamos todos os tipos de `angular2/web_worker/ui` e sobre o lado do Worker nós importamos de `angular2/web_worker/worker`. Estes módulos incluem todos as declarações no pacote WebWorker. Por importar destas URLs invés de `angular2/angular2` podemos assegurar estaticamente que nossa aplicação não faz referências a um tipo que não existe no context que se procura executar. Por exemplo, se tentássemos importar DomRenderer em um WebWorker ou NgFor na Interface nós podemos obter um erro de compilação.
* A Interface carrega o Angular do arquivo `angular2/web_worker/ui.js` e o Worker carrega o angular do `angular2/web_worker/worker.js`. Estes pacotes são criados especificamente para ser usados com WebWorkers e devem ser usados ao invés do arquivo `angular2.js` normal. Ambos os arquivos contêm  subconjuntos de código base que foi desenvolvido para ser executado especificamente sobre a Interface de Usuário ou Worker. Adicionalmente, eles contêm a infraestrutura núcleo de mensageria que é usado para comunicação entre o IU ou Worker. Este código de mensageria não está no arquivo `angular2.js` padrão.
* Passamos `loader.js`para Inicialização e não `app.ts`. Você pode pensar de `loader.js` como uma `index.html` de um Worker. Como WebWorkers não compartilham memória com a IU precisamos recarregar as dependências do Angular 2 antes de iniciarmos nossa aplicação. Fazemos isto com importScripts. Adicionalmente, precisamos fazer o carregamente em um arquivo diferente do arquivo `app.ts`porque nosso carregador de módulos(System.js neste exemplo) não foi carregado ainda, e `app.ts` irá ser compilado com uma chamada a System.define no seu topo. 
* O componente HelloWorld parece exatamente como um componente Angular 2 HelloWorld comum! O objetivo do suporte a WebWorkers foi para permitir que muito do Angular vivesse em um worker o quanto possível. Como tal, *maior* parte dos componentes Angular 2 pode ser inicializados em um WebWorker com nenhuma ou pouquíssimas mudanças requeridas. 

Para referência, aqui o mesmo exemplo de HelloWorld em Dart.
```HTML
<html>
  <body>
    <script type="application/dart" src="index.dart"></script>
    <script src="packages/browser.dart.js"></script>
  </body>
</html>
```
```Dart
// index.dart
import "package:angular2/web_worker/ui.dart";
import "package:angular2/src/core/reflection/reflection.dart";
import "package:angular2/src/core/reflection/reflection_capabilities.dart";

main() {
  reflector.reflectionCapabilities = new ReflectionCabilities();
  bootstrap("app.dart");
}
```
```Dart
import "package:angular2/web_worker/worker.dart";
import "package:angular2/src/core/reflection/reflection.dart";
import "package:angular2/src/core/reflection/reflection_capabilities.dart";

@Component(
  selector: "hello-world"
)
@View(
  template: "<h1>Olá {{name}}</h1>"
)
class HelloWorld {
  String name = "Jane";
}

main(List<String> args, SendPort replyTo) {
  reflector.reflectionCapabilities = new ReflectionCapabilities();
  bootstrapWebWorker(replyTo, HelloWorld);
}

```
Este código é quase o mesmo da versão em TypeScript com apenas algumas poucas diferenças chave:  
* Não temos um arquivo `loader.js`. Aplicações Dart não precisam deste arquivo porque você não vai precisar de um carregador de módulos.
* Passamos um `SendPort` para `bootstrapWebWorker`. Aplicações Dart usam a API Isola, o qual comunica-se por abstrações de portas do Dart. Quando você chama `bootstrap`da thread de IU, Angular inicia um novo Isolate para executar sua lógica de aplicação. Quando Dart inicia um novo Isolate ele passa um `SendPort`ára o Isolate para que possa se comunicar com o Isolate que a criou. Você precisa passar uma `SendPort` para `boostrapWebWorker`para que o Angular possa se comunicar com IU.
* Você precisará configurar `ReflectionCapabilities(Capacidades de Reflexão)` em ambos IU e Worker. Da mesma forma de escrita de aplicações as não-concorrentes em Angular 2 Dart você precisa configurar o refletor(reflector). Você não deve usar Reflecção(Reflection) em produção, mas deve usar o transformador Angular 2 para removê-lo em seu código final em JavaScript. Note que há um bug com a execução do transformado com seu código IU (#3971). Você pode (e deve) passar o arquivo onde você chama `bootstrapWebWorker` como um ponto de entrada para o transformador, mas você não deve passar seu arquivo index da IU para o transformado até o bug ser consertado. 

## Escrevendos componentes Compatíveis com WebWorker
Você pode fazer quase tudo em um componente WebWorker que possa ser feito em um Componente típico Angular 2.
A princípal execeção é que **não** há acesso ao DOM de um componente WebWorker. Em Dart isto significa que você não pode importar nada de `dart:html` e em JavaScript isto significa que você não pode usar `document` ou `window`. Ao invés disso, você deve usar vínculos de dados e ser for necessário você pode injetar o `Renderer` junto com seu `ElementRef` do componente e usar métodos como `setElementProperty`, `setElementAttribute`, `setElementClass`, `setElementStyle`, e `setText`. Não que você *não possa*  chamar `getNativeElementSync`. Fazendo isto irá retornar sempre `null` quando executado em um WebWorker. 
Se você precisa de acesso ao DOM veja [Executando código sobre a IU](#xecutando-código-sobre-a-iu).

## Visão geral do Design WebWorker
Quanto sua aplicação é executada em um Webworker, a maior parte do núcleo do Angular justo com a lógica de aplicação é executando em um Worker. Os dois principais componentes que executam sebre a IU são `Renderer` e o `RendererCompiler`. Quando se executa Angular em um WebWorker os vínculos destes dois componentes são substituidos por `WebWorkerRenderer` e `WebWorkerRenderCompiler`. Quando estes componentes são usados em tempo de execução, eles passam mensagens através do [MessageBroker (Coordenador de Mensagens)](#messagebroker) instruindo a IU para executar o método de fato e retorna o resultado. A abstração do [MessageBroker](#messagebroker) permite que qualquer lado da fronteira do WebWorker agendar código a ser executado  no lado oposto e receber o resultado. 
Adicionalmente, o [MessageBroker](#messagebroker) roda sobre o [MessageBus (barramento de mensagens)](#messagebus).
MessageBus é uma abstração de baixo nível que provês uma API agnóstica a linguagens para comunicação de componentes Angular através de qualquer fronteira de tempo de execução como a comuinicação `WebWorker <--> IU` , comunicação `UI <--> Servidor`. ou comunicação `Window <--> Window`. 

Veja o diagrama abaixo para uma Visão Geral de Alto Nível de como este código é estruturado:

![Diagrama WebWorker](http://stanford.edu/~jteplitz/ng_2_worker.png)

## Executando Código Sobre a IU
Se sua aplicação necessitar de executar código sobre a IU, há algumas opções. O modo mais fácil é usar um CustomElement(Elemento Customizado) em sua View. Você pode registrar este elemento customizado de seu arquivo HTML e executar código em resposta ao ganchos de ciclo de vida do elemento. Note, Elementos Customizados ainda são esperimentais. Veja [MDN](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements) para os últimos detalhes sobre como usá-los.

Se você requer mais robustez na comunicação entre WebWorker e a IU você pode usar o [MessageBroker](#usando-o-messagebroker-em-sua-aplicação) ou [MessageBus](#usando-o-messagebus-em-sua-aplicação) diretamente.

## MessageBus (Barramento de Mensagenss)
O MessageBus é uma abstração de baixo nível que provê uma API agnóstica a linguagem para comunicação com componentes Angular através da fronteira de tempo de execução. Suporta comunicação multiplex através do uso de abstração de canais.

Angular atualmente inclui duas implementações estáveis de MessageBus, o qual é utilizado por padrão quando você executa sua aplicação em WebWorker,

1. O `PostMessageBus(Enviar para o Barramento de Mensagens)` é utilizado em aplicações JavaScript para comunicação entre um Isolate em backgroud e a IU.
2. O `IsolateMessageBus (Barramento de Mensagens Isolada) é usado por aplicações Dart para comunicação entre um Isolate em Background para a IU.

Angular também inclui três implementações experimentais de MensageBus: 


1. O `WebSocketMessageBus`é um MessageBus Dart que vvi na IU e comunica-se com uma aplicação Angular rodando no servidor. Foi feito para ser utilizado tanto com `SingleClientServerMessageBus` quanto `MultiClientServerMessageBus`. 
2. O `SingleClientServerMessageBus`é um MesageBus Dart que vivew em um servidor Dart. Permite que uma aplicação Angular rode em um servidor e se comunique com um único browser que rode `WebSocketMessageBus`.
3. O `MultiClientServerMessageBus` é como o `SingleClientServerMessageBus` exceto que permite que um número arbitrário de clientes se conectem ao servidor. Isto mantém todos os browser conectados em sincronia e se um evento é disparado em algum browser conectado o resultado é propagado para todos os clentes conectados. Isto pode ser especialmente útil como ferramenta de depuração, por permitir a você conectar a multiplos browser / dispositivos para a mesma aplicação Angular, mudar o estado daquela aplicação. e assegurar que todos os clientes renderizam a View corretamente. Usando estas ferramentas facilitam encontrar problemas de compatibilidade de browser mais rápido. 

### Usando o MessageBus em sua Aplicação
**Nota**: Se você quiser passar mensagens customizadas entre a IU e WebWorker, é recomendável você usar o [MessafeBroker](#usando-o-messagebroker-em-sua-aplicação). Porém, se queser controler o protocolo de mensagem por você mesmo, você pode usar o MessageBus diretamente.

Para usar o MessageBus você precisa inicializar um novo canal sobre o IU e WebWorker.
Em TypeScript isto irá parecer com o seguinte:
```TypeScript
// index.ts, which is running on the UI.
var instance = bootstrap("loader.js");
var bus = instance.bus;
bus.initChannel("Meu canal customizado");
```
```TypeScript
// background_index.ts, que está rodando sobre WebWorker
import {MessageBus} from 'angular2/web_worker/worker';
@Component({...})
@View({...})
export class MyComponent {
  constructor (bus: MessageBus) {
    bus.initChannel("Meu canal customizado");
  }
}
```

Uma vez que o canal tenha sido inicializado ambos os lados podem ser usados  tanto os métodos `from` e `to` em MessageBus para enviar e receber mensagens respectivamente. Ambos os métodos retornam EventEmitter. Expandindo o exemplo anterior: 
```TypeScript
// index.ts, o quall está rodando na IU.
import {bootstrap} from 'angukar2/web_worker/ui';
var instance = bootstrap("loader.js");
var bus = instance.bus;
bus.initChannel("Meu canal customizado");
bus.to("Meu canal customizado").next("Olá da IU");
```
```TypeScript
// background_index.ts, o qual está rodando no WebWorker
import {MessageBus, Component, View} from 'angular2/web_worker/worker';
@Component({...})
@View({...})
export class MyComponent {
  constructor (bus: MessageBus) {
    bus.initChannel("Meu canal customizado");
    bus.from("Meu canal customizado").observer((message) => {
      console.log(message); // irá imprimir "Olá da UI"
    });
  }
}
```
Este exemplo é bastante identico em Dart, e incluimos por referência:
```Dart
// index.dart, que está sendo rodade na IU.
import 'package:angular2/web_workers/ui.dart';

main() {
  var instance = bootstrap("background_index.dart");
  var bus = instance.bus;
  bus.initChannel("Meu canal customizado");
  bus.to("Meu canal customizado").add("Olá da IU");
}

```
```Dart
// background_index.dart, which is running on the WebWorker
import 'package:angular2/web_worker/worker.dart';
@Component(...)
@View(...)
class MyComponent {
  MyComponent (MessageBus bus) {
    bus.initChannel("Meu canal customizado");
    bus.from("Meu canal customizado").listen((message) {
      print(message); // irá imprimir "Olá da IU"
    });
  }
}
```

A única diferença substancial entre APIs Dart e TypeScript é a API para o `EventEmitter`.

**Nota** Por causa das mensagens passadas através do MessageBus crusa a fronteira do WebWorker, ele precisam ser serializados.
Se você usar o MessageBus diretamente, você é responsável por serializar suas mensagens.
Em JavaScript / TypeScript isto significa que ele deve ser serializável via [algoritmo de clone estruturado] (https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Structured_clone_algorithm).

Em Dart isto significa que as mensagens devem ser validas para serem passadas por um [SendPort](https://api.dartlang.org/1.12.1/dart-isolate/SendPort/send.html).

### MessageBus e Zones
A API MessageBus inclui support para [Zones(Zonas)](http://www.github.com/angular/zone.js).
Um MessageBus pode ser anexada a uma específia zona(zone)(através de chamada a `attachToZone`). Então canais específicos podem ser especificados para executar na zona quando eles forem inicializados.
Se um canal é executado na zona, isto significa que qualquer evento emitido daquele canal irá ser executado dentro de uma dada zona. Por exemplo, por padrão Angular executa o canal EventDispatch (Dispachador de Eventos) dentro da zona do Angular. Isto significa que quando um evento é disparado de um DOM e recebido em um WebWorker o manipulador de evento automaticamente executa dentro da zona Angular Isto é desejado porque depois do manipulador de evento sair nós desejamos sair da zona para que disparemos a detecção de mudanças. Geralmente, você quer seus canais rodem dentro de zonas a menos que você tenha uma boa razão para executar fora da zona.

### Implementando e Usando um MessageBus Customizada
**Nota** Implementar e usar um MessageBuss Customizada é experimental e requer importar de APIs privadas.

Se você quer execuar sua aplicação de alguma outra coisa que um WebWorker você pode implementar um barramento de mensagens customizada. Implementar apenas significa criar uma classe que preencha a API especificada pela classe abstrata MessageBus.

Se você está implementando seu MessageBus em Dart você pode extender a classe `GenericMessageBus` incluso no Angular. Se o fizer, você não precisará implementar suporte a canais nem zonas você mesmo. Você apenas precisará implementar uma `MessageBusSink` que extende `GenericMessageBusSink`e um `MessageBusSource` que extende de `GenericMessageBusSource`. O `MessageBusSink` deve sobrescrever o método `sendMessages`. Este método é dado uma lista de mensagens serializadas que é requerido para enviar pelo coletor.
O `MessageBusSource` precisa provê um [Stream(fonte)](https://api.dartlang.org/1.12.1/dart-async/Stream-class.html) de mensagens recebidas (tanto por passar a fonte para o constructor de `GenericMessaBusSource` ou chamar attachTo() com a fonte). Também é necessário sobrescrever o método abstrato `decodeMessages`. Este método é dado uma lista de mensagens serializadas recebidas pela origem e deve executar qualquer trabalho de decodificação que seja necessário antes de a aplicação possa ler as mensagens.

Por exemplo, se seu MessageBus envia e recebe dados em JSON você poderia fazer o seguinte: 
```Dart
import 'package:angular2/src/web_workers/shared/generic_message_bus.dart';
import 'dart:convert';

class JsonMessageBusSink extends GenericMessageBusSink {
  @override
  void sendMessages(List<dynamic> messages) {
    String encodedMessages = JSON.encode(messages);
    // Send encodedMessages here
  }
}

class JsonMessageBusSource extends GenericMessageBuSource {
  JsonMessageBusSource(Stream incomingMessages) : super (incomingMessages);

  @override
  List<dynamic> decodeMessages(dynamic messages) {
    return JSON.decode(messages);
  }
}
```
Uma vez que você tenha implementado seu MessageBus customizado tanto em TypeScript ou Dart você pode dizer ao Angular para usar como segue:
Em TypeScript:
```TypeScript
// index.ts, executando no lado do IU
import {bootstrapUICommon} from 'angular2/src/web_workers/ui/impl';
var bus = new MyAwesomeMessageBus();
bootstrapUICommon(bus);
```
```TypeScript
// background_index.ts, executando no lado da aplicação
import {bootstrapWebWorkerCommon} from 'angular2/src/web_workers/worker/application_common';
import {MyApp} from './app';
var bus = new MyAwesomeMessageBus();
bootstrapWebWorkerCommon(MyApp, bus);
```
Em Dart:
```Dart
// index.dart, executando no lado do IU
import 'package:angular2/src/web_workers/ui/impl.dart' show bootstrapUICommon;
import "package:angular2/src/core/reflection/reflection.dart";
import "package:angular2/src/core/reflection/reflection_capabilities.dart";

main() {
  reflector.reflectionCapabilities = new ReflectionCapabilities();
  var bus = new MyAwesomeMessageBus();
  bootstrapUiCommon(bus);
}
```
```Dart
// background_index.dart, executando no lado da aplicação
import "package:angular2/src/web_workers/worker/application_common.dart" show bootstrapWebWorkerCommon;
import "package:angular2/src/core/reflection/reflection.dart";
import "package:angular2/src/core/reflection/reflection_capabilities.dart";
import "./app.dart" show MyApp;

main() {
    reflector.reflectionCapabilities = new ReflectionCapabilities();
    var bus = new MyAwesomeMessageBus();
    bootstrapWebWorkerCommon(MyApp, bus);
}
```

Note como chamamos `bootstrapUICommon`invés de `bootstrap` do lado da IU. `bootstrap` cria um novo WebWorker / Isolate e anexa o MessageBus padrão do Angular nele. Se você estiver usando um customizado você deve ser o responsável por configurar a aplicação e inicializar a comunicação usando ele. `bootstrapUiCommon` assume que o dado MessageBus já está configurado e pode comunicar com a aplicação. De modo similar, podemos chamar `bootstrapWebWorkerCommon` invés de `bootstrapWebWorker` do lado da aplicação. Isto é porque `bootstrapWebWorker` assume que você está usando o MessageBus padrão e inicializa um novo para você. 

## MessageBroker
O MessageBroker é uma abstração de alto nível que ficar acima do MessageBus. É usado quando você quer executar código no outro lado de uma fronteira de tempo de execução e pode querer receber o resultado. 
Há dois tipos de MessageBrokers. O `ServiceMessageBroker`é usado pelo lado que realmente executa uma operação e pode retornar um resultado. Inversamente, o `ClientMessageBroker` é usado pelo lado que requisita  uma operação para ser executado e pode querer receber o resultado.

### Usando o MessageBroker em sua Aplicação
Para usar MessageBrokers em sua aplicação você deve inicializar ambos um `ClientMessageBroker` e um `ServiceMessageBroker` em um mesmo canal. Você pode registrar métodos com  o `ServiceMessageBroker`e instruir o `ClientMessageBroker` a executar estes métodos. Abaixo está um exemplo leve do uso de MessageBrokers em uma aplicação. Para um exemplo mais completo, veja o `WebWorkerRenderer`e `MessageBasedRenderer` dentro de um código Angular WebWorker.

#### Usando o MessageBroker em TypeScript
```TypeScript
// index.ts, que está rodando na IU com um método que nósqueremos expôr a um WebWorker
import {bootstrap} from 'angular2/web_worker/ui';

var instance = bootstrap("loader.js");
var broker = instance.app.createServiceMessageBroker("Meu canal Broker");

// assuma que você tenha alguma função chamada doCoolThings que dado uma string retorna uma Promise<string>
broker.registerMethod("awesomeMethod", [PRIMITIVE], (arg1: string) => doCoolThing(arg1), PRIMITIVE);
```
```TypeScript
// background.ts, o qual roda em um WebWorker e quer executar um método na IU
import {Component, View, ClientMessageBrokerFactory, PRIMITIVE, UiArguments, FnArgs}
from 'angular2/web_worker/worker';

@Component(...)
@View(...)
export class MyComponent {
  constructor(brokerFactory: ClientMessageBrokerFactory) {
    var broker = brokerFactory.createMessageBroker("Meu canal Broker");

    var arguments = [new FnArg(value, PRIMITIVE)];
    var methodInfo = new UiArguments("awesomeMethod", arguments);
    broker.runOnService(methodInfo, PRIMTIVE).then((result: string) => {
      // resultado irá ser igual ao valor retornado em doCoolThing(valor) que foi executado na IU.
    });
  }
}
```
#### Usando o MessageBroker em Dart
```Dart
// index.dart, o qual está rodando na IU com um método que queremos expor em um WebWorker
import 'package:angular2/web_worker/ui.dart';

main() {
  var instance = bootstrap("background.dart");
  var broker = instance.app.createServiceMessageBroker("Meu canal Broker");

  // assuma que temo alguma função doCoolThings que tem como argumento de entrada uma String e retona um Future<String>
  broker.registerMethod("awesomeMethod", [PRIMITIVE], (String arg1) => doCoolThing(arg1), PRIMITIVE);
}

```
```Dart
// background.dart, o qual está sendo executado em um WebWorker e espera executar um método na IU
import 'package:angular2/web_worker/worker.dart';

@Component(...)
@View(...)
class MyComponent {
  MyComponent(ClientMessageBrokerFactory brokerFactory) {
    var broker = brokerFactory.createMessageBroker("Meu canal Broker");

    var arguments = [new FnArg(value, PRIMITIVE)];
    var methodInfo = new UiArguments("awesomeMethod", arguments);
    broker.runOnService(methodInfo, PRIMTIVE).then((String result) {
      // o resultado será igual ao valor retornado de doCoolThin(gvalor) que rodou na IU.
    });
  }
}
```
Ambos o cliente e o serviço criam novos MessageBrokers e anexam a eles ao mesmo canal. 
O serviço então chama `registerMethod` para registrar o método que se espera escutar. `registerMethod` possui quatro argumentos O primeiro é o nome do método, o segundo são os tipos(Types) dos parâmetros daquele método, o terceiro é o próprio método, e o quarto (que é opcional) é o tipo de retorno do método. 
O MessageBroker controla a serialização / deserialização de seus parâmetros e retorna tipos usando o serializador do Angular. Porém, no momento o serializador apenas sabe como serializar classes Angular como aqueles usados por Renderer.
Se você passar qualquer outro além daquele tipos em volta da sua aplicação você pode controlar a serialização por você mesmo e então usar o tipo `PRIMITIVE` par dizer ao MessageBroker para prevenir a serialização de seus dados.

A última coisa está acontecendo é o cliente chama `runOnService` com o nome do método que deseja roda, uma lista de argumentos e seus tipos, e (opcionalmente) o tipo de retorno experado.