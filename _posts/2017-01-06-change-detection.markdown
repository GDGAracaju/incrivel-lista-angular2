# Detecção de mudanças
-----------------------------------

![Detecção de mudanças](http://36.media.tumblr.com/70d4551eef20b55b195c3232bf3d4e1b/tumblr_njb2puhhEa1qc0howo2_1280.png)
Todo componente tem um detector de mudanças responsável por checar os vínculos definidos em seu template. Exemplos de vínculos: `{{todo.text}}` e `[todo]="t"`. Detectores de mudança propaga vínculos[c] da raiz para as folhas em uma busca em profundidade.

Por padrão a detecção de mudança ocorre por todo nó da árvore para verificar o que foi mudado, e o que fazer a cada evento do navegador. Embora isto pareça terrivelmente ineficiente, o sistema de detecção de mudanças do Angular 2 pode percorrer milhares de checagens simples (números são dependentes da plataforma) em poucos milissegundos.

## Objetos imutáveis
Se um componente depender somente de seus vínculos, e seus vínculos são imutáveis, então o componente pode mudar, e somente mudar, se um de seus vínculos mudar. Portanto, podemos cortar a detecção de mudança nas subárvores de um componente até que um evento ocorra. Quando isto acontece, podemos checar a subárvore apenas uma única vez, e então, desabilitar até a próxima mudança (caixas em cinza indicam os detectores desabilitados)
![CD - Objetos imutáveis](http://40.media.tumblr.com/0f43874fd6b8967f777ac9384122b589/tumblr_njb2puhhEa1qc0howo4_1280.png)

Para implementar isto basta configurar a estratégia de detecção de mudança para `ON_PUSH`

``` javascript
@Component({changeDetection:ON_PUSH})
class ImmutableTodoCmp {
  todo:Todo;
}
```

## Objetos Observáveis
Se o componente depende apenas de seus vínculos, e os vínculos são observáveis, então este componente pode mudar se, e somente se, um de seus vínculos emitir um evento. Portanto, podemos cortar a detecção de mudanças na subárvore do componente até que um evento ocorra. Quando isto acontece, podemos checar a subárvore apenas uma única vez, e então desabilitá-la até a próxima mudança.

**NOTA:** Se você tem uma árvore de componentes com vínculos imutáveis, uma mudança tem que passar por todos os componentes desde a raiz. Este não é o caso quando tratamos com observáveis

``` javascript
type ObservableTodo = Observable<Todo>;
type ObservableTodos = Observable<Array<ObservableTodo>>;

@Component({selector:’todos’})
class ObservableTodosCmp {
  todos:ObservableTodos;
  //...
}
```

``` javascript
<todo *ng-for=”var t of todos” todo=”t”></todo>
```

``` javascript
@Component({selector:'todo'})
class ObservableTodoCmp {
  todo:ObservableTodo;
  //...
}
```

Digamos que a aplicação usa somente objetos observáveis. Quando ele inicia, Angular irá checar todos os objetos.

![CD - Ob 1](http://40.media.tumblr.com/b9a743a15d23c3db9f910f4c7566b928/tumblr_njb2puhhEa1qc0howo5_1280.png)

Após o primeiro passo irá parece com o seguinte:

![CD - Ob 2](http://40.media.tumblr.com/5f4ba2e53fb3de05f9c199199f4aae77/tumblr_njb2puhhEa1qc0howo6_1280.png)

Digamos que o primeiro observável todo dispara um evento. O sistema irá mudar para o seguinte estado:

![CD - Ob 3](http://40.media.tumblr.com/cb54aedb3479e1b0578ae2c6c8c7ccc2/tumblr_njb2puhhEa1qc0howo7_1280.png)

E depois de checar `App_ChangeDetector`, `Todos_ChangeDetector`, e o primeiro `Todo_ChangeDetector`, irá voltar para este estado.

![CD - Ob 4](http://40.media.tumblr.com/5f4ba2e53fb3de05f9c199199f4aae77/tumblr_njb2puhhEa1qc0howo6_1280.png)

Assumindo que as mudanças ocorrem raramente e os componentes formam uma árvore balanceada, usando mudanças observáveis a complexidade de detecção de mudanças vai de `O(N)` para `O(logN)`, onde N é o número de vínculos em um sistema.

**LEMBRE-SE**
- Uma aplicação Angular 2 é um sistema reativo.
- O sistema de detecção propaga dos vínculos da raiz as folhas.
- Diferente do Angular 1.X, o Grafo de detecção de mudanças é uma árvore direcionada. Como resultado, o sistema é mais performático e previsível.
- Por padrão, o sistema de detecção de mudanças percorre toda a árvore. Mas se você usa objetos imutáveis ou observáveis, você pode tomar a vantagem deles e apenas checar partes da árvore que "realmente mudam".
- Estas otimizações compõem e não quebram as garantias que o detector de mudanças provê.
