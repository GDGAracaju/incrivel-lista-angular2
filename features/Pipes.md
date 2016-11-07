# Pipes
-----------------------------------
Pipes can be appended on the end of the expressions to translate the value to a different format. Typically used
to control the stringification of numbers, dates, and other data, but can also be used for ordering, mapping, and
reducing arrays. Pipes can be chained.

Pipes podem ser acrescentados ao final de expressoes para traduzir valores em um formato diferente. Typicamente usado para controlar a stringificação de números, datas e outros formatos, mas também usado para ordenação, mapeamento, e redução de arrays. Pipes podem ser encadeados.

**NOTA:** Pipes são conhecidos com filtros em Angular 1.X.

A sintaxe de Pipes é:

``` javascript
<div class="movie-copy">
  <p>
    {{ model.getValue('copy') | async | uppercase }}
  </p>
  <ul>
    <li>
      <b>Starring</b>: {{ model.getValue('starring') | async }}
    </li>
    <li>
      <b>Genres</b>: {{ model.getValue('genres') | async }}
    </li>
  <ul>
</div>
```
