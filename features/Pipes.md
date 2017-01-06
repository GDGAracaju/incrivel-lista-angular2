# Pipes
-----------------------------------
Pipes podem ser anexados no final de expressões para traduzir o valor para diversos formatos. Tipicamente usado para controlar a transformação para strings de números, datas, e outros dados, mas também pode ordenar, mapear, e reduzir arrays. Pipes podem ser encadeados. 

**NOTA:** Pipes são conhecidos com filtros em Angular 1.X.

A sintaxe de Pipes é:

``` javascript
<div class="movie-copy">
  <p>
    \{\{ model.getValue('copy') | async | uppercase \}\}
  </p>
  <ul>
    <li>
      <b>Estrelando</b>: \{\{ model.getValue('starring') | async \}\}
    </li>
    <li>
      <b>Gêneros</b>: \{\{ model.getValue('genres') | async \}\}
    </li>
  <ul>
</div>
```
