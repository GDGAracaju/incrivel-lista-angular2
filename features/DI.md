# Injeção de Dependências
-----------------------------------

## Principais abstrações

A biblioteca é construída sobre as seguintes abstrações: `Injetor`, `Vínculo` e `Dependência`.

 * Um injetor é criado de um conjunto de vínculos.
 * Um injetor resolve dependências e cria objetos.
 * Um vínculo mapeia um token, como uma string ou classe, a uma função de fábrica e uma lista de dependências. Então um vínculo define como é criado um objeto.
 * Uma dependência aponta para um token e contém mais informações detalhadas em como um objeto correspondente a aquele token deve ser injetado.

```
[Injetor]
    |
    |
    |*
[Vínculo]
   |----------|-----------------|
   |          |                 |*
[Token][Função de Fábrica] [Dependência]
                               |---------|
                               |         |
                            [Token]   [Flags]
```

## Exemplo

``` javascript
class Engine {
}

class Car {
	constructor(@Inject(Engine) engine) {
	}
}

var inj = Injector.resolveAndCreate([
	bind(Car).toClass(Car),
	bind(Engine).toClass(Engine)
]);
var car = inj.get(Car);
```

Neste exemplo criamos dois vínculos: um para Car e outro para Engine. `@Inject(Engine)` declara uma dependência em Engine.

## Injetor

Um injetor instancia objetos preguiçosamente(ou lazy load), somente quando for chamado, e então faz cache dele.

Compare

``` javascript
var car = inj.get(Car); //instancia ambos Car e Engine
```

com

``` javascript
var engine = inj.get(Engine); //Instancia Engine
var car = inj.get(Car); //Instancia um Car (reusa Engine)
```

e com

``` javascript
var car = inj.get(Car); //instancia ambos Engine e Car
var engine = inj.get(Engine); //lê Engine do cache
```
Para evitar bugs, tenha certeza que os objetos registrados tenham construtores livres de efeitos colaterais. Neste caso, um injetor atua como um hashmap, onde a ordem com o que os objetos são criado não importa.

## Injetores filhos e Dependências

Injetores são hierárquicos.

``` js
var parent = Injector.resolveAndCreate([
  bind(Engine).toClass(TurboEngine)
]);
var child = parent.resolveAndCreateChild([Car]);

var car = child.get(Car); // usa o vínculo de Car do Injetor filho e Engine do injetor pai.
```
Injetores formam uma árvore.

```
      InjectorAvô
   /              \
InjectorPai1  InjectorPai2
  |
InjetorFilho1
```
O algoritmo de injeção de dependência funciona como segue abaixo:

``` js
// isto é um pseudocódigo.
var inj = this;
while (inj) {
  if (inj.hasKey(requestedKey)) {
    return inj.get(requestedKey);
  } else {
    inj = inj.parent;
  }
}
throw new NoProviderError(requestedKey);
```

Então no exemplo seguinte:

``` js
class Car {
  constructor(e: Engine){}
}
```

Injetor de Dependências começa resolvendo `Engine` com o mesmo injetor que o vínculo `Car` está definido. Irá checar caso aquele injetor tem vínculo com `Engine`. Se este for o caso, irá retornar a instância. Se não, o injetor irá perguntar ao pai caso tenha uma instância de `Engine`. O processo continua até encontrar uma instância de `Engine`, ou atingirmos a raiz da árvore de injetores.

### Restrições

Você pode colocar restrições superior e inferior a uma dependência. Por exemplo, o decorador `@Self` diz ao Injetor de Dependência para procurar por `Engine` somente no mesmo injetor onde `Car` foi definido. Então não irá passar por toda a árvore.

``` js
class Car {
  constructor(@Self() e: Engine){}
}
```

Um exemplo mais realista é tendo dois vínculos que tem de ser providos juntos(Ex.: NgModel e NgRequiredValidator.)

O decorador `@Host` diz ao Injetor para procurar `Engine` em seu injetor, partente, até alcançar seu hospedeiro(host)(ver próxima seção sobre hospedeiros.)

``` js
class Car {
  constructor(@Host() e: Engine){}
}
```

O decorador `@SkipSelf` diz ao Injetor para procurar por `Engine` em toda a árvore começando pelo pai do injetor.

``` js
class Car {
  constructor(@SkipSelf() e: Engine){}
}
```

#### Injetor de Dependências não navega para baixo

A resolução de dependências somente navega para cima sua árvore. O exemplo a seguir irá lançar exceção pois o Injetor irá procurar uma instância de `Engine` começando do `pai`.

``` js
var parent = Injector.resolveAndCreate([Car]);
var child = parent.resolveAndCreateChild([
  bind(Engine).toClass(TurboEngine)
]);

parent.get(Car); // irá lançar NoProviderError
```


## Vínculos

Você mpode vincular uma classe, um valor, ou uma fábrica. É possível criar alias para vínculos existentes.

``` js
var inj = Injector.resolveAndCreate([
  bind(Car).toClass(Car),
  bind(Engine).toClass(Engine)
]);

var inj = Injector.resolveAndCreate([
  Car,  // açucar sintático para bind(Car).toClass(Car)
  Engine
]);

var inj = Injector.resolveAndCreate([
  bind(Car).toValue(new Car(new Engine()))
]);

var inj = Injector.resolveAndCreate([
  bind(Car).toFactory((e) => new Car(e), [Engine]),
  bind(Engine).toFactory(() => new Engine())
]);
```

Você pode vincular qualquer token.

``` js
var inj = Injector.resolveAndCreate([
  bind(Car).toFactory((e) => new Car(), ["engine!"]),
  bind("engine!").toClass(Engine)
]);
```
Se quiser criar um alias para um vínculo existente, você pode usar `toAlias`:

``` js
var inj = Injector.resolveAndCreate([
  bind(Engine).toClass(Engine),
  bind("engine!").toAlias(Engine)
]);
```
O que implica em `inj.get(Engine) === inj.get("engine!")`.

Note quew tokens e fábricas são desacoplados.

``` js
bind("algum token").toFactory(someFactory);
```
A função `someFactory` não tem que saber quem cria um objeto `algum token`.

### Resolvendo vínculos

Quando o Injetor recebe `bind(Car).toClass(Car)`, precisa fazer algumas coisas antes de criar uma instância de `Car`:

- Precisa refletir sobre `Car` para criar uma função de fábrica.
- Precisa normalizar as dependências (Ex., calcular os limites alto e baixo)

O resultado destas duas operações é um `ResolvedBinding`.

As funções  `resolveAndCreate` e `resolveAndCreateChild` resolve os vínculos antes de criar um injetor. Mas você pode criar resoluções de vínculos manualmente através de `Injector.resolve([bind(Car).toClass(Car)])`. Criando um injetor de um injetor pré resolvido é muito mais performático, e poderá ser necessário para áreas que sejam sensíveis a performance.

Você pode criar um injetor usando uma lista de vínculos resolvidos.

``` js
var listOfResolvingBindings = Injector.resolve([Binding1, Binding2]);
var inj = Injector.fromResolvedBindings(listOfResolvingBindings);
inj.createChildFromResolvedBindings(listOfResolvedBindings);
```


### Dependências Transientes

Um injetor tem apenas uma instância criada para cada vínculo registrado.

``` js
inj.get(MyClass) === inj.get(MyClass); //always holds
```

Se precisarmos de dependências transientes, alguma coisa que queremos uma nova instância a cada vez temos duas opções.

Podemos criar um injetor filho para cada nova instância:

``` js
var child = inj.resolveAndCreateChild([MyClass]);
child.get(MyClass);
```
Ou podemos registror uma função de fábrica:

``` js
var inj = Injector.resolveAndCreate([
  bind('MyClassFactory').toFactory(dep => () => new MyClass(dep), [SomeDependency])
]);

var factory = inj.get('MyClassFactory');
var instance1 = factory(), instance2 = factory();
// Dependente da implementação de MyClass, mas geralmente a propriedade mantém.
expect(instance1).not.toBe(instance2);
```
