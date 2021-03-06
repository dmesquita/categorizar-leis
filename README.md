
# Categorização de Leis a partir das ementas
by Déborah Mesquita



## Etapa 1: carregar o dataset


```python
import pandas as pd

file_path = 'Leis_1992_2016.csv'
leis = pd.read_csv(file_path)
leis.ementa = leis.ementa.astype(str)
leis.ementa = leis.ementa.str.lower()
```


```python
# Durante a análise descobri que algumas ementas estavam com a primeira palavra duplicada

def clear_first_word_duplicate(ementa):
    words = ementa.split(" ")
    if(len(words) > 2 and words[0] == words[1]):
        return " ".join(words[1:])
    else:
        return ementa

leis['ementa'] = leis.apply(lambda row: clear_first_word_duplicate(row['ementa']), axis=1)

leis.ementa = leis.ementa.astype(str)
```


```python
print("Quantidade de leis: ", len(leis))
```

    Quantidade de leis:  821



```python
leis.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1012</td>
      <td>1992</td>
      <td>reajusta os vencimentos do pessoal da prefeitu...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1013</td>
      <td>1992</td>
      <td>autoriza ao poder executivo desafetar áreas de...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1014</td>
      <td>1992</td>
      <td>concede pensão financeira à viuva do ex-vice-p...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1015</td>
      <td>1992</td>
      <td>cria cargos (funções) para efeito de concursos...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1016</td>
      <td>1992</td>
      <td>dá nova redação à alínea "a" do artigo 1º da l...</td>
    </tr>
  </tbody>
</table>
</div>



## Etapa 2: identificar uma teoria de predição
Ao explorar o dataset, foi possível notar que a primeira palavra de cada ementa representa bem uma categorização inicial


```python
from collections import Counter

total_word_frequency = Counter()

for index,row in leis.iterrows():
    ementa = row['ementa']
    for word in ementa.split(" "):
        total_word_frequency[word] += 1
```


```python
total_word_frequency.most_common()
```




    [('de', 1280),
     ('e', 908),
     ('do', 735),
     ('outras', 590),
     ('dá', 585),
     ('providências.', 483),
     ('a', 475),
     ('o', 426),
     ('da', 385),
     ('municipal', 274),
     ('município', 223),
     ('sobre', 209),
     ('lei', 200),
     ('dispõe', 191),
     ('para', 188),
     ('no', 167),
     ('ipojuca', 158),
     ...
     ('(funções)', 1),
     ...]




```python
total_words_reduced = Counter()

for index,row in leis.iterrows():
    ementa = row['ementa']
    for word in ementa.split(" ")[:1]:
        total_words_reduced[word] += 1
```


```python
print("Quantidade de palavras diferentes utilizando todas as palavras das ementas: ",len(total_word_frequency))
print("Quantiade de palavras diferentes utilizando apenas a primeira palavra: ",len(total_words_reduced))
```

    Quantidade de palavras diferentes utilizando todas as palavras das ementas:  2427
    Quantiade de palavras diferentes utilizando apenas a primeira palavra:  67



```python
total_words_reduced.most_common()
```




    [('dispõe', 166),
     ('altera', 108),
     ('autoriza', 93),
     ('cria', 77),
     ('institui', 55),
     ('denomina', 53),
     ('estabelece', 29),
     ('reajusta', 25),
     ('concede', 25),
     ('dá', 17),
     ('fixa', 17),
     ('atribui', 13),
     ('estima', 12),
     ('?', 10),
     ('revoga', 9),
     ('modifica', 8),
     ('desafeta', 7),
     ('disciplina', 6),
     ('institui,', 6),
     ('dota', 5),
     ('abre', 5),
     ('aprova', 5),
     ('reestrutura', 5),
     ('orça', 4),
     ('acrescenta', 4),
     ('lei', 4),
     ('prorroga', 3),
     ('nan', 3),
     ('implanta', 3),
     ('regulamenta', 3),
     ('oferece', 2),
     ('declara', 2),
     ('aumente', 2),
     ('introduz', 2),
     ('insere', 1),
     ('autorizo', 1),
     ('da', 1),
     ('adapta', 1),
     ('autorizao', 1),
     ('eleva', 1),
     ('dispõeõe', 1),
     ('isenta', 1),
     ('reorganiza,', 1),
     ('adequa', 1),
     ('converte,', 1),
     ('define', 1),
     ('criação', 1),
     ('redefine', 1),
     ('vigilância', 1),
     ('designa', 1),
     ('aletera', 1),
     ('aumenta', 1),
     ('equipara', 1),
     ('incentiva', 1),
     ('reduz', 1),
     ('revisa', 1),
     ('doa', 1),
     ('colocação', 1),
     ('complementa', 1),
     ('suspensão', 1),
     ('disposições', 1),
     ('amplia', 1),
     ('-', 1),
     ('proíbe', 1),
     ('prevê', 1),
     ('determina', 1),
     ('extingue', 1)]



### Comentários
Usar apenas a primeira palavra apresentou um bom resultado em classificar as leis;

Vamos usar as palavras com no mínimo 6 ocorrências e criar uma nova coluna no dataset, representando a classificação das leis


```python
categorias = [x for x,cnt in total_words_reduced.most_common() if cnt > 5 ]

def get_categoria(ementa):
    first_word = ementa.split(" ")[0]
    if (first_word in categorias):
        return first_word
    else:
        return "outra"

leis['categoria_geral'] = leis.apply(lambda row: get_categoria(row['ementa']), axis=1)
```


```python
import numpy as np
from bokeh.models import ColumnDataSource, LabelSet
from bokeh.plotting import figure, show, output_file
from bokeh.io import output_notebook
from bokeh.charts import Bar, show
output_notebook()
```



    <div class="bk-root">
        <a href="http://bokeh.pydata.org" target="_blank" class="bk-logo bk-logo-small bk-logo-notebook"></a>
        <span id="f8efd7b9-bb77-4f46-8b2f-038e8d44e36a">Loading BokehJS ...</span>
    </div>





```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por categoria", legend='bottom_right')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="e27ca5d0-e540-423d-ad1b-ca1415599c04"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("e27ca5d0-e540-423d-ad1b-ca1415599c04").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("e27ca5d0-e540-423d-ad1b-ca1415599c04");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid 'e27ca5d0-e540-423d-ad1b-ca1415599c04' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"8cb1987a-e9c8-46bc-bd61-66b6ec0eab3a":{"roots":{"references":[{"attributes":{"data_source":{"id":"c3fed1e3-5c06-4d5a-ba35-85360a56df98","type":"ColumnDataSource"},"glyph":{"id":"2de79842-3a76-4a2a-b299-3de1c997d8fd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2b831298-b051-4137-81de-b44ec542f431","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"c2d1f3d5-7f2d-4fa0-8341-262ce77b5850","type":"ColumnDataSource"},{"attributes":{"label":{"value":"denomina"},"renderers":[{"id":"ba885bf4-33e9-44b6-b5d3-d2355f0493c0","type":"GlyphRenderer"}]},"id":"ccbefb7f-2092-4d70-b3dd-b95764c7f532","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"513892fb-499e-4dd4-b255-8a5487ed98dc","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"776828d7-06be-40aa-b2cf-6d9db0c6a36b","type":"ColumnDataSource"},"glyph":{"id":"73e07443-e851-4dff-84b6-985b8671ebb6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"aa4134f0-18f0-485c-89a7-882ea88a06f9","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"26dfc46f-5045-4a85-897f-3977e417f0e9","type":"ColumnDataSource"},"glyph":{"id":"e8a66fbe-04e4-4491-9303-22686eba99c3","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f6bbfcef-e51d-4e19-9d2a-00fbae4f4f2a","type":"GlyphRenderer"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"6ea2727a-f51c-497e-9abd-9f00228c3c10","type":"BasicTickFormatter"},"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"},"ticker":{"id":"2fc075f1-dcd2-4dac-9cae-2ee60d2c2dc5","type":"BasicTicker"}},"id":"90d08a82-9c94-41c1-ad15-7cd437f273f8","type":"LinearAxis"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"3365d243-3e19-487e-b0bb-3376fe2dafe5","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"13286a2d-7d4d-4f2a-9c21-939b3eddcbdf","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"d3816560-f93b-44bc-8f4f-a6ca253c28cd","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f083d9a1-0baf-4a95-b2db-f80274fc8ce2","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["modifica"],"chart_index":[{"categoria_geral":"modifica"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"modifica"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["modifica"],"y":[4.0]}},"id":"c957878d-3a31-46e1-adf7-d5e722098cd8","type":"ColumnDataSource"},{"attributes":{"label":{"value":"institui,"},"renderers":[{"id":"4b0d7cba-7636-4e15-88fe-62833fc75318","type":"GlyphRenderer"}]},"id":"d24cc16d-4bc5-4427-8fbb-5bc1109760f6","type":"LegendItem"},{"attributes":{"data_source":{"id":"d3816560-f93b-44bc-8f4f-a6ca253c28cd","type":"ColumnDataSource"},"glyph":{"id":"3e880d5f-a038-4619-8cb5-8baa67ecccc7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a36c154e-b4b9-4140-94d9-9d8d76e44690","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"513892fb-499e-4dd4-b255-8a5487ed98dc","type":"ColumnDataSource"},"glyph":{"id":"f083d9a1-0baf-4a95-b2db-f80274fc8ce2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"63e4cd87-1980-40ad-b15b-13dd5660177e","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3e880d5f-a038-4619-8cb5-8baa67ecccc7","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"776828d7-06be-40aa-b2cf-6d9db0c6a36b","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"264ad8a1-5809-4fe7-9ed7-941dc862c6f9","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4e63f10a-bb64-41db-93d9-96fecd63a11d","type":"Rect"},{"attributes":{"label":{"value":"institui"},"renderers":[{"id":"fc576e23-8ec7-4147-a3b5-29cf240995dd","type":"GlyphRenderer"}]},"id":"ee705d16-279e-4d1b-9bba-24b6f45a5dfe","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui,"],"chart_index":[{"categoria_geral":"institui,"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"institui,"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["institui,"],"y":[3.0]}},"id":"64ab2ee3-886b-41f2-ac9c-7d26e874360e","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"73e07443-e851-4dff-84b6-985b8671ebb6","type":"Rect"},{"attributes":{"label":{"value":"disciplina"},"renderers":[{"id":"3763dc8f-f07b-42d5-abc6-05d05cc480b6","type":"GlyphRenderer"}]},"id":"cf09a8ce-4cba-451d-b187-26f856610335","type":"LegendItem"},{"attributes":{"callback":null,"factors":["?","altera","atribui","autoriza","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","institui,","modifica","outra","reajusta","revoga"]},"id":"e2cb3894-1da7-4f01-887f-cca94ede7a13","type":"FactorRange"},{"attributes":{"data_source":{"id":"c2d1f3d5-7f2d-4fa0-8341-262ce77b5850","type":"ColumnDataSource"},"glyph":{"id":"4e63f10a-bb64-41db-93d9-96fecd63a11d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3763dc8f-f07b-42d5-abc6-05d05cc480b6","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"acb368ca-9680-4bd6-8fb7-f1b82d37fb42","type":"Rect"},{"attributes":{"label":{"value":"modifica"},"renderers":[{"id":"d2451bf7-bfba-4fc7-a97f-15b20ee89374","type":"GlyphRenderer"}]},"id":"5f63648b-7065-4237-8359-88caaed527bf","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[55.0],"label":[{"categoria_geral":"institui"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["institui"],"y":[27.5]}},"id":"891018b3-0777-400b-8424-00e9c7745577","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0fab2fb3-ff38-41df-8783-130b9df53d60","type":"Rect"},{"attributes":{"data_source":{"id":"3365d243-3e19-487e-b0bb-3376fe2dafe5","type":"ColumnDataSource"},"glyph":{"id":"acb368ca-9680-4bd6-8fb7-f1b82d37fb42","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f88c4792-133b-4b60-8614-1527f605b0d2","type":"GlyphRenderer"},{"attributes":{"label":{"value":"?"},"renderers":[{"id":"f6bbfcef-e51d-4e19-9d2a-00fbae4f4f2a","type":"GlyphRenderer"}]},"id":"ba9c45e7-849b-4b2c-aaa6-d8f20217ff1b","type":"LegendItem"},{"attributes":{"data_source":{"id":"c957878d-3a31-46e1-adf7-d5e722098cd8","type":"ColumnDataSource"},"glyph":{"id":"0fab2fb3-ff38-41df-8783-130b9df53d60","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d2451bf7-bfba-4fc7-a97f-15b20ee89374","type":"GlyphRenderer"},{"attributes":{},"id":"6ea2727a-f51c-497e-9abd-9f00228c3c10","type":"BasicTickFormatter"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6783c7e1-99ef-449c-99c7-25d298b70ead","type":"Rect"},{"attributes":{"label":{"value":"estima"},"renderers":[{"id":"a36c154e-b4b9-4140-94d9-9d8d76e44690","type":"GlyphRenderer"}]},"id":"90103387-e8e3-4137-a294-a21546c5a6ef","type":"LegendItem"},{"attributes":{"data_source":{"id":"9833bdae-cd74-4db7-b497-3dc6beedb39f","type":"ColumnDataSource"},"glyph":{"id":"13286a2d-7d4d-4f2a-9c21-939b3eddcbdf","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"854c66b9-1405-4a57-87e8-22176736b856","type":"GlyphRenderer"},{"attributes":{"label":{"value":"fixa"},"renderers":[{"id":"aa4134f0-18f0-485c-89a7-882ea88a06f9","type":"GlyphRenderer"}]},"id":"39d4bafc-a6a1-46b8-acdb-594049706845","type":"LegendItem"},{"attributes":{},"id":"2fc075f1-dcd2-4dac-9cae-2ee60d2c2dc5","type":"BasicTicker"},{"attributes":{"data_source":{"id":"264ad8a1-5809-4fe7-9ed7-941dc862c6f9","type":"ColumnDataSource"},"glyph":{"id":"6783c7e1-99ef-449c-99c7-25d298b70ead","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ba885bf4-33e9-44b6-b5d3-d2355f0493c0","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4c92a6f1-2e1f-4432-95ca-aaf6ea3e5925","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e8a66fbe-04e4-4491-9303-22686eba99c3","type":"Rect"},{"attributes":{"label":{"value":"desafeta"},"renderers":[{"id":"9d830505-250c-43a5-b26d-6cba3d104f55","type":"GlyphRenderer"}]},"id":"d3f32abf-24da-430c-ae7e-c0a1aca3da04","type":"LegendItem"},{"attributes":{"dimension":1,"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"},"ticker":{"id":"2fc075f1-dcd2-4dac-9cae-2ee60d2c2dc5","type":"BasicTicker"}},"id":"471351bf-2c9f-402b-99fa-eaee3dacd60a","type":"Grid"},{"attributes":{"label":{"value":"estabelece"},"renderers":[{"id":"854c66b9-1405-4a57-87e8-22176736b856","type":"GlyphRenderer"}]},"id":"a215bf86-141c-45f0-82d2-99786175f6da","type":"LegendItem"},{"attributes":{"data_source":{"id":"64ab2ee3-886b-41f2-ac9c-7d26e874360e","type":"ColumnDataSource"},"glyph":{"id":"4c92a6f1-2e1f-4432-95ca-aaf6ea3e5925","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4b0d7cba-7636-4e15-88fe-62833fc75318","type":"GlyphRenderer"},{"attributes":{"label":{"value":"revoga"},"renderers":[{"id":"63e4cd87-1980-40ad-b15b-13dd5660177e","type":"GlyphRenderer"}]},"id":"d33322e6-34d6-4558-9fb4-7d385ac3d08b","type":"LegendItem"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"1df38102-30dc-47c0-9247-f605f02c2313","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"},"ticker":{"id":"dc3fb85f-a5ac-4db4-99b1-90f130c29a4f","type":"CategoricalTicker"}},"id":"623b45e5-d8bd-41c8-918b-fe43883fbfa4","type":"CategoricalAxis"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9a8d1fdc-0863-4a46-ba6e-8602658ae960","type":"Rect"},{"attributes":{},"id":"1df38102-30dc-47c0-9247-f605f02c2313","type":"CategoricalTickFormatter"},{"attributes":{"data_source":{"id":"891018b3-0777-400b-8424-00e9c7745577","type":"ColumnDataSource"},"glyph":{"id":"9a8d1fdc-0863-4a46-ba6e-8602658ae960","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"fc576e23-8ec7-4147-a3b5-29cf240995dd","type":"GlyphRenderer"},{"attributes":{},"id":"dc3fb85f-a5ac-4db4-99b1-90f130c29a4f","type":"CategoricalTicker"},{"attributes":{"label":{"value":"disp\u00f5e"},"renderers":[{"id":"2b831298-b051-4137-81de-b44ec542f431","type":"GlyphRenderer"}]},"id":"88b607bb-f8ea-4917-aabd-a2ceee027ae2","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[29.0],"label":[{"categoria_geral":"estabelece"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["estabelece"],"y":[14.5]}},"id":"9833bdae-cd74-4db7-b497-3dc6beedb39f","type":"ColumnDataSource"},{"attributes":{"label":{"value":"altera"},"renderers":[{"id":"d19f2ef8-4254-4c41-82bb-5b059d93b3a6","type":"GlyphRenderer"}]},"id":"ed4b1898-c3e6-401f-8aa8-da95bfc83441","type":"LegendItem"},{"attributes":{"label":{"value":"atribui"},"renderers":[{"id":"f88c4792-133b-4b60-8614-1527f605b0d2","type":"GlyphRenderer"}]},"id":"fcf350bd-5e8c-491b-9c86-19ced56a09e1","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"26dfc46f-5045-4a85-897f-3977e417f0e9","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"78498c40-670d-43cb-804a-e83a0db5100d","type":"ColumnDataSource"},"glyph":{"id":"f5af6df8-843a-43de-b82a-58b2065ab086","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8b85b1ad-9a42-43dc-985f-7ad393c4a458","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera"],"chart_index":[{"categoria_geral":"altera"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[108.0],"label":[{"categoria_geral":"altera"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["altera"],"y":[54.0]}},"id":"7e6a5a64-eb8b-4417-96a3-2c134a0c561e","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"0ea84b4f-28ca-4140-81c7-475a4e69508e","type":"ColumnDataSource"},"glyph":{"id":"c13b65a6-b8db-4405-81f9-4263250f64fd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"9c48ae74-ea1b-4be5-9d69-66080e9e85d3","type":"GlyphRenderer"},{"attributes":{"label":{"value":"d\u00e1"},"renderers":[{"id":"506df850-f62f-4707-999b-cd0887d07877","type":"GlyphRenderer"}]},"id":"2caabdfa-6c57-4a8f-abcc-b033b8215d9c","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[77.0],"label":[{"categoria_geral":"cria"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["cria"],"y":[38.5]}},"id":"78498c40-670d-43cb-804a-e83a0db5100d","type":"ColumnDataSource"},{"attributes":{"label":{"value":"autoriza"},"renderers":[{"id":"9c48ae74-ea1b-4be5-9d69-66080e9e85d3","type":"GlyphRenderer"}]},"id":"b25b4d64-a5e8-44d2-8cce-8bedd73d3070","type":"LegendItem"},{"attributes":{"items":[{"id":"4ce675ee-f840-4f88-a30c-0d8bef866ea4","type":"LegendItem"},{"id":"b25b4d64-a5e8-44d2-8cce-8bedd73d3070","type":"LegendItem"},{"id":"6dbc56a2-9d70-46ef-a9f1-ffbeafea81d0","type":"LegendItem"},{"id":"6e7b6b6a-eb55-4ef6-9f3f-5f8f7471a2c4","type":"LegendItem"},{"id":"2caabdfa-6c57-4a8f-abcc-b033b8215d9c","type":"LegendItem"},{"id":"fa36d26a-c83c-4a33-b993-8826be908454","type":"LegendItem"},{"id":"88b607bb-f8ea-4917-aabd-a2ceee027ae2","type":"LegendItem"},{"id":"d3f32abf-24da-430c-ae7e-c0a1aca3da04","type":"LegendItem"},{"id":"d33322e6-34d6-4558-9fb4-7d385ac3d08b","type":"LegendItem"},{"id":"39d4bafc-a6a1-46b8-acdb-594049706845","type":"LegendItem"},{"id":"fcf350bd-5e8c-491b-9c86-19ced56a09e1","type":"LegendItem"},{"id":"ccbefb7f-2092-4d70-b3dd-b95764c7f532","type":"LegendItem"},{"id":"ee705d16-279e-4d1b-9bba-24b6f45a5dfe","type":"LegendItem"},{"id":"ed4b1898-c3e6-401f-8aa8-da95bfc83441","type":"LegendItem"},{"id":"a215bf86-141c-45f0-82d2-99786175f6da","type":"LegendItem"},{"id":"ba9c45e7-849b-4b2c-aaa6-d8f20217ff1b","type":"LegendItem"},{"id":"90103387-e8e3-4137-a294-a21546c5a6ef","type":"LegendItem"},{"id":"cf09a8ce-4cba-451d-b187-26f856610335","type":"LegendItem"},{"id":"5f63648b-7065-4237-8359-88caaed527bf","type":"LegendItem"},{"id":"d24cc16d-4bc5-4427-8fbb-5bc1109760f6","type":"LegendItem"}],"location":"bottom_right","plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"9b7a99c0-178f-498c-b149-90bdc62abecf","type":"Legend"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"cf6ce180-a4d1-4efe-aa17-67e268f07e54","type":"Rect"},{"attributes":{},"id":"e089edce-7393-49e3-ae86-b00016dd9ee1","type":"ToolEvents"},{"attributes":{"data_source":{"id":"defc65fa-aa27-4c89-a56a-747a3cb231de","type":"ColumnDataSource"},"glyph":{"id":"cf6ce180-a4d1-4efe-aa17-67e268f07e54","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"9e81a400-bed3-4540-8a3e-bb2d466ece6b","type":"GlyphRenderer"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"feff56d3-a58b-423f-881e-5c07cea7d20b","type":"GlyphRenderer"}]},"id":"fa36d26a-c83c-4a33-b993-8826be908454","type":"LegendItem"},{"attributes":{"data_source":{"id":"b778287c-6c1a-46d7-9398-13b8467d1c1b","type":"ColumnDataSource"},"glyph":{"id":"0d840b03-c5ec-46a6-820e-94b62bbdcc65","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"506df850-f62f-4707-999b-cd0887d07877","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"b5bed6c5-5cd8-4073-a0b2-396062de6c6e","type":"ColumnDataSource"},"glyph":{"id":"6630cec9-5523-44e6-af37-9432af20f530","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"9d830505-250c-43a5-b26d-6cba3d104f55","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c13b65a6-b8db-4405-81f9-4263250f64fd","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[93.0],"label":[{"categoria_geral":"autoriza"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["autoriza"],"y":[46.5]}},"id":"0ea84b4f-28ca-4140-81c7-475a4e69508e","type":"ColumnDataSource"},{"attributes":{"below":[{"id":"623b45e5-d8bd-41c8-918b-fe43883fbfa4","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"90d08a82-9c94-41c1-ad15-7cd437f273f8","type":"LinearAxis"}],"renderers":[{"id":"f07f862d-2fa4-4193-8c4e-b89bea7736e5","type":"BoxAnnotation"},{"id":"9e81a400-bed3-4540-8a3e-bb2d466ece6b","type":"GlyphRenderer"},{"id":"9c48ae74-ea1b-4be5-9d69-66080e9e85d3","type":"GlyphRenderer"},{"id":"4b0fcedc-4a52-4cd9-a755-c1483aecc5ee","type":"GlyphRenderer"},{"id":"8b85b1ad-9a42-43dc-985f-7ad393c4a458","type":"GlyphRenderer"},{"id":"506df850-f62f-4707-999b-cd0887d07877","type":"GlyphRenderer"},{"id":"feff56d3-a58b-423f-881e-5c07cea7d20b","type":"GlyphRenderer"},{"id":"2b831298-b051-4137-81de-b44ec542f431","type":"GlyphRenderer"},{"id":"9d830505-250c-43a5-b26d-6cba3d104f55","type":"GlyphRenderer"},{"id":"63e4cd87-1980-40ad-b15b-13dd5660177e","type":"GlyphRenderer"},{"id":"aa4134f0-18f0-485c-89a7-882ea88a06f9","type":"GlyphRenderer"},{"id":"f88c4792-133b-4b60-8614-1527f605b0d2","type":"GlyphRenderer"},{"id":"ba885bf4-33e9-44b6-b5d3-d2355f0493c0","type":"GlyphRenderer"},{"id":"fc576e23-8ec7-4147-a3b5-29cf240995dd","type":"GlyphRenderer"},{"id":"d19f2ef8-4254-4c41-82bb-5b059d93b3a6","type":"GlyphRenderer"},{"id":"854c66b9-1405-4a57-87e8-22176736b856","type":"GlyphRenderer"},{"id":"f6bbfcef-e51d-4e19-9d2a-00fbae4f4f2a","type":"GlyphRenderer"},{"id":"a36c154e-b4b9-4140-94d9-9d8d76e44690","type":"GlyphRenderer"},{"id":"3763dc8f-f07b-42d5-abc6-05d05cc480b6","type":"GlyphRenderer"},{"id":"d2451bf7-bfba-4fc7-a97f-15b20ee89374","type":"GlyphRenderer"},{"id":"4b0d7cba-7636-4e15-88fe-62833fc75318","type":"GlyphRenderer"},{"id":"9b7a99c0-178f-498c-b149-90bdc62abecf","type":"Legend"},{"id":"623b45e5-d8bd-41c8-918b-fe43883fbfa4","type":"CategoricalAxis"},{"id":"90d08a82-9c94-41c1-ad15-7cd437f273f8","type":"LinearAxis"},{"id":"471351bf-2c9f-402b-99fa-eaee3dacd60a","type":"Grid"}],"title":{"id":"2bdbfe70-155f-4dc4-a6e3-f00a3c89a2fd","type":"Title"},"tool_events":{"id":"e089edce-7393-49e3-ae86-b00016dd9ee1","type":"ToolEvents"},"toolbar":{"id":"d5a3638c-2d59-401b-8d8f-673530e7e9be","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"e2cb3894-1da7-4f01-887f-cca94ede7a13","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"ce2059c4-a94a-412c-a221-66b647ad88b0","type":"Range1d"}},"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"},{"attributes":{"callback":null,"end":174.3},"id":"ce2059c4-a94a-412c-a221-66b647ad88b0","type":"Range1d"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"9c02b687-8cb4-4ac4-8067-a891ba4162ad","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"9c02b687-8cb4-4ac4-8067-a891ba4162ad","type":"ColumnDataSource"},"glyph":{"id":"030cec0f-8e81-4f32-8395-4ba62277c1ea","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4b0fcedc-4a52-4cd9-a755-c1483aecc5ee","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"defc65fa-aa27-4c89-a56a-747a3cb231de","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"b778287c-6c1a-46d7-9398-13b8467d1c1b","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"b5bed6c5-5cd8-4073-a0b2-396062de6c6e","type":"ColumnDataSource"},{"attributes":{"label":{"value":"reajusta"},"renderers":[{"id":"9e81a400-bed3-4540-8a3e-bb2d466ece6b","type":"GlyphRenderer"}]},"id":"4ce675ee-f840-4f88-a30c-0d8bef866ea4","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"030cec0f-8e81-4f32-8395-4ba62277c1ea","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f5af6df8-843a-43de-b82a-58b2065ab086","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[85.0],"label":[{"categoria_geral":"outra"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["outra"],"y":[42.5]}},"id":"465ab4df-ae91-4845-be16-9b5dd23d88a2","type":"ColumnDataSource"},{"attributes":{"label":{"value":"cria"},"renderers":[{"id":"8b85b1ad-9a42-43dc-985f-7ad393c4a458","type":"GlyphRenderer"}]},"id":"6e7b6b6a-eb55-4ef6-9f3f-5f8f7471a2c4","type":"LegendItem"},{"attributes":{"plot":null,"text":"Quantidade de Leis por categoria"},"id":"2bdbfe70-155f-4dc4-a6e3-f00a3c89a2fd","type":"Title"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6f1ec603-9ec7-4332-ab70-aa663d3c1899","type":"Rect"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"f2378858-a573-426e-97e5-0380c5c3b9df","type":"PanTool"},{"id":"0f06a3d8-3443-484d-bdf3-c40979297356","type":"WheelZoomTool"},{"id":"4a480e8c-ced6-4547-b654-e78d5ade0d46","type":"BoxZoomTool"},{"id":"13baa2b2-2fe8-4034-ae57-0e6aad2e8dc1","type":"SaveTool"},{"id":"908025b1-4cf9-4099-8909-ea9798df336f","type":"ResetTool"},{"id":"bf3920ca-283e-4dc1-bc36-22ab08771823","type":"HelpTool"}]},"id":"d5a3638c-2d59-401b-8d8f-673530e7e9be","type":"Toolbar"},{"attributes":{"label":{"value":"concede"},"renderers":[{"id":"4b0fcedc-4a52-4cd9-a755-c1483aecc5ee","type":"GlyphRenderer"}]},"id":"6dbc56a2-9d70-46ef-a9f1-ffbeafea81d0","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0d840b03-c5ec-46a6-820e-94b62bbdcc65","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"2de79842-3a76-4a2a-b299-3de1c997d8fd","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3dc17ad5-0a67-4032-87a2-451f0fa266c8","type":"Rect"},{"attributes":{"data_source":{"id":"465ab4df-ae91-4845-be16-9b5dd23d88a2","type":"ColumnDataSource"},"glyph":{"id":"6f1ec603-9ec7-4332-ab70-aa663d3c1899","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"feff56d3-a58b-423f-881e-5c07cea7d20b","type":"GlyphRenderer"},{"attributes":{"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"f2378858-a573-426e-97e5-0380c5c3b9df","type":"PanTool"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"f07f862d-2fa4-4193-8c4e-b89bea7736e5","type":"BoxAnnotation"},{"attributes":{"data_source":{"id":"7e6a5a64-eb8b-4417-96a3-2c134a0c561e","type":"ColumnDataSource"},"glyph":{"id":"3dc17ad5-0a67-4032-87a2-451f0fa266c8","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d19f2ef8-4254-4c41-82bb-5b059d93b3a6","type":"GlyphRenderer"},{"attributes":{"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"0f06a3d8-3443-484d-bdf3-c40979297356","type":"WheelZoomTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[166.0],"label":[{"categoria_geral":"disp\u00f5e"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["disp\u00f5e"],"y":[83.0]}},"id":"c3fed1e3-5c06-4d5a-ba35-85360a56df98","type":"ColumnDataSource"},{"attributes":{"overlay":{"id":"f07f862d-2fa4-4193-8c4e-b89bea7736e5","type":"BoxAnnotation"},"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"4a480e8c-ced6-4547-b654-e78d5ade0d46","type":"BoxZoomTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6630cec9-5523-44e6-af37-9432af20f530","type":"Rect"},{"attributes":{"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"13baa2b2-2fe8-4034-ae57-0e6aad2e8dc1","type":"SaveTool"},{"attributes":{"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"908025b1-4cf9-4099-8909-ea9798df336f","type":"ResetTool"},{"attributes":{"plot":{"id":"a0e9902f-6b63-4256-89c2-cc5a05a3795b","subtype":"Chart","type":"Plot"}},"id":"bf3920ca-283e-4dc1-bc36-22ab08771823","type":"HelpTool"}],"root_ids":["a0e9902f-6b63-4256-89c2-cc5a05a3795b"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"8cb1987a-e9c8-46bc-bd61-66b6ec0eab3a","elementid":"e27ca5d0-e540-423d-ad1b-ca1415599c04","modelid":"a0e9902f-6b63-4256-89c2-cc5a05a3795b"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("e27ca5d0-e540-423d-ad1b-ca1415599c04")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'dispõe'

Podemos ver que a categoria 'dispõe' parece ser a mais geral, portanto vamos começar a criar sub-categorias para este grupo;

Também é possível ver que algumas categorias podem ser combinadas


```python
leis[leis['categoria_geral'] == 'dispõe'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>9</th>
      <td>1021</td>
      <td>1992</td>
      <td>dispõe sobre as diretrizes orçamentárias para ...</td>
      <td>dispõe</td>
    </tr>
    <tr>
      <th>38</th>
      <td>1050</td>
      <td>1993</td>
      <td>dispõe sobre as diretrizes para elaboraçãao do...</td>
      <td>dispõe</td>
    </tr>
    <tr>
      <th>58</th>
      <td>1070</td>
      <td>1994</td>
      <td>dispõe sobre as diretizes para a elaboração de...</td>
      <td>dispõe</td>
    </tr>
    <tr>
      <th>70</th>
      <td>1083</td>
      <td>1995</td>
      <td>dispõe sobre a revalidação dos certames públic...</td>
      <td>dispõe</td>
    </tr>
    <tr>
      <th>72</th>
      <td>1085</td>
      <td>1995</td>
      <td>dispõe sobre as diretrizes para elaboração e e...</td>
      <td>dispõe</td>
    </tr>
  </tbody>
</table>
</div>



### Comentários
A quarta palavra das ementas pertencentes a categoria 'dispõe' parece identificar o propósito da lei, vamos usar isto como sub-categoria


```python
sub_categoria_dispoe = Counter()

for index,row in leis.iterrows():
    if (row['categoria_geral'] == 'dispõe'):
        ementa = row['ementa'].split(" ")
        fourth_word = ementa[3]
        sub_categoria_dispoe[fourth_word] += 1

print("Quantidade de sub-categorias: ",len(sub_categoria_dispoe))
sub_categoria_dispoe.most_common()
```

    Quantidade de sub-categorias:  64





    [('criação', 27),
     ('plano', 10),
     ('abertura', 9),
     ('revisão', 8),
     ('de', 7),
     ('instituição', 7),
     ('alteração', 7),
     ('estrutura', 6),
     ('diretrizes', 5),
     ('denominação', 5),
     ('reajuste', 4),
     ('vencimentos', 4),
     ('contratação', 3),
     ('aumento', 3),
     ('reestruturação', 3),
     ('reorganização', 2),
     ('interveniência', 2),
     ('verba', 2),
     ('destinação', 2),
     ('isenção', 2),
     ('regime', 2),
     ('do', 2),
     ('conselho', 2),
     ('dos', 2),
     ('diretizes', 1),
     ('revalidação', 1),
     ('feriados', 1),
     ('adicionais', 1),
     ('percentual', 1),
     ('programa', 1),
     ('adequação', 1),
     ('de~crédito', 1),
     ('de^rédito', 1),
     ('direírizes', 1),
     ('introdução', 1),
     ('plantio,', 1),
     ('atendimento', 1),
     ('conselhos', 1),
     ('aplicação', 1),
     ('estatuto', 1),
     ('estruturação', 1),
     ('inclusão', 1),
     ('piano', 1),
     ('municipal', 1),
     ('para', 1),
     ('regulamentação', 1),
     ('nos', 1),
     ('concessão', 1),
     ('ampliação', 1),
     ('e', 1),
     ('criação,', 1),
     ('obrigatoriedade', 1),
     ('proibição', 1),
     ('licenciamento', 1),
     ('auxílios,', 1),
     ('cancelamento', 1),
     ('doação', 1),
     ('parcelamento', 1),
     ('lei', 1),
     ('extinção', 1),
     ('turísticas', 1),
     ('gratificação', 1),
     ('remuneração', 1),
     ('organizações', 1)]



### Observações
A categoria 'de' ficou com um número considerável de ocorrências, vamos remover as stop_words e usar a próxima palavra da ementa nesses casos


```python
sub_categoria_dispoe = Counter()
stop_words = ['a', 'o' , 'de', 'as', 'do', 'e', 'os' , 'dos', 'no', 'da', 'para']

for index,row in leis.iterrows():
    if (row['categoria_geral'] == 'dispõe'):
        ementa = row['ementa'].split(" ")
        fourth_word = ementa[3]
        fifth_word = ementa[4]
        if (fourth_word in stop_words):
            sub_categoria_dispoe[fifth_word] += 1
        else:
            sub_categoria_dispoe[fourth_word] += 1

print("Quantidade de sub-categorias: ",len(sub_categoria_dispoe))       
sub_categoria_dispoe.most_common()
```

    Quantidade de sub-categorias:  65





    [('criação', 27),
     ('plano', 10),
     ('abertura', 9),
     ('revisão', 8),
     ('instituição', 7),
     ('alteração', 7),
     ('estrutura', 6),
     ('vencimentos', 6),
     ('diretrizes', 5),
     ('crédito', 5),
     ('denominação', 5),
     ('reajuste', 4),
     ('contratação', 3),
     ('aumento', 3),
     ('reestruturação', 3),
     ('reorganização', 2),
     ('interveniência', 2),
     ('verba', 2),
     ('destinação', 2),
     ('isenção', 2),
     ('regime', 2),
     ('conselho', 2),
     ('doação', 2),
     ('diretizes', 1),
     ('revalidação', 1),
     ('feriados', 1),
     ('adicionais', 1),
     ('percentual', 1),
     ('programa', 1),
     ('créditos', 1),
     ('adequação', 1),
     ('de~crédito', 1),
     ('de^rédito', 1),
     ('direírizes', 1),
     ('introdução', 1),
     ('plantio,', 1),
     ('atendimento', 1),
     ('diagnóstico', 1),
     ('conselhos', 1),
     ('aplicação', 1),
     ('estatuto', 1),
     ('estruturação', 1),
     ('inclusão', 1),
     ('piano', 1),
     ('municipal', 1),
     ('regulamentação', 1),
     ('nos', 1),
     ('concessão', 1),
     ('ampliação', 1),
     ('competência', 1),
     ('remanejamento', 1),
     ('criação,', 1),
     ('obrigatoriedade', 1),
     ('uso', 1),
     ('proibição', 1),
     ('licenciamento', 1),
     ('auxílios,', 1),
     ('cancelamento', 1),
     ('parcelamento', 1),
     ('lei', 1),
     ('extinção', 1),
     ('turísticas', 1),
     ('gratificação', 1),
     ('remuneração', 1),
     ('organizações', 1)]



### Comentários
Agora sim estamos com um resultado legal, vamos usar como sub-categorias as palavras com mais de 2 ocorrências


```python
sub_categorias_dispoe = [x for x,cnt in sub_categoria_dispoe.most_common() if cnt > 2 ]

def get_sub_categoria_dispoe(ementa):
    if (len(ementa.split(" ")) > 3):
        fourth_word = ementa.split(" ")[3]
        fifth_word = ementa.split(" ")[4]
        if (fourth_word in sub_categorias_dispoe):
            return fourth_word
        elif (fifth_word in sub_categorias_dispoe):
            return fifth_word
        else:
            return "outra"
    else:
        return "outra"

leis['sub_categoria'] = leis.apply(lambda row: get_sub_categoria_dispoe(row['ementa']), axis=1)
```


```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por sub-categoria", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="7685b1c0-8beb-4763-96aa-6f0b5b4783ad"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("7685b1c0-8beb-4763-96aa-6f0b5b4783ad").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("7685b1c0-8beb-4763-96aa-6f0b5b4783ad");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid '7685b1c0-8beb-4763-96aa-6f0b5b4783ad' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"1caffb7c-5f35-4705-8ef8-7d51dd3d5753":{"roots":{"references":[{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera"],"chart_index":[{"categoria_geral":"altera","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[108.0],"label":[{"categoria_geral":"altera","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["altera"],"y":[54.0]}},"id":"a84d9bf6-f7a7-4522-89bb-aec8fbc97cf2","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["disp\u00f5e"],"y":[151.0]}},"id":"0b5b4814-a888-4194-bcc3-dea7b1c0ed63","type":"ColumnDataSource"},{"attributes":{"label":{"value":"reestrutura\u00e7\u00e3o"},"renderers":[{"id":"c1e7d830-f802-4664-ad09-a06923668353","type":"GlyphRenderer"}]},"id":"1fb9b79f-9d86-4dc9-baae-e58da3847556","type":"LegendItem"},{"attributes":{"data_source":{"id":"770fbc6b-de4f-4e0c-bd42-71f35268a3d5","type":"ColumnDataSource"},"glyph":{"id":"5ba3d3d3-b0dd-44d3-8935-108355f345d9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"eac9730d-466d-4508-bc66-7ba3124b0b20","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["institui\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[144.5]}},"id":"5eb56e4b-0661-41bd-a3a7-b2a29626df00","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["diretrizes"],"width":[0.8],"x":["disp\u00f5e"],"y":[2.5]}},"id":"2bb1bebf-72f5-4d4e-aa4a-976b63c3ee28","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[55.0],"label":[{"categoria_geral":"institui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui"],"y":[27.5]}},"id":"f171bf98-064f-458f-99bf-baf470e1409d","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a4da9886-eca1-4ce2-8209-23a982c4c879","type":"ColumnDataSource"},"glyph":{"id":"27bfa560-8556-4fc9-b1b7-888d8e79ebe5","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"761f5025-7a24-415a-8efb-c36805af74d1","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"6815487f-0a92-4322-9528-b21e168330db","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"aff2d5c6-0b27-4e33-a432-fa6b29cf6a96","type":"Rect"},{"attributes":{"data_source":{"id":"2bb1bebf-72f5-4d4e-aa4a-976b63c3ee28","type":"ColumnDataSource"},"glyph":{"id":"af6264cc-2c54-4d44-a09e-31cb0303f1e7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2ba70e81-c46a-48ad-9ffc-0db8581f51f0","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"f171bf98-064f-458f-99bf-baf470e1409d","type":"ColumnDataSource"},"glyph":{"id":"426877ac-d625-4f69-8b5d-6ea9a2435227","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"03e52158-c516-4112-872d-1b0c28a57e02","type":"GlyphRenderer"},{"attributes":{"label":{"value":"cr\u00e9dito"},"renderers":[{"id":"5f799efb-fc5d-4499-b392-f9c226c3fab9","type":"GlyphRenderer"}]},"id":"d3cfb565-d87b-444b-929c-81af2a071f0e","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["altera\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[157.5]}},"id":"447388e3-9074-48a7-baa3-94adb789e2e1","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"27bfa560-8556-4fc9-b1b7-888d8e79ebe5","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"426877ac-d625-4f69-8b5d-6ea9a2435227","type":"Rect"},{"attributes":{"label":{"value":"aumento"},"renderers":[{"id":"2b8f314d-5905-40aa-81e0-776d903183ee","type":"GlyphRenderer"}]},"id":"7aed4421-aff9-488c-ac62-f3009ad128ba","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reestrutura\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[139.5]}},"id":"376f75c7-335d-4e48-a0a7-91dde405f914","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6188cf68-27a3-4bad-9456-33e5671df19c","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"59876611-f73c-4028-8b8c-051f0c812168","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"818bcb45-be90-48ef-a382-efc61fdd48c2","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"067d832a-10fe-41f2-a805-7978a60bcc23","type":"ColumnDataSource"},"glyph":{"id":"7c846ad6-925f-42ed-a29a-4f74997c5d84","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"5f799efb-fc5d-4499-b392-f9c226c3fab9","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b7947dcf-fe38-41f8-8a7b-674974953442","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reajuste"],"width":[0.8],"x":["disp\u00f5e"],"y":[136.0]}},"id":"d3552387-b0f6-40f1-bd71-00ef4253a22c","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["estrutura"],"width":[0.8],"x":["disp\u00f5e"],"y":[128.0]}},"id":"aea2750b-4fda-46a3-808f-a770520c98ac","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a84d9bf6-f7a7-4522-89bb-aec8fbc97cf2","type":"ColumnDataSource"},"glyph":{"id":"b7947dcf-fe38-41f8-8a7b-674974953442","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e41dbe8c-b48d-4b03-ae58-25a6757cfd1e","type":"GlyphRenderer"},{"attributes":{"label":{"value":"cria\u00e7\u00e3o"},"renderers":[{"id":"297d2af8-18c5-466b-89f6-6b6bfed82997","type":"GlyphRenderer"}]},"id":"8b0a315e-6453-4e9f-8b76-62781562c906","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7c846ad6-925f-42ed-a29a-4f74997c5d84","type":"Rect"},{"attributes":{},"id":"55df21ed-a224-4146-91e2-abde855eebe6","type":"CategoricalTicker"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estabelece"],"y":[12.5]}},"id":"e640cf52-a7de-46ce-9d57-d407fab56eea","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a7ec75a0-01d8-467f-8879-6b7fd76c57fb","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["estabelece"],"y":[27.0]}},"id":"80b0067b-fa01-40b2-922d-3e27d45c6311","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["denomina\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[163.5]}},"id":"a4da9886-eca1-4ce2-8209-23a982c4c879","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"702aaecd-54f2-4d7a-938c-028bfb8bbe92","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["aumento"],"width":[0.8],"x":["disp\u00f5e"],"y":[132.5]}},"id":"3f2bcd15-8ce7-45ef-b247-846a62884e57","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"953cb3ae-ef36-4615-a3b9-017bdde6d2b7","type":"ColumnDataSource"},"glyph":{"id":"dd193fca-e069-4659-8217-395da2032023","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"55561680-c17c-47bb-b274-9f693465f835","type":"GlyphRenderer"},{"attributes":{"label":{"value":"plano"},"renderers":[{"id":"fddd23f5-998f-4cba-afdd-4a01cbe5b157","type":"GlyphRenderer"}]},"id":"a68d6560-7a5d-4b65-8d9e-1dac72c7289e","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"298d87a0-7300-4698-a312-8145136055db","type":"ColumnDataSource"},{"attributes":{"label":{"value":"contrata\u00e7\u00e3o"},"renderers":[{"id":"eac9730d-466d-4508-bc66-7ba3124b0b20","type":"GlyphRenderer"}]},"id":"af4fb797-44e0-4351-a80d-e94337a96ef6","type":"LegendItem"},{"attributes":{"label":{"value":"abertura"},"renderers":[{"id":"261c6520-0973-4c79-9a42-20678ebf2da4","type":"GlyphRenderer"}]},"id":"2dc94be3-6432-418f-8e01-603b4b5ba4fe","type":"LegendItem"},{"attributes":{"data_source":{"id":"6815487f-0a92-4322-9528-b21e168330db","type":"ColumnDataSource"},"glyph":{"id":"702aaecd-54f2-4d7a-938c-028bfb8bbe92","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8a051ecf-aaec-4d51-b2e6-130b72e0ad27","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5ba3d3d3-b0dd-44d3-8935-108355f345d9","type":"Rect"},{"attributes":{"label":{"value":"estrutura"},"renderers":[{"id":"ef580c9c-752a-4acb-a6b2-c4276b6a87ff","type":"GlyphRenderer"}]},"id":"937e17f9-28b9-4d40-8b48-9b27cb2433af","type":"LegendItem"},{"attributes":{"data_source":{"id":"73e8e4d3-2463-46ed-bb60-05231c1ebc3a","type":"ColumnDataSource"},"glyph":{"id":"6188cf68-27a3-4bad-9456-33e5671df19c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"89f8126c-2dd0-4d58-9ae9-59933d2d79d2","type":"GlyphRenderer"},{"attributes":{"label":{"value":"revis\u00e3o"},"renderers":[{"id":"06cac2b7-9be9-429a-b130-7d27d081fcda","type":"GlyphRenderer"}]},"id":"01872457-d852-4e64-b499-959e3e33696f","type":"LegendItem"},{"attributes":{"label":{"value":"vencimentos"},"renderers":[{"id":"c0044790-5351-49b1-9d44-836cf8210d2e","type":"GlyphRenderer"}]},"id":"8e6628d9-1873-431e-8e06-bed8224b18ad","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"bdb7b653-1253-4fff-afab-c3975f2b5347","type":"ColumnDataSource"},{"attributes":{"label":{"value":"reajuste"},"renderers":[{"id":"baedbe20-2e6e-4d22-89b5-0c30e7815e7e","type":"GlyphRenderer"}]},"id":"0ccdd9ec-32d6-4f03-98dc-b321898c827e","type":"LegendItem"},{"attributes":{"label":{"value":"altera\u00e7\u00e3o"},"renderers":[{"id":"a93c543e-eb9c-4639-88e4-08d22c2ec1f3","type":"GlyphRenderer"}]},"id":"4b9e9247-24b8-4254-899c-ab6a7a3d7d96","type":"LegendItem"},{"attributes":{"label":{"value":"institui\u00e7\u00e3o"},"renderers":[{"id":"881ff678-8288-44ea-93d1-c7e01684991b","type":"GlyphRenderer"}]},"id":"afcd5111-c3c4-498b-9658-e524667ac348","type":"LegendItem"},{"attributes":{"label":{"value":"denomina\u00e7\u00e3o"},"renderers":[{"id":"761f5025-7a24-415a-8efb-c36805af74d1","type":"GlyphRenderer"}]},"id":"5b9174d5-e8ac-4f95-b97f-8f81385575e8","type":"LegendItem"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"817e0261-aa7b-4e89-9303-ff2b01ba52b1","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"},"ticker":{"id":"55df21ed-a224-4146-91e2-abde855eebe6","type":"CategoricalTicker"}},"id":"025be0ca-4cf4-4fed-8ff6-e509323f154f","type":"CategoricalAxis"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"582fc690-082c-4713-bdc9-ccdd334f008d","type":"Rect"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"9a943af3-c65d-4acf-8845-5f71461ec261","type":"BasicTickFormatter"},"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"},"ticker":{"id":"c549a4b9-5815-4905-abff-eccc1375cd1d","type":"BasicTicker"}},"id":"b828762d-e0f6-424b-964e-88f222a147c2","type":"LinearAxis"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[78.0],"label":[{"categoria_geral":"autoriza","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["autoriza"],"y":[39.0]}},"id":"15ce7cb8-7535-4594-aeeb-77bbb3a34bc6","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d28f5120-b8f0-40e6-887c-728d11057746","type":"Rect"},{"attributes":{},"id":"c549a4b9-5815-4905-abff-eccc1375cd1d","type":"BasicTicker"},{"attributes":{"dimension":1,"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"},"ticker":{"id":"c549a4b9-5815-4905-abff-eccc1375cd1d","type":"BasicTicker"}},"id":"787a73ee-d6d7-4240-9242-f387e5f07a5e","type":"Grid"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"8a5aa3f5-b774-4743-bc32-2ae54b140a97","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"818bcb45-be90-48ef-a382-efc61fdd48c2","type":"ColumnDataSource"},"glyph":{"id":"582fc690-082c-4713-bdc9-ccdd334f008d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e6bdd7c6-932e-4767-9a80-6a72d50b1dae","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"25e53878-bc4f-421d-91e6-c5ffc0b7594b","type":"Rect"},{"attributes":{},"id":"817e0261-aa7b-4e89-9303-ff2b01ba52b1","type":"CategoricalTickFormatter"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"4be65834-cc5d-4781-9572-5006350660f6","type":"ColumnDataSource"},{"attributes":{},"id":"9a943af3-c65d-4acf-8845-5f71461ec261","type":"BasicTickFormatter"},{"attributes":{"data_source":{"id":"8a5aa3f5-b774-4743-bc32-2ae54b140a97","type":"ColumnDataSource"},"glyph":{"id":"d28f5120-b8f0-40e6-887c-728d11057746","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"82a82a52-080f-4045-b96a-7c7ac4d7ef02","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"e640cf52-a7de-46ce-9d57-d407fab56eea","type":"ColumnDataSource"},"glyph":{"id":"25e53878-bc4f-421d-91e6-c5ffc0b7594b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"440085ce-3134-4306-b3bb-337bcd8ec421","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8580ebb3-92ae-4f2f-933f-495279376c07","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"73e8e4d3-2463-46ed-bb60-05231c1ebc3a","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"298d87a0-7300-4698-a312-8145136055db","type":"ColumnDataSource"},"glyph":{"id":"8580ebb3-92ae-4f2f-933f-495279376c07","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"9adfaa90-fa4c-4c84-8baf-1320035a6114","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"bdb7b653-1253-4fff-afab-c3975f2b5347","type":"ColumnDataSource"},"glyph":{"id":"a23ac9a0-2ae4-42c7-b7f3-0b4a7b408785","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"6283ac5f-fb27-40d2-9d0a-5d719829d306","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"dd193fca-e069-4659-8217-395da2032023","type":"Rect"},{"attributes":{"data_source":{"id":"4be65834-cc5d-4781-9572-5006350660f6","type":"ColumnDataSource"},"glyph":{"id":"aff2d5c6-0b27-4e33-a432-fa6b29cf6a96","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"51514218-f46d-43b3-ad08-450c8529acb1","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9a198ec5-14d3-4832-88a2-2389aee18a91","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8fbcffb2-70f9-4662-92da-d28d009790bc","type":"Rect"},{"attributes":{"data_source":{"id":"b97a6a0e-6307-4d71-a15c-9a9793e4ffd8","type":"ColumnDataSource"},"glyph":{"id":"9a198ec5-14d3-4832-88a2-2389aee18a91","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"fddd23f5-998f-4cba-afdd-4a01cbe5b157","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"plano"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["outra"],"y":[84.5]}},"id":"6d029acd-0623-43cd-bc04-bdceb508ee18","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a23ac9a0-2ae4-42c7-b7f3-0b4a7b408785","type":"Rect"},{"attributes":{"data_source":{"id":"aea2750b-4fda-46a3-808f-a770520c98ac","type":"ColumnDataSource"},"glyph":{"id":"8fbcffb2-70f9-4662-92da-d28d009790bc","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ef580c9c-752a-4acb-a6b2-c4276b6a87ff","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"201b6082-3fbd-4927-8e87-156c2dac1362","type":"ColumnDataSource"},"glyph":{"id":"59876611-f73c-4028-8b8c-051f0c812168","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8699532b-b8f0-488f-9055-1f5f4a24279c","type":"GlyphRenderer"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"dd74cd73-e3d4-47d6-9a4d-9466f2e33678","type":"BoxAnnotation"},{"attributes":{"data_source":{"id":"0f6dec50-5f56-4db7-87fa-2645840356d3","type":"ColumnDataSource"},"glyph":{"id":"bc314a5c-0098-4d99-8b1e-9ba69a876e3d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8d236c76-7920-4eca-ae14-4ca2985f45d9","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"42fa348e-d589-4a3c-af69-3277fd1b5aa4","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"99b33916-6fea-4238-a562-6567ee78955d","type":"ColumnDataSource"},"glyph":{"id":"c324032c-1998-4b40-b941-2c28f64ce02f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2a3320c1-6eea-4d43-9a59-e7b8f6560e29","type":"GlyphRenderer"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"923b913b-fbab-4999-9dfa-034af8bcebe0","type":"PanTool"},{"id":"25333594-8bec-475a-8a61-d6c5b3ba5abe","type":"WheelZoomTool"},{"id":"c5b1bad1-366b-4bff-87b4-89f3ec602086","type":"BoxZoomTool"},{"id":"0afba55b-1bec-4507-bbd8-2194a3aff134","type":"SaveTool"},{"id":"628fcada-069a-452d-a775-3ed218c56bf5","type":"ResetTool"},{"id":"a85279ce-f6e7-4804-ba3b-01cb282de6aa","type":"HelpTool"}]},"id":"04352ed0-96d1-4d92-ae1c-58e93ba01dd5","type":"Toolbar"},{"attributes":{"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"923b913b-fbab-4999-9dfa-034af8bcebe0","type":"PanTool"},{"attributes":{"plot":null,"text":"Quantidade de Leis por sub-categoria"},"id":"e227383d-293b-4b47-a647-508bd47073c9","type":"Title"},{"attributes":{"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"25333594-8bec-475a-8a61-d6c5b3ba5abe","type":"WheelZoomTool"},{"attributes":{"overlay":{"id":"dd74cd73-e3d4-47d6-9a4d-9466f2e33678","type":"BoxAnnotation"},"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"c5b1bad1-366b-4bff-87b4-89f3ec602086","type":"BoxZoomTool"},{"attributes":{"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"0afba55b-1bec-4507-bbd8-2194a3aff134","type":"SaveTool"},{"attributes":{"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"628fcada-069a-452d-a775-3ed218c56bf5","type":"ResetTool"},{"attributes":{"plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"a85279ce-f6e7-4804-ba3b-01cb282de6aa","type":"HelpTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[77.0],"label":[{"categoria_geral":"cria","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["cria"],"y":[38.5]}},"id":"99b33916-6fea-4238-a562-6567ee78955d","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"3c74748d-2a5b-4ce7-ab8d-fd7962e6a4da","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ae166a48-4ae3-4fe3-aec0-6d34655116e0","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c324032c-1998-4b40-b941-2c28f64ce02f","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"af6264cc-2c54-4d44-a09e-31cb0303f1e7","type":"Rect"},{"attributes":{"data_source":{"id":"3c74748d-2a5b-4ce7-ab8d-fd7962e6a4da","type":"ColumnDataSource"},"glyph":{"id":"ae166a48-4ae3-4fe3-aec0-6d34655116e0","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"dfff848e-c57e-45f4-9c42-dc8704d5bb30","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"bc314a5c-0098-4d99-8b1e-9ba69a876e3d","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"139f76c9-22a7-494c-b1c5-ebfd511ebbc4","type":"Rect"},{"attributes":{"data_source":{"id":"42fa348e-d589-4a3c-af69-3277fd1b5aa4","type":"ColumnDataSource"},"glyph":{"id":"139f76c9-22a7-494c-b1c5-ebfd511ebbc4","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4f0175c5-6f8f-451f-83d2-81ee4fb13007","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6ceda2a4-00b4-4563-b141-63514007a84c","type":"Rect"},{"attributes":{"data_source":{"id":"da3b4b8b-3fc0-496c-b8b8-94cd30a020f1","type":"ColumnDataSource"},"glyph":{"id":"04a65187-45d6-4a79-a62f-ce46a4224d74","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"261c6520-0973-4c79-9a42-20678ebf2da4","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["modifica"],"chart_index":[{"categoria_geral":"modifica","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"modifica","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["modifica"],"y":[4.0]}},"id":"f2b11d55-e77f-40b6-b2c1-1916f1873eeb","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"f2b11d55-e77f-40b6-b2c1-1916f1873eeb","type":"ColumnDataSource"},"glyph":{"id":"8b300244-18b7-41b9-a6a4-98ef568f3470","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"747650d0-71b2-4250-b371-65e39dca666f","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"656a5976-e6fd-49e6-80f2-d46846ce2187","type":"ColumnDataSource"},"glyph":{"id":"a7ec75a0-01d8-467f-8879-6b7fd76c57fb","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"297d2af8-18c5-466b-89f6-6b6bfed82997","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["disp\u00f5e"],"y":[122.5]}},"id":"057f007e-9e27-4b26-b7af-b345ebc04cf5","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5d81b0c3-e01b-404e-9266-2fb18799feb7","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"841dff23-15e9-47c4-87ee-38415e087adb","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"201b6082-3fbd-4927-8e87-156c2dac1362","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"fc0faf0f-78cf-4ca0-92f9-dccbb1c23332","type":"ColumnDataSource"},"glyph":{"id":"8be9e9e4-8177-40cd-867b-2ad494266a0a","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"0acce490-0b83-4671-a505-162fecca9e7a","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"376f75c7-335d-4e48-a0a7-91dde405f914","type":"ColumnDataSource"},"glyph":{"id":"d7739a0c-48fd-4f50-8c0b-6061694d43fd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c1e7d830-f802-4664-ad09-a06923668353","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"04a65187-45d6-4a79-a62f-ce46a4224d74","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui,"],"chart_index":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui,"],"y":[3.0]}},"id":"bb5bcb06-5464-483d-ba06-83e2064c6454","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d7739a0c-48fd-4f50-8c0b-6061694d43fd","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c9fc3968-d124-4c22-ac16-68465f02ff5d","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disp\u00f5e"],"y":[34.0]}},"id":"953cb3ae-ef36-4615-a3b9-017bdde6d2b7","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["contrata\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[64.5]}},"id":"770fbc6b-de4f-4e0c-bd42-71f35268a3d5","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"962638f3-ba2c-4ccf-ae96-bb1b8508d1a4","type":"Rect"},{"attributes":{"data_source":{"id":"135eac66-e89f-433e-9acb-7932aca3d288","type":"ColumnDataSource"},"glyph":{"id":"c9fc3968-d124-4c22-ac16-68465f02ff5d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"06cac2b7-9be9-429a-b130-7d27d081fcda","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"d3552387-b0f6-40f1-bd71-00ef4253a22c","type":"ColumnDataSource"},"glyph":{"id":"4ce9ccdb-89f8-4787-a408-77fded15cb1a","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"baedbe20-2e6e-4d22-89b5-0c30e7815e7e","type":"GlyphRenderer"},{"attributes":{"label":{"value":"diretrizes"},"renderers":[{"id":"2ba70e81-c46a-48ad-9ffc-0db8581f51f0","type":"GlyphRenderer"}]},"id":"0eca91eb-01e1-4a1d-bb91-ac7dc7691b1d","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[83.0],"label":[{"categoria_geral":"outra","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[41.5]}},"id":"0f6dec50-5f56-4db7-87fa-2645840356d3","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"5eb56e4b-0661-41bd-a3a7-b2a29626df00","type":"ColumnDataSource"},"glyph":{"id":"962638f3-ba2c-4ccf-ae96-bb1b8508d1a4","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"881ff678-8288-44ea-93d1-c7e01684991b","type":"GlyphRenderer"},{"attributes":{},"id":"ce418743-ee20-4dea-a594-8a95f10514c5","type":"ToolEvents"},{"attributes":{"data_source":{"id":"15ce7cb8-7535-4594-aeeb-77bbb3a34bc6","type":"ColumnDataSource"},"glyph":{"id":"6ceda2a4-00b4-4563-b141-63514007a84c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f5573fce-27b3-49ca-aa35-5d38e5969d21","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[27.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[89.5]}},"id":"656a5976-e6fd-49e6-80f2-d46846ce2187","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7cd9eebe-8b62-471f-90e3-70de73295f5f","type":"Rect"},{"attributes":{"data_source":{"id":"0b5b4814-a888-4194-bcc3-dea7b1c0ed63","type":"ColumnDataSource"},"glyph":{"id":"841dff23-15e9-47c4-87ee-38415e087adb","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c0044790-5351-49b1-9d44-836cf8210d2e","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"fc0faf0f-78cf-4ca0-92f9-dccbb1c23332","type":"ColumnDataSource"},{"attributes":{"callback":null,"factors":["?","altera","atribui","autoriza","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","institui,","modifica","outra","reajusta","revoga"]},"id":"964bd447-afb9-44f2-9b5e-fd99a400a33c","type":"FactorRange"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8be9e9e4-8177-40cd-867b-2ad494266a0a","type":"Rect"},{"attributes":{"data_source":{"id":"057f007e-9e27-4b26-b7af-b345ebc04cf5","type":"ColumnDataSource"},"glyph":{"id":"7cd9eebe-8b62-471f-90e3-70de73295f5f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"22a27438-0e58-4b9d-9a93-bdc5821a07b2","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["disp\u00f5e"],"y":[71.0]}},"id":"b97a6a0e-6307-4d71-a15c-9a9793e4ffd8","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ffd495e9-72cc-4c9b-a410-45b2321b5373","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza","sub_categoria":"cr\u00e9dito"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[15.0],"label":[{"categoria_geral":"autoriza","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["autoriza"],"y":[85.5]}},"id":"067d832a-10fe-41f2-a805-7978a60bcc23","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["abertura"],"width":[0.8],"x":["disp\u00f5e"],"y":[107.5]}},"id":"da3b4b8b-3fc0-496c-b8b8-94cd30a020f1","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"bb5bcb06-5464-483d-ba06-83e2064c6454","type":"ColumnDataSource"},"glyph":{"id":"2723b0af-4e09-4bf6-90a7-fa01e9234059","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b34e2b6b-2bcc-4210-8350-9f9b15c52a3e","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"2723b0af-4e09-4bf6-90a7-fa01e9234059","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"104e9036-230d-485c-abfd-e1a8c1ce8de6","type":"Rect"},{"attributes":{"data_source":{"id":"80b0067b-fa01-40b2-922d-3e27d45c6311","type":"ColumnDataSource"},"glyph":{"id":"104e9036-230d-485c-abfd-e1a8c1ce8de6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f11987b9-ef86-4136-97ed-f0177aeb8c78","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"6d029acd-0623-43cd-bc04-bdceb508ee18","type":"ColumnDataSource"},"glyph":{"id":"93ac5f51-baba-490c-a3bf-ec70329a30eb","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b0cecae7-3aba-41c7-a4e1-19641ef356da","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"49848ed2-d180-4867-9c90-1ad6863b3ea5","type":"ColumnDataSource"},"glyph":{"id":"5d81b0c3-e01b-404e-9266-2fb18799feb7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"1c56236a-8a1b-429f-9976-0d53b447dc68","type":"GlyphRenderer"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"4f0175c5-6f8f-451f-83d2-81ee4fb13007","type":"GlyphRenderer"}]},"id":"63957e47-098c-41ac-8792-ab6b844331eb","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8b300244-18b7-41b9-a6a4-98ef568f3470","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["revis\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[116.0]}},"id":"135eac66-e89f-433e-9acb-7932aca3d288","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1c6a44fe-bd7e-41c0-b1f7-517559423bda","type":"Rect"},{"attributes":{"data_source":{"id":"3f2bcd15-8ce7-45ef-b247-846a62884e57","type":"ColumnDataSource"},"glyph":{"id":"ffd495e9-72cc-4c9b-a410-45b2321b5373","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2b8f314d-5905-40aa-81e0-776d903183ee","type":"GlyphRenderer"},{"attributes":{"callback":null,"end":174.3},"id":"568bd3bd-92a7-4919-9c84-d9f8a039984e","type":"Range1d"},{"attributes":{"data_source":{"id":"447388e3-9074-48a7-baa3-94adb789e2e1","type":"ColumnDataSource"},"glyph":{"id":"1c6a44fe-bd7e-41c0-b1f7-517559423bda","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a93c543e-eb9c-4639-88e4-08d22c2ec1f3","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"93ac5f51-baba-490c-a3bf-ec70329a30eb","type":"Rect"},{"attributes":{"below":[{"id":"025be0ca-4cf4-4fed-8ff6-e509323f154f","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"b828762d-e0f6-424b-964e-88f222a147c2","type":"LinearAxis"}],"renderers":[{"id":"dd74cd73-e3d4-47d6-9a4d-9466f2e33678","type":"BoxAnnotation"},{"id":"4f0175c5-6f8f-451f-83d2-81ee4fb13007","type":"GlyphRenderer"},{"id":"f5573fce-27b3-49ca-aa35-5d38e5969d21","type":"GlyphRenderer"},{"id":"8699532b-b8f0-488f-9055-1f5f4a24279c","type":"GlyphRenderer"},{"id":"2a3320c1-6eea-4d43-9a59-e7b8f6560e29","type":"GlyphRenderer"},{"id":"dfff848e-c57e-45f4-9c42-dc8704d5bb30","type":"GlyphRenderer"},{"id":"8d236c76-7920-4eca-ae14-4ca2985f45d9","type":"GlyphRenderer"},{"id":"2ba70e81-c46a-48ad-9ffc-0db8581f51f0","type":"GlyphRenderer"},{"id":"5f799efb-fc5d-4499-b392-f9c226c3fab9","type":"GlyphRenderer"},{"id":"8a051ecf-aaec-4d51-b2e6-130b72e0ad27","type":"GlyphRenderer"},{"id":"e6bdd7c6-932e-4767-9a80-6a72d50b1dae","type":"GlyphRenderer"},{"id":"9adfaa90-fa4c-4c84-8baf-1320035a6114","type":"GlyphRenderer"},{"id":"55561680-c17c-47bb-b274-9f693465f835","type":"GlyphRenderer"},{"id":"89f8126c-2dd0-4d58-9ae9-59933d2d79d2","type":"GlyphRenderer"},{"id":"6283ac5f-fb27-40d2-9d0a-5d719829d306","type":"GlyphRenderer"},{"id":"03e52158-c516-4112-872d-1b0c28a57e02","type":"GlyphRenderer"},{"id":"e41dbe8c-b48d-4b03-ae58-25a6757cfd1e","type":"GlyphRenderer"},{"id":"eac9730d-466d-4508-bc66-7ba3124b0b20","type":"GlyphRenderer"},{"id":"440085ce-3134-4306-b3bb-337bcd8ec421","type":"GlyphRenderer"},{"id":"82a82a52-080f-4045-b96a-7c7ac4d7ef02","type":"GlyphRenderer"},{"id":"51514218-f46d-43b3-ad08-450c8529acb1","type":"GlyphRenderer"},{"id":"fddd23f5-998f-4cba-afdd-4a01cbe5b157","type":"GlyphRenderer"},{"id":"297d2af8-18c5-466b-89f6-6b6bfed82997","type":"GlyphRenderer"},{"id":"261c6520-0973-4c79-9a42-20678ebf2da4","type":"GlyphRenderer"},{"id":"06cac2b7-9be9-429a-b130-7d27d081fcda","type":"GlyphRenderer"},{"id":"22a27438-0e58-4b9d-9a93-bdc5821a07b2","type":"GlyphRenderer"},{"id":"ef580c9c-752a-4acb-a6b2-c4276b6a87ff","type":"GlyphRenderer"},{"id":"0acce490-0b83-4671-a505-162fecca9e7a","type":"GlyphRenderer"},{"id":"2b8f314d-5905-40aa-81e0-776d903183ee","type":"GlyphRenderer"},{"id":"baedbe20-2e6e-4d22-89b5-0c30e7815e7e","type":"GlyphRenderer"},{"id":"747650d0-71b2-4250-b371-65e39dca666f","type":"GlyphRenderer"},{"id":"c1e7d830-f802-4664-ad09-a06923668353","type":"GlyphRenderer"},{"id":"881ff678-8288-44ea-93d1-c7e01684991b","type":"GlyphRenderer"},{"id":"c0044790-5351-49b1-9d44-836cf8210d2e","type":"GlyphRenderer"},{"id":"b34e2b6b-2bcc-4210-8350-9f9b15c52a3e","type":"GlyphRenderer"},{"id":"1c56236a-8a1b-429f-9976-0d53b447dc68","type":"GlyphRenderer"},{"id":"a93c543e-eb9c-4639-88e4-08d22c2ec1f3","type":"GlyphRenderer"},{"id":"f11987b9-ef86-4136-97ed-f0177aeb8c78","type":"GlyphRenderer"},{"id":"761f5025-7a24-415a-8efb-c36805af74d1","type":"GlyphRenderer"},{"id":"b0cecae7-3aba-41c7-a4e1-19641ef356da","type":"GlyphRenderer"},{"id":"7f1a4ae0-e68b-412a-b53a-203228e02be0","type":"Legend"},{"id":"025be0ca-4cf4-4fed-8ff6-e509323f154f","type":"CategoricalAxis"},{"id":"b828762d-e0f6-424b-964e-88f222a147c2","type":"LinearAxis"},{"id":"787a73ee-d6d7-4240-9242-f387e5f07a5e","type":"Grid"}],"title":{"id":"e227383d-293b-4b47-a647-508bd47073c9","type":"Title"},"tool_events":{"id":"ce418743-ee20-4dea-a594-8a95f10514c5","type":"ToolEvents"},"toolbar":{"id":"04352ed0-96d1-4d92-ae1c-58e93ba01dd5","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"964bd447-afb9-44f2-9b5e-fd99a400a33c","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"568bd3bd-92a7-4919-9c84-d9f8a039984e","type":"Range1d"}},"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["outra"],"y":[83.5]}},"id":"49848ed2-d180-4867-9c90-1ad6863b3ea5","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4ce9ccdb-89f8-4787-a408-77fded15cb1a","type":"Rect"},{"attributes":{"items":[{"id":"63957e47-098c-41ac-8792-ab6b844331eb","type":"LegendItem"},{"id":"0eca91eb-01e1-4a1d-bb91-ac7dc7691b1d","type":"LegendItem"},{"id":"d3cfb565-d87b-444b-929c-81af2a071f0e","type":"LegendItem"},{"id":"af4fb797-44e0-4351-a80d-e94337a96ef6","type":"LegendItem"},{"id":"a68d6560-7a5d-4b65-8d9e-1dac72c7289e","type":"LegendItem"},{"id":"8b0a315e-6453-4e9f-8b76-62781562c906","type":"LegendItem"},{"id":"2dc94be3-6432-418f-8e01-603b4b5ba4fe","type":"LegendItem"},{"id":"01872457-d852-4e64-b499-959e3e33696f","type":"LegendItem"},{"id":"937e17f9-28b9-4d40-8b48-9b27cb2433af","type":"LegendItem"},{"id":"7aed4421-aff9-488c-ac62-f3009ad128ba","type":"LegendItem"},{"id":"0ccdd9ec-32d6-4f03-98dc-b321898c827e","type":"LegendItem"},{"id":"1fb9b79f-9d86-4dc9-baae-e58da3847556","type":"LegendItem"},{"id":"afcd5111-c3c4-498b-9658-e524667ac348","type":"LegendItem"},{"id":"8e6628d9-1873-431e-8e06-bed8224b18ad","type":"LegendItem"},{"id":"4b9e9247-24b8-4254-899c-ab6a7a3d7d96","type":"LegendItem"},{"id":"5b9174d5-e8ac-4f95-b97f-8f81385575e8","type":"LegendItem"}],"location":"top_left","plot":{"id":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf","subtype":"Chart","type":"Plot"}},"id":"7f1a4ae0-e68b-412a-b53a-203228e02be0","type":"Legend"}],"root_ids":["fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"1caffb7c-5f35-4705-8ef8-7d51dd3d5753","elementid":"7685b1c0-8beb-4763-96aa-6f0b5b4783ad","modelid":"fd0eb9cc-e6cb-4ae4-a04a-c8125db17daf"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("7685b1c0-8beb-4763-96aa-6f0b5b4783ad")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'modifica'
Vamos investigar se as categorias 'altera' e 'modifica' podem ser combinadas


```python
print(leis[leis['categoria_geral'] == 'altera']['ementa'].head())
print(leis[leis['categoria_geral'] == 'modifica']['ementa'].head())
```

    89     altera o art.2 da lei municipal 1046/93 e dá o...
    130    altera classificação de cargos comissionados d...
    134    altera a lei n° 1.100/95 e dá outras providenc...
    135    altera vantagens dos professores do ensino de ...
    153    altera simbologia de cargo de provimento em co...
    Name: ementa, dtype: object
    353    modifica e renumera a lei municipal n° 1.139/9...
    355    modifica a lei municipal n.° 1.312/02 e dá out...
    392    modifica os deveres e atribuições dos cargos d...
    402    modifica o anexo único da lei 1414/2005, crian...
    415    modifica o artigo 4° da lei 1337/2002 e dá out...
    Name: ementa, dtype: object


### Comentários
As duas parecem alterar Leis, vamos combiná-las


```python
def change_category_label(to_change, old_category, new_category):
    if(old_category == to_change):
        return new_category
    else:
        return old_category

leis['categoria_geral'] = leis.apply(lambda row: change_category_label('modifica', row['categoria_geral'],'altera'), axis=1)
```


```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por sub-categoria", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="d3c09797-a4ba-45f0-914a-1f319b1a7d14"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("d3c09797-a4ba-45f0-914a-1f319b1a7d14").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("d3c09797-a4ba-45f0-914a-1f319b1a7d14");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid 'd3c09797-a4ba-45f0-914a-1f319b1a7d14' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"90fe5fde-596f-4721-a7a6-f57708b20c58":{"roots":{"references":[{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[78.0],"label":[{"categoria_geral":"autoriza","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["autoriza"],"y":[39.0]}},"id":"ec35d4ab-76eb-481d-b4ac-2d3330675904","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"d3c25591-b9f2-4000-a6ab-cae22c41ba5d","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"686c8004-6f34-4aaa-b345-e24574e56ad2","type":"ColumnDataSource"},"glyph":{"id":"03b9b7a3-f11c-4c9f-baa4-33e7b9e7e60e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"080656ec-840f-47d6-9786-6dcfd424d79a","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b8ce5a61-77b9-4541-95c1-f68c42b760b8","type":"Rect"},{"attributes":{"data_source":{"id":"ac931838-17fe-4e23-a8f7-6b02310c7f7f","type":"ColumnDataSource"},"glyph":{"id":"9a8b04fe-9edb-4b32-9fd2-ac3974f69a83","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"712406ef-6180-4e66-bf3e-f933f0d139b1","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"b25a61cc-480b-42ce-92c2-427ef4a721b1","type":"ColumnDataSource"},"glyph":{"id":"a19cf9d6-7c8d-4cc5-b01c-93de98100b5f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8efbbb0d-6c52-4523-8f19-c8a3f69a42f2","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"24e5c8b2-1d6d-44d2-ba33-e4224ec01527","type":"Rect"},{"attributes":{"callback":null,"end":174.3},"id":"9283ed03-01c4-4925-abbc-61759c41f131","type":"Range1d"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[77.0],"label":[{"categoria_geral":"cria","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["cria"],"y":[38.5]}},"id":"b7a90d7d-a3f8-4327-80b6-d819e03e61bc","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"8bd54487-108a-45b8-8874-93ac4ddea8b7","type":"ColumnDataSource"},"glyph":{"id":"4e3ae053-96f3-4304-9d4d-3b7f27415475","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"784dc2dd-5f0b-41d7-b2d7-4b3d9d06a465","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"e04218a2-0811-4f80-ace7-921a60cda5ec","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a5a17f4f-e478-4135-b000-40c4e4939776","type":"ColumnDataSource"},"glyph":{"id":"f5b2a87f-d783-48f2-8810-e2cd6c9024f0","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"16a7a8d2-bc9c-4a04-8429-37a7e5fd6737","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"8bd54487-108a-45b8-8874-93ac4ddea8b7","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"e04218a2-0811-4f80-ace7-921a60cda5ec","type":"ColumnDataSource"},"glyph":{"id":"821f1d4b-114f-40d3-901d-29bace9aa3d0","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"25ba8226-5add-457a-a17b-c9fc1004604f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9a8b04fe-9edb-4b32-9fd2-ac3974f69a83","type":"Rect"},{"attributes":{"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"a26a27f5-2dea-4115-b13f-882bd88595fb","type":"HelpTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["diretrizes"],"width":[0.8],"x":["disp\u00f5e"],"y":[2.5]}},"id":"9087b51a-8c5a-48d9-b58c-efb39be6787f","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"ec35d4ab-76eb-481d-b4ac-2d3330675904","type":"ColumnDataSource"},"glyph":{"id":"d035cc50-d7d4-4991-b0ca-dc3808abbd1d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3069d767-e816-4198-a4ed-dcd91405abfe","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"73fa476a-cdbc-4b85-9650-6a67efa4bf5c","type":"ColumnDataSource"},"glyph":{"id":"843d25a5-d230-4db0-a53d-0242eca5dde1","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3fc7fcf0-38da-450e-83ed-0455da02e6ab","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"c28e1a0b-6cff-4e20-bc6e-088043550aef","type":"ColumnDataSource"},"glyph":{"id":"b8ce5a61-77b9-4541-95c1-f68c42b760b8","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"badf03e4-ca5e-4418-8968-2ac8b64f0140","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e89dc515-7a8f-457b-a8e2-97a99ebb3797","type":"Rect"},{"attributes":{"label":{"value":"abertura"},"renderers":[{"id":"8efbbb0d-6c52-4523-8f19-c8a3f69a42f2","type":"GlyphRenderer"}]},"id":"bcb36a67-a442-4a95-b83f-362bfa8d42a5","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"821f1d4b-114f-40d3-901d-29bace9aa3d0","type":"Rect"},{"attributes":{"data_source":{"id":"e0e4d09b-afe5-417e-ae82-868048ef43c6","type":"ColumnDataSource"},"glyph":{"id":"4c98f499-c08c-4ece-9dab-82c760ec20d3","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8a366482-bfbc-4597-b9f4-ca03c6c5a40c","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4c98f499-c08c-4ece-9dab-82c760ec20d3","type":"Rect"},{"attributes":{"label":{"value":"altera\u00e7\u00e3o"},"renderers":[{"id":"8a366482-bfbc-4597-b9f4-ca03c6c5a40c","type":"GlyphRenderer"}]},"id":"5d0183a8-d42f-4cf8-ab05-ac723789c99a","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4e3ae053-96f3-4304-9d4d-3b7f27415475","type":"Rect"},{"attributes":{"label":{"value":"institui\u00e7\u00e3o"},"renderers":[{"id":"930214ba-7d1c-4c03-aa3b-ef95afc33844","type":"GlyphRenderer"}]},"id":"8f0739f8-5dbf-435d-a1b0-48fa91a6e288","type":"LegendItem"},{"attributes":{"data_source":{"id":"d3c25591-b9f2-4000-a6ab-cae22c41ba5d","type":"ColumnDataSource"},"glyph":{"id":"e89dc515-7a8f-457b-a8e2-97a99ebb3797","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ef040920-d49d-400c-abfe-5ed2cb345aad","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera"],"chart_index":[{"categoria_geral":"altera","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[116.0],"label":[{"categoria_geral":"altera","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["altera"],"y":[58.0]}},"id":"f416fab3-876a-439d-ae32-b77b375f4e9c","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["contrata\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[64.5]}},"id":"83c86fa0-fe94-4327-9e78-b3026a8c84ca","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"b9fda161-b000-4ec3-972e-40e8660de7b8","type":"ColumnDataSource"},"glyph":{"id":"fa1169d0-9859-4454-be57-31acd0d11a91","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bf37b4ba-ff17-4cf2-8ccb-92a85630dd9a","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reajuste"],"width":[0.8],"x":["disp\u00f5e"],"y":[136.0]}},"id":"e90d1320-f910-429d-953d-0de0bdd82bcd","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[27.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[89.5]}},"id":"20c2803e-21e6-44db-8db3-169c087424df","type":"ColumnDataSource"},{"attributes":{"label":{"value":"denomina\u00e7\u00e3o"},"renderers":[{"id":"badf03e4-ca5e-4418-8968-2ac8b64f0140","type":"GlyphRenderer"}]},"id":"0a9f50c0-af04-4cd9-abbf-76b545b95a7d","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"578f08e6-49e7-4e13-b09f-8e82f95a93f9","type":"Rect"},{"attributes":{"label":{"value":"cr\u00e9dito"},"renderers":[{"id":"bc0b7f9a-d0bd-44b4-92cb-eb23a0d2168b","type":"GlyphRenderer"}]},"id":"8e706486-3c87-487a-b4a5-923b1ffa4a25","type":"LegendItem"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"fb71452b-9fb2-4ad3-9433-c0a0e89f981f","type":"BasicTickFormatter"},"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"},"ticker":{"id":"94add172-8600-4bde-87b1-ff012f37a962","type":"BasicTicker"}},"id":"874eccc1-f72a-4828-aa4e-4bf54519f7b7","type":"LinearAxis"},{"attributes":{"label":{"value":"vencimentos"},"renderers":[{"id":"1b820274-9a35-40b9-b9c7-42d79124f7b3","type":"GlyphRenderer"}]},"id":"16a26489-a478-4d92-abcc-588463295d49","type":"LegendItem"},{"attributes":{"data_source":{"id":"14541eb1-147b-45fa-84ea-75c27885dd77","type":"ColumnDataSource"},"glyph":{"id":"578f08e6-49e7-4e13-b09f-8e82f95a93f9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"254bcdbf-e073-40cb-960f-18e4b73ded19","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["disp\u00f5e"],"y":[151.0]}},"id":"2c474b75-99b0-4b5d-9d59-70e5d6d1e300","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["altera\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[157.5]}},"id":"e0e4d09b-afe5-417e-ae82-868048ef43c6","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["revis\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[116.0]}},"id":"b9fda161-b000-4ec3-972e-40e8660de7b8","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"9087b51a-8c5a-48d9-b58c-efb39be6787f","type":"ColumnDataSource"},"glyph":{"id":"626dca50-57cc-4b9b-8381-b11e08ce7576","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f6c41406-68e9-4254-872d-c73ce1d2d96f","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["disp\u00f5e"],"y":[122.5]}},"id":"1f023461-3fa6-4cbc-9464-a5f596a5c1c7","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["institui\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[144.5]}},"id":"4e198755-0cc9-46f3-962d-e79d091fe516","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"843d25a5-d230-4db0-a53d-0242eca5dde1","type":"Rect"},{"attributes":{"overlay":{"id":"4211c36e-1593-4e85-8ca4-ce264306bd32","type":"BoxAnnotation"},"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"30954370-d9fe-4493-b2ea-7a66a3e1c849","type":"BoxZoomTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fed27b94-cfda-43b1-bf96-64a7e55b66cc","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["aumento"],"width":[0.8],"x":["disp\u00f5e"],"y":[132.5]}},"id":"e16a8c59-b938-45b1-af99-63f77dfcc79d","type":"ColumnDataSource"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"4211c36e-1593-4e85-8ca4-ce264306bd32","type":"BoxAnnotation"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d035cc50-d7d4-4991-b0ca-dc3808abbd1d","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reestrutura\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[139.5]}},"id":"6565ed58-7cfe-43d6-ac3e-577ee2e6e192","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"19d9ae53-c664-42cf-8d20-4e19af8e0ae6","type":"ResetTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["estrutura"],"width":[0.8],"x":["disp\u00f5e"],"y":[128.0]}},"id":"a207d9d3-5c37-4442-8c9b-8af5b9326d0b","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"626dca50-57cc-4b9b-8381-b11e08ce7576","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["abertura"],"width":[0.8],"x":["disp\u00f5e"],"y":[107.5]}},"id":"b25a61cc-480b-42ce-92c2-427ef4a721b1","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"20c2803e-21e6-44db-8db3-169c087424df","type":"ColumnDataSource"},"glyph":{"id":"fed27b94-cfda-43b1-bf96-64a7e55b66cc","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"21b495e3-4126-492e-a874-4d7f2caea9d6","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["disp\u00f5e"],"y":[71.0]}},"id":"14541eb1-147b-45fa-84ea-75c27885dd77","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["denomina\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[163.5]}},"id":"c28e1a0b-6cff-4e20-bc6e-088043550aef","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"e90d1320-f910-429d-953d-0de0bdd82bcd","type":"ColumnDataSource"},"glyph":{"id":"46979660-5b2b-4a36-ba06-2e2975ecf1dd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"87565157-02e8-445d-a089-e480435bcc8f","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disp\u00f5e"],"y":[34.0]}},"id":"7eb2e12a-e58d-4c12-be38-1deffff74b87","type":"ColumnDataSource"},{"attributes":{"dimension":1,"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"},"ticker":{"id":"94add172-8600-4bde-87b1-ff012f37a962","type":"BasicTicker"}},"id":"7217d6dd-e00e-492e-93e9-0a15cc9cd77d","type":"Grid"},{"attributes":{"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"013e79a7-c705-43d6-9671-6dabd1f9e908","type":"SaveTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7f3c191e-8c6e-4499-95ea-dfa4ae18ff35","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["estabelece"],"y":[27.0]}},"id":"1cf3c6f3-74b0-4bde-86db-6b3cd9fcbb24","type":"ColumnDataSource"},{"attributes":{"label":{"value":"contrata\u00e7\u00e3o"},"renderers":[{"id":"e47f87a7-6963-4b9c-80bf-0508714bc81e","type":"GlyphRenderer"}]},"id":"aedfb1c3-0140-4293-93ce-a4951af7117a","type":"LegendItem"},{"attributes":{"data_source":{"id":"26011cbd-93ef-487f-8986-beccdd0a5822","type":"ColumnDataSource"},"glyph":{"id":"d7d3f63b-f683-4e91-8be3-396f08443d56","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"62e07a2e-66f5-4acd-a8a4-d0ee1d3c3a99","type":"GlyphRenderer"},{"attributes":{"label":{"value":"cria\u00e7\u00e3o"},"renderers":[{"id":"21b495e3-4126-492e-a874-4d7f2caea9d6","type":"GlyphRenderer"}]},"id":"bdf211e1-80d1-4225-af26-4802fc5ee3ba","type":"LegendItem"},{"attributes":{"label":{"value":"plano"},"renderers":[{"id":"254bcdbf-e073-40cb-960f-18e4b73ded19","type":"GlyphRenderer"}]},"id":"8a82d484-ab1b-4d94-99ac-bb96229b6e51","type":"LegendItem"},{"attributes":{"callback":null,"factors":["?","altera","atribui","autoriza","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","institui,","outra","reajusta","revoga"]},"id":"759b0606-3ade-4f36-84b5-422a55c3af74","type":"FactorRange"},{"attributes":{"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"e43f24e5-1155-406f-936c-4f684dd70647","type":"PanTool"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"56c64148-0d9f-4707-9f66-0cf495d6851d","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"},"ticker":{"id":"807da81d-dd72-415f-a1ef-e041da76d5de","type":"CategoricalTicker"}},"id":"9666069c-dca7-4002-a78b-aac5d38b9c76","type":"CategoricalAxis"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5b4437df-1367-40bb-9e84-3c390bfad382","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"dd8d144a-a927-44d8-b9e2-02257ec6cc52","type":"ColumnDataSource"},{"attributes":{},"id":"94add172-8600-4bde-87b1-ff012f37a962","type":"BasicTicker"},{"attributes":{"data_source":{"id":"c9013e10-070c-431f-aa25-a21d368afac1","type":"ColumnDataSource"},"glyph":{"id":"5b4437df-1367-40bb-9e84-3c390bfad382","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bc0b7f9a-d0bd-44b4-92cb-eb23a0d2168b","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5f2a5d95-6fcb-4a21-8c18-bf1d2a62e83b","type":"Rect"},{"attributes":{},"id":"807da81d-dd72-415f-a1ef-e041da76d5de","type":"CategoricalTicker"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fa1169d0-9859-4454-be57-31acd0d11a91","type":"Rect"},{"attributes":{},"id":"56c64148-0d9f-4707-9f66-0cf495d6851d","type":"CategoricalTickFormatter"},{"attributes":{"plot":null,"text":"Quantidade de Leis por sub-categoria"},"id":"72cd2227-f289-4e5b-a0ed-c6d662543c93","type":"Title"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"26011cbd-93ef-487f-8986-beccdd0a5822","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"46979660-5b2b-4a36-ba06-2e2975ecf1dd","type":"Rect"},{"attributes":{},"id":"fb71452b-9fb2-4ad3-9433-c0a0e89f981f","type":"BasicTickFormatter"},{"attributes":{"data_source":{"id":"dd8d144a-a927-44d8-b9e2-02257ec6cc52","type":"ColumnDataSource"},"glyph":{"id":"8002d431-9928-43a2-b7be-c08131d62e70","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e8c3fe5a-da4c-4f49-8532-d219990dde94","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"1f023461-3fa6-4cbc-9464-a5f596a5c1c7","type":"ColumnDataSource"},"glyph":{"id":"5f2a5d95-6fcb-4a21-8c18-bf1d2a62e83b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"cdab02bc-2c76-40d0-86b1-421d5ce443d7","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d7d3f63b-f683-4e91-8be3-396f08443d56","type":"Rect"},{"attributes":{"plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"346840d2-ecfc-4ab9-9d5a-d56a61e5435e","type":"WheelZoomTool"},{"attributes":{"label":{"value":"revis\u00e3o"},"renderers":[{"id":"bf37b4ba-ff17-4cf2-8ccb-92a85630dd9a","type":"GlyphRenderer"}]},"id":"261ec002-6546-4b7e-8d77-77b0fb31a348","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"485b6ae9-04b7-40f3-899e-3685475e54f7","type":"ColumnDataSource"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"e43f24e5-1155-406f-936c-4f684dd70647","type":"PanTool"},{"id":"346840d2-ecfc-4ab9-9d5a-d56a61e5435e","type":"WheelZoomTool"},{"id":"30954370-d9fe-4493-b2ea-7a66a3e1c849","type":"BoxZoomTool"},{"id":"013e79a7-c705-43d6-9671-6dabd1f9e908","type":"SaveTool"},{"id":"19d9ae53-c664-42cf-8d20-4e19af8e0ae6","type":"ResetTool"},{"id":"a26a27f5-2dea-4115-b13f-882bd88595fb","type":"HelpTool"}]},"id":"fdf530f8-f4d3-4026-babc-abaf2645355b","type":"Toolbar"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"3f964efe-4ef5-489f-bc9a-58781a6c2189","type":"ColumnDataSource"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"784dc2dd-5f0b-41d7-b2d7-4b3d9d06a465","type":"GlyphRenderer"}]},"id":"f19926f4-a7f0-465b-860f-cd5ab202c051","type":"LegendItem"},{"attributes":{"items":[{"id":"f19926f4-a7f0-465b-860f-cd5ab202c051","type":"LegendItem"},{"id":"275efd8a-fa26-4944-9992-2075f0c13555","type":"LegendItem"},{"id":"8e706486-3c87-487a-b4a5-923b1ffa4a25","type":"LegendItem"},{"id":"aedfb1c3-0140-4293-93ce-a4951af7117a","type":"LegendItem"},{"id":"8a82d484-ab1b-4d94-99ac-bb96229b6e51","type":"LegendItem"},{"id":"bdf211e1-80d1-4225-af26-4802fc5ee3ba","type":"LegendItem"},{"id":"bcb36a67-a442-4a95-b83f-362bfa8d42a5","type":"LegendItem"},{"id":"261ec002-6546-4b7e-8d77-77b0fb31a348","type":"LegendItem"},{"id":"9dd8e3a1-0417-439b-a76c-287426bc3559","type":"LegendItem"},{"id":"24144962-b4db-4230-ab8f-9c7292232f28","type":"LegendItem"},{"id":"cf28ee8f-bdeb-49e1-b356-2c8911a39f42","type":"LegendItem"},{"id":"d767466f-bdda-4208-9d95-8ea820bb704c","type":"LegendItem"},{"id":"8f0739f8-5dbf-435d-a1b0-48fa91a6e288","type":"LegendItem"},{"id":"16a26489-a478-4d92-abcc-588463295d49","type":"LegendItem"},{"id":"5d0183a8-d42f-4cf8-ab05-ac723789c99a","type":"LegendItem"},{"id":"0a9f50c0-af04-4cd9-abbf-76b545b95a7d","type":"LegendItem"}],"location":"top_left","plot":{"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"}},"id":"9508c8db-dfa8-4ed9-94dd-679e728fae2d","type":"Legend"},{"attributes":{"data_source":{"id":"3f964efe-4ef5-489f-bc9a-58781a6c2189","type":"ColumnDataSource"},"glyph":{"id":"1d2d6d64-954d-46cf-9d0f-4f50f39bfd65","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"72c43674-ad8d-4533-b7c1-018bb61d66d1","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8002d431-9928-43a2-b7be-c08131d62e70","type":"Rect"},{"attributes":{},"id":"dd383674-e9a4-4ce7-baf8-2fca55658a49","type":"ToolEvents"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1d2d6d64-954d-46cf-9d0f-4f50f39bfd65","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"92d7e361-5890-476c-b3b9-9583ce3588c9","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b9c5aac1-ab25-43b4-a31a-10bf07ebefa9","type":"Rect"},{"attributes":{"label":{"value":"diretrizes"},"renderers":[{"id":"f6c41406-68e9-4254-872d-c73ce1d2d96f","type":"GlyphRenderer"}]},"id":"275efd8a-fa26-4944-9992-2075f0c13555","type":"LegendItem"},{"attributes":{"below":[{"id":"9666069c-dca7-4002-a78b-aac5d38b9c76","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"874eccc1-f72a-4828-aa4e-4bf54519f7b7","type":"LinearAxis"}],"renderers":[{"id":"4211c36e-1593-4e85-8ca4-ce264306bd32","type":"BoxAnnotation"},{"id":"784dc2dd-5f0b-41d7-b2d7-4b3d9d06a465","type":"GlyphRenderer"},{"id":"3069d767-e816-4198-a4ed-dcd91405abfe","type":"GlyphRenderer"},{"id":"72c43674-ad8d-4533-b7c1-018bb61d66d1","type":"GlyphRenderer"},{"id":"19a5661a-bfdc-49e4-b19d-36cd7f15c2a9","type":"GlyphRenderer"},{"id":"e8c3fe5a-da4c-4f49-8532-d219990dde94","type":"GlyphRenderer"},{"id":"712406ef-6180-4e66-bf3e-f933f0d139b1","type":"GlyphRenderer"},{"id":"f6c41406-68e9-4254-872d-c73ce1d2d96f","type":"GlyphRenderer"},{"id":"bc0b7f9a-d0bd-44b4-92cb-eb23a0d2168b","type":"GlyphRenderer"},{"id":"d0ec1df7-4cf3-4c12-b8f2-ceeaa31362ca","type":"GlyphRenderer"},{"id":"62e07a2e-66f5-4acd-a8a4-d0ee1d3c3a99","type":"GlyphRenderer"},{"id":"84448483-d04f-4f91-a2b3-717a18982957","type":"GlyphRenderer"},{"id":"e4ce37b8-fffc-4eb6-8cdf-18255dbc584c","type":"GlyphRenderer"},{"id":"a05dc215-c531-4ba7-ba7d-67bb81811d10","type":"GlyphRenderer"},{"id":"f85a1a03-acc2-40c6-b5b1-136c5021aaf4","type":"GlyphRenderer"},{"id":"48f1be8e-287d-473c-8c7c-7c4a540126a0","type":"GlyphRenderer"},{"id":"48ccdee8-9fb1-4259-95db-65132fd3f3d0","type":"GlyphRenderer"},{"id":"e47f87a7-6963-4b9c-80bf-0508714bc81e","type":"GlyphRenderer"},{"id":"080656ec-840f-47d6-9786-6dcfd424d79a","type":"GlyphRenderer"},{"id":"25ba8226-5add-457a-a17b-c9fc1004604f","type":"GlyphRenderer"},{"id":"ef040920-d49d-400c-abfe-5ed2cb345aad","type":"GlyphRenderer"},{"id":"254bcdbf-e073-40cb-960f-18e4b73ded19","type":"GlyphRenderer"},{"id":"21b495e3-4126-492e-a874-4d7f2caea9d6","type":"GlyphRenderer"},{"id":"8efbbb0d-6c52-4523-8f19-c8a3f69a42f2","type":"GlyphRenderer"},{"id":"bf37b4ba-ff17-4cf2-8ccb-92a85630dd9a","type":"GlyphRenderer"},{"id":"cdab02bc-2c76-40d0-86b1-421d5ce443d7","type":"GlyphRenderer"},{"id":"f5b40776-cfed-423d-94d9-4934123949e3","type":"GlyphRenderer"},{"id":"689afc5c-241a-4b07-8457-0c4ce5eb0de7","type":"GlyphRenderer"},{"id":"b2b46b96-5b75-4e0a-a071-f083b4c0fe9b","type":"GlyphRenderer"},{"id":"87565157-02e8-445d-a089-e480435bcc8f","type":"GlyphRenderer"},{"id":"bebf576c-f622-4998-af36-40f02c9827bc","type":"GlyphRenderer"},{"id":"930214ba-7d1c-4c03-aa3b-ef95afc33844","type":"GlyphRenderer"},{"id":"1b820274-9a35-40b9-b9c7-42d79124f7b3","type":"GlyphRenderer"},{"id":"4b31a017-e2c6-4b70-bc36-c734a85c2518","type":"GlyphRenderer"},{"id":"16a7a8d2-bc9c-4a04-8429-37a7e5fd6737","type":"GlyphRenderer"},{"id":"8a366482-bfbc-4597-b9f4-ca03c6c5a40c","type":"GlyphRenderer"},{"id":"2f33f444-7c39-4f96-af98-47187f648738","type":"GlyphRenderer"},{"id":"badf03e4-ca5e-4418-8968-2ac8b64f0140","type":"GlyphRenderer"},{"id":"3fc7fcf0-38da-450e-83ed-0455da02e6ab","type":"GlyphRenderer"},{"id":"9508c8db-dfa8-4ed9-94dd-679e728fae2d","type":"Legend"},{"id":"9666069c-dca7-4002-a78b-aac5d38b9c76","type":"CategoricalAxis"},{"id":"874eccc1-f72a-4828-aa4e-4bf54519f7b7","type":"LinearAxis"},{"id":"7217d6dd-e00e-492e-93e9-0a15cc9cd77d","type":"Grid"}],"title":{"id":"72cd2227-f289-4e5b-a0ed-c6d662543c93","type":"Title"},"tool_events":{"id":"dd383674-e9a4-4ce7-baf8-2fca55658a49","type":"ToolEvents"},"toolbar":{"id":"fdf530f8-f4d3-4026-babc-abaf2645355b","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"759b0606-3ade-4f36-84b5-422a55c3af74","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"9283ed03-01c4-4925-abbc-61759c41f131","type":"Range1d"}},"id":"d476f79e-79a3-4b58-991b-73df5568e9aa","subtype":"Chart","type":"Plot"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"e9b3f8c4-9d3f-45bd-9c47-440319f7bcd2","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a207d9d3-5c37-4442-8c9b-8af5b9326d0b","type":"ColumnDataSource"},"glyph":{"id":"7f3c191e-8c6e-4499-95ea-dfa4ae18ff35","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f5b40776-cfed-423d-94d9-4934123949e3","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"90ba460b-a0ee-42ad-9aec-5a5f3b94e8d2","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"034258fc-95ac-45cb-ad7f-944fe8bcdbeb","type":"Rect"},{"attributes":{"data_source":{"id":"e9b3f8c4-9d3f-45bd-9c47-440319f7bcd2","type":"ColumnDataSource"},"glyph":{"id":"1c792634-28b2-4d16-9df8-1288ec5c258e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"84448483-d04f-4f91-a2b3-717a18982957","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"cf667cd7-306b-4bd3-920f-25cc1f35ec34","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"485b6ae9-04b7-40f3-899e-3685475e54f7","type":"ColumnDataSource"},"glyph":{"id":"24e5c8b2-1d6d-44d2-ba33-e4224ec01527","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d0ec1df7-4cf3-4c12-b8f2-ceeaa31362ca","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1c792634-28b2-4d16-9df8-1288ec5c258e","type":"Rect"},{"attributes":{"data_source":{"id":"cf667cd7-306b-4bd3-920f-25cc1f35ec34","type":"ColumnDataSource"},"glyph":{"id":"c63c91d9-81c0-46f2-abf5-3cc71172fb98","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"689afc5c-241a-4b07-8457-0c4ce5eb0de7","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["outra"],"y":[83.5]}},"id":"a5a17f4f-e478-4135-b000-40c4e4939776","type":"ColumnDataSource"},{"attributes":{"label":{"value":"reajuste"},"renderers":[{"id":"87565157-02e8-445d-a089-e480435bcc8f","type":"GlyphRenderer"}]},"id":"cf28ee8f-bdeb-49e1-b356-2c8911a39f42","type":"LegendItem"},{"attributes":{"data_source":{"id":"4e198755-0cc9-46f3-962d-e79d091fe516","type":"ColumnDataSource"},"glyph":{"id":"039a234d-1528-4951-804b-833f08b93f0f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"930214ba-7d1c-4c03-aa3b-ef95afc33844","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza","sub_categoria":"cr\u00e9dito"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[15.0],"label":[{"categoria_geral":"autoriza","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["autoriza"],"y":[85.5]}},"id":"c9013e10-070c-431f-aa25-a21d368afac1","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"d2098ba4-32d0-4732-a382-fc8f1437c820","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c63c91d9-81c0-46f2-abf5-3cc71172fb98","type":"Rect"},{"attributes":{"data_source":{"id":"7eb2e12a-e58d-4c12-be38-1deffff74b87","type":"ColumnDataSource"},"glyph":{"id":"ae885eb5-c528-4bbf-bcf0-154c7291c731","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e4ce37b8-fffc-4eb6-8cdf-18255dbc584c","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e45dd281-7777-41ed-8eb8-a0d950ad10e3","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e6b06671-8e6d-43b5-86ca-35ed1d72248c","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ae885eb5-c528-4bbf-bcf0-154c7291c731","type":"Rect"},{"attributes":{"data_source":{"id":"e16a8c59-b938-45b1-af99-63f77dfcc79d","type":"ColumnDataSource"},"glyph":{"id":"e6b06671-8e6d-43b5-86ca-35ed1d72248c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b2b46b96-5b75-4e0a-a071-f083b4c0fe9b","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f60cbd65-a333-480b-a494-373ae6d6d7ae","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[55.0],"label":[{"categoria_geral":"institui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui"],"y":[27.5]}},"id":"a5760d45-fe9c-4af4-9bd0-0e58bd2c9207","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"90ba460b-a0ee-42ad-9aec-5a5f3b94e8d2","type":"ColumnDataSource"},"glyph":{"id":"f60cbd65-a333-480b-a494-373ae6d6d7ae","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a05dc215-c531-4ba7-ba7d-67bb81811d10","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9c331dd0-10d0-4b95-9d4b-cf183ed5235c","type":"Rect"},{"attributes":{"label":{"value":"reestrutura\u00e7\u00e3o"},"renderers":[{"id":"bebf576c-f622-4998-af36-40f02c9827bc","type":"GlyphRenderer"}]},"id":"d767466f-bdda-4208-9d95-8ea820bb704c","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"270b5b8e-aba4-491a-b8c4-7e980d84aa82","type":"Rect"},{"attributes":{"data_source":{"id":"f416fab3-876a-439d-ae32-b77b375f4e9c","type":"ColumnDataSource"},"glyph":{"id":"e45dd281-7777-41ed-8eb8-a0d950ad10e3","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"48ccdee8-9fb1-4259-95db-65132fd3f3d0","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"d2098ba4-32d0-4732-a382-fc8f1437c820","type":"ColumnDataSource"},"glyph":{"id":"9c331dd0-10d0-4b95-9d4b-cf183ed5235c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f85a1a03-acc2-40c6-b5b1-136c5021aaf4","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"6565ed58-7cfe-43d6-ac3e-577ee2e6e192","type":"ColumnDataSource"},"glyph":{"id":"034258fc-95ac-45cb-ad7f-944fe8bcdbeb","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bebf576c-f622-4998-af36-40f02c9827bc","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"03b9b7a3-f11c-4c9f-baa4-33e7b9e7e60e","type":"Rect"},{"attributes":{"data_source":{"id":"1cf3c6f3-74b0-4bde-86db-6b3cd9fcbb24","type":"ColumnDataSource"},"glyph":{"id":"0a4fe654-15ec-4549-90e7-1b508575d16c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2f33f444-7c39-4f96-af98-47187f648738","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui,"],"chart_index":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui,"],"y":[3.0]}},"id":"a18bf3a6-f11b-4527-ac1e-5f041f42a332","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"b7a90d7d-a3f8-4327-80b6-d819e03e61bc","type":"ColumnDataSource"},"glyph":{"id":"92d7e361-5890-476c-b3b9-9583ce3588c9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"19a5661a-bfdc-49e4-b19d-36cd7f15c2a9","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ea35626a-5d2b-4389-8d55-eb86cc4a5a40","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"039a234d-1528-4951-804b-833f08b93f0f","type":"Rect"},{"attributes":{"data_source":{"id":"a5760d45-fe9c-4af4-9bd0-0e58bd2c9207","type":"ColumnDataSource"},"glyph":{"id":"ea35626a-5d2b-4389-8d55-eb86cc4a5a40","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"48f1be8e-287d-473c-8c7c-7c4a540126a0","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"79433e3e-768a-44a9-ba12-2c4bf899df7d","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a19cf9d6-7c8d-4cc5-b01c-93de98100b5f","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f5b2a87f-d783-48f2-8810-e2cd6c9024f0","type":"Rect"},{"attributes":{"data_source":{"id":"2c474b75-99b0-4b5d-9d59-70e5d6d1e300","type":"ColumnDataSource"},"glyph":{"id":"79433e3e-768a-44a9-ba12-2c4bf899df7d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"1b820274-9a35-40b9-b9c7-42d79124f7b3","type":"GlyphRenderer"},{"attributes":{"label":{"value":"aumento"},"renderers":[{"id":"b2b46b96-5b75-4e0a-a071-f083b4c0fe9b","type":"GlyphRenderer"}]},"id":"24144962-b4db-4230-ab8f-9c7292232f28","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[83.0],"label":[{"categoria_geral":"outra","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[41.5]}},"id":"ac931838-17fe-4e23-a8f7-6b02310c7f7f","type":"ColumnDataSource"},{"attributes":{"label":{"value":"estrutura"},"renderers":[{"id":"f5b40776-cfed-423d-94d9-4934123949e3","type":"GlyphRenderer"}]},"id":"9dd8e3a1-0417-439b-a76c-287426bc3559","type":"LegendItem"},{"attributes":{"data_source":{"id":"83c86fa0-fe94-4327-9e78-b3026a8c84ca","type":"ColumnDataSource"},"glyph":{"id":"b9c5aac1-ab25-43b4-a31a-10bf07ebefa9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e47f87a7-6963-4b9c-80bf-0508714bc81e","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"plano"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["outra"],"y":[84.5]}},"id":"73fa476a-cdbc-4b85-9650-6a67efa4bf5c","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estabelece"],"y":[12.5]}},"id":"686c8004-6f34-4aaa-b345-e24574e56ad2","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0a4fe654-15ec-4549-90e7-1b508575d16c","type":"Rect"},{"attributes":{"data_source":{"id":"a18bf3a6-f11b-4527-ac1e-5f041f42a332","type":"ColumnDataSource"},"glyph":{"id":"270b5b8e-aba4-491a-b8c4-7e980d84aa82","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4b31a017-e2c6-4b70-bc36-c734a85c2518","type":"GlyphRenderer"}],"root_ids":["d476f79e-79a3-4b58-991b-73df5568e9aa"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"90fe5fde-596f-4721-a7a6-f57708b20c58","elementid":"d3c09797-a4ba-45f0-914a-1f319b1a7d14","modelid":"d476f79e-79a3-4b58-991b-73df5568e9aa"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("d3c09797-a4ba-45f0-914a-1f319b1a7d14")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'altera'
Aparentemente todas as leis com 'altera' na primeira palavra da ementa alteram uma lei, vamos investigar isso


```python
def get_counter_with_category(category, keyword, index_word):
    counter = Counter()
    for index,row in leis.iterrows():
        if (row[category] == keyword):
            ementa = row['ementa'].split(" ")
            if(len(ementa) > index_word):
                word = ementa[index_word]
                counter[word] += 1
    return counter

sub_categoria_altera = get_counter_with_category('categoria_geral','altera',2)

sub_categoria_altera.most_common()
```




    [('lei', 31),
     ('anexo', 15),
     ('da', 11),
     ('art.', 10),
     ('artigo', 6),
     ('redação', 5),
     ('§', 4),
     ('composição', 3),
     ('arts.', 3),
     ('estrutura', 3),
     ('§§', 3),
     ('de', 2),
     ('dos', 2),
     ('inciso', 2),
     ('acrescenta', 2),
     ('incisos', 2),
     ('art.2', 1),
     ('introduz', 1),
     ('valor', 1),
     ('renumera', 1),
     ('deveres', 1),
     ('artigos', 1),
     ('anexos', 1),
     ('disposições', 1),
     ('alíquota', 1),
     ('att', 1),
     ('parágrafo', 1),
     ('suprime', 1)]



### Comentários
Confirmado, as leis com 'altera' sempre se referem a leis

Vamos mudar a label de 'altera' para 'altera lei', assim deixamos mais claro; outra obervação é que nesse caso não precisamos de sub-categoria, então substituimos esse campo com '-'


```python
def change_category_label(to_change, old_category, new_category):
    if(old_category == to_change):
        return new_category
    else:
        return old_category

def change_sub_category_label(to_change, category,old_sub_category, new_sub_category):
    if (category == to_change):
        return new_sub_category
    else:
        return old_sub_category

leis['categoria_geral'] = leis.apply(lambda row: change_category_label('altera', row['categoria_geral'],'altera lei'), axis=1)
leis['sub_categoria'] = leis.apply(lambda row: change_sub_category_label('altera lei',row['categoria_geral'],row['sub_categoria'],'-'), axis=1)
```


```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por sub-categoria", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="311292f3-0b03-431d-a77b-46a80bcac038"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("311292f3-0b03-431d-a77b-46a80bcac038").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("311292f3-0b03-431d-a77b-46a80bcac038");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid '311292f3-0b03-431d-a77b-46a80bcac038' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"b3502bfa-1402-4e0b-9d15-196c2237191e":{"roots":{"references":[{"attributes":{"data_source":{"id":"63b9222b-1386-417b-b956-4596a5ae5c47","type":"ColumnDataSource"},"glyph":{"id":"da2c03f5-2504-4f17-abbf-d0c1a3173eaf","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c7fcd1d8-31d2-4be8-a335-5e2b76a3c1d9","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"4d0ea096-28b9-4ec9-9d60-dca8e1dd0da0","type":"ColumnDataSource"},"glyph":{"id":"320b01be-8667-48b5-8b62-f1bba8e5ed1b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4decc147-05c3-422b-abd7-5159aefd9e15","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"7d8466af-711d-457a-a8d0-eba6b716ca57","type":"ColumnDataSource"},"glyph":{"id":"8182e8da-8318-4bae-bd05-a8f312b6851d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"1a9c263c-47e4-4ea1-868f-6aced4481def","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8182e8da-8318-4bae-bd05-a8f312b6851d","type":"Rect"},{"attributes":{"label":{"value":"diretrizes"},"renderers":[{"id":"3cfff907-0174-4385-b4dc-1bf6a3e9b514","type":"GlyphRenderer"}]},"id":"d3811afe-2a73-443d-94d6-b9f66229722a","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3fde501e-e261-46d4-b40a-c5cd59ad3fbe","type":"Rect"},{"attributes":{"data_source":{"id":"ec5a1b2d-3b85-4eee-91e4-dc56313d7754","type":"ColumnDataSource"},"glyph":{"id":"3fde501e-e261-46d4-b40a-c5cd59ad3fbe","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"71079dd1-59e3-4acd-ba19-cf97ca64c689","type":"GlyphRenderer"},{"attributes":{"plot":null,"text":"Quantidade de Leis por sub-categoria"},"id":"cd89f31e-6ee0-4537-8c3b-2c5f970b1d0f","type":"Title"},{"attributes":{"callback":null,"end":174.3},"id":"3531a511-d889-4163-8c61-8d748f0a7103","type":"Range1d"},{"attributes":{"data_source":{"id":"5bffcb76-e5fd-482a-a0fb-12dcbd6309fe","type":"ColumnDataSource"},"glyph":{"id":"0c25470a-e358-4b0f-8136-fc53903dfacd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ad8dcf45-4fe3-4eae-af6a-f50c9cdd4f8a","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0c25470a-e358-4b0f-8136-fc53903dfacd","type":"Rect"},{"attributes":{"callback":null,"factors":["?","altera lei","atribui","autoriza","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","institui,","outra","reajusta","revoga"]},"id":"b4ab9918-bdb5-4a3a-b4e4-13ec2d397be5","type":"FactorRange"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6906376c-cc97-492e-9834-e9f0c10b4c64","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"f0b14e72-4e56-4b2e-b24b-42b3b18140a4","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"66dcffdb-2671-4c00-a0a2-ba49da9b9bbc","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e403a9a5-0acb-411b-ab30-08b5a2596f0e","type":"Rect"},{"attributes":{"data_source":{"id":"20ac143a-c627-46e7-899e-7e53822a4ad7","type":"ColumnDataSource"},"glyph":{"id":"6906376c-cc97-492e-9834-e9f0c10b4c64","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"14063c06-c400-4abf-bd22-82d07d3cc3c4","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8cf64fdb-738d-4a3e-ba08-087d8602e0e8","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a3e15046-c466-458a-80b2-ad06e1bf43cf","type":"Rect"},{"attributes":{"data_source":{"id":"e82762da-b635-4bb5-9cb5-f2981e03a14d","type":"ColumnDataSource"},"glyph":{"id":"e403a9a5-0acb-411b-ab30-08b5a2596f0e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"0f9ef696-0fbf-4e03-9410-8ce2461ff079","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"61a4b097-d623-4797-b13c-60ec5d745f75","type":"ColumnDataSource"},"glyph":{"id":"c3c0c0a8-78d7-4595-82d8-05fadd1da16c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"25ae6572-43d5-42ec-b180-4f4588ff151d","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"756d5991-72d5-4168-a460-f5370645b6ff","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"ec5a1b2d-3b85-4eee-91e4-dc56313d7754","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c099d371-1b2d-4a7d-96bf-0b5fe74216b5","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"ad7466ba-4e25-4d64-8eb1-546d283c93d1","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"1892c1d9-b06b-4acb-97a7-5b7ab705113a","type":"ColumnDataSource"},"glyph":{"id":"c099d371-1b2d-4a7d-96bf-0b5fe74216b5","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"502227d8-da6f-4042-897c-932cf19ed6ce","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"ce2a6608-236f-49aa-b1f1-fe70d9ab62c2","type":"ColumnDataSource"},"glyph":{"id":"8cf64fdb-738d-4a3e-ba08-087d8602e0e8","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b420a11e-a33b-4737-85da-46ba2889342c","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c3c0c0a8-78d7-4595-82d8-05fadd1da16c","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["denomina\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[163.5]}},"id":"3b12b5fe-3093-415b-832d-cbc32f745a0a","type":"ColumnDataSource"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"eaa9c031-e053-4ed8-b3b9-9f4bf2affecf","type":"PanTool"},{"id":"a069a71d-09f1-4d6a-bffa-165f80ba304b","type":"WheelZoomTool"},{"id":"3356564f-d058-4e57-9fec-47c4fa716860","type":"BoxZoomTool"},{"id":"6cc47f23-2bae-4201-835b-e3a624240a7e","type":"SaveTool"},{"id":"32c3ead8-6ecf-4746-8e3e-0246bb9e42f2","type":"ResetTool"},{"id":"cd3337c1-aa39-4818-9c92-4795f03bc631","type":"HelpTool"}]},"id":"0855baf2-526f-42d2-ad21-fef6adebda62","type":"Toolbar"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"52896bdc-040c-403d-af01-69757610c916","type":"Rect"},{"attributes":{"data_source":{"id":"f0b14e72-4e56-4b2e-b24b-42b3b18140a4","type":"ColumnDataSource"},"glyph":{"id":"52896bdc-040c-403d-af01-69757610c916","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d8d9639b-f6c4-41b0-8752-576c631ef669","type":"GlyphRenderer"},{"attributes":{"below":[{"id":"d7e65b92-af09-4f3e-9dbc-a4a8752a5240","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"1a379c82-3277-4ca9-8faf-77e7c1d1e9a1","type":"LinearAxis"}],"renderers":[{"id":"ef7bcf0e-dc61-4558-9c65-117c919907be","type":"BoxAnnotation"},{"id":"394c710a-005f-4f53-a10a-0affedcc3338","type":"GlyphRenderer"},{"id":"b420a11e-a33b-4737-85da-46ba2889342c","type":"GlyphRenderer"},{"id":"71079dd1-59e3-4acd-ba19-cf97ca64c689","type":"GlyphRenderer"},{"id":"4a19047d-58bf-48d8-a934-94f5668cec8b","type":"GlyphRenderer"},{"id":"f2d12fb8-fa4f-4dc0-ad1f-29ee4c6b808d","type":"GlyphRenderer"},{"id":"4af30efc-57fc-49e0-bee4-6f264f0e8dec","type":"GlyphRenderer"},{"id":"3cfff907-0174-4385-b4dc-1bf6a3e9b514","type":"GlyphRenderer"},{"id":"6aa82fc4-9318-4fb8-8104-d687df43fcd3","type":"GlyphRenderer"},{"id":"25ae6572-43d5-42ec-b180-4f4588ff151d","type":"GlyphRenderer"},{"id":"4decc147-05c3-422b-abd7-5159aefd9e15","type":"GlyphRenderer"},{"id":"01264398-63c3-4f4d-b1cb-f879f4e99e05","type":"GlyphRenderer"},{"id":"c8903a2c-ceb2-4479-9c97-dc47881f09a2","type":"GlyphRenderer"},{"id":"4bb5d83d-0d19-4d2d-a859-a6dc4209e231","type":"GlyphRenderer"},{"id":"7c6e7b1a-eb82-47be-94c2-610eb7d40486","type":"GlyphRenderer"},{"id":"92edccc6-5942-4298-9626-f8cd9149d5f0","type":"GlyphRenderer"},{"id":"3668df3c-0eb2-42bf-b7bf-c33881ebc8cc","type":"GlyphRenderer"},{"id":"9033e95b-8cba-4548-a487-48646b347c36","type":"GlyphRenderer"},{"id":"94812902-47d0-4cc3-accb-81962ac6f0a8","type":"GlyphRenderer"},{"id":"338985fe-fa38-4ac8-b942-930dcf121f6d","type":"GlyphRenderer"},{"id":"bc5a1978-2b4f-4359-a7c0-6af21c625752","type":"GlyphRenderer"},{"id":"c7fcd1d8-31d2-4be8-a335-5e2b76a3c1d9","type":"GlyphRenderer"},{"id":"4389d0bd-7817-4540-9379-a5985fda35cd","type":"GlyphRenderer"},{"id":"1a9c263c-47e4-4ea1-868f-6aced4481def","type":"GlyphRenderer"},{"id":"ad8dcf45-4fe3-4eae-af6a-f50c9cdd4f8a","type":"GlyphRenderer"},{"id":"14063c06-c400-4abf-bd22-82d07d3cc3c4","type":"GlyphRenderer"},{"id":"502227d8-da6f-4042-897c-932cf19ed6ce","type":"GlyphRenderer"},{"id":"d8d9639b-f6c4-41b0-8752-576c631ef669","type":"GlyphRenderer"},{"id":"76da3348-2871-467f-98f3-a9777c0847a9","type":"GlyphRenderer"},{"id":"0f9ef696-0fbf-4e03-9410-8ce2461ff079","type":"GlyphRenderer"},{"id":"29cab464-083f-4969-b832-bba4ff2c0a38","type":"GlyphRenderer"},{"id":"e23ddbed-d486-4d49-a070-e3c723445bfd","type":"GlyphRenderer"},{"id":"e75a1818-4e58-4187-afbf-df0cdbcba59c","type":"GlyphRenderer"},{"id":"0582869d-d5f1-4136-a935-5d5f7d9c4268","type":"GlyphRenderer"},{"id":"2ce87850-3f0e-4408-86e9-757941eac2a4","type":"GlyphRenderer"},{"id":"f1954b5b-f4d2-49ef-a704-c7f38c9bb51f","type":"GlyphRenderer"},{"id":"4af22370-00a2-4052-83ca-bf8950a90038","type":"GlyphRenderer"},{"id":"26043dcc-10a2-46df-8683-34dd29c340f4","type":"GlyphRenderer"},{"id":"bdccdcc0-ac42-4399-b87b-f084b238a4e6","type":"GlyphRenderer"},{"id":"7063963c-b86b-442b-9a3b-ac693972322d","type":"Legend"},{"id":"d7e65b92-af09-4f3e-9dbc-a4a8752a5240","type":"CategoricalAxis"},{"id":"1a379c82-3277-4ca9-8faf-77e7c1d1e9a1","type":"LinearAxis"},{"id":"1dc63e8e-dfed-43e8-8248-99e4d2cb9041","type":"Grid"}],"title":{"id":"cd89f31e-6ee0-4537-8c3b-2c5f970b1d0f","type":"Title"},"tool_events":{"id":"8a79302d-2d8d-4c19-8458-7cb9681a0655","type":"ToolEvents"},"toolbar":{"id":"0855baf2-526f-42d2-ad21-fef6adebda62","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"b4ab9918-bdb5-4a3a-b4e4-13ec2d397be5","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"3531a511-d889-4163-8c61-8d748f0a7103","type":"Range1d"}},"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9dbc2dc7-8ebc-4520-a46e-75704e9bd74e","type":"Rect"},{"attributes":{},"id":"8a79302d-2d8d-4c19-8458-7cb9681a0655","type":"ToolEvents"},{"attributes":{"data_source":{"id":"fea4dbfd-3afb-4ebe-8cc1-059f3fa83ae4","type":"ColumnDataSource"},"glyph":{"id":"a3e15046-c466-458a-80b2-ad06e1bf43cf","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"76da3348-2871-467f-98f3-a9777c0847a9","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reajuste"],"width":[0.8],"x":["disp\u00f5e"],"y":[136.0]}},"id":"e82762da-b635-4bb5-9cb5-f2981e03a14d","type":"ColumnDataSource"},{"attributes":{"label":{"value":"vencimentos"},"renderers":[{"id":"e75a1818-4e58-4187-afbf-df0cdbcba59c","type":"GlyphRenderer"}]},"id":"8d2c1e8b-811c-4252-872d-52d13dfc0469","type":"LegendItem"},{"attributes":{"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"32c3ead8-6ecf-4746-8e3e-0246bb9e42f2","type":"ResetTool"},{"attributes":{"label":{"value":"altera\u00e7\u00e3o"},"renderers":[{"id":"f1954b5b-f4d2-49ef-a704-c7f38c9bb51f","type":"GlyphRenderer"}]},"id":"d80d4352-6075-49f6-99d8-df7f70b30ee0","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[27.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[89.5]}},"id":"b2d2ec2a-7eba-4023-8f12-355dfeade510","type":"ColumnDataSource"},{"attributes":{"label":{"value":"cr\u00e9dito"},"renderers":[{"id":"6aa82fc4-9318-4fb8-8104-d687df43fcd3","type":"GlyphRenderer"}]},"id":"fa6e1247-d761-466c-94a7-1628bea9d9b1","type":"LegendItem"},{"attributes":{"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"cd3337c1-aa39-4818-9c92-4795f03bc631","type":"HelpTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["contrata\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[64.5]}},"id":"4df7a956-1a6a-47db-bcda-dcdbfa5fe483","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f8ce36db-911a-4310-b58a-0e44c88b10e3","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["disp\u00f5e"],"y":[151.0]}},"id":"7e1fd528-c11a-4c5f-aed5-3d8bc44efa2e","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["outra"],"y":[83.5]}},"id":"994264e5-3834-4976-bf4e-97f615dfab56","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["disp\u00f5e"],"y":[122.5]}},"id":"20ac143a-c627-46e7-899e-7e53822a4ad7","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["altera\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[157.5]}},"id":"1667ebfb-b4a2-4890-bb2c-a9e87437e0dd","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"35f902e4-bd62-4544-81ec-077a43ca218d","type":"ColumnDataSource"},"glyph":{"id":"9cb33752-643e-4ea2-a7d3-43c13012f143","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bdccdcc0-ac42-4399-b87b-f084b238a4e6","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["revis\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[116.0]}},"id":"5bffcb76-e5fd-482a-a0fb-12dcbd6309fe","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["institui\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[144.5]}},"id":"90e144c1-dd94-466d-84a3-c4180fbb9abc","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9cb33752-643e-4ea2-a7d3-43c13012f143","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["aumento"],"width":[0.8],"x":["disp\u00f5e"],"y":[132.5]}},"id":"fea4dbfd-3afb-4ebe-8cc1-059f3fa83ae4","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reestrutura\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[139.5]}},"id":"a86df966-3857-4288-bc9a-aa7b961b249c","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["estrutura"],"width":[0.8],"x":["disp\u00f5e"],"y":[128.0]}},"id":"1892c1d9-b06b-4acb-97a7-5b7ab705113a","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["abertura"],"width":[0.8],"x":["disp\u00f5e"],"y":[107.5]}},"id":"7d8466af-711d-457a-a8d0-eba6b716ca57","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"plano"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["outra"],"y":[84.5]}},"id":"35f902e4-bd62-4544-81ec-077a43ca218d","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["disp\u00f5e"],"y":[71.0]}},"id":"63b9222b-1386-417b-b956-4596a5ae5c47","type":"ColumnDataSource"},{"attributes":{},"id":"150fb784-ff6e-4fae-bf51-b7147f415d1d","type":"CategoricalTicker"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disp\u00f5e"],"y":[34.0]}},"id":"91fe2c45-3d00-497d-9aa1-ae8bd04f5b5c","type":"ColumnDataSource"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"394c710a-005f-4f53-a10a-0affedcc3338","type":"GlyphRenderer"}]},"id":"a40648a9-7151-42da-8b7c-d4e4f4e0072d","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["estabelece"],"y":[27.0]}},"id":"5a2f5d6c-5227-451c-99ae-c6c59b721e85","type":"ColumnDataSource"},{"attributes":{"label":{"value":"reestrutura\u00e7\u00e3o"},"renderers":[{"id":"29cab464-083f-4969-b832-bba4ff2c0a38","type":"GlyphRenderer"}]},"id":"1b9aa550-5ae2-4a5e-972a-86c774e5604a","type":"LegendItem"},{"attributes":{"label":{"value":"reajuste"},"renderers":[{"id":"0f9ef696-0fbf-4e03-9410-8ce2461ff079","type":"GlyphRenderer"}]},"id":"a515b415-c651-4c01-b27d-18c0d6cbf873","type":"LegendItem"},{"attributes":{"label":{"value":"institui\u00e7\u00e3o"},"renderers":[{"id":"e23ddbed-d486-4d49-a070-e3c723445bfd","type":"GlyphRenderer"}]},"id":"b324bd0e-156a-4a51-98f1-1c42a56b892a","type":"LegendItem"},{"attributes":{"label":{"value":"denomina\u00e7\u00e3o"},"renderers":[{"id":"26043dcc-10a2-46df-8683-34dd29c340f4","type":"GlyphRenderer"}]},"id":"e8277428-8698-48d6-a990-c9e0be9e0dc6","type":"LegendItem"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"a49c772a-d4b4-4012-bd44-ec27827880d2","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"},"ticker":{"id":"150fb784-ff6e-4fae-bf51-b7147f415d1d","type":"CategoricalTicker"}},"id":"d7e65b92-af09-4f3e-9dbc-a4a8752a5240","type":"CategoricalAxis"},{"attributes":{"data_source":{"id":"7ff8c991-3cc3-40d5-8ec3-6d30b065013a","type":"ColumnDataSource"},"glyph":{"id":"cfe50fcf-0828-4b1e-a378-298a32cb738d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4a19047d-58bf-48d8-a934-94f5668cec8b","type":"GlyphRenderer"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"ba217c0c-0b7d-4716-956e-35f878e3a6f1","type":"BasicTickFormatter"},"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"},"ticker":{"id":"2ebb0146-f59a-46ab-b1bb-b96b54321aaf","type":"BasicTicker"}},"id":"1a379c82-3277-4ca9-8faf-77e7c1d1e9a1","type":"LinearAxis"},{"attributes":{},"id":"2ebb0146-f59a-46ab-b1bb-b96b54321aaf","type":"BasicTicker"},{"attributes":{"dimension":1,"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"},"ticker":{"id":"2ebb0146-f59a-46ab-b1bb-b96b54321aaf","type":"BasicTicker"}},"id":"1dc63e8e-dfed-43e8-8248-99e4d2cb9041","type":"Grid"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[77.0],"label":[{"categoria_geral":"cria","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["cria"],"y":[38.5]}},"id":"7ff8c991-3cc3-40d5-8ec3-6d30b065013a","type":"ColumnDataSource"},{"attributes":{},"id":"a49c772a-d4b4-4012-bd44-ec27827880d2","type":"CategoricalTickFormatter"},{"attributes":{},"id":"ba217c0c-0b7d-4716-956e-35f878e3a6f1","type":"BasicTickFormatter"},{"attributes":{"data_source":{"id":"8219af27-a825-4ac8-8e99-0ff2c508389e","type":"ColumnDataSource"},"glyph":{"id":"48a61d78-7d7e-48dd-b173-d5483a355c22","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"394c710a-005f-4f53-a10a-0affedcc3338","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"8219af27-a825-4ac8-8e99-0ff2c508389e","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza","sub_categoria":"cr\u00e9dito"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[15.0],"label":[{"categoria_geral":"autoriza","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["autoriza"],"y":[85.5]}},"id":"8683001c-103e-4b99-a105-10fc1f2a3c48","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"6cc47f23-2bae-4201-835b-e3a624240a7e","type":"SaveTool"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"ef7bcf0e-dc61-4558-9c65-117c919907be","type":"BoxAnnotation"},{"attributes":{"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"a069a71d-09f1-4d6a-bffa-165f80ba304b","type":"WheelZoomTool"},{"attributes":{"overlay":{"id":"ef7bcf0e-dc61-4558-9c65-117c919907be","type":"BoxAnnotation"},"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"3356564f-d058-4e57-9fec-47c4fa716860","type":"BoxZoomTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"cfe50fcf-0828-4b1e-a378-298a32cb738d","type":"Rect"},{"attributes":{"plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"eaa9c031-e053-4ed8-b3b9-9f4bf2affecf","type":"PanTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[55.0],"label":[{"categoria_geral":"institui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui"],"y":[27.5]}},"id":"6caf7869-26f2-48eb-8d7d-a8a79569696f","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"999615a7-7e93-4582-b282-ef1712a1ae83","type":"ColumnDataSource"},"glyph":{"id":"f0a6cd6a-a147-4d47-8a0f-874a071c0b75","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4bb5d83d-0d19-4d2d-a859-a6dc4209e231","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"8583c783-c4e1-4a22-b002-4c67fbd19418","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a86df966-3857-4288-bc9a-aa7b961b249c","type":"ColumnDataSource"},"glyph":{"id":"9dbc2dc7-8ebc-4520-a46e-75704e9bd74e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"29cab464-083f-4969-b832-bba4ff2c0a38","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"8583c783-c4e1-4a22-b002-4c67fbd19418","type":"ColumnDataSource"},"glyph":{"id":"d228db3a-9b1e-4a8c-ac5b-970823359dc3","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"7c6e7b1a-eb82-47be-94c2-610eb7d40486","type":"GlyphRenderer"},{"attributes":{"label":{"value":"aumento"},"renderers":[{"id":"76da3348-2871-467f-98f3-a9777c0847a9","type":"GlyphRenderer"}]},"id":"f9b4ab59-4f84-4118-8e5c-28ed1e1045a9","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d228db3a-9b1e-4a8c-ac5b-970823359dc3","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui,"],"chart_index":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui,"],"y":[3.0]}},"id":"e845b780-e317-4712-816e-f9b98fce75e8","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estabelece"],"y":[12.5]}},"id":"55419a3b-e004-4a40-98a1-d4f9b6629032","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"90e144c1-dd94-466d-84a3-c4180fbb9abc","type":"ColumnDataSource"},"glyph":{"id":"6018bb74-ed12-4f30-8558-194021e64eca","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e23ddbed-d486-4d49-a070-e3c723445bfd","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"bf820bf0-1deb-4803-9911-f99613bbbb96","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6018bb74-ed12-4f30-8558-194021e64eca","type":"Rect"},{"attributes":{"data_source":{"id":"6caf7869-26f2-48eb-8d7d-a8a79569696f","type":"ColumnDataSource"},"glyph":{"id":"bf820bf0-1deb-4803-9911-f99613bbbb96","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"92edccc6-5942-4298-9626-f8cd9149d5f0","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"81a50825-8577-4b73-8e42-d15ad55601ed","type":"ColumnDataSource"},"glyph":{"id":"66dcffdb-2671-4c00-a0a2-ba49da9b9bbc","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3668df3c-0eb2-42bf-b7bf-c33881ebc8cc","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"61a4b097-d623-4797-b13c-60ec5d745f75","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e3463de0-1722-4046-a584-b181969ceb8d","type":"Rect"},{"attributes":{"label":{"value":"-"},"renderers":[{"id":"3668df3c-0eb2-42bf-b7bf-c33881ebc8cc","type":"GlyphRenderer"}]},"id":"524394f2-5691-4602-9ac7-6dbe882e6cc9","type":"LegendItem"},{"attributes":{"data_source":{"id":"ceb86419-3b45-48a0-95e6-e2f5088d3f03","type":"ColumnDataSource"},"glyph":{"id":"b5dde054-5c15-42d4-9a4b-665a5e7d1e5a","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"338985fe-fa38-4ac8-b942-930dcf121f6d","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"7e1fd528-c11a-4c5f-aed5-3d8bc44efa2e","type":"ColumnDataSource"},"glyph":{"id":"e3463de0-1722-4046-a584-b181969ceb8d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e75a1818-4e58-4187-afbf-df0cdbcba59c","type":"GlyphRenderer"},{"attributes":{"items":[{"id":"a40648a9-7151-42da-8b7c-d4e4f4e0072d","type":"LegendItem"},{"id":"d3811afe-2a73-443d-94d6-b9f66229722a","type":"LegendItem"},{"id":"fa6e1247-d761-466c-94a7-1628bea9d9b1","type":"LegendItem"},{"id":"524394f2-5691-4602-9ac7-6dbe882e6cc9","type":"LegendItem"},{"id":"a9ddb1d1-e8ca-4fa2-b671-5050b2f02881","type":"LegendItem"},{"id":"72906f52-ff30-430a-8190-4e0bd9b2a79c","type":"LegendItem"},{"id":"15fe39ae-265e-4131-ad0e-9b1f0c7d0b05","type":"LegendItem"},{"id":"106c1b41-07d9-4cc7-831f-9cc03f5a045d","type":"LegendItem"},{"id":"76e82f6e-3ece-4680-8238-c1e274cf77f7","type":"LegendItem"},{"id":"6f4a11f6-a937-4a08-bad2-554495bc03f4","type":"LegendItem"},{"id":"f9b4ab59-4f84-4118-8e5c-28ed1e1045a9","type":"LegendItem"},{"id":"a515b415-c651-4c01-b27d-18c0d6cbf873","type":"LegendItem"},{"id":"1b9aa550-5ae2-4a5e-972a-86c774e5604a","type":"LegendItem"},{"id":"b324bd0e-156a-4a51-98f1-1c42a56b892a","type":"LegendItem"},{"id":"8d2c1e8b-811c-4252-872d-52d13dfc0469","type":"LegendItem"},{"id":"d80d4352-6075-49f6-99d8-df7f70b30ee0","type":"LegendItem"},{"id":"e8277428-8698-48d6-a990-c9e0be9e0dc6","type":"LegendItem"}],"location":"top_left","plot":{"id":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c","subtype":"Chart","type":"Plot"}},"id":"7063963c-b86b-442b-9a3b-ac693972322d","type":"Legend"},{"attributes":{"data_source":{"id":"5a2f5d6c-5227-451c-99ae-c6c59b721e85","type":"ColumnDataSource"},"glyph":{"id":"52c89f91-fedf-425d-9a75-edcfc6904a9a","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4af22370-00a2-4052-83ca-bf8950a90038","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"4df7a956-1a6a-47db-bcda-dcdbfa5fe483","type":"ColumnDataSource"},"glyph":{"id":"8d6ca9d2-3be9-42a0-9858-b96c2b1dfa61","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"9033e95b-8cba-4548-a487-48646b347c36","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8d6ca9d2-3be9-42a0-9858-b96c2b1dfa61","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"2d23e351-08dd-40c0-aaeb-890bb12aa97e","type":"Rect"},{"attributes":{"label":{"value":"estrutura"},"renderers":[{"id":"502227d8-da6f-4042-897c-932cf19ed6ce","type":"GlyphRenderer"}]},"id":"6f4a11f6-a937-4a08-bad2-554495bc03f4","type":"LegendItem"},{"attributes":{"data_source":{"id":"c4a749c0-684e-4136-836f-f83d4ed87b4a","type":"ColumnDataSource"},"glyph":{"id":"756d5991-72d5-4168-a460-f5370645b6ff","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f2d12fb8-fa4f-4dc0-ad1f-29ee4c6b808d","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"ceb86419-3b45-48a0-95e6-e2f5088d3f03","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"52c89f91-fedf-425d-9a75-edcfc6904a9a","type":"Rect"},{"attributes":{"data_source":{"id":"e845b780-e317-4712-816e-f9b98fce75e8","type":"ColumnDataSource"},"glyph":{"id":"2d23e351-08dd-40c0-aaeb-890bb12aa97e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"0582869d-d5f1-4136-a935-5d5f7d9c4268","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a3380d18-8d33-4870-af72-70d7b45963e9","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ba061f04-8991-4db4-ad9b-ffe0bf69024e","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"891f5cf4-89c4-480c-80f2-a2ecd643aed7","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"55419a3b-e004-4a40-98a1-d4f9b6629032","type":"ColumnDataSource"},"glyph":{"id":"a3380d18-8d33-4870-af72-70d7b45963e9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"94812902-47d0-4cc3-accb-81962ac6f0a8","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera lei"],"chart_index":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[116.0],"label":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["-"],"width":[0.8],"x":["altera lei"],"y":[58.0]}},"id":"81a50825-8577-4b73-8e42-d15ad55601ed","type":"ColumnDataSource"},{"attributes":{"label":{"value":"abertura"},"renderers":[{"id":"1a9c263c-47e4-4ea1-868f-6aced4481def","type":"GlyphRenderer"}]},"id":"106c1b41-07d9-4cc7-831f-9cc03f5a045d","type":"LegendItem"},{"attributes":{"label":{"value":"plano"},"renderers":[{"id":"c7fcd1d8-31d2-4be8-a335-5e2b76a3c1d9","type":"GlyphRenderer"}]},"id":"72906f52-ff30-430a-8190-4e0bd9b2a79c","type":"LegendItem"},{"attributes":{"data_source":{"id":"ef041ca8-cf3a-4ae9-8c6d-6dcfc44ff463","type":"ColumnDataSource"},"glyph":{"id":"d0ffbcbd-7d6c-4c79-8e8f-f0ade28f5126","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4af30efc-57fc-49e0-bee4-6f264f0e8dec","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b5dde054-5c15-42d4-9a4b-665a5e7d1e5a","type":"Rect"},{"attributes":{"data_source":{"id":"994264e5-3834-4976-bf4e-97f615dfab56","type":"ColumnDataSource"},"glyph":{"id":"ba061f04-8991-4db4-ad9b-ffe0bf69024e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2ce87850-3f0e-4408-86e9-757941eac2a4","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"c4a749c0-684e-4136-836f-f83d4ed87b4a","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"320b01be-8667-48b5-8b62-f1bba8e5ed1b","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"da2c03f5-2504-4f17-abbf-d0c1a3173eaf","type":"Rect"},{"attributes":{"label":{"value":"revis\u00e3o"},"renderers":[{"id":"ad8dcf45-4fe3-4eae-af6a-f50c9cdd4f8a","type":"GlyphRenderer"}]},"id":"76e82f6e-3ece-4680-8238-c1e274cf77f7","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[83.0],"label":[{"categoria_geral":"outra","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[41.5]}},"id":"ef041ca8-cf3a-4ae9-8c6d-6dcfc44ff463","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"b2d2ec2a-7eba-4023-8f12-355dfeade510","type":"ColumnDataSource"},"glyph":{"id":"cd6debb2-312f-41aa-9e0a-6ceec8d57a16","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4389d0bd-7817-4540-9379-a5985fda35cd","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"3b12b5fe-3093-415b-832d-cbc32f745a0a","type":"ColumnDataSource"},"glyph":{"id":"185f17fc-8ea7-46c6-802b-1c36d655085b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"26043dcc-10a2-46df-8683-34dd29c340f4","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d0ffbcbd-7d6c-4c79-8e8f-f0ade28f5126","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e1b4f310-0e13-4c82-af3e-905a608afdca","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"06de7fc5-9acc-4e4d-ac52-dc69b287f8a6","type":"Rect"},{"attributes":{"data_source":{"id":"1667ebfb-b4a2-4890-bb2c-a9e87437e0dd","type":"ColumnDataSource"},"glyph":{"id":"06de7fc5-9acc-4e4d-ac52-dc69b287f8a6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f1954b5b-f4d2-49ef-a704-c7f38c9bb51f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f312574f-a1ea-4448-834e-e8f4b78b1238","type":"Rect"},{"attributes":{"data_source":{"id":"891f5cf4-89c4-480c-80f2-a2ecd643aed7","type":"ColumnDataSource"},"glyph":{"id":"f312574f-a1ea-4448-834e-e8f4b78b1238","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bc5a1978-2b4f-4359-a7c0-6af21c625752","type":"GlyphRenderer"},{"attributes":{"label":{"value":"cria\u00e7\u00e3o"},"renderers":[{"id":"4389d0bd-7817-4540-9379-a5985fda35cd","type":"GlyphRenderer"}]},"id":"15fe39ae-265e-4131-ad0e-9b1f0c7d0b05","type":"LegendItem"},{"attributes":{"label":{"value":"contrata\u00e7\u00e3o"},"renderers":[{"id":"9033e95b-8cba-4548-a487-48646b347c36","type":"GlyphRenderer"}]},"id":"a9ddb1d1-e8ca-4fa2-b671-5050b2f02881","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"cd6debb2-312f-41aa-9e0a-6ceec8d57a16","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"185f17fc-8ea7-46c6-802b-1c36d655085b","type":"Rect"},{"attributes":{"data_source":{"id":"91fe2c45-3d00-497d-9aa1-ae8bd04f5b5c","type":"ColumnDataSource"},"glyph":{"id":"3cbc8fd5-70ae-4a5d-8277-26072c9dd6f1","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c8903a2c-ceb2-4479-9c97-dc47881f09a2","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"999615a7-7e93-4582-b282-ef1712a1ae83","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f0a6cd6a-a147-4d47-8a0f-874a071c0b75","type":"Rect"},{"attributes":{"data_source":{"id":"8683001c-103e-4b99-a105-10fc1f2a3c48","type":"ColumnDataSource"},"glyph":{"id":"e2c421a0-e66e-4d63-a3de-fec43bd74a74","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"6aa82fc4-9318-4fb8-8104-d687df43fcd3","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e2c421a0-e66e-4d63-a3de-fec43bd74a74","type":"Rect"},{"attributes":{"data_source":{"id":"ad7466ba-4e25-4d64-8eb1-546d283c93d1","type":"ColumnDataSource"},"glyph":{"id":"f8ce36db-911a-4310-b58a-0e44c88b10e3","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"01264398-63c3-4f4d-b1cb-f879f4e99e05","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["diretrizes"],"width":[0.8],"x":["disp\u00f5e"],"y":[2.5]}},"id":"88479b0d-bac1-4845-8e1a-85abbdcd86e8","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"48a61d78-7d7e-48dd-b173-d5483a355c22","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"4d0ea096-28b9-4ec9-9d60-dca8e1dd0da0","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"88479b0d-bac1-4845-8e1a-85abbdcd86e8","type":"ColumnDataSource"},"glyph":{"id":"e1b4f310-0e13-4c82-af3e-905a608afdca","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3cfff907-0174-4385-b4dc-1bf6a3e9b514","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3cbc8fd5-70ae-4a5d-8277-26072c9dd6f1","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza"],"chart_index":[{"categoria_geral":"autoriza","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[78.0],"label":[{"categoria_geral":"autoriza","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["autoriza"],"y":[39.0]}},"id":"ce2a6608-236f-49aa-b1f1-fe70d9ab62c2","type":"ColumnDataSource"}],"root_ids":["6dae9e9a-c9db-4a05-be64-c4a0cb19132c"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"b3502bfa-1402-4e0b-9d15-196c2237191e","elementid":"311292f3-0b03-431d-a77b-46a80bcac038","modelid":"6dae9e9a-c9db-4a05-be64-c4a0cb19132c"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("311292f3-0b03-431d-a77b-46a80bcac038")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'autoriza'
Todas as leis com 'autoriza' se referem ao poder executiva, vamos mudar a label para 'autoriza poder executivo'


```python
leis[leis['categoria_geral'] == 'autoriza'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>1013</td>
      <td>1992</td>
      <td>autoriza ao poder executivo desafetar áreas de...</td>
      <td>autoriza</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1023</td>
      <td>1992</td>
      <td>autoriza ao poder executivo desmembrar e desaf...</td>
      <td>autoriza</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1025</td>
      <td>1992</td>
      <td>autoriza a abertura de crédito especial e dá o...</td>
      <td>autoriza</td>
      <td>crédito</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1034</td>
      <td>1993</td>
      <td>autoriza a dispensa de multa, juros e redução ...</td>
      <td>autoriza</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>23</th>
      <td>1035</td>
      <td>1993</td>
      <td>autoriza os chefes dos poderes executivos e le...</td>
      <td>autoriza</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub_categoria_autoriza = get_counter_with_category('categoria_geral','autoriza',2)

sub_categoria_autoriza.most_common()
```




    [('poder', 44),
     ('abertura', 13),
     ('de', 8),
     ('reajuste', 6),
     ('contratação', 5),
     ('chefe', 3),
     ('município', 3),
     ('cessão', 2),
     ('dispensa', 1),
     ('chefes', 1),
     ('concessão', 1),
     ('poderes', 1),
     ('pbcfer', 1),
     ('da', 1),
     ('pagamento', 1),
     ('câmara', 1),
     ('celebração', 1)]



### Observações
Podemos ver que 'poder' tem quase todas as ocorrências (referente ao poder executivo); vamos mudar a label para 'autoriza poder executivo'


```python
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('autoriza', row['categoria_geral'],'autoriza poder executivo'), axis=1)
```


```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por sub-categoria", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="5c081063-2c27-4f64-97c6-d3bee031ad76"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("5c081063-2c27-4f64-97c6-d3bee031ad76").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("5c081063-2c27-4f64-97c6-d3bee031ad76");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid '5c081063-2c27-4f64-97c6-d3bee031ad76' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"19f8daab-b2cd-4d7e-8a7e-713d18e51736":{"roots":{"references":[{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"571604e4-6074-48d6-956b-8121a4e41878","type":"BasicTickFormatter"},"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"},"ticker":{"id":"366db1dc-87c3-40ec-b9c3-7f71df8b0544","type":"BasicTicker"}},"id":"4a1f4e12-3552-453d-ac99-f0ccf0f2c998","type":"LinearAxis"},{"attributes":{},"id":"c6213cdf-37b7-447b-86d7-127dd10d2e74","type":"CategoricalTicker"},{"attributes":{},"id":"366db1dc-87c3-40ec-b9c3-7f71df8b0544","type":"BasicTicker"},{"attributes":{"dimension":1,"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"},"ticker":{"id":"366db1dc-87c3-40ec-b9c3-7f71df8b0544","type":"BasicTicker"}},"id":"a41eb696-197e-4620-a78b-4e4286d1c8f0","type":"Grid"},{"attributes":{"callback":null,"factors":["?","altera lei","atribui","autoriza poder executivo","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","institui,","outra","reajusta","revoga"]},"id":"0048b1fa-ae30-459d-b6e1-0989262e4560","type":"FactorRange"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[27.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[89.5]}},"id":"e781146c-1b23-4431-9612-98f9a42027d1","type":"ColumnDataSource"},{"attributes":{},"id":"0badc65f-a694-438b-a331-651553dfc7b6","type":"CategoricalTickFormatter"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["disp\u00f5e"],"y":[71.0]}},"id":"37bfb529-53bd-4956-8045-ea09c50d9d41","type":"ColumnDataSource"},{"attributes":{},"id":"571604e4-6074-48d6-956b-8121a4e41878","type":"BasicTickFormatter"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["contrata\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[64.5]}},"id":"69556566-7b1e-42b5-9683-37ba2e001144","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"32232382-339f-4c73-9b31-e4e3dd030c80","type":"HelpTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"36a50dd4-aeb7-46ee-b4db-7e219fbb42f2","type":"Rect"},{"attributes":{"data_source":{"id":"c82d22f6-042f-4bd3-b8f1-60fa7a678d47","type":"ColumnDataSource"},"glyph":{"id":"7c043633-aced-47d1-9c00-f563ba729589","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a8aa9ac6-ac1a-4ed9-865c-2dfb747667cd","type":"GlyphRenderer"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"f5897507-9edd-4830-8fc8-f2ac7072f54c","type":"BoxAnnotation"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7baa79f9-f191-4207-9130-8ce0af5946eb","type":"Rect"},{"attributes":{"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"b7afe58b-8862-47aa-bde3-0af1352fe772","type":"SaveTool"},{"attributes":{"overlay":{"id":"f5897507-9edd-4830-8fc8-f2ac7072f54c","type":"BoxAnnotation"},"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"3a773eb5-3a6e-472b-8f1e-e81478f9f0d4","type":"BoxZoomTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disp\u00f5e"],"y":[34.0]}},"id":"71ff0979-5ad2-4360-a130-f6ec502658b4","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"41d541a7-c2bd-46c8-999d-ad5bfa13ba76","type":"WheelZoomTool"},{"attributes":{"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"f424a18a-3dd1-4bea-ade8-475a52dbceac","type":"ResetTool"},{"attributes":{},"id":"32570b2b-1749-4e83-b0b1-3b182c1354da","type":"ToolEvents"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"cr\u00e9dito"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[15.0],"label":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["autoriza poder executivo"],"y":[85.5]}},"id":"bcf9c676-8462-40cb-8136-a2cfbf4e98c5","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["outra"],"y":[83.5]}},"id":"1704719f-7072-47aa-b834-b7167a942d0b","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"bb173a42-e59c-416b-8db1-3e8ca72be7a0","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"b4731e3d-5220-45f1-904a-08350d353c23","type":"PanTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"plano"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["outra"],"y":[84.5]}},"id":"fdb37baa-5fe6-48ee-9e5b-98dcf7c35621","type":"ColumnDataSource"},{"attributes":{"below":[{"id":"77da3b14-729f-4956-b459-5813cbde4a82","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"4a1f4e12-3552-453d-ac99-f0ccf0f2c998","type":"LinearAxis"}],"renderers":[{"id":"f5897507-9edd-4830-8fc8-f2ac7072f54c","type":"BoxAnnotation"},{"id":"fcb0b7cd-9c87-4a69-aac6-1bd588216d3d","type":"GlyphRenderer"},{"id":"abefd671-6f01-4220-9c47-fc2b916da10f","type":"GlyphRenderer"},{"id":"c2afbbbb-3a72-43a2-9592-480b87b6ecab","type":"GlyphRenderer"},{"id":"120a0cf0-7531-4da3-a0a9-6c20c4e26e28","type":"GlyphRenderer"},{"id":"a8aa9ac6-ac1a-4ed9-865c-2dfb747667cd","type":"GlyphRenderer"},{"id":"47ce1e9f-d253-477e-aeba-5c997e648f7b","type":"GlyphRenderer"},{"id":"d92eec4f-f78f-4f8e-8ed0-5891f6ab80dd","type":"GlyphRenderer"},{"id":"3e5592ec-e069-4622-adce-f5a333b01c28","type":"GlyphRenderer"},{"id":"5afe9af0-2c3a-4ed1-aabf-a2489685ecaf","type":"GlyphRenderer"},{"id":"41395de7-db37-406d-9c0a-f595ec8a8ef1","type":"GlyphRenderer"},{"id":"f65ec41d-2ee2-4152-9ca7-c8717790ce84","type":"GlyphRenderer"},{"id":"4bfb7950-26d0-4366-98d3-d2990cc59d28","type":"GlyphRenderer"},{"id":"d45a55d2-96d5-4445-acf6-1b3b712f5f06","type":"GlyphRenderer"},{"id":"7d3608e0-78ef-400e-9573-3b17f66a49fd","type":"GlyphRenderer"},{"id":"a1b0dffa-f21b-4f30-94fe-5f9117e32d0a","type":"GlyphRenderer"},{"id":"feaa7e2a-f6c9-4e82-b816-7494f9a5c2d0","type":"GlyphRenderer"},{"id":"ae2dacd0-2d3f-4703-86a1-2a12f5dca6bd","type":"GlyphRenderer"},{"id":"88a2781f-c1e3-4253-820c-666fde9e14c9","type":"GlyphRenderer"},{"id":"43610051-6ed3-4bf2-9379-004197f4607f","type":"GlyphRenderer"},{"id":"c59d7188-0e4b-42af-8b12-d01ae8026eee","type":"GlyphRenderer"},{"id":"a1643ac2-3e0e-4aaa-91d2-c1c2bc84d768","type":"GlyphRenderer"},{"id":"c99bb893-a24c-4099-b751-ae4841710a89","type":"GlyphRenderer"},{"id":"d41aa21c-8a9d-4641-94ac-9370fcba8b0a","type":"GlyphRenderer"},{"id":"23a5b413-ebf0-463f-8151-206682637048","type":"GlyphRenderer"},{"id":"b92c4563-2d27-4169-be85-67482e172bd2","type":"GlyphRenderer"},{"id":"f9718f3d-6135-433b-9adf-f0d29474c11f","type":"GlyphRenderer"},{"id":"28e595bd-14bb-450a-938f-a1fc8c40b1d6","type":"GlyphRenderer"},{"id":"fbb62372-53a8-456f-b1fd-0bf0ad73b8be","type":"GlyphRenderer"},{"id":"d8ef8168-d29b-4ab0-8303-4dcc99830b9f","type":"GlyphRenderer"},{"id":"f976b3b5-d85b-4fc0-8454-04c681d57c55","type":"GlyphRenderer"},{"id":"e17693d9-adce-436a-947b-b1abdc145b1a","type":"GlyphRenderer"},{"id":"a623402c-8bf8-4477-9329-334354f3220c","type":"GlyphRenderer"},{"id":"c8a25535-1821-4c31-a59e-2e878c2db916","type":"GlyphRenderer"},{"id":"999a517b-63e7-4c5c-b16d-9b305e207d3b","type":"GlyphRenderer"},{"id":"648fae51-abbb-4558-ade1-3196f397ce5a","type":"GlyphRenderer"},{"id":"2a564e1b-c477-4371-8036-6e10e61f7519","type":"GlyphRenderer"},{"id":"baff117e-5deb-4538-83b1-6fc68509a8d3","type":"GlyphRenderer"},{"id":"5e6f35bd-5cbe-42f6-ae55-97efd179c653","type":"GlyphRenderer"},{"id":"cb410f8a-15e2-45d5-9681-6bcd74efcfd2","type":"Legend"},{"id":"77da3b14-729f-4956-b459-5813cbde4a82","type":"CategoricalAxis"},{"id":"4a1f4e12-3552-453d-ac99-f0ccf0f2c998","type":"LinearAxis"},{"id":"a41eb696-197e-4620-a78b-4e4286d1c8f0","type":"Grid"}],"title":{"id":"5603df9e-78a3-414b-9058-785a967708ed","type":"Title"},"tool_events":{"id":"32570b2b-1749-4e83-b0b1-3b182c1354da","type":"ToolEvents"},"toolbar":{"id":"376b70c4-c085-475c-b48f-82462dbac3b2","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"0048b1fa-ae30-459d-b6e1-0989262e4560","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"87dc8200-4d84-4736-88e6-893b774dc230","type":"Range1d"}},"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["estabelece"],"y":[27.0]}},"id":"3f032ec9-25b1-44b1-9bdb-1d27b7ca8bc0","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["estrutura"],"width":[0.8],"x":["disp\u00f5e"],"y":[128.0]}},"id":"92d2e051-11a0-4314-a1bf-45ccff586ac6","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["revis\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[116.0]}},"id":"1957b20d-ac09-4bac-8320-0451516e3721","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"1704719f-7072-47aa-b834-b7167a942d0b","type":"ColumnDataSource"},"glyph":{"id":"03815c1b-532d-43e0-a728-f2baece8b4f7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"999a517b-63e7-4c5c-b16d-9b305e207d3b","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"03815c1b-532d-43e0-a728-f2baece8b4f7","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a39a2584-772b-4e01-9e13-0e2772b7f2df","type":"Rect"},{"attributes":{"data_source":{"id":"72a469c3-4b40-4328-a3d5-9c4f7d6d7c7b","type":"ColumnDataSource"},"glyph":{"id":"a39a2584-772b-4e01-9e13-0e2772b7f2df","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"648fae51-abbb-4558-ade1-3196f397ce5a","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["abertura"],"width":[0.8],"x":["disp\u00f5e"],"y":[107.5]}},"id":"49b939ce-e84b-41e5-8ead-d347cf3a6ecd","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"2fb77a26-e93e-42d3-9170-e063e778e48e","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reajuste"],"width":[0.8],"x":["disp\u00f5e"],"y":[136.0]}},"id":"c776091d-ebb6-463e-9795-3b18e97efe79","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["disp\u00f5e"],"y":[122.5]}},"id":"51db9cf3-8647-468b-97ce-86e70c003504","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"3f032ec9-25b1-44b1-9bdb-1d27b7ca8bc0","type":"ColumnDataSource"},"glyph":{"id":"2fb77a26-e93e-42d3-9170-e063e778e48e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2a564e1b-c477-4371-8036-6e10e61f7519","type":"GlyphRenderer"},{"attributes":{"label":{"value":"reestrutura\u00e7\u00e3o"},"renderers":[{"id":"f976b3b5-d85b-4fc0-8454-04c681d57c55","type":"GlyphRenderer"}]},"id":"b664a104-7a66-47bf-93d7-c0fa2d69acda","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["aumento"],"width":[0.8],"x":["disp\u00f5e"],"y":[132.5]}},"id":"5b439911-77d7-43ec-82ee-d842047b8f6e","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7cc62c78-000c-4112-a8f5-89d9908ae429","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7c043633-aced-47d1-9c00-f563ba729589","type":"Rect"},{"attributes":{"label":{"value":"cr\u00e9dito"},"renderers":[{"id":"3e5592ec-e069-4622-adce-f5a333b01c28","type":"GlyphRenderer"}]},"id":"e4249c31-b76f-4ffb-a01c-4742625c380b","type":"LegendItem"},{"attributes":{"data_source":{"id":"dc6f72f7-32a8-4c8e-ba57-976225a31097","type":"ColumnDataSource"},"glyph":{"id":"25c987e3-1fa5-4fc0-b348-7600d09516d2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"fcb0b7cd-9c87-4a69-aac6-1bd588216d3d","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["institui\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[144.5]}},"id":"a32fe480-a0df-4aa2-87e6-f15f9d8b84b0","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"b61040a9-fb5d-4f5c-86ff-113f5b519a93","type":"ColumnDataSource"},"glyph":{"id":"7cc62c78-000c-4112-a8f5-89d9908ae429","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"baff117e-5deb-4538-83b1-6fc68509a8d3","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"cfd75d4d-60de-4ea5-a08a-066b500e7e82","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reestrutura\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[139.5]}},"id":"d0479b57-4531-4f9f-9846-12e9131b3eaf","type":"ColumnDataSource"},{"attributes":{"label":{"value":"aumento"},"renderers":[{"id":"fbb62372-53a8-456f-b1fd-0bf0ad73b8be","type":"GlyphRenderer"}]},"id":"cffba28e-b6d1-4032-aac4-d8acdc3527cc","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["disp\u00f5e"],"y":[151.0]}},"id":"c504b643-7577-489c-8d7a-1baad3d8aa0b","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"25c987e3-1fa5-4fc0-b348-7600d09516d2","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["denomina\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[163.5]}},"id":"b61040a9-fb5d-4f5c-86ff-113f5b519a93","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["altera\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[157.5]}},"id":"72a469c3-4b40-4328-a3d5-9c4f7d6d7c7b","type":"ColumnDataSource"},{"attributes":{"label":{"value":"contrata\u00e7\u00e3o"},"renderers":[{"id":"ae2dacd0-2d3f-4703-86a1-2a12f5dca6bd","type":"GlyphRenderer"}]},"id":"ea5151d4-6dab-4329-95a9-0da99b197d0c","type":"LegendItem"},{"attributes":{"data_source":{"id":"fdb37baa-5fe6-48ee-9e5b-98dcf7c35621","type":"ColumnDataSource"},"glyph":{"id":"cfd75d4d-60de-4ea5-a08a-066b500e7e82","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"5e6f35bd-5cbe-42f6-ae55-97efd179c653","type":"GlyphRenderer"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"0badc65f-a694-438b-a331-651553dfc7b6","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"},"ticker":{"id":"c6213cdf-37b7-447b-86d7-127dd10d2e74","type":"CategoricalTicker"}},"id":"77da3b14-729f-4956-b459-5813cbde4a82","type":"CategoricalAxis"},{"attributes":{"label":{"value":"-"},"renderers":[{"id":"feaa7e2a-f6c9-4e82-b816-7494f9a5c2d0","type":"GlyphRenderer"}]},"id":"73c14c4d-173c-4565-9825-8ed329f74e03","type":"LegendItem"},{"attributes":{"label":{"value":"cria\u00e7\u00e3o"},"renderers":[{"id":"c99bb893-a24c-4099-b751-ae4841710a89","type":"GlyphRenderer"}]},"id":"f55a2010-4507-4593-b19f-b75e59b6b4e2","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"c82d22f6-042f-4bd3-b8f1-60fa7a678d47","type":"ColumnDataSource"},{"attributes":{"label":{"value":"plano"},"renderers":[{"id":"a1643ac2-3e0e-4aaa-91d2-c1c2bc84d768","type":"GlyphRenderer"}]},"id":"906ac39b-e31d-4aa6-9ce8-7e0678ec774f","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"dc6f72f7-32a8-4c8e-ba57-976225a31097","type":"ColumnDataSource"},{"attributes":{"label":{"value":"abertura"},"renderers":[{"id":"d41aa21c-8a9d-4641-94ac-9370fcba8b0a","type":"GlyphRenderer"}]},"id":"2222c915-5ebb-4d11-ab95-d0b1e13f4068","type":"LegendItem"},{"attributes":{"label":{"value":"estrutura"},"renderers":[{"id":"f9718f3d-6135-433b-9adf-f0d29474c11f","type":"GlyphRenderer"}]},"id":"9b3cebc8-fbf2-452d-ab69-3a63dbf073fa","type":"LegendItem"},{"attributes":{"label":{"value":"revis\u00e3o"},"renderers":[{"id":"23a5b413-ebf0-463f-8151-206682637048","type":"GlyphRenderer"}]},"id":"e3f7f9b7-ee4e-46f6-934f-d4e562c23a52","type":"LegendItem"},{"attributes":{"label":{"value":"vencimentos"},"renderers":[{"id":"a623402c-8bf8-4477-9329-334354f3220c","type":"GlyphRenderer"}]},"id":"3255f047-375c-430d-9089-0815bb41bcfe","type":"LegendItem"},{"attributes":{"label":{"value":"reajuste"},"renderers":[{"id":"d8ef8168-d29b-4ab0-8303-4dcc99830b9f","type":"GlyphRenderer"}]},"id":"a342172c-611a-462d-ae06-14f9d551dec4","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3c6fe8a7-7f39-4a84-9c9e-0e0b015553e2","type":"Rect"},{"attributes":{"label":{"value":"altera\u00e7\u00e3o"},"renderers":[{"id":"648fae51-abbb-4558-ade1-3196f397ce5a","type":"GlyphRenderer"}]},"id":"043e44cd-912f-4682-8b0b-b26fdb1bd8f4","type":"LegendItem"},{"attributes":{"label":{"value":"institui\u00e7\u00e3o"},"renderers":[{"id":"e17693d9-adce-436a-947b-b1abdc145b1a","type":"GlyphRenderer"}]},"id":"41fae51a-8014-4f27-9b35-accc6cd17cf5","type":"LegendItem"},{"attributes":{"label":{"value":"denomina\u00e7\u00e3o"},"renderers":[{"id":"baff117e-5deb-4538-83b1-6fc68509a8d3","type":"GlyphRenderer"}]},"id":"1ad7798e-6e74-4410-8197-17c6d399be81","type":"LegendItem"},{"attributes":{"data_source":{"id":"bb173a42-e59c-416b-8db1-3e8ca72be7a0","type":"ColumnDataSource"},"glyph":{"id":"0556774e-c911-447a-9f01-1aecddca1f5e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d45a55d2-96d5-4445-acf6-1b3b712f5f06","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"f8c75eb9-112f-45b6-ae0c-39e4a6dcf543","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"c7351bbe-6bb5-4a1f-94b8-37bf37c27f6b","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"334a5b05-72e5-40db-8b7d-8eec3f38da76","type":"ColumnDataSource"},"glyph":{"id":"36a50dd4-aeb7-46ee-b4db-7e219fbb42f2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d92eec4f-f78f-4f8e-8ed0-5891f6ab80dd","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"c7351bbe-6bb5-4a1f-94b8-37bf37c27f6b","type":"ColumnDataSource"},"glyph":{"id":"0bd85e49-7d36-4bab-ad0d-2cafa4cae96b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"28e595bd-14bb-450a-938f-a1fc8c40b1d6","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"69556566-7b1e-42b5-9683-37ba2e001144","type":"ColumnDataSource"},"glyph":{"id":"9bfa0422-17ad-4007-9e43-5cbff6308923","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ae2dacd0-2d3f-4703-86a1-2a12f5dca6bd","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"bcf9c676-8462-40cb-8136-a2cfbf4e98c5","type":"ColumnDataSource"},"glyph":{"id":"4cefa9ac-0e0c-4636-8a0f-d5e0606e7e0b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3e5592ec-e069-4622-adce-f5a333b01c28","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"4be76530-a8d0-42e3-b36e-ed0a5ad031c0","type":"ColumnDataSource"},"glyph":{"id":"59dd6297-157e-480a-a8be-8f3073394ec6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c59d7188-0e4b-42af-8b12-d01ae8026eee","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"4be76530-a8d0-42e3-b36e-ed0a5ad031c0","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"86c129c0-fd6c-4ad3-9b1a-cd0455fc0ba8","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estabelece"],"y":[12.5]}},"id":"cdb52655-7318-471d-b22f-1cb2c53f35ad","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"92d2e051-11a0-4314-a1bf-45ccff586ac6","type":"ColumnDataSource"},"glyph":{"id":"7baa79f9-f191-4207-9130-8ce0af5946eb","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f9718f3d-6135-433b-9adf-f0d29474c11f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4cefa9ac-0e0c-4636-8a0f-d5e0606e7e0b","type":"Rect"},{"attributes":{"data_source":{"id":"cdb52655-7318-471d-b22f-1cb2c53f35ad","type":"ColumnDataSource"},"glyph":{"id":"276edb8c-f00b-41da-ad31-5dcc3677d215","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"88a2781f-c1e3-4253-820c-666fde9e14c9","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"4fd12e92-e4d1-4654-9d89-07857286d52f","type":"ColumnDataSource"},"glyph":{"id":"d94a9bd4-6269-4464-8819-f73a194c6bc4","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"43610051-6ed3-4bf2-9379-004197f4607f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"51b71894-60ac-4e15-bc93-4a3dea216a06","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"276edb8c-f00b-41da-ad31-5dcc3677d215","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"c93419f1-5880-4069-b250-d330d52c4459","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0bd85e49-7d36-4bab-ad0d-2cafa4cae96b","type":"Rect"},{"attributes":{"data_source":{"id":"f8c75eb9-112f-45b6-ae0c-39e4a6dcf543","type":"ColumnDataSource"},"glyph":{"id":"51b71894-60ac-4e15-bc93-4a3dea216a06","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"5afe9af0-2c3a-4ed1-aabf-a2489685ecaf","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[55.0],"label":[{"categoria_geral":"institui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui"],"y":[27.5]}},"id":"a6a88085-4cb0-4c58-a07a-61bf60fd6bbe","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a8183f7d-a043-4ec1-b82a-ca68c42ba33e","type":"ColumnDataSource"},"glyph":{"id":"37ec88f8-90f8-4b93-b84d-936e885c3404","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"7d3608e0-78ef-400e-9573-3b17f66a49fd","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3cccc0ae-139b-4d40-bdf8-10fdbbdd9a21","type":"Rect"},{"attributes":{"data_source":{"id":"49b939ce-e84b-41e5-8ead-d347cf3a6ecd","type":"ColumnDataSource"},"glyph":{"id":"60a774a7-969a-479a-b679-c6e234ca1367","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d41aa21c-8a9d-4641-94ac-9370fcba8b0a","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"59dd6297-157e-480a-a8be-8f3073394ec6","type":"Rect"},{"attributes":{"data_source":{"id":"5b439911-77d7-43ec-82ee-d842047b8f6e","type":"ColumnDataSource"},"glyph":{"id":"3cccc0ae-139b-4d40-bdf8-10fdbbdd9a21","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"fbb62372-53a8-456f-b1fd-0bf0ad73b8be","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"544a048b-122d-4ffc-bab7-b2b16a5dc523","type":"Rect"},{"attributes":{"items":[{"id":"344c1de9-9a85-458e-bc68-f1be5f5ef1f8","type":"LegendItem"},{"id":"eba55c26-af57-4b78-ade3-99b19630a2b8","type":"LegendItem"},{"id":"e4249c31-b76f-4ffb-a01c-4742625c380b","type":"LegendItem"},{"id":"73c14c4d-173c-4565-9825-8ed329f74e03","type":"LegendItem"},{"id":"ea5151d4-6dab-4329-95a9-0da99b197d0c","type":"LegendItem"},{"id":"906ac39b-e31d-4aa6-9ce8-7e0678ec774f","type":"LegendItem"},{"id":"f55a2010-4507-4593-b19f-b75e59b6b4e2","type":"LegendItem"},{"id":"2222c915-5ebb-4d11-ab95-d0b1e13f4068","type":"LegendItem"},{"id":"e3f7f9b7-ee4e-46f6-934f-d4e562c23a52","type":"LegendItem"},{"id":"9b3cebc8-fbf2-452d-ab69-3a63dbf073fa","type":"LegendItem"},{"id":"cffba28e-b6d1-4032-aac4-d8acdc3527cc","type":"LegendItem"},{"id":"a342172c-611a-462d-ae06-14f9d551dec4","type":"LegendItem"},{"id":"b664a104-7a66-47bf-93d7-c0fa2d69acda","type":"LegendItem"},{"id":"41fae51a-8014-4f27-9b35-accc6cd17cf5","type":"LegendItem"},{"id":"3255f047-375c-430d-9089-0815bb41bcfe","type":"LegendItem"},{"id":"043e44cd-912f-4682-8b0b-b26fdb1bd8f4","type":"LegendItem"},{"id":"1ad7798e-6e74-4410-8197-17c6d399be81","type":"LegendItem"}],"location":"top_left","plot":{"id":"621fd28c-ae76-437a-a882-1a3b127e9fc8","subtype":"Chart","type":"Plot"}},"id":"cb410f8a-15e2-45d5-9681-6bcd74efcfd2","type":"Legend"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d94a9bd4-6269-4464-8819-f73a194c6bc4","type":"Rect"},{"attributes":{"callback":null,"end":174.3},"id":"87dc8200-4d84-4736-88e6-893b774dc230","type":"Range1d"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"e69fd1dc-d6ec-4b17-b67f-d3c0712555be","type":"Rect"},{"attributes":{"data_source":{"id":"86c129c0-fd6c-4ad3-9b1a-cd0455fc0ba8","type":"ColumnDataSource"},"glyph":{"id":"544a048b-122d-4ffc-bab7-b2b16a5dc523","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"41395de7-db37-406d-9c0a-f595ec8a8ef1","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"37bfb529-53bd-4956-8045-ea09c50d9d41","type":"ColumnDataSource"},"glyph":{"id":"e69fd1dc-d6ec-4b17-b67f-d3c0712555be","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a1643ac2-3e0e-4aaa-91d2-c1c2bc84d768","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera lei"],"chart_index":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[116.0],"label":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["-"],"width":[0.8],"x":["altera lei"],"y":[58.0]}},"id":"866fb8f0-8a6e-4c57-abfa-c9cc24f79057","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c2c965ac-2f35-4f13-996c-9143d5d241a2","type":"Rect"},{"attributes":{"data_source":{"id":"c776091d-ebb6-463e-9795-3b18e97efe79","type":"ColumnDataSource"},"glyph":{"id":"c2c965ac-2f35-4f13-996c-9143d5d241a2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d8ef8168-d29b-4ab0-8303-4dcc99830b9f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b0d13e9f-0cc5-4a8a-a8a7-3403735c6cbf","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d20a9717-5499-4dba-b9ea-a78474801a9f","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"32037b24-9c24-478d-b657-56d6a17a5715","type":"Rect"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"fcb0b7cd-9c87-4a69-aac6-1bd588216d3d","type":"GlyphRenderer"}]},"id":"344c1de9-9a85-458e-bc68-f1be5f5ef1f8","type":"LegendItem"},{"attributes":{"data_source":{"id":"c93419f1-5880-4069-b250-d330d52c4459","type":"ColumnDataSource"},"glyph":{"id":"b0d13e9f-0cc5-4a8a-a8a7-3403735c6cbf","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f65ec41d-2ee2-4152-9ca7-c8717790ce84","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"e781146c-1b23-4431-9612-98f9a42027d1","type":"ColumnDataSource"},"glyph":{"id":"32037b24-9c24-478d-b657-56d6a17a5715","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c99bb893-a24c-4099-b751-ae4841710a89","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"37ec88f8-90f8-4b93-b84d-936e885c3404","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f1ef3ef0-2344-41c3-bd19-996be1c95ac1","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"a8183f7d-a043-4ec1-b82a-ca68c42ba33e","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"51db9cf3-8647-468b-97ce-86e70c003504","type":"ColumnDataSource"},"glyph":{"id":"7e9ea22a-8ea1-408a-afc9-66e892ce505e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b92c4563-2d27-4169-be85-67482e172bd2","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"d0479b57-4531-4f9f-9846-12e9131b3eaf","type":"ColumnDataSource"},"glyph":{"id":"f1ef3ef0-2344-41c3-bd19-996be1c95ac1","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f976b3b5-d85b-4fc0-8454-04c681d57c55","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"c504b643-7577-489c-8d7a-1baad3d8aa0b","type":"ColumnDataSource"},"glyph":{"id":"bb394528-a4af-43b3-ad63-1457665d0dc8","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a623402c-8bf8-4477-9329-334354f3220c","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5833046c-3f87-4fa4-b58e-9e9a44d41e85","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"60a774a7-969a-479a-b679-c6e234ca1367","type":"Rect"},{"attributes":{"label":{"value":"diretrizes"},"renderers":[{"id":"d92eec4f-f78f-4f8e-8ed0-5891f6ab80dd","type":"GlyphRenderer"}]},"id":"eba55c26-af57-4b78-ade3-99b19630a2b8","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9bfa0422-17ad-4007-9e43-5cbff6308923","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7e9ea22a-8ea1-408a-afc9-66e892ce505e","type":"Rect"},{"attributes":{"data_source":{"id":"a6a88085-4cb0-4c58-a07a-61bf60fd6bbe","type":"ColumnDataSource"},"glyph":{"id":"5833046c-3f87-4fa4-b58e-9e9a44d41e85","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a1b0dffa-f21b-4f30-94fe-5f9117e32d0a","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"9e2c4d53-64f3-48a8-9953-ac67fd3a3fe5","type":"ColumnDataSource"},"glyph":{"id":"1fdc18cd-854c-4f7b-8e5c-9ce1b33de1e5","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c8a25535-1821-4c31-a59e-2e878c2db916","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"bb394528-a4af-43b3-ad63-1457665d0dc8","type":"Rect"},{"attributes":{"data_source":{"id":"a32fe480-a0df-4aa2-87e6-f15f9d8b84b0","type":"ColumnDataSource"},"glyph":{"id":"d20a9717-5499-4dba-b9ea-a78474801a9f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e17693d9-adce-436a-947b-b1abdc145b1a","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fb7ca7bf-f573-4b5f-a7c6-290341a10cae","type":"Rect"},{"attributes":{"data_source":{"id":"71ff0979-5ad2-4360-a130-f6ec502658b4","type":"ColumnDataSource"},"glyph":{"id":"3c6fe8a7-7f39-4a84-9c9e-0e0b015553e2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4bfb7950-26d0-4366-98d3-d2990cc59d28","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1fdc18cd-854c-4f7b-8e5c-9ce1b33de1e5","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"40504eb0-c57d-4410-83ab-dfb39ff20156","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"eca96401-c2b0-4337-9da5-dd9c3c9d3478","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui,"],"chart_index":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui,"],"y":[3.0]}},"id":"9e2c4d53-64f3-48a8-9953-ac67fd3a3fe5","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"1957b20d-ac09-4bac-8320-0451516e3721","type":"ColumnDataSource"},"glyph":{"id":"eca96401-c2b0-4337-9da5-dd9c3c9d3478","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"23a5b413-ebf0-463f-8151-206682637048","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"866fb8f0-8a6e-4c57-abfa-c9cc24f79057","type":"ColumnDataSource"},"glyph":{"id":"40504eb0-c57d-4410-83ab-dfb39ff20156","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"feaa7e2a-f6c9-4e82-b816-7494f9a5c2d0","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0556774e-c911-447a-9f01-1aecddca1f5e","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[77.0],"label":[{"categoria_geral":"cria","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["cria"],"y":[38.5]}},"id":"8c79567b-9933-4171-9cd9-70c17ff4a87a","type":"ColumnDataSource"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"b4731e3d-5220-45f1-904a-08350d353c23","type":"PanTool"},{"id":"41d541a7-c2bd-46c8-999d-ad5bfa13ba76","type":"WheelZoomTool"},{"id":"3a773eb5-3a6e-472b-8f1e-e81478f9f0d4","type":"BoxZoomTool"},{"id":"b7afe58b-8862-47aa-bde3-0af1352fe772","type":"SaveTool"},{"id":"f424a18a-3dd1-4bea-ade8-475a52dbceac","type":"ResetTool"},{"id":"32232382-339f-4c73-9b31-e4e3dd030c80","type":"HelpTool"}]},"id":"376b70c4-c085-475c-b48f-82462dbac3b2","type":"Toolbar"},{"attributes":{"plot":null,"text":"Quantidade de Leis por sub-categoria"},"id":"5603df9e-78a3-414b-9058-785a967708ed","type":"Title"},{"attributes":{"data_source":{"id":"fa7359a3-e339-418e-a121-d97547b17d0f","type":"ColumnDataSource"},"glyph":{"id":"fb7ca7bf-f573-4b5f-a7c6-290341a10cae","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"abefd671-6f01-4220-9c47-fc2b916da10f","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"8c79567b-9933-4171-9cd9-70c17ff4a87a","type":"ColumnDataSource"},"glyph":{"id":"058ca9e5-ece1-49c4-abbc-b0b2c1072d42","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"120a0cf0-7531-4da3-a0a9-6c20c4e26e28","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[78.0],"label":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["autoriza poder executivo"],"y":[39.0]}},"id":"fa7359a3-e339-418e-a121-d97547b17d0f","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"6ca660fa-0667-4fb2-8a2d-7149bd2dc9b5","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"6ca660fa-0667-4fb2-8a2d-7149bd2dc9b5","type":"ColumnDataSource"},"glyph":{"id":"6e44136a-73a7-4e2d-b5ad-baad1f9e1475","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c2afbbbb-3a72-43a2-9592-480b87b6ecab","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6e44136a-73a7-4e2d-b5ad-baad1f9e1475","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"058ca9e5-ece1-49c4-abbc-b0b2c1072d42","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"4fd12e92-e4d1-4654-9d89-07857286d52f","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"outra"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[83.0],"label":[{"categoria_geral":"outra","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[41.5]}},"id":"1c9da740-a788-45de-b50d-bc53774f082a","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"59a54eaf-6884-4413-980b-ba7af0bdc5d6","type":"Rect"},{"attributes":{"data_source":{"id":"1c9da740-a788-45de-b50d-bc53774f082a","type":"ColumnDataSource"},"glyph":{"id":"59a54eaf-6884-4413-980b-ba7af0bdc5d6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"47ce1e9f-d253-477e-aeba-5c997e648f7b","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["diretrizes"],"width":[0.8],"x":["disp\u00f5e"],"y":[2.5]}},"id":"334a5b05-72e5-40db-8b7d-8eec3f38da76","type":"ColumnDataSource"}],"root_ids":["621fd28c-ae76-437a-a882-1a3b127e9fc8"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"19f8daab-b2cd-4d7e-8a7e-713d18e51736","elementid":"5c081063-2c27-4f64-97c6-d3bee031ad76","modelid":"621fd28c-ae76-437a-a882-1a3b127e9fc8"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("5c081063-2c27-4f64-97c6-d3bee031ad76")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'cria'
Esta categoria pode ter sub-categorizas interessantes, vamos investigar isso


```python
leis[leis['categoria_geral'] == 'cria'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>1015</td>
      <td>1992</td>
      <td>cria cargos (funções) para efeito de concursos...</td>
      <td>cria</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1033</td>
      <td>1993</td>
      <td>cria cargos de assessores dos secretarios muni...</td>
      <td>cria</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1038</td>
      <td>1993</td>
      <td>cria o departamento de agricultura, cargos com...</td>
      <td>cria</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>35</th>
      <td>1047</td>
      <td>1993</td>
      <td>cria cargos em comissão e dá outras providências</td>
      <td>cria</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>52</th>
      <td>1064</td>
      <td>1994</td>
      <td>cria o conselho municipal de defesa do meio am...</td>
      <td>cria</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub_categoria_cria = Counter()

for index,row in leis.iterrows():
    if (row['categoria_geral'] == 'cria'):
        ementa = row['ementa'].split(" ")
        fourth_word = ementa[1]
        fifth_word = ementa[2]
        if (fourth_word in stop_words):
            sub_categoria_cria[fifth_word] += 1
        else:
            sub_categoria_cria[fourth_word] += 1


sub_categoria_cria.most_common()
```




    [('cargos', 25),
     ('nome', 17),
     ('conselho', 9),
     ('secretaria', 4),
     ('cargo', 3),
     ('fundo', 3),
     ('plano', 2),
     ('sistema', 2),
     ('departamento', 1),
     ('diretoria', 1),
     ('medalha', 1),
     ('trofeu', 1),
     ('duas', 1),
     ('guarda', 1),
     ('programa', 1),
     ('coordenadoria', 1),
     ('âmbito', 1),
     ('reserva', 1),
     ('02', 1),
     ('2', 1)]



### Comentários
Vamos utilizar as sub-categorias com mais de 3 ocorrências


```python
categorias_cria = [x for x,cnt in sub_categoria_cria.most_common() if cnt > 3 ]

def get_sub_categoria_cria(ementa):
    ementa = ementa.split(" ")
    fourth_word = ementa[1]
    fifth_word = ementa[2]
    if (fourth_word in categorias_cria):
        return fourth_word
    elif (fifth_word in categorias_cria):
        return fifth_word
    else:
        return "outra"

def change_sub_category_label_cria(to_change, category,old_sub_category, ementa):
    if (category == to_change):
        return get_sub_categoria_cria(ementa)
    else:
        return old_sub_category

leis['sub_categoria'] = leis.apply(lambda row: change_sub_category_label_cria('cria',row['categoria_geral'],row['sub_categoria'],row['ementa']), axis=1)
```


```python
leis[leis['categoria_geral'] == 'cria'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>1015</td>
      <td>1992</td>
      <td>cria cargos (funções) para efeito de concursos...</td>
      <td>cria</td>
      <td>cargos</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1033</td>
      <td>1993</td>
      <td>cria cargos de assessores dos secretarios muni...</td>
      <td>cria</td>
      <td>cargos</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1038</td>
      <td>1993</td>
      <td>cria o departamento de agricultura, cargos com...</td>
      <td>cria</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>35</th>
      <td>1047</td>
      <td>1993</td>
      <td>cria cargos em comissão e dá outras providências</td>
      <td>cria</td>
      <td>cargos</td>
    </tr>
    <tr>
      <th>52</th>
      <td>1064</td>
      <td>1994</td>
      <td>cria o conselho municipal de defesa do meio am...</td>
      <td>cria</td>
      <td>conselho</td>
    </tr>
  </tbody>
</table>
</div>




```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por sub-categoria", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="fa2541dc-bf1f-459b-92fd-c15087461a39"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("fa2541dc-bf1f-459b-92fd-c15087461a39").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("fa2541dc-bf1f-459b-92fd-c15087461a39");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid 'fa2541dc-bf1f-459b-92fd-c15087461a39' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"b7c8f4a7-0a5c-452b-ba79-b5da70100b4e":{"roots":{"references":[{"attributes":{"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"9625d82f-36db-4ee2-9e3d-fb3a0e6dfea5","type":"SaveTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[83.0],"label":[{"categoria_geral":"outra","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[41.5]}},"id":"e39b6395-18c1-4dd8-a4ff-e79c7589729d","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"37b70500-cea5-4c6a-a8c0-e5c3eea9db56","type":"ResetTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"457f98a0-a5cf-4cfe-94a8-442f6a7f8f79","type":"Rect"},{"attributes":{"data_source":{"id":"f222e8f4-a1fd-472e-af81-6d7a9fac22f3","type":"ColumnDataSource"},"glyph":{"id":"57a9691f-76f6-4ed9-9333-14cf3aee78f7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a881af44-4b3d-441f-8fdf-c6cf53f3c559","type":"GlyphRenderer"},{"attributes":{"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"4be62ce2-f149-412b-bc4e-6b5e0ae225a0","type":"HelpTool"},{"attributes":{"data_source":{"id":"96d672ed-4122-44f2-a5cd-f5f956f24473","type":"ColumnDataSource"},"glyph":{"id":"3ba11558-a964-43f4-91d7-fa1d5710e447","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"73a4fa3f-a4c2-4bd9-ba0d-14a9adcaddaf","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1082c6c2-31a3-4192-bc6a-edae5facf627","type":"Rect"},{"attributes":{"data_source":{"id":"cf163660-3519-44c7-ac8f-ca3ba86ac331","type":"ColumnDataSource"},"glyph":{"id":"cf372661-e5d1-4293-9c95-692f42f15c90","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"0f78c90c-cfb6-4340-a646-246a190f545c","type":"GlyphRenderer"},{"attributes":{"overlay":{"id":"2f559d90-5e98-4dd7-921f-bce946adba50","type":"BoxAnnotation"},"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"b17bd746-1bd6-45d8-ad6c-7b5aa8b68d9e","type":"BoxZoomTool"},{"attributes":{"data_source":{"id":"a7e1681e-36df-471b-9f09-0d124d23bb42","type":"ColumnDataSource"},"glyph":{"id":"a5dc2305-5931-4b1f-b3ec-4ca9c8d93a70","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a721bd6a-2bda-4ba5-a0de-95c3c3dd6852","type":"GlyphRenderer"},{"attributes":{"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"a9ec0b35-7362-4216-a01f-dad130a69c3f","type":"WheelZoomTool"},{"attributes":{"plot":null,"text":"Quantidade de Leis por sub-categoria"},"id":"f7b3f4f4-3fa2-46e2-82b5-18065033b470","type":"Title"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"a7e1681e-36df-471b-9f09-0d124d23bb42","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"0e95c67a-9834-47a3-b90e-dd0a6ef51e51","type":"PanTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"57a9691f-76f6-4ed9-9333-14cf3aee78f7","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a5dc2305-5931-4b1f-b3ec-4ca9c8d93a70","type":"Rect"},{"attributes":{"data_source":{"id":"e39b6395-18c1-4dd8-a4ff-e79c7589729d","type":"ColumnDataSource"},"glyph":{"id":"d990f35a-510c-479c-a2e0-8f7cbcb11b11","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"74c3f02b-a538-4bc1-a92d-d334caee25fe","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[78.0],"label":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["autoriza poder executivo"],"y":[39.0]}},"id":"909efea6-40fd-482e-9adf-7f2a899456f0","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d990f35a-510c-479c-a2e0-8f7cbcb11b11","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"cr\u00e9dito"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[15.0],"label":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["autoriza poder executivo"],"y":[85.5]}},"id":"f9ee25c9-d8f6-4aed-9b3c-1ad6aaff322e","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["diretrizes"],"width":[0.8],"x":["disp\u00f5e"],"y":[2.5]}},"id":"96d672ed-4122-44f2-a5cd-f5f956f24473","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"4fd5bb8b-5b85-4805-9517-8d90d2c0ff42","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1e12711a-e12c-4467-9185-a3e6baf1bf11","type":"Rect"},{"attributes":{},"id":"04960c8d-30c9-4f48-b5e1-50cbcd41a31a","type":"CategoricalTicker"},{"attributes":{"data_source":{"id":"909efea6-40fd-482e-9adf-7f2a899456f0","type":"ColumnDataSource"},"glyph":{"id":"1082c6c2-31a3-4192-bc6a-edae5facf627","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"abafeda2-e1de-46cf-b38a-fbdff38b3821","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"01b8c7ea-7c4c-4dab-aef3-6e5f364fab3b","type":"ColumnDataSource"},"glyph":{"id":"ef7b634e-0219-4599-9547-a30002dc8220","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"1fe6004f-8ec1-4ea2-a23a-2be08157d771","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[22.0],"label":[{"categoria_geral":"cria","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["cria"],"y":[36.0]}},"id":"7c7bdc77-f2ec-477f-9866-5fb8b835cf3d","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3ba11558-a964-43f4-91d7-fa1d5710e447","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"secretaria"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"cria","sub_categoria":"secretaria"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["secretaria"],"width":[0.8],"x":["cria"],"y":[58.0]}},"id":"01b8c7ea-7c4c-4dab-aef3-6e5f364fab3b","type":"ColumnDataSource"},{"attributes":{},"id":"d120195c-ffed-46eb-bd64-e880171f2252","type":"BasicTickFormatter"},{"attributes":{"data_source":{"id":"816b864c-cf3f-4144-8538-944ad8ed56ab","type":"ColumnDataSource"},"glyph":{"id":"1e12711a-e12c-4467-9185-a3e6baf1bf11","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"eb060eed-3b80-432c-94e0-42e2743540f3","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"conselho"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"cria","sub_categoria":"conselho"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["conselho"],"width":[0.8],"x":["cria"],"y":[51.5]}},"id":"5120ba6b-a7ef-4465-be1b-76642e891843","type":"ColumnDataSource"},{"attributes":{"label":{"value":"cargos"},"renderers":[{"id":"5bb468e9-4793-4c66-86eb-e77a96c4f4df","type":"GlyphRenderer"}]},"id":"04642c72-2652-48a0-81ae-081f4855df3f","type":"LegendItem"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"2f559d90-5e98-4dd7-921f-bce946adba50","type":"BoxAnnotation"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"67b674c1-0a4b-4fc0-a6f3-aeed3a64ec47","type":"Rect"},{"attributes":{"data_source":{"id":"f9ee25c9-d8f6-4aed-9b3c-1ad6aaff322e","type":"ColumnDataSource"},"glyph":{"id":"67b674c1-0a4b-4fc0-a6f3-aeed3a64ec47","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"5755a838-3bd9-4afa-8d1e-3a7a2d75465f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3c7466ab-d3d4-429c-9865-fd07c8ec3f43","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"f1d006f0-de55-4d5d-bea8-69207c6bc553","type":"ColumnDataSource"},{"attributes":{},"id":"1bc605f4-e201-4e43-89b6-bc6cf0fa4a66","type":"ToolEvents"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"cf163660-3519-44c7-ac8f-ca3ba86ac331","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"1b434025-0e88-4bce-a3e0-677977361518","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"7c7bdc77-f2ec-477f-9866-5fb8b835cf3d","type":"ColumnDataSource"},"glyph":{"id":"c07228b7-4a49-460b-98a9-4c6c6d092727","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4242d5ea-d799-4a01-b8b1-bdce2abd60ef","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"be6b34ca-51e0-485e-8917-f67a5c467a2d","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"25ec65db-d68b-4728-8a3b-d1ebd065a836","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"242681ca-de18-43cd-bc1c-1da5f29c7255","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"94234902-cbf9-4933-a1cb-a5cdc95c4544","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c07228b7-4a49-460b-98a9-4c6c6d092727","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"f222e8f4-a1fd-472e-af81-6d7a9fac22f3","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ef7b634e-0219-4599-9547-a30002dc8220","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"cf372661-e5d1-4293-9c95-692f42f15c90","type":"Rect"},{"attributes":{"data_source":{"id":"f1d006f0-de55-4d5d-bea8-69207c6bc553","type":"ColumnDataSource"},"glyph":{"id":"3c7466ab-d3d4-429c-9865-fd07c8ec3f43","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"65eb021e-f35d-43e7-a73f-5cc56eb03a62","type":"GlyphRenderer"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"d120195c-ffed-46eb-bd64-e880171f2252","type":"BasicTickFormatter"},"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"},"ticker":{"id":"41a0318a-a563-4f67-b45b-79ea122d0c65","type":"BasicTicker"}},"id":"4b1cbe08-6571-4bac-9ff8-37999edd1b82","type":"LinearAxis"},{"attributes":{"items":[{"id":"5069febf-6d1c-49bf-a3a2-01a9223e00fc","type":"LegendItem"},{"id":"04642c72-2652-48a0-81ae-081f4855df3f","type":"LegendItem"},{"id":"53029ff0-9742-4018-942b-b92b2c1f2b6a","type":"LegendItem"},{"id":"aef7c7b9-50a3-4076-bb9f-350cf90cc018","type":"LegendItem"},{"id":"bfa8935f-52b7-4830-9625-15e48847f3ea","type":"LegendItem"},{"id":"2891d0f5-152a-480b-9021-0756f17bbdf9","type":"LegendItem"},{"id":"95ab8436-9152-4831-a32e-079dc5bfb77c","type":"LegendItem"},{"id":"f1d89587-a098-4beb-af76-056ce6719204","type":"LegendItem"},{"id":"f27da2c5-55bb-4b84-9783-662558c627fd","type":"LegendItem"},{"id":"63437993-8bc7-49a3-9154-4cbcc5ed05f9","type":"LegendItem"},{"id":"f4ce359d-c754-440d-b5f6-380abac13596","type":"LegendItem"},{"id":"f6b60aa2-c636-4b25-92b9-7475a2604043","type":"LegendItem"},{"id":"44f9b444-f8ae-4520-8e4b-4807ff526de2","type":"LegendItem"},{"id":"6a0d3ff2-cf5f-44da-b025-1453bb7536b2","type":"LegendItem"},{"id":"4c4ee23a-54f5-4ad5-beda-3cce21f0e808","type":"LegendItem"},{"id":"54de99d9-0801-431a-b3d3-1d0e56d3bee5","type":"LegendItem"},{"id":"4ec8fcf0-fa46-4771-83ae-9a608a428ed6","type":"LegendItem"},{"id":"459592dd-607d-4a97-a207-2ea5d98793cf","type":"LegendItem"},{"id":"9849be06-9f91-4dae-abef-f6c66d53e698","type":"LegendItem"},{"id":"27656b8b-7866-4b92-8c0f-53a5177b1be3","type":"LegendItem"},{"id":"65fcb2c5-7c52-40f0-891e-20e2777584c7","type":"LegendItem"}],"location":"top_left","plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"}},"id":"8f4976fc-60b1-456e-8ad6-a9d688273949","type":"Legend"},{"attributes":{},"id":"41a0318a-a563-4f67-b45b-79ea122d0c65","type":"BasicTicker"},{"attributes":{"dimension":1,"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"},"ticker":{"id":"41a0318a-a563-4f67-b45b-79ea122d0c65","type":"BasicTicker"}},"id":"4562106b-0ba8-4cb0-9166-971d841f16f4","type":"Grid"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"0e95c67a-9834-47a3-b90e-dd0a6ef51e51","type":"PanTool"},{"id":"a9ec0b35-7362-4216-a01f-dad130a69c3f","type":"WheelZoomTool"},{"id":"b17bd746-1bd6-45d8-ad6c-7b5aa8b68d9e","type":"BoxZoomTool"},{"id":"9625d82f-36db-4ee2-9e3d-fb3a0e6dfea5","type":"SaveTool"},{"id":"37b70500-cea5-4c6a-a8c0-e5c3eea9db56","type":"ResetTool"},{"id":"4be62ce2-f149-412b-bc4e-6b5e0ae225a0","type":"HelpTool"}]},"id":"952f4ff5-5811-4b5b-a616-ebf3bd24b608","type":"Toolbar"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"cargos"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"cria","sub_categoria":"cargos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cargos"],"width":[0.8],"x":["cria"],"y":[12.5]}},"id":"978b421d-06b2-41d3-b55c-ec667a3515a9","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5d0729c0-f332-431b-a91a-0c97db0d199a","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f603a85d-6621-4e1c-9e2b-287785d32ed1","type":"Rect"},{"attributes":{"data_source":{"id":"4fd5bb8b-5b85-4805-9517-8d90d2c0ff42","type":"ColumnDataSource"},"glyph":{"id":"25ec65db-d68b-4728-8a3b-d1ebd065a836","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"cee5d97c-a409-4a06-9471-1e62346eeaf0","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"15f53844-05a8-4c59-b14c-44991935d169","type":"ColumnDataSource"},"glyph":{"id":"0e0c4fab-8af8-4a69-bf40-f831d5217c39","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a86224d0-b643-4811-9e82-d713ce3bbfb4","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["revis\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[116.0]}},"id":"791837cb-090c-45c0-a82f-0dd5622c07f6","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"03b9b082-9458-4b00-ab49-c6c3bcbc7a10","type":"ColumnDataSource"},"glyph":{"id":"d9315eed-05b8-4fcf-9d64-95f30fb03ed8","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"afebd3e6-6fc3-480c-86b1-8832cc3846af","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"15216eff-f3ca-4cf6-9fef-fe5cb6d004f0","type":"ColumnDataSource"},"glyph":{"id":"c36ef2e0-76a9-4871-8fff-ec8420de17de","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"78a2bd46-6a62-4d72-9962-f6f2370e21d9","type":"GlyphRenderer"},{"attributes":{"callback":null,"factors":["?","altera lei","atribui","autoriza poder executivo","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","institui,","outra","reajusta","revoga"]},"id":"1cdbf9c5-f46c-4d79-9e8c-df5f8c52a732","type":"FactorRange"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[27.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[89.5]}},"id":"369a9fcd-0405-41cf-9312-05a3a09ed971","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[55.0],"label":[{"categoria_geral":"institui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui"],"y":[27.5]}},"id":"18f021ce-fd08-4982-b1d2-b662fe2fd7c9","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c36ef2e0-76a9-4871-8fff-ec8420de17de","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"nome"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"cria","sub_categoria":"nome"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["nome"],"width":[0.8],"x":["cria"],"y":[68.5]}},"id":"15216eff-f3ca-4cf6-9fef-fe5cb6d004f0","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"5120ba6b-a7ef-4465-be1b-76642e891843","type":"ColumnDataSource"},"glyph":{"id":"fff2da13-9f58-4931-9305-55bbd196ea9d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"0aad22ad-a2e6-4c6e-b445-34b38462ccf8","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"9f0b12cf-ede5-492b-b2ed-9859631402b0","type":"ColumnDataSource"},"glyph":{"id":"d39262c6-e209-4587-82e4-5f2c670d2ec7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4254f2e7-d8d1-4a95-b339-df8ba8f03bc4","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"fdaa3ea4-25f6-4838-aab8-f24b757789c7","type":"ColumnDataSource"},"glyph":{"id":"f6f20d73-1998-4288-9418-9b992ef12e66","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"80c09d79-95cc-4733-b822-6e4444e9ce8f","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fff2da13-9f58-4931-9305-55bbd196ea9d","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"669ed6a8-8116-44fc-9027-1f0f895c307c","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d39262c6-e209-4587-82e4-5f2c670d2ec7","type":"Rect"},{"attributes":{"data_source":{"id":"1b434025-0e88-4bce-a3e0-677977361518","type":"ColumnDataSource"},"glyph":{"id":"94234902-cbf9-4933-a1cb-a5cdc95c4544","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e74639c2-bf46-478a-861f-93a905b461de","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"249bfb40-e1b5-4b96-8656-3d581426d3d3","type":"ColumnDataSource"},"glyph":{"id":"669ed6a8-8116-44fc-9027-1f0f895c307c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"14b74d84-b472-4c36-9ae4-d83d6030e6f8","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"7d2e5cdb-b7f8-43d6-97ee-c2b0c4adb011","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c1ac3560-e87f-44d1-b918-8299b194a961","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reajuste"],"width":[0.8],"x":["disp\u00f5e"],"y":[136.0]}},"id":"e2edfa0c-b733-4c38-a27c-e76db644a5d3","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["aumento"],"width":[0.8],"x":["disp\u00f5e"],"y":[132.5]}},"id":"249bfb40-e1b5-4b96-8656-3d581426d3d3","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"bbe3a4ca-6d5e-478a-b378-dff9f279868f","type":"ColumnDataSource"},"glyph":{"id":"7d2e5cdb-b7f8-43d6-97ee-c2b0c4adb011","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4c6131a9-806d-41f8-9a98-4be7e0aa58a7","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"a707721b-95a3-467a-bb73-d8658ffa1bdd","type":"ColumnDataSource"},"glyph":{"id":"c1ac3560-e87f-44d1-b918-8299b194a961","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ac3a551f-533a-461f-8153-c5bcf18c341f","type":"GlyphRenderer"},{"attributes":{"label":{"value":"plano"},"renderers":[{"id":"bc5729a9-0d08-4fa8-849c-89a71004a860","type":"GlyphRenderer"}]},"id":"f27da2c5-55bb-4b84-9783-662558c627fd","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["estrutura"],"width":[0.8],"x":["disp\u00f5e"],"y":[128.0]}},"id":"1edc2454-4ffd-4e04-8e67-78af072b143f","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"2fefe752-a785-4244-b02d-7bc422976d42","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera lei"],"chart_index":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[116.0],"label":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["-"],"width":[0.8],"x":["altera lei"],"y":[58.0]}},"id":"fdaa3ea4-25f6-4838-aab8-f24b757789c7","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"e2edfa0c-b733-4c38-a27c-e76db644a5d3","type":"ColumnDataSource"},"glyph":{"id":"2fefe752-a785-4244-b02d-7bc422976d42","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b20e8e73-846c-49ec-ac5f-15308900fce7","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"dc422a62-c05f-46c1-bdbc-5293a92c31f2","type":"Rect"},{"attributes":{"label":{"value":"diretrizes"},"renderers":[{"id":"73a4fa3f-a4c2-4bd9-ba0d-14a9adcaddaf","type":"GlyphRenderer"}]},"id":"53029ff0-9742-4018-942b-b92b2c1f2b6a","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"plano"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["outra"],"y":[84.5]}},"id":"03b9b082-9458-4b00-ab49-c6c3bcbc7a10","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"18f021ce-fd08-4982-b1d2-b662fe2fd7c9","type":"ColumnDataSource"},"glyph":{"id":"a45307b7-8e24-4590-a015-151d329c9a7d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"54b1ac68-b22f-4c60-965b-5df0f4d11341","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"237416db-aff0-40ba-8d31-6ee4acf19420","type":"ColumnDataSource"},"glyph":{"id":"dc422a62-c05f-46c1-bdbc-5293a92c31f2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c48e1684-8028-4dbf-8404-d7654239025f","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["disp\u00f5e"],"y":[122.5]}},"id":"bf4df790-c84a-4578-b564-bca14cfdd4a7","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["disp\u00f5e"],"y":[71.0]}},"id":"ac1b9eba-8691-40f8-b394-499de1039424","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"03cfda26-61c5-449d-996b-cfa553a323fe","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a45307b7-8e24-4590-a015-151d329c9a7d","type":"Rect"},{"attributes":{"label":{"value":"-"},"renderers":[{"id":"80c09d79-95cc-4733-b822-6e4444e9ce8f","type":"GlyphRenderer"}]},"id":"95ab8436-9152-4831-a32e-079dc5bfb77c","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["abertura"],"width":[0.8],"x":["disp\u00f5e"],"y":[107.5]}},"id":"b17414e3-e217-48f5-bc4b-769a96723d5b","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"4d9fbeb1-e2d2-47e1-9415-116300e3d688","type":"ColumnDataSource"},"glyph":{"id":"03cfda26-61c5-449d-996b-cfa553a323fe","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"87182a03-9941-4245-b66d-453e0d4bdac8","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["contrata\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[64.5]}},"id":"005b8559-6d07-430b-93d5-9534427f03c4","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["outra"],"y":[83.5]}},"id":"15f53844-05a8-4c59-b14c-44991935d169","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f6f20d73-1998-4288-9418-9b992ef12e66","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["estabelece"],"y":[27.0]}},"id":"a707721b-95a3-467a-bb73-d8658ffa1bdd","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disp\u00f5e"],"y":[34.0]}},"id":"bbe3a4ca-6d5e-478a-b378-dff9f279868f","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estabelece"],"y":[12.5]}},"id":"84a560f4-142b-4310-8ebe-82dfee295d13","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reestrutura\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[139.5]}},"id":"4d9fbeb1-e2d2-47e1-9415-116300e3d688","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0b536a80-b89a-4be3-adc3-35edfdd7b936","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8b490f85-300e-4d59-a418-84a2d08e5028","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["institui\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[144.5]}},"id":"0971e812-f9d1-4be5-ad7f-71acdbb53391","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui,"],"chart_index":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"institui,","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui,"],"y":[3.0]}},"id":"d3b5ceae-7438-4d79-b7b5-feb195b220c5","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["altera\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[157.5]}},"id":"9f0b12cf-ede5-492b-b2ed-9859631402b0","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["disp\u00f5e"],"y":[151.0]}},"id":"436e4834-b6a9-4fda-991a-ce706b7d640e","type":"ColumnDataSource"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"4084f236-81c1-419c-907e-c8264bdd7d58","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"},"ticker":{"id":"04960c8d-30c9-4f48-b5e1-50cbcd41a31a","type":"CategoricalTicker"}},"id":"37efb891-a110-4697-8df1-e2103ea6123d","type":"CategoricalAxis"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["denomina\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[163.5]}},"id":"237416db-aff0-40ba-8d31-6ee4acf19420","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0e0c4fab-8af8-4a69-bf40-f831d5217c39","type":"Rect"},{"attributes":{"label":{"value":"cr\u00e9dito"},"renderers":[{"id":"5755a838-3bd9-4afa-8d1e-3a7a2d75465f","type":"GlyphRenderer"}]},"id":"aef7c7b9-50a3-4076-bb9f-350cf90cc018","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"50c472c4-38bd-4fb5-ac9e-a378b4dbba17","type":"Rect"},{"attributes":{"label":{"value":"conselho"},"renderers":[{"id":"0aad22ad-a2e6-4c6e-b445-34b38462ccf8","type":"GlyphRenderer"}]},"id":"bfa8935f-52b7-4830-9625-15e48847f3ea","type":"LegendItem"},{"attributes":{"label":{"value":"secretaria"},"renderers":[{"id":"1fe6004f-8ec1-4ea2-a23a-2be08157d771","type":"GlyphRenderer"}]},"id":"2891d0f5-152a-480b-9021-0756f17bbdf9","type":"LegendItem"},{"attributes":{"label":{"value":"abertura"},"renderers":[{"id":"b8385752-1386-4e9e-a873-f79aa88caf2c","type":"GlyphRenderer"}]},"id":"f4ce359d-c754-440d-b5f6-380abac13596","type":"LegendItem"},{"attributes":{"label":{"value":"contrata\u00e7\u00e3o"},"renderers":[{"id":"ebed8508-fd8a-4e5e-bac6-c7cbf5404700","type":"GlyphRenderer"}]},"id":"f1d89587-a098-4beb-af76-056ce6719204","type":"LegendItem"},{"attributes":{"data_source":{"id":"005b8559-6d07-430b-93d5-9534427f03c4","type":"ColumnDataSource"},"glyph":{"id":"50c472c4-38bd-4fb5-ac9e-a378b4dbba17","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ebed8508-fd8a-4e5e-bac6-c7cbf5404700","type":"GlyphRenderer"},{"attributes":{"label":{"value":"revis\u00e3o"},"renderers":[{"id":"1b96fbb7-802c-4c86-9cbd-0263ce3de745","type":"GlyphRenderer"}]},"id":"f6b60aa2-c636-4b25-92b9-7475a2604043","type":"LegendItem"},{"attributes":{"data_source":{"id":"436e4834-b6a9-4fda-991a-ce706b7d640e","type":"ColumnDataSource"},"glyph":{"id":"f603a85d-6621-4e1c-9e2b-287785d32ed1","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"61750228-dba2-4921-818d-202e064ee341","type":"GlyphRenderer"},{"attributes":{"label":{"value":"cria\u00e7\u00e3o"},"renderers":[{"id":"57fdbbf9-de9d-43cb-9025-6581a511477b","type":"GlyphRenderer"}]},"id":"63437993-8bc7-49a3-9154-4cbcc5ed05f9","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d9315eed-05b8-4fcf-9d64-95f30fb03ed8","type":"Rect"},{"attributes":{"label":{"value":"estrutura"},"renderers":[{"id":"23b912b6-7ddf-4ab5-8a3a-7cf1c45970bd","type":"GlyphRenderer"}]},"id":"44f9b444-f8ae-4520-8e4b-4807ff526de2","type":"LegendItem"},{"attributes":{"label":{"value":"nome"},"renderers":[{"id":"78a2bd46-6a62-4d72-9962-f6f2370e21d9","type":"GlyphRenderer"}]},"id":"6a0d3ff2-cf5f-44da-b025-1453bb7536b2","type":"LegendItem"},{"attributes":{"label":{"value":"aumento"},"renderers":[{"id":"14b74d84-b472-4c36-9ae4-d83d6030e6f8","type":"GlyphRenderer"}]},"id":"4c4ee23a-54f5-4ad5-beda-3cce21f0e808","type":"LegendItem"},{"attributes":{"label":{"value":"reajuste"},"renderers":[{"id":"b20e8e73-846c-49ec-ac5f-15308900fce7","type":"GlyphRenderer"}]},"id":"54de99d9-0801-431a-b3d3-1d0e56d3bee5","type":"LegendItem"},{"attributes":{"callback":null,"end":174.3},"id":"70006074-281e-4785-8c40-adb5df25f409","type":"Range1d"},{"attributes":{"label":{"value":"reestrutura\u00e7\u00e3o"},"renderers":[{"id":"87182a03-9941-4245-b66d-453e0d4bdac8","type":"GlyphRenderer"}]},"id":"4ec8fcf0-fa46-4771-83ae-9a608a428ed6","type":"LegendItem"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"a881af44-4b3d-441f-8fdf-c6cf53f3c559","type":"GlyphRenderer"}]},"id":"5069febf-6d1c-49bf-a3a2-01a9223e00fc","type":"LegendItem"},{"attributes":{"label":{"value":"institui\u00e7\u00e3o"},"renderers":[{"id":"21fc5503-0627-4cb9-94fd-380ace74e450","type":"GlyphRenderer"}]},"id":"459592dd-607d-4a97-a207-2ea5d98793cf","type":"LegendItem"},{"attributes":{"data_source":{"id":"d3b5ceae-7438-4d79-b7b5-feb195b220c5","type":"ColumnDataSource"},"glyph":{"id":"0b536a80-b89a-4be3-adc3-35edfdd7b936","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3697176d-8baf-4835-82e1-7193154a3874","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ffa43d54-cd8d-4b20-aab3-4502c4dc0034","type":"Rect"},{"attributes":{"label":{"value":"vencimentos"},"renderers":[{"id":"61750228-dba2-4921-818d-202e064ee341","type":"GlyphRenderer"}]},"id":"9849be06-9f91-4dae-abef-f6c66d53e698","type":"LegendItem"},{"attributes":{"data_source":{"id":"242681ca-de18-43cd-bc1c-1da5f29c7255","type":"ColumnDataSource"},"glyph":{"id":"457f98a0-a5cf-4cfe-94a8-442f6a7f8f79","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"98f7976f-29af-41c4-8d68-dbfdce7efef8","type":"GlyphRenderer"},{"attributes":{"label":{"value":"altera\u00e7\u00e3o"},"renderers":[{"id":"4254f2e7-d8d1-4a95-b339-df8ba8f03bc4","type":"GlyphRenderer"}]},"id":"27656b8b-7866-4b92-8c0f-53a5177b1be3","type":"LegendItem"},{"attributes":{"data_source":{"id":"84a560f4-142b-4310-8ebe-82dfee295d13","type":"ColumnDataSource"},"glyph":{"id":"ffa43d54-cd8d-4b20-aab3-4502c4dc0034","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"aa00892d-de5d-4984-a055-e7f703d8f7df","type":"GlyphRenderer"},{"attributes":{"label":{"value":"denomina\u00e7\u00e3o"},"renderers":[{"id":"c48e1684-8028-4dbf-8404-d7654239025f","type":"GlyphRenderer"}]},"id":"65fcb2c5-7c52-40f0-891e-20e2777584c7","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fc701cb7-5d8f-42d6-847a-ac239056db83","type":"Rect"},{"attributes":{"data_source":{"id":"d2ecec61-0278-4cda-bee9-015e1e87c95c","type":"ColumnDataSource"},"glyph":{"id":"8b490f85-300e-4d59-a418-84a2d08e5028","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b858704d-386d-444e-8a71-2ce3981c7ad4","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"d2ecec61-0278-4cda-bee9-015e1e87c95c","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"b17414e3-e217-48f5-bc4b-769a96723d5b","type":"ColumnDataSource"},"glyph":{"id":"2d77408a-d945-4041-b906-28cb5a919bec","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b8385752-1386-4e9e-a873-f79aa88caf2c","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"816b864c-cf3f-4144-8538-944ad8ed56ab","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"ac1b9eba-8691-40f8-b394-499de1039424","type":"ColumnDataSource"},"glyph":{"id":"bdb74a15-9fb3-455d-9c40-6de45ee1bb54","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bc5729a9-0d08-4fa8-849c-89a71004a860","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"bdb74a15-9fb3-455d-9c40-6de45ee1bb54","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"9c40d9a0-ab68-40ea-9320-1d72cb34ff72","type":"Rect"},{"attributes":{"data_source":{"id":"e09c14dd-4a98-41ea-90e1-6f49a097b50e","type":"ColumnDataSource"},"glyph":{"id":"48910f5d-5c56-4ed9-8c5f-b5b50e4853af","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"7af92d8b-69e2-4ced-a210-ac5cc31a8a79","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"369a9fcd-0405-41cf-9312-05a3a09ed971","type":"ColumnDataSource"},"glyph":{"id":"9c40d9a0-ab68-40ea-9320-1d72cb34ff72","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"57fdbbf9-de9d-43cb-9025-6581a511477b","type":"GlyphRenderer"},{"attributes":{"below":[{"id":"37efb891-a110-4697-8df1-e2103ea6123d","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"4b1cbe08-6571-4bac-9ff8-37999edd1b82","type":"LinearAxis"}],"renderers":[{"id":"2f559d90-5e98-4dd7-921f-bce946adba50","type":"BoxAnnotation"},{"id":"a881af44-4b3d-441f-8fdf-c6cf53f3c559","type":"GlyphRenderer"},{"id":"abafeda2-e1de-46cf-b38a-fbdff38b3821","type":"GlyphRenderer"},{"id":"3690dc29-4de5-436f-88f6-39af04e203af","type":"GlyphRenderer"},{"id":"5bb468e9-4793-4c66-86eb-e77a96c4f4df","type":"GlyphRenderer"},{"id":"a721bd6a-2bda-4ba5-a0de-95c3c3dd6852","type":"GlyphRenderer"},{"id":"74c3f02b-a538-4bc1-a92d-d334caee25fe","type":"GlyphRenderer"},{"id":"73a4fa3f-a4c2-4bd9-ba0d-14a9adcaddaf","type":"GlyphRenderer"},{"id":"5755a838-3bd9-4afa-8d1e-3a7a2d75465f","type":"GlyphRenderer"},{"id":"0f78c90c-cfb6-4340-a646-246a190f545c","type":"GlyphRenderer"},{"id":"4242d5ea-d799-4a01-b8b1-bdce2abd60ef","type":"GlyphRenderer"},{"id":"65eb021e-f35d-43e7-a73f-5cc56eb03a62","type":"GlyphRenderer"},{"id":"cee5d97c-a409-4a06-9471-1e62346eeaf0","type":"GlyphRenderer"},{"id":"0aad22ad-a2e6-4c6e-b445-34b38462ccf8","type":"GlyphRenderer"},{"id":"4c6131a9-806d-41f8-9a98-4be7e0aa58a7","type":"GlyphRenderer"},{"id":"1fe6004f-8ec1-4ea2-a23a-2be08157d771","type":"GlyphRenderer"},{"id":"eb060eed-3b80-432c-94e0-42e2743540f3","type":"GlyphRenderer"},{"id":"e74639c2-bf46-478a-861f-93a905b461de","type":"GlyphRenderer"},{"id":"54b1ac68-b22f-4c60-965b-5df0f4d11341","type":"GlyphRenderer"},{"id":"80c09d79-95cc-4733-b822-6e4444e9ce8f","type":"GlyphRenderer"},{"id":"ebed8508-fd8a-4e5e-bac6-c7cbf5404700","type":"GlyphRenderer"},{"id":"aa00892d-de5d-4984-a055-e7f703d8f7df","type":"GlyphRenderer"},{"id":"98f7976f-29af-41c4-8d68-dbfdce7efef8","type":"GlyphRenderer"},{"id":"b858704d-386d-444e-8a71-2ce3981c7ad4","type":"GlyphRenderer"},{"id":"bc5729a9-0d08-4fa8-849c-89a71004a860","type":"GlyphRenderer"},{"id":"57fdbbf9-de9d-43cb-9025-6581a511477b","type":"GlyphRenderer"},{"id":"b8385752-1386-4e9e-a873-f79aa88caf2c","type":"GlyphRenderer"},{"id":"1b96fbb7-802c-4c86-9cbd-0263ce3de745","type":"GlyphRenderer"},{"id":"cdb2b59c-4e38-4f45-8be4-a8e14af18764","type":"GlyphRenderer"},{"id":"23b912b6-7ddf-4ab5-8a3a-7cf1c45970bd","type":"GlyphRenderer"},{"id":"7af92d8b-69e2-4ced-a210-ac5cc31a8a79","type":"GlyphRenderer"},{"id":"78a2bd46-6a62-4d72-9962-f6f2370e21d9","type":"GlyphRenderer"},{"id":"14b74d84-b472-4c36-9ae4-d83d6030e6f8","type":"GlyphRenderer"},{"id":"b20e8e73-846c-49ec-ac5f-15308900fce7","type":"GlyphRenderer"},{"id":"87182a03-9941-4245-b66d-453e0d4bdac8","type":"GlyphRenderer"},{"id":"21fc5503-0627-4cb9-94fd-380ace74e450","type":"GlyphRenderer"},{"id":"61750228-dba2-4921-818d-202e064ee341","type":"GlyphRenderer"},{"id":"3697176d-8baf-4835-82e1-7193154a3874","type":"GlyphRenderer"},{"id":"a86224d0-b643-4811-9e82-d713ce3bbfb4","type":"GlyphRenderer"},{"id":"4254f2e7-d8d1-4a95-b339-df8ba8f03bc4","type":"GlyphRenderer"},{"id":"ac3a551f-533a-461f-8153-c5bcf18c341f","type":"GlyphRenderer"},{"id":"c48e1684-8028-4dbf-8404-d7654239025f","type":"GlyphRenderer"},{"id":"afebd3e6-6fc3-480c-86b1-8832cc3846af","type":"GlyphRenderer"},{"id":"8f4976fc-60b1-456e-8ad6-a9d688273949","type":"Legend"},{"id":"37efb891-a110-4697-8df1-e2103ea6123d","type":"CategoricalAxis"},{"id":"4b1cbe08-6571-4bac-9ff8-37999edd1b82","type":"LinearAxis"},{"id":"4562106b-0ba8-4cb0-9166-971d841f16f4","type":"Grid"}],"title":{"id":"f7b3f4f4-3fa2-46e2-82b5-18065033b470","type":"Title"},"tool_events":{"id":"1bc605f4-e201-4e43-89b6-bc6cf0fa4a66","type":"ToolEvents"},"toolbar":{"id":"952f4ff5-5811-4b5b-a616-ebf3bd24b608","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"1cdbf9c5-f46c-4d79-9e8c-df5f8c52a732","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"70006074-281e-4785-8c40-adb5df25f409","type":"Range1d"}},"id":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20","subtype":"Chart","type":"Plot"},{"attributes":{"data_source":{"id":"1edc2454-4ffd-4e04-8e67-78af072b143f","type":"ColumnDataSource"},"glyph":{"id":"1b2ffeaf-3640-49e8-91c5-7968d2a119a0","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"23b912b6-7ddf-4ab5-8a3a-7cf1c45970bd","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1b2ffeaf-3640-49e8-91c5-7968d2a119a0","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"2d77408a-d945-4041-b906-28cb5a919bec","type":"Rect"},{"attributes":{"data_source":{"id":"0971e812-f9d1-4be5-ad7f-71acdbb53391","type":"ColumnDataSource"},"glyph":{"id":"fc701cb7-5d8f-42d6-847a-ac239056db83","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"21fc5503-0627-4cb9-94fd-380ace74e450","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"26788d86-a871-442c-a1b9-02febfe977a9","type":"Rect"},{"attributes":{"data_source":{"id":"b10d4da6-8292-4826-97a6-d7a73061c116","type":"ColumnDataSource"},"glyph":{"id":"be6b34ca-51e0-485e-8917-f67a5c467a2d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3690dc29-4de5-436f-88f6-39af04e203af","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"791837cb-090c-45c0-a82f-0dd5622c07f6","type":"ColumnDataSource"},"glyph":{"id":"64315f55-01df-4fd6-bd5c-a0db6d1c943d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"1b96fbb7-802c-4c86-9cbd-0263ce3de745","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"64315f55-01df-4fd6-bd5c-a0db6d1c943d","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"e09c14dd-4a98-41ea-90e1-6f49a097b50e","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"b10d4da6-8292-4826-97a6-d7a73061c116","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"978b421d-06b2-41d3-b55c-ec667a3515a9","type":"ColumnDataSource"},"glyph":{"id":"5d0729c0-f332-431b-a91a-0c97db0d199a","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"5bb468e9-4793-4c66-86eb-e77a96c4f4df","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"48910f5d-5c56-4ed9-8c5f-b5b50e4853af","type":"Rect"},{"attributes":{"data_source":{"id":"bf4df790-c84a-4578-b564-bca14cfdd4a7","type":"ColumnDataSource"},"glyph":{"id":"26788d86-a871-442c-a1b9-02febfe977a9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"cdb2b59c-4e38-4f45-8be4-a8e14af18764","type":"GlyphRenderer"},{"attributes":{},"id":"4084f236-81c1-419c-907e-c8264bdd7d58","type":"CategoricalTickFormatter"}],"root_ids":["cb0c3b7a-b552-446b-a8fe-88ed4f709a20"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"b7c8f4a7-0a5c-452b-ba79-b5da70100b4e","elementid":"fa2541dc-bf1f-459b-92fd-c15087461a39","modelid":"cb0c3b7a-b552-446b-a8fe-88ed4f709a20"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("fa2541dc-bf1f-459b-92fd-c15087461a39")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'institui'
Primeiro vamos combinar 'institui,' e 'institui'


```python
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('institui,', row['categoria_geral'],'institui'), axis=1)
```


```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por sub-categoria", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="4a5d7422-c2fb-4f79-945b-ca593d15682f"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("4a5d7422-c2fb-4f79-945b-ca593d15682f").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("4a5d7422-c2fb-4f79-945b-ca593d15682f");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid '4a5d7422-c2fb-4f79-945b-ca593d15682f' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"e664dacd-e503-4385-ad00-132252083e34":{"roots":{"references":[{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"aumento"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["aumento"],"width":[0.8],"x":["disp\u00f5e"],"y":[132.5]}},"id":"c334a68b-6f04-4772-826c-4867e007df46","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"plano"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["outra"],"y":[84.5]}},"id":"e6bcc715-ed02-4da7-b8d4-f7793275b635","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"2cec7b06-1fea-4e77-8dfc-8531a3367c84","type":"ColumnDataSource"},"glyph":{"id":"354eb425-80f2-41ed-99ef-84d9f6f0f9e7","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8d7bb156-5ce5-414b-808a-806cce08c7aa","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"be2d4d67-6af6-445f-b3a5-249bf3a2f3ad","type":"ColumnDataSource"},"glyph":{"id":"164b292f-0ea6-4a79-a260-eb3daafe86e9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"63950d7c-f087-40cd-b032-ad4e1c5c0621","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"secretaria"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"cria","sub_categoria":"secretaria"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["secretaria"],"width":[0.8],"x":["cria"],"y":[58.0]}},"id":"e0223ee5-4a11-438b-9f94-b52d658c3361","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"088e9ed2-0e41-4d91-b48c-e11652a2f335","type":"ColumnDataSource"},"glyph":{"id":"cf764e94-280c-480c-8565-5b9371fab7bd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"cd1fd160-8140-42e3-a5a5-59f3d93a74ec","type":"GlyphRenderer"},{"attributes":{},"id":"67c417b3-09db-45a5-8d17-bd97636e8a10","type":"ToolEvents"},{"attributes":{"data_source":{"id":"64ff969d-27a9-4b31-b8c6-133829ef14c9","type":"ColumnDataSource"},"glyph":{"id":"5dd76bd2-4b96-4d99-afb3-bfb4e99c3977","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8e61635f-0248-4839-9e04-5fd98d9e32c5","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"c70a9df7-16d6-45a1-8fa5-633a8c651428","type":"ColumnDataSource"},"glyph":{"id":"6b80a024-13c9-4424-80fb-4b4137cf1d13","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8f1d0590-0600-4180-9a5f-0540acd1a5fc","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ceb3bc42-c5b2-4756-9613-6db449d3503a","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"cf764e94-280c-480c-8565-5b9371fab7bd","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"0dc95556-e7bb-4bcc-9a80-6c1e05c4dbac","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["disp\u00f5e"],"y":[122.5]}},"id":"9b42a72d-7909-470d-9612-729e276e4113","type":"ColumnDataSource"},{"attributes":{"label":{"value":"conselho"},"renderers":[{"id":"f6eb70e7-b2bd-409b-8a8d-c8502fd0a465","type":"GlyphRenderer"}]},"id":"a1b43be9-3628-4e3c-b54c-7affd0b2e920","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6b80a024-13c9-4424-80fb-4b4137cf1d13","type":"Rect"},{"attributes":{"label":{"value":"cr\u00e9dito"},"renderers":[{"id":"314ab9ff-5dca-4224-b5c1-db5339237f63","type":"GlyphRenderer"}]},"id":"ae2d64ed-f1f6-4421-8f89-3510287fe100","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4615a229-9e70-4d2e-801a-0233883284f6","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"14fdbff7-ed7c-4e35-98d8-f37f579a17bc","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"plano"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["plano"],"width":[0.8],"x":["disp\u00f5e"],"y":[71.0]}},"id":"0fc552ee-79c5-4d3d-bc71-ba7690bc370c","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[8.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"revis\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["revis\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[116.0]}},"id":"a3c6b57f-39ab-45e3-ac54-a26bf9eb5c00","type":"ColumnDataSource"},{"attributes":{"label":{"value":"aumento"},"renderers":[{"id":"92046d1d-ca68-4207-82df-f110894ef342","type":"GlyphRenderer"}]},"id":"8d852e74-bdbb-4396-8a09-97405a25317b","type":"LegendItem"},{"attributes":{"label":{"value":"altera\u00e7\u00e3o"},"renderers":[{"id":"63950d7c-f087-40cd-b032-ad4e1c5c0621","type":"GlyphRenderer"}]},"id":"3d0c6d48-11d8-4284-9509-2a547a4e17d6","type":"LegendItem"},{"attributes":{"data_source":{"id":"c334a68b-6f04-4772-826c-4867e007df46","type":"ColumnDataSource"},"glyph":{"id":"14fdbff7-ed7c-4e35-98d8-f37f579a17bc","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"92046d1d-ca68-4207-82df-f110894ef342","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"nome"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"cria","sub_categoria":"nome"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["nome"],"width":[0.8],"x":["cria"],"y":[68.5]}},"id":"088e9ed2-0e41-4d91-b48c-e11652a2f335","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a95550d5-2f58-4967-88b6-a20d2ff3a960","type":"Rect"},{"attributes":{"data_source":{"id":"7cae4de9-0a40-4b79-90fc-922cda7cbe5e","type":"ColumnDataSource"},"glyph":{"id":"89bd4caa-9ae1-4a23-8805-afe0c2241d84","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2544a1bd-29f7-49c9-90b5-841df1753fb4","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reajuste"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reajuste"],"width":[0.8],"x":["disp\u00f5e"],"y":[136.0]}},"id":"fe850190-69d6-4510-9cf5-b607a8988ff5","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"estrutura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["estrutura"],"width":[0.8],"x":["disp\u00f5e"],"y":[128.0]}},"id":"bbda7416-d363-4d0e-a126-d01fd9097157","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"e6bcc715-ed02-4da7-b8d4-f7793275b635","type":"ColumnDataSource"},"glyph":{"id":"a95550d5-2f58-4967-88b6-a20d2ff3a960","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"26d88aed-206d-4466-8d94-7292e6d16190","type":"GlyphRenderer"},{"attributes":{"label":{"value":"institui\u00e7\u00e3o"},"renderers":[{"id":"8cb0ed7f-78c2-4c69-8edb-0458e9d81df3","type":"GlyphRenderer"}]},"id":"d48fd320-be1c-4872-be42-90114581f8c8","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"abertura"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["abertura"],"width":[0.8],"x":["disp\u00f5e"],"y":[107.5]}},"id":"28aabb3e-2916-49e6-b906-c22ad39592cd","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0ae1a563-5544-41f1-b1c2-8d9c222b8668","type":"Rect"},{"attributes":{"data_source":{"id":"0dc95556-e7bb-4bcc-9a80-6c1e05c4dbac","type":"ColumnDataSource"},"glyph":{"id":"4615a229-9e70-4d2e-801a-0233883284f6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2805e094-6624-47cd-9b16-39fe2e97e16f","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"contrata\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["contrata\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[64.5]}},"id":"c14a9e27-8cc9-44f5-b96a-bf5bbc9af690","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[27.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[89.5]}},"id":"035af110-408a-4585-8a50-cea40db742b5","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"fe850190-69d6-4510-9cf5-b607a8988ff5","type":"ColumnDataSource"},"glyph":{"id":"0ae1a563-5544-41f1-b1c2-8d9c222b8668","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b98a4a40-5b61-4e6a-9e50-75e0c93907dc","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disp\u00f5e"],"y":[34.0]}},"id":"1650e047-6de7-4fff-8bb5-59714fa6d6fe","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[3.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"reestrutura\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["reestrutura\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[139.5]}},"id":"7cae4de9-0a40-4b79-90fc-922cda7cbe5e","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[1.0],"label":[{"categoria_geral":"outra","sub_categoria":"cria\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cria\u00e7\u00e3o"],"width":[0.8],"x":["outra"],"y":[83.5]}},"id":"de76c399-8d3a-48d7-9b46-a4d2a2abb9c8","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"institui\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["institui\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[144.5]}},"id":"815cdb5e-10c8-4b16-b9e9-6f3fd42bc6c7","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["disp\u00f5e"],"y":[151.0]}},"id":"cb1ba73c-f531-42d4-b980-e1c110c1b128","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"altera\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["altera\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[157.5]}},"id":"be2d4d67-6af6-445f-b3a5-249bf3a2f3ad","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"denomina\u00e7\u00e3o"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["denomina\u00e7\u00e3o"],"width":[0.8],"x":["disp\u00f5e"],"y":[163.5]}},"id":"7905073c-570a-4f2e-b5aa-282eed00ce74","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[4.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"vencimentos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["vencimentos"],"width":[0.8],"x":["estabelece"],"y":[27.0]}},"id":"c70a9df7-16d6-45a1-8fa5-633a8c651428","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5dd76bd2-4b96-4d99-afb3-bfb4e99c3977","type":"Rect"},{"attributes":{},"id":"326374cf-b73b-42ee-9b17-ba21e68e74fa","type":"BasicTicker"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"38b38fe1-9521-44fb-8e4a-14daf760a460","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"},"ticker":{"id":"d52caa49-ddf2-4200-98c5-a5393be12dfa","type":"CategoricalTicker"}},"id":"3d3e329e-eecd-4655-aace-2e7df7dd869f","type":"CategoricalAxis"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"2805e094-6624-47cd-9b16-39fe2e97e16f","type":"GlyphRenderer"}]},"id":"816d2b57-b1bc-4424-a461-9a50556ebc9e","type":"LegendItem"},{"attributes":{"label":{"value":"secretaria"},"renderers":[{"id":"229ee45c-a17f-400e-8b2b-b3beadc07232","type":"GlyphRenderer"}]},"id":"5ff9ed04-1d8a-40b9-a11c-30f7fe787f3d","type":"LegendItem"},{"attributes":{"data_source":{"id":"815cdb5e-10c8-4b16-b9e9-6f3fd42bc6c7","type":"ColumnDataSource"},"glyph":{"id":"269389b0-c32d-42ef-9dd9-db078bf01600","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8cb0ed7f-78c2-4c69-8edb-0458e9d81df3","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"269389b0-c32d-42ef-9dd9-db078bf01600","type":"Rect"},{"attributes":{"label":{"value":"-"},"renderers":[{"id":"fae95767-ddcf-4593-a2a3-26029abc36e8","type":"GlyphRenderer"}]},"id":"5a4be83e-78b6-4bf8-995b-b469683c7949","type":"LegendItem"},{"attributes":{"label":{"value":"contrata\u00e7\u00e3o"},"renderers":[{"id":"f31aa681-1767-4bbb-b5eb-6bd2a66b5f2e","type":"GlyphRenderer"}]},"id":"9567d43c-c844-433d-8a03-2e2b63003fd0","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[5.0],"label":[{"categoria_geral":"disp\u00f5e","sub_categoria":"diretrizes"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["diretrizes"],"width":[0.8],"x":["disp\u00f5e"],"y":[2.5]}},"id":"64ff969d-27a9-4b31-b8c6-133829ef14c9","type":"ColumnDataSource"},{"attributes":{"label":{"value":"plano"},"renderers":[{"id":"197b06be-cd6d-44c3-96e5-b6db6878c04e","type":"GlyphRenderer"}]},"id":"c2795c5d-7ff1-4caa-af05-cae14ea2f0aa","type":"LegendItem"},{"attributes":{"label":{"value":"cria\u00e7\u00e3o"},"renderers":[{"id":"95fd74e9-e21c-48b8-8fba-d7d89174c2cc","type":"GlyphRenderer"}]},"id":"ec96fa4b-a892-4bc7-9845-cb29b6f3cd72","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"efe0ed4a-3708-4149-a441-e2ae829d94c9","type":"Rect"},{"attributes":{"label":{"value":"abertura"},"renderers":[{"id":"beaa78aa-ba87-434b-b3b5-aaba0b6c9ba0","type":"GlyphRenderer"}]},"id":"74d8dc61-8144-495b-a7de-b7b380b895e9","type":"LegendItem"},{"attributes":{"label":{"value":"revis\u00e3o"},"renderers":[{"id":"e1d64865-bf68-4a06-9e26-67106dd51962","type":"GlyphRenderer"}]},"id":"d9d03f75-9ac2-4b5e-a530-4428b34e39e2","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"164b292f-0ea6-4a79-a260-eb3daafe86e9","type":"Rect"},{"attributes":{"label":{"value":"estrutura"},"renderers":[{"id":"e1370b63-71ce-460a-a0d2-0844115918ee","type":"GlyphRenderer"}]},"id":"07828529-278e-4bf5-ae95-ff39f15076ab","type":"LegendItem"},{"attributes":{"items":[{"id":"816d2b57-b1bc-4424-a461-9a50556ebc9e","type":"LegendItem"},{"id":"0a0d8c6b-ad6c-4677-9732-19b286847c4d","type":"LegendItem"},{"id":"2a8e540b-2d2b-40e7-88f9-1dd6aecb3c66","type":"LegendItem"},{"id":"ae2d64ed-f1f6-4421-8f89-3510287fe100","type":"LegendItem"},{"id":"a1b43be9-3628-4e3c-b54c-7affd0b2e920","type":"LegendItem"},{"id":"5ff9ed04-1d8a-40b9-a11c-30f7fe787f3d","type":"LegendItem"},{"id":"5a4be83e-78b6-4bf8-995b-b469683c7949","type":"LegendItem"},{"id":"9567d43c-c844-433d-8a03-2e2b63003fd0","type":"LegendItem"},{"id":"c2795c5d-7ff1-4caa-af05-cae14ea2f0aa","type":"LegendItem"},{"id":"ec96fa4b-a892-4bc7-9845-cb29b6f3cd72","type":"LegendItem"},{"id":"74d8dc61-8144-495b-a7de-b7b380b895e9","type":"LegendItem"},{"id":"d9d03f75-9ac2-4b5e-a530-4428b34e39e2","type":"LegendItem"},{"id":"07828529-278e-4bf5-ae95-ff39f15076ab","type":"LegendItem"},{"id":"ff097dd2-9932-4c44-b1e1-cfe37364348b","type":"LegendItem"},{"id":"8d852e74-bdbb-4396-8a09-97405a25317b","type":"LegendItem"},{"id":"c24b94c8-de75-4568-9dcf-c73fada9fffe","type":"LegendItem"},{"id":"99aead6a-453b-45d3-8eb6-2662b60b34aa","type":"LegendItem"},{"id":"d48fd320-be1c-4872-be42-90114581f8c8","type":"LegendItem"},{"id":"7ee6f49b-61fc-40b1-9208-e8b8332f6353","type":"LegendItem"},{"id":"3d0c6d48-11d8-4284-9509-2a547a4e17d6","type":"LegendItem"},{"id":"e097ba14-881c-46f0-923d-554df35192c1","type":"LegendItem"}],"location":"top_left","plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"ccc40af2-7a01-467a-93b1-0a05edffbf8c","type":"Legend"},{"attributes":{"label":{"value":"nome"},"renderers":[{"id":"cd1fd160-8140-42e3-a5a5-59f3d93a74ec","type":"GlyphRenderer"}]},"id":"ff097dd2-9932-4c44-b1e1-cfe37364348b","type":"LegendItem"},{"attributes":{"data_source":{"id":"cb1ba73c-f531-42d4-b980-e1c110c1b128","type":"ColumnDataSource"},"glyph":{"id":"efe0ed4a-3708-4149-a441-e2ae829d94c9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ffd0a5c9-9f67-4255-9147-18c075b08f05","type":"GlyphRenderer"},{"attributes":{},"id":"d52caa49-ddf2-4200-98c5-a5393be12dfa","type":"CategoricalTicker"},{"attributes":{"label":{"value":"denomina\u00e7\u00e3o"},"renderers":[{"id":"26308228-1d0e-486a-b211-9e232d245c9f","type":"GlyphRenderer"}]},"id":"e097ba14-881c-46f0-923d-554df35192c1","type":"LegendItem"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"016c444d-76ba-4efb-b32a-7fe168cabfce","type":"BasicTickFormatter"},"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"},"ticker":{"id":"326374cf-b73b-42ee-9b17-ba21e68e74fa","type":"BasicTicker"}},"id":"b5c7d361-8680-4f4c-97ad-fe9e73929920","type":"LinearAxis"},{"attributes":{"label":{"value":"diretrizes"},"renderers":[{"id":"8e61635f-0248-4839-9e04-5fd98d9e32c5","type":"GlyphRenderer"}]},"id":"2a8e540b-2d2b-40e7-88f9-1dd6aecb3c66","type":"LegendItem"},{"attributes":{"data_source":{"id":"7905073c-570a-4f2e-b5aa-282eed00ce74","type":"ColumnDataSource"},"glyph":{"id":"ceb3bc42-c5b2-4756-9613-6db449d3503a","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"26308228-1d0e-486a-b211-9e232d245c9f","type":"GlyphRenderer"},{"attributes":{"dimension":1,"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"},"ticker":{"id":"326374cf-b73b-42ee-9b17-ba21e68e74fa","type":"BasicTicker"}},"id":"cef2dc10-9272-4436-968a-0732ff043ad0","type":"Grid"},{"attributes":{},"id":"38b38fe1-9521-44fb-8e4a-14daf760a460","type":"CategoricalTickFormatter"},{"attributes":{},"id":"016c444d-76ba-4efb-b32a-7fe168cabfce","type":"BasicTickFormatter"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"5ca3884e-3979-4597-bb85-f56d53a38d8f","type":"Rect"},{"attributes":{"label":{"value":"vencimentos"},"renderers":[{"id":"ffd0a5c9-9f67-4255-9147-18c075b08f05","type":"GlyphRenderer"}]},"id":"7ee6f49b-61fc-40b1-9208-e8b8332f6353","type":"LegendItem"},{"attributes":{"data_source":{"id":"de76c399-8d3a-48d7-9b46-a4d2a2abb9c8","type":"ColumnDataSource"},"glyph":{"id":"5ca3884e-3979-4597-bb85-f56d53a38d8f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"faa94308-b7fd-4fd1-a69a-dc5089a1cf2a","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"a1a7995a-f518-451a-9fad-58ecc10f5394","type":"ColumnDataSource"},"glyph":{"id":"b3865137-b8e6-422f-91c4-fb240fd4b18b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"147bea69-c244-4274-abf0-7a2f496406d0","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"f3428be3-57b3-4860-b353-3b654f2400c7","type":"ColumnDataSource"},"glyph":{"id":"0adb7836-cec1-411c-8306-dbf852e9b532","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c2079f9a-4348-411c-8cf2-84753106b5b9","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[83.0],"label":[{"categoria_geral":"outra","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[41.5]}},"id":"8b101cd1-02cf-4fcb-976a-0fbb1492b20b","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estima"],"chart_index":[{"categoria_geral":"estima","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[12.0],"label":[{"categoria_geral":"estima","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estima"],"y":[6.0]}},"id":"8d8296ea-5d95-4a56-aa6a-7c7e22648b11","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"89bd4caa-9ae1-4a23-8805-afe0c2241d84","type":"Rect"},{"attributes":{"data_source":{"id":"8d8296ea-5d95-4a56-aa6a-7c7e22648b11","type":"ColumnDataSource"},"glyph":{"id":"fcf1e8e0-25a1-4303-bc21-13a87eb56038","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"ca43befd-c911-46a0-9989-4b901106f9a8","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["d\u00e1"],"chart_index":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"d\u00e1","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["d\u00e1"],"y":[8.5]}},"id":"a1a7995a-f518-451a-9fad-58ecc10f5394","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"28aabb3e-2916-49e6-b906-c22ad39592cd","type":"ColumnDataSource"},"glyph":{"id":"4a4bf369-fc53-4c83-8ff6-ba6204d5753d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"beaa78aa-ba87-434b-b3b5-aaba0b6c9ba0","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"29aaa622-3806-48b3-b58d-455a1599ac08","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fcf1e8e0-25a1-4303-bc21-13a87eb56038","type":"Rect"},{"attributes":{"data_source":{"id":"38e2a312-7128-48af-bd3b-e29dff763958","type":"ColumnDataSource"},"glyph":{"id":"0a74a86f-2d20-4731-b753-f7516971af20","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"6098194a-82fe-4a2c-b533-48e8e24b8d03","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b3865137-b8e6-422f-91c4-fb240fd4b18b","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8c613934-bbc6-4ccd-95c5-a90575db175d","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6aa2b316-082a-4302-9196-0bc2e161d402","type":"Rect"},{"attributes":{"data_source":{"id":"0fc552ee-79c5-4d3d-bc71-ba7690bc370c","type":"ColumnDataSource"},"glyph":{"id":"8c613934-bbc6-4ccd-95c5-a90575db175d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"197b06be-cd6d-44c3-96e5-b6db6878c04e","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"9b42a72d-7909-470d-9612-729e276e4113","type":"ColumnDataSource"},"glyph":{"id":"3f89abec-a6bf-4a0f-9f6e-4a0bdb7f4a01","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a88212aa-4228-468c-a07a-0c15924e18db","type":"GlyphRenderer"},{"attributes":{"callback":null,"end":174.3},"id":"a49e98eb-eda2-4616-8de0-50d64460f8ba","type":"Range1d"},{"attributes":{"data_source":{"id":"8b101cd1-02cf-4fcb-976a-0fbb1492b20b","type":"ColumnDataSource"},"glyph":{"id":"6aa2b316-082a-4302-9196-0bc2e161d402","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"d7f724c2-0e9f-4ebe-9d76-833e85772f04","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"75f835cd-d31e-4984-988c-5c9601bded80","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"38e2a312-7128-48af-bd3b-e29dff763958","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"a3c6b57f-39ab-45e3-ac54-a26bf9eb5c00","type":"ColumnDataSource"},"glyph":{"id":"91e2c386-56a5-4068-8d20-3fe7b058ce70","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e1d64865-bf68-4a06-9e26-67106dd51962","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"035af110-408a-4585-8a50-cea40db742b5","type":"ColumnDataSource"},"glyph":{"id":"75f835cd-d31e-4984-988c-5c9601bded80","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"95fd74e9-e21c-48b8-8fba-d7d89174c2cc","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"afe84215-c439-4268-bfca-c36322580643","type":"Rect"},{"attributes":{"data_source":{"id":"1650e047-6de7-4fff-8bb5-59714fa6d6fe","type":"ColumnDataSource"},"glyph":{"id":"29aaa622-3806-48b3-b58d-455a1599ac08","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"5cfc1033-2fbe-49f0-8008-bc138f1884ed","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4a4bf369-fc53-4c83-8ff6-ba6204d5753d","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b8f2a57f-d298-428a-9147-5c29b99a3219","type":"Rect"},{"attributes":{"data_source":{"id":"7be59a9d-9be4-4c23-89f8-bd6c2e8dd41c","type":"ColumnDataSource"},"glyph":{"id":"afe84215-c439-4268-bfca-c36322580643","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"314ab9ff-5dca-4224-b5c1-db5339237f63","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3f89abec-a6bf-4a0f-9f6e-4a0bdb7f4a01","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0a74a86f-2d20-4731-b753-f7516971af20","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"cdce15f6-0410-47da-bb6c-4acdbe3ffb76","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"91e2c386-56a5-4068-8d20-3fe7b058ce70","type":"Rect"},{"attributes":{"label":{"value":"reajuste"},"renderers":[{"id":"b98a4a40-5b61-4e6a-9e50-75e0c93907dc","type":"GlyphRenderer"}]},"id":"c24b94c8-de75-4568-9dcf-c73fada9fffe","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"3b522266-44f7-4909-af60-ccdf425dbad6","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"20042481-8e71-4bad-be5f-82a13a46b79e","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a446a7d4-86a4-4d7f-9caf-54c1ccf620c9","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disciplina"],"chart_index":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[6.0],"label":[{"categoria_geral":"disciplina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["disciplina"],"y":[3.0]}},"id":"2cec7b06-1fea-4e77-8dfc-8531a3367c84","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"1a0f36af-17e0-464a-ab3e-b41d079419a3","type":"Rect"},{"attributes":{"data_source":{"id":"f15e52c9-4ff8-4de3-91cb-02023a9db19a","type":"ColumnDataSource"},"glyph":{"id":"20042481-8e71-4bad-be5f-82a13a46b79e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4c26162b-6c7d-426f-acb5-701b6c79992c","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"cdce15f6-0410-47da-bb6c-4acdbe3ffb76","type":"ColumnDataSource"},"glyph":{"id":"ee5a358a-ac42-45c7-bf6f-e644f391860d","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"289fa3c1-1963-4d2c-81bc-8ea70c418338","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"354eb425-80f2-41ed-99ef-84d9f6f0f9e7","type":"Rect"},{"attributes":{"data_source":{"id":"bbda7416-d363-4d0e-a126-d01fd9097157","type":"ColumnDataSource"},"glyph":{"id":"3b522266-44f7-4909-af60-ccdf425dbad6","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e1370b63-71ce-460a-a0d2-0844115918ee","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ee5a358a-ac42-45c7-bf6f-e644f391860d","type":"Rect"},{"attributes":{"data_source":{"id":"e0223ee5-4a11-438b-9f94-b52d658c3361","type":"ColumnDataSource"},"glyph":{"id":"b8f2a57f-d298-428a-9147-5c29b99a3219","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"229ee45c-a17f-400e-8b2b-b3beadc07232","type":"GlyphRenderer"},{"attributes":{"below":[{"id":"3d3e329e-eecd-4655-aace-2e7df7dd869f","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"b5c7d361-8680-4f4c-97ad-fe9e73929920","type":"LinearAxis"}],"renderers":[{"id":"ebef3529-1ef7-4a73-834c-44f9ac9257b3","type":"BoxAnnotation"},{"id":"2805e094-6624-47cd-9b16-39fe2e97e16f","type":"GlyphRenderer"},{"id":"b680ffcc-58d9-49d4-8d93-26d7a8b328f7","type":"GlyphRenderer"},{"id":"dfb7a109-df5f-454d-bf52-1f1edb2d496a","type":"GlyphRenderer"},{"id":"c2079f9a-4348-411c-8cf2-84753106b5b9","type":"GlyphRenderer"},{"id":"147bea69-c244-4274-abf0-7a2f496406d0","type":"GlyphRenderer"},{"id":"d7f724c2-0e9f-4ebe-9d76-833e85772f04","type":"GlyphRenderer"},{"id":"8e61635f-0248-4839-9e04-5fd98d9e32c5","type":"GlyphRenderer"},{"id":"314ab9ff-5dca-4224-b5c1-db5339237f63","type":"GlyphRenderer"},{"id":"6098194a-82fe-4a2c-b533-48e8e24b8d03","type":"GlyphRenderer"},{"id":"4c26162b-6c7d-426f-acb5-701b6c79992c","type":"GlyphRenderer"},{"id":"289fa3c1-1963-4d2c-81bc-8ea70c418338","type":"GlyphRenderer"},{"id":"c49adeff-930f-433c-ba18-fb41bc05f53e","type":"GlyphRenderer"},{"id":"f6eb70e7-b2bd-409b-8a8d-c8502fd0a465","type":"GlyphRenderer"},{"id":"5cfc1033-2fbe-49f0-8008-bc138f1884ed","type":"GlyphRenderer"},{"id":"229ee45c-a17f-400e-8b2b-b3beadc07232","type":"GlyphRenderer"},{"id":"2a8343b1-f214-490f-8325-737cd40f6c81","type":"GlyphRenderer"},{"id":"9fb3f6c7-73c4-4ed3-9a40-6d1530f83f3d","type":"GlyphRenderer"},{"id":"630935a5-2d30-4c27-81d6-0e6594f17208","type":"GlyphRenderer"},{"id":"fae95767-ddcf-4593-a2a3-26029abc36e8","type":"GlyphRenderer"},{"id":"f31aa681-1767-4bbb-b5eb-6bd2a66b5f2e","type":"GlyphRenderer"},{"id":"bb7aefe4-1865-4822-bbeb-b83875ce5ba7","type":"GlyphRenderer"},{"id":"685eb4b8-ae20-4972-80cd-6c15190867c1","type":"GlyphRenderer"},{"id":"ca43befd-c911-46a0-9989-4b901106f9a8","type":"GlyphRenderer"},{"id":"197b06be-cd6d-44c3-96e5-b6db6878c04e","type":"GlyphRenderer"},{"id":"95fd74e9-e21c-48b8-8fba-d7d89174c2cc","type":"GlyphRenderer"},{"id":"beaa78aa-ba87-434b-b3b5-aaba0b6c9ba0","type":"GlyphRenderer"},{"id":"e1d64865-bf68-4a06-9e26-67106dd51962","type":"GlyphRenderer"},{"id":"a88212aa-4228-468c-a07a-0c15924e18db","type":"GlyphRenderer"},{"id":"e1370b63-71ce-460a-a0d2-0844115918ee","type":"GlyphRenderer"},{"id":"8d7bb156-5ce5-414b-808a-806cce08c7aa","type":"GlyphRenderer"},{"id":"cd1fd160-8140-42e3-a5a5-59f3d93a74ec","type":"GlyphRenderer"},{"id":"92046d1d-ca68-4207-82df-f110894ef342","type":"GlyphRenderer"},{"id":"b98a4a40-5b61-4e6a-9e50-75e0c93907dc","type":"GlyphRenderer"},{"id":"2544a1bd-29f7-49c9-90b5-841df1753fb4","type":"GlyphRenderer"},{"id":"8cb0ed7f-78c2-4c69-8edb-0458e9d81df3","type":"GlyphRenderer"},{"id":"ffd0a5c9-9f67-4255-9147-18c075b08f05","type":"GlyphRenderer"},{"id":"faa94308-b7fd-4fd1-a69a-dc5089a1cf2a","type":"GlyphRenderer"},{"id":"63950d7c-f087-40cd-b032-ad4e1c5c0621","type":"GlyphRenderer"},{"id":"8f1d0590-0600-4180-9a5f-0540acd1a5fc","type":"GlyphRenderer"},{"id":"26308228-1d0e-486a-b211-9e232d245c9f","type":"GlyphRenderer"},{"id":"26d88aed-206d-4466-8d94-7292e6d16190","type":"GlyphRenderer"},{"id":"ccc40af2-7a01-467a-93b1-0a05edffbf8c","type":"Legend"},{"id":"3d3e329e-eecd-4655-aace-2e7df7dd869f","type":"CategoricalAxis"},{"id":"b5c7d361-8680-4f4c-97ad-fe9e73929920","type":"LinearAxis"},{"id":"cef2dc10-9272-4436-968a-0732ff043ad0","type":"Grid"}],"title":{"id":"982934fe-78e4-43bf-9dd8-5c9706b0149e","type":"Title"},"tool_events":{"id":"67c417b3-09db-45a5-8d17-bd97636e8a10","type":"ToolEvents"},"toolbar":{"id":"eccef5e3-0abd-4f45-ad4f-8d074794c932","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"a7926521-fe2c-4d78-8191-c7abe940ec2d","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"a49e98eb-eda2-4616-8de0-50d64460f8ba","type":"Range1d"}},"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"},{"attributes":{"data_source":{"id":"f86907d4-19b7-4457-aa85-212ced138ec9","type":"ColumnDataSource"},"glyph":{"id":"a446a7d4-86a4-4d7f-9caf-54c1ccf620c9","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"c49adeff-930f-433c-ba18-fb41bc05f53e","type":"GlyphRenderer"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"ebef3529-1ef7-4a73-834c-44f9ac9257b3","type":"BoxAnnotation"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"concede","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["concede"],"y":[12.5]}},"id":"50f2e9d1-465f-4dc5-a9e4-8213a2fc6274","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0adb7836-cec1-411c-8306-dbf852e9b532","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["fixa"],"chart_index":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[17.0],"label":[{"categoria_geral":"fixa","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["fixa"],"y":[8.5]}},"id":"f86907d4-19b7-4457-aa85-212ced138ec9","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[61.0],"label":[{"categoria_geral":"institui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["institui"],"y":[30.5]}},"id":"628f7593-cbb1-4de6-be47-c5ee53b6b50e","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"89201d67-523e-45c2-bcd4-38c444390d0f","type":"ColumnDataSource"},"glyph":{"id":"ac16dd61-1644-4ed0-9c21-95751466eaee","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f6eb70e7-b2bd-409b-8a8d-c8502fd0a465","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"99ff71c8-f998-43c7-bbaa-338355e3543e","type":"ColumnDataSource"},"glyph":{"id":"1a0f36af-17e0-464a-ab3e-b41d079419a3","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2a8343b1-f214-490f-8325-737cd40f6c81","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"0ef6d353-0248-4292-a6ef-668849d4b7ab","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera lei"],"chart_index":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[116.0],"label":[{"categoria_geral":"altera lei","sub_categoria":"-"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["-"],"width":[0.8],"x":["altera lei"],"y":[58.0]}},"id":"38474c10-b52b-45de-87b1-86ccde742578","type":"ColumnDataSource"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"8601c922-1bc0-434f-a8cd-473366ff1f62","type":"PanTool"},{"id":"2e9c8f62-6490-4e42-977a-501debd8ecbb","type":"WheelZoomTool"},{"id":"52e7ed2c-46a4-4468-9110-aa578489eb16","type":"BoxZoomTool"},{"id":"078e9709-743d-4b1d-9865-83d93e3b610d","type":"SaveTool"},{"id":"7462797e-0c1c-4cd9-a352-024008ab58ce","type":"ResetTool"},{"id":"ca548908-8694-41bf-acef-b05a433f75aa","type":"HelpTool"}]},"id":"eccef5e3-0abd-4f45-ad4f-8d074794c932","type":"Toolbar"},{"attributes":{"data_source":{"id":"a2688cea-dfd5-4f4e-8324-dd893a953c3b","type":"ColumnDataSource"},"glyph":{"id":"4c725d16-e0c4-49e7-a561-a1b5bb951b93","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"9fb3f6c7-73c4-4ed3-9a40-6d1530f83f3d","type":"GlyphRenderer"},{"attributes":{"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"8601c922-1bc0-434f-a8cd-473366ff1f62","type":"PanTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[53.0],"label":[{"categoria_geral":"denomina","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["denomina"],"y":[26.5]}},"id":"a2688cea-dfd5-4f4e-8324-dd893a953c3b","type":"ColumnDataSource"},{"attributes":{"plot":null,"text":"Quantidade de Leis por sub-categoria"},"id":"982934fe-78e4-43bf-9dd8-5c9706b0149e","type":"Title"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["atribui"],"chart_index":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[13.0],"label":[{"categoria_geral":"atribui","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["atribui"],"y":[6.5]}},"id":"99ff71c8-f998-43c7-bbaa-338355e3543e","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"25882454-6b30-4e0e-ad56-b00429bc63e4","type":"ColumnDataSource"},{"attributes":{"callback":null,"factors":["?","altera lei","atribui","autoriza poder executivo","concede","cria","denomina","desafeta","disciplina","disp\u00f5e","d\u00e1","estabelece","estima","fixa","institui","outra","reajusta","revoga"]},"id":"a7926521-fe2c-4d78-8191-c7abe940ec2d","type":"FactorRange"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"ac16dd61-1644-4ed0-9c21-95751466eaee","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"413ff66a-8924-4460-a878-344a300bbafd","type":"Rect"},{"attributes":{"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"2e9c8f62-6490-4e42-977a-501debd8ecbb","type":"WheelZoomTool"},{"attributes":{"overlay":{"id":"ebef3529-1ef7-4a73-834c-44f9ac9257b3","type":"BoxAnnotation"},"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"52e7ed2c-46a4-4468-9110-aa578489eb16","type":"BoxZoomTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4c725d16-e0c4-49e7-a561-a1b5bb951b93","type":"Rect"},{"attributes":{"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"078e9709-743d-4b1d-9865-83d93e3b610d","type":"SaveTool"},{"attributes":{"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"7462797e-0c1c-4cd9-a352-024008ab58ce","type":"ResetTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"df73b9a7-2da1-4fa9-830b-177e4ddc301e","type":"Rect"},{"attributes":{"plot":{"id":"27cc21be-1c55-4413-b365-6859440dd3ae","subtype":"Chart","type":"Plot"}},"id":"ca548908-8694-41bf-acef-b05a433f75aa","type":"HelpTool"},{"attributes":{"data_source":{"id":"628f7593-cbb1-4de6-be47-c5ee53b6b50e","type":"ColumnDataSource"},"glyph":{"id":"df73b9a7-2da1-4fa9-830b-177e4ddc301e","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"630935a5-2d30-4c27-81d6-0e6594f17208","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"41c5db43-f6f9-4788-ba9e-8da3719da1c1","type":"ColumnDataSource"},"glyph":{"id":"413ff66a-8924-4460-a878-344a300bbafd","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b680ffcc-58d9-49d4-8d93-26d7a8b328f7","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[22.0],"label":[{"categoria_geral":"cria","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["cria"],"y":[36.0]}},"id":"f15e52c9-4ff8-4de3-91cb-02023a9db19a","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b67dbb03-bbb7-4c1e-a4cb-7d22c448fa34","type":"Rect"},{"attributes":{"label":{"value":"cargos"},"renderers":[{"id":"c2079f9a-4348-411c-8cf2-84753106b5b9","type":"GlyphRenderer"}]},"id":"0a0d8c6b-ad6c-4677-9732-19b286847c4d","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"cr\u00e9dito"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[15.0],"label":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"cr\u00e9dito"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cr\u00e9dito"],"width":[0.8],"x":["autoriza poder executivo"],"y":[85.5]}},"id":"7be59a9d-9be4-4c23-89f8-bd6c2e8dd41c","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"conselho"}],"color":["#c33ff3"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"cria","sub_categoria":"conselho"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["conselho"],"width":[0.8],"x":["cria"],"y":[51.5]}},"id":"89201d67-523e-45c2-bcd4-38c444390d0f","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"25882454-6b30-4e0e-ad56-b00429bc63e4","type":"ColumnDataSource"},"glyph":{"id":"a3297023-ca58-43d3-ac37-e437632f1650","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"685eb4b8-ae20-4972-80cd-6c15190867c1","type":"GlyphRenderer"},{"attributes":{"label":{"value":"reestrutura\u00e7\u00e3o"},"renderers":[{"id":"2544a1bd-29f7-49c9-90b5-841df1753fb4","type":"GlyphRenderer"}]},"id":"99aead6a-453b-45d3-8eb6-2662b60b34aa","type":"LegendItem"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[78.0],"label":[{"categoria_geral":"autoriza poder executivo","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["autoriza poder executivo"],"y":[39.0]}},"id":"41c5db43-f6f9-4788-ba9e-8da3719da1c1","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"38474c10-b52b-45de-87b1-86ccde742578","type":"ColumnDataSource"},"glyph":{"id":"b67dbb03-bbb7-4c1e-a4cb-7d22c448fa34","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"fae95767-ddcf-4593-a2a3-26029abc36e8","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria","sub_categoria":"cargos"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"cria","sub_categoria":"cargos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cargos"],"width":[0.8],"x":["cria"],"y":[12.5]}},"id":"f3428be3-57b3-4860-b353-3b654f2400c7","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["estabelece"],"chart_index":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"estabelece","sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["estabelece"],"y":[12.5]}},"id":"211826bf-ff65-4212-8fea-a413a305f6ce","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"c14a9e27-8cc9-44f5-b96a-bf5bbc9af690","type":"ColumnDataSource"},"glyph":{"id":"8ffd3f20-6360-466c-bdac-134094f811e2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f31aa681-1767-4bbb-b5eb-6bd2a66b5f2e","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"f49f96b9-6151-48a1-9d9a-e34873fb56da","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8ffd3f20-6360-466c-bdac-134094f811e2","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a3297023-ca58-43d3-ac37-e437632f1650","type":"Rect"},{"attributes":{"data_source":{"id":"50f2e9d1-465f-4dc5-a9e4-8213a2fc6274","type":"ColumnDataSource"},"glyph":{"id":"f49f96b9-6151-48a1-9d9a-e34873fb56da","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"dfb7a109-df5f-454d-bf52-1f1edb2d496a","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"211826bf-ff65-4212-8fea-a413a305f6ce","type":"ColumnDataSource"},"glyph":{"id":"0ef6d353-0248-4292-a6ef-668849d4b7ab","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"bb7aefe4-1865-4822-bbeb-b83875ce5ba7","type":"GlyphRenderer"}],"root_ids":["27cc21be-1c55-4413-b365-6859440dd3ae"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"e664dacd-e503-4385-ad00-132252083e34","elementid":"4a5d7422-c2fb-4f79-945b-ca593d15682f","modelid":"27cc21be-1c55-4413-b365-6859440dd3ae"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("4a5d7422-c2fb-4f79-945b-ca593d15682f")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>



```python
leis[leis['categoria_geral'] == 'institui'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>87</th>
      <td>1100</td>
      <td>1995</td>
      <td>institui a gratificação de produtividade fisca...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>125</th>
      <td>1138</td>
      <td>1997</td>
      <td>institui o fundo municipal de saúde e dá outra...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>168</th>
      <td>1181</td>
      <td>1998</td>
      <td>institui o código tributário do município do i...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>184</th>
      <td>1197</td>
      <td>1999</td>
      <td>institui o programa de garantia de renda mínim...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>185</th>
      <td>1198</td>
      <td>1999</td>
      <td>institui a gratificação adicional de risco de ...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub_categoria_institui = get_counter_with_category('categoria_geral','institui',2)
sub_categoria_institui.most_common()
```




    [('programa', 11),
     ('fundo', 5),
     ('código', 5),
     ('dia', 4),
     ('âmbito', 4),
     ('"dia', 3),
     ('gratificação', 2),
     ('rede', 2),
     ('município,', 2),
     ('semana', 2),
     ('auxílio-transporte', 2),
     ('e', 1),
     ('para', 1),
     ('plano', 1),
     ('contribuição', 1),
     ('percentual', 1),
     ('calendário', 1),
     ('verba', 1),
     ('de', 1),
     ('auxilio', 1),
     ('gratificações', 1),
     ('"prêmio', 1),
     ('sistema', 1),
     ('auxílio-saúde', 1),
     ('regulamenta', 1),
     ('comissão', 1),
     ('suprimento', 1),
     ('tratamento', 1),
     ('“semana', 1),
     ('município', 1)]




```python
categorias_institui = [x for x,cnt in sub_categoria_institui.most_common() if cnt > 3 ]

def get_sub_categoria_institui(ementa):
    ementa = ementa.split(" ")
    word = ementa[2]
    if (word in categorias_institui):
        return word
    elif (word == '"dia'):
        return 'dia'
    else:
        return "outra"

def change_sub_category_label_institui(to_change, category,old_sub_category, ementa):
    if (category == to_change):
        return get_sub_categoria_institui(ementa)
    else:
        return old_sub_category

leis['sub_categoria'] = leis.apply(lambda row: change_sub_category_label_institui('institui',row['categoria_geral'],row['sub_categoria'],row['ementa']), axis=1)
```


```python
leis[leis['categoria_geral'] == 'institui'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>87</th>
      <td>1100</td>
      <td>1995</td>
      <td>institui a gratificação de produtividade fisca...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>125</th>
      <td>1138</td>
      <td>1997</td>
      <td>institui o fundo municipal de saúde e dá outra...</td>
      <td>institui</td>
      <td>fundo</td>
    </tr>
    <tr>
      <th>168</th>
      <td>1181</td>
      <td>1998</td>
      <td>institui o código tributário do município do i...</td>
      <td>institui</td>
      <td>código</td>
    </tr>
    <tr>
      <th>184</th>
      <td>1197</td>
      <td>1999</td>
      <td>institui o programa de garantia de renda mínim...</td>
      <td>institui</td>
      <td>programa</td>
    </tr>
    <tr>
      <th>185</th>
      <td>1198</td>
      <td>1999</td>
      <td>institui a gratificação adicional de risco de ...</td>
      <td>institui</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>



### Categorias 'estima', 'fixa', 'estabelece'
Vamos mudar estas categorias para 'receitas e despesas'


```python
print(leis[leis['categoria_geral'] == 'estima']['ementa'].head())
print(leis[leis['categoria_geral'] == 'fixa']['ementa'].head())
print(leis[leis['categoria_geral'] == 'estabelece']['ementa'].head())
```

    136    estima a receita e fixa a despesa para o exerc...
    187    estima a reita e fixa a despesa para o exercíc...
    219    estima a receita e fixa despesas para o exercí...
    321    estima a receita e fixa a despesa da prefeitur...
    351    estima a receita e fixa a despesa da prefeitur...
    Name: ementa, dtype: object
    49     fixa o salário mínimo em urv e dá outras provi...
    69     fixa o valor do salário mínimo para os funcion...
    170    fixa o subsídio dos vereadores deste município...
    171    fixa os subsídios do prefeito, do viceprefeito...
    210    fixa os subsídios do prefeito, do vice-prefeit...
    Name: ementa, dtype: object
    123    estabelece as diretrizes orçamen tárias do nün...
    155    estabelece as diretrizes orçamentárias do muni...
    206    estabelece as diretrizes orçamentarias para o ...
    214    estabelece o chancfélamento do ingressos para ...
    215    estabelece o subsfi|uto tributário do iss e dá...
    Name: ementa, dtype: object



```python
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('estima', row['categoria_geral'],'receitas e despesas'), axis=1)
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('fixa', row['categoria_geral'],'receitas e despesas'), axis=1)
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('estabelece', row['categoria_geral'],'receitas e despesas'), axis=1)
```


```python
leis[leis['categoria_geral'] == 'receitas e despesas'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>49</th>
      <td>1061</td>
      <td>1994</td>
      <td>fixa o salário mínimo em urv e dá outras provi...</td>
      <td>receitas e despesas</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>69</th>
      <td>1082</td>
      <td>1995</td>
      <td>fixa o valor do salário mínimo para os funcion...</td>
      <td>receitas e despesas</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>123</th>
      <td>1136</td>
      <td>1997</td>
      <td>estabelece as diretrizes orçamen tárias do nün...</td>
      <td>receitas e despesas</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>136</th>
      <td>1149</td>
      <td>1997</td>
      <td>estima a receita e fixa a despesa para o exerc...</td>
      <td>receitas e despesas</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>155</th>
      <td>1168</td>
      <td>1998</td>
      <td>estabelece as diretrizes orçamentárias do muni...</td>
      <td>receitas e despesas</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>




```python
categorias_receitas_despesas = ['estima', 'fixa', 'estabelece']

def get_sub_categoria_receita_despesas(ementa):
    ementa = ementa.split(" ")
    word = ementa[0]
    return word

def change_sub_category_label_cria(to_change, category,old_sub_category, ementa):
    if (category == to_change):
        return get_sub_categoria_receita_despesas(ementa)
    else:
        return old_sub_category

leis['sub_categoria'] = leis.apply(lambda row: change_sub_category_label_cria('receitas e despesas',row['categoria_geral'],row['sub_categoria'],row['ementa']), axis=1)
```


```python
leis[leis['categoria_geral'] == 'receitas e despesas'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>49</th>
      <td>1061</td>
      <td>1994</td>
      <td>fixa o salário mínimo em urv e dá outras provi...</td>
      <td>receitas e despesas</td>
      <td>fixa</td>
    </tr>
    <tr>
      <th>69</th>
      <td>1082</td>
      <td>1995</td>
      <td>fixa o valor do salário mínimo para os funcion...</td>
      <td>receitas e despesas</td>
      <td>fixa</td>
    </tr>
    <tr>
      <th>123</th>
      <td>1136</td>
      <td>1997</td>
      <td>estabelece as diretrizes orçamen tárias do nün...</td>
      <td>receitas e despesas</td>
      <td>estabelece</td>
    </tr>
    <tr>
      <th>136</th>
      <td>1149</td>
      <td>1997</td>
      <td>estima a receita e fixa a despesa para o exerc...</td>
      <td>receitas e despesas</td>
      <td>estima</td>
    </tr>
    <tr>
      <th>155</th>
      <td>1168</td>
      <td>1998</td>
      <td>estabelece as diretrizes orçamentárias do muni...</td>
      <td>receitas e despesas</td>
      <td>estabelece</td>
    </tr>
  </tbody>
</table>
</div>



## Categoria 'atribui'
Vamos combinar a categoria 'atribui' com a categoria 'denomina'


```python
print(leis[leis['categoria_geral'] == 'atribui']['ementa'].head())
print(leis[leis['categoria_geral'] == 'denomina']['ementa'].head())
```

    77    atribui denominação a bem público municipal e ...
    81    atribui a bem público municipal e da outras pr...
    93    atribui denominação a bem público e dá outras ...
    94    atribui denominação a bem público e dá outras ...
    95    atribui denominação a bem público e dá outras ...
    Name: ementa, dtype: object
    78     denomina rua josé hipolito monteiro a rua atua...
    79     denomina rua theodomiro josé camargo silva a r...
    80     denomina rua mário luiz cavalcanti a atualment...
    164    denomina artéria pública e dá outras providênc...
    174    denomina artéria pública e dá outras providênc...
    Name: ementa, dtype: object



```python
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('atribui', row['categoria_geral'],'denomina'), axis=1)
```

## Categorias 'disicplina', 'dá', 'concede'
Podemos combinar estas categorias


```python
print(leis[leis['categoria_geral'] == 'disciplina']['ementa'])
```

    251    disciplina a concessão de incentivos fiscais a...
    391    disciplina a concessão de incentivos fiscais a...
    479    disciplina a concessão de incentivos fiscais p...
    489    disciplina a permissão de serviços de transpor...
    545    disciplina a obrigatoriedade e a utilização do...
    614    disciplina o regime de extinção dos empregos p...
    Name: ementa, dtype: object



```python
leis['categoria_geral'] = leis.apply(lambda row: change_category_label('disciplina', row['categoria_geral'],'concede'), axis=1)
```


```python
leis[leis['categoria_geral'] == 'dá']
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>4</th>
      <td>1016</td>
      <td>1992</td>
      <td>dá nova redação à alínea "a" do artigo 1º da l...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1022</td>
      <td>1992</td>
      <td>dá nova redação no artigo 1º da lei 1009 de 1....</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1036</td>
      <td>1993</td>
      <td>dá designação a logradouros públicos - rua hil...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1041</td>
      <td>1993</td>
      <td>dá designação a logradouros av. gilvan leonico...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>30</th>
      <td>1042</td>
      <td>1993</td>
      <td>dá designação ao estádio municipal de nossa se...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>31</th>
      <td>1043</td>
      <td>1993</td>
      <td>dá designação a bem público centro de saúde ve...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>43</th>
      <td>1055</td>
      <td>1993</td>
      <td>dá nova redação ao art.2 da lei 1052/93 de 10/...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>50</th>
      <td>1062</td>
      <td>1994</td>
      <td>dá nova redação do art. 1 da lei 1048/93 de 05...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>75</th>
      <td>1088</td>
      <td>1995</td>
      <td>dá nova redação aos arts. 72. da lei n. 1070/9...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>282</th>
      <td>1296</td>
      <td>2001</td>
      <td>dá nova redação a dispositivos da lei n.° 1279...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>349</th>
      <td>1365</td>
      <td>2003</td>
      <td>dá nova redação e acrescenta dispositivos à le...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>385</th>
      <td>1406</td>
      <td>2005</td>
      <td>dá o nome de escola modelo exprefeito jaime ag...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>442</th>
      <td>1463</td>
      <td>2007</td>
      <td>dá o nome de avenida dona maria do carmo pontu...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>443</th>
      <td>1464</td>
      <td>2007</td>
      <td>dá o nome de rua marcos pontual de petribú a r...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>444</th>
      <td>1465</td>
      <td>2007</td>
      <td>dá o nome de praça engenheiro pedro pessoa cav...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>449</th>
      <td>1470</td>
      <td>2007</td>
      <td>dá o nome de escola municipal luiz manoel nogu...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>636</th>
      <td>1667</td>
      <td>2013</td>
      <td>dá nova redação à ementa e ao art. 1º da lei m...</td>
      <td>dá</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>




```python
def change_category_da(to_change, old_category, ementa):
    if(old_category == to_change):
        ementa = ementa.split(" ")
        if(ementa[2] == 'nome' or ementa[1] == 'designação'):
            return 'denomina'
        elif(ementa[2] == 'redação'):
            return 'altera lei'
        else:
            return 'outra'
    else:
        return old_category


leis['categoria_geral'] = leis.apply(lambda row: change_category_da('dá', row['categoria_geral'],row['ementa']), axis=1)
leis['sub_categoria'] = leis.apply(lambda row: change_sub_category_label('altera lei',row['categoria_geral'],row['sub_categoria'],'-'), axis=1)
```

## Categoria 'denomina'


```python
leis[leis['categoria_geral'] == 'denomina'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>24</th>
      <td>1036</td>
      <td>1993</td>
      <td>dá designação a logradouros públicos - rua hil...</td>
      <td>denomina</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1041</td>
      <td>1993</td>
      <td>dá designação a logradouros av. gilvan leonico...</td>
      <td>denomina</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>30</th>
      <td>1042</td>
      <td>1993</td>
      <td>dá designação ao estádio municipal de nossa se...</td>
      <td>denomina</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>31</th>
      <td>1043</td>
      <td>1993</td>
      <td>dá designação a bem público centro de saúde ve...</td>
      <td>denomina</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>77</th>
      <td>1090</td>
      <td>1995</td>
      <td>atribui denominação a bem público municipal e ...</td>
      <td>denomina</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>




```python
sub_categoria_denomina = Counter()

for index,row in leis.iterrows():
    if (row['categoria_geral'] == 'denomina'):
        ementa = row['ementa'].split(" ")
        second_word = ementa[1]
        third_word = ementa[2]
        if (second_word in stop_words):
            sub_categoria_denomina[third_word] += 1
        else:
            if(second_word == 'designação' or second_word == 'denominação'):
                word = ementa[3]
                sub_categoria_denomina[word] +=1
            else:
                sub_categoria_denomina[second_word] += 1


sub_categoria_denomina.most_common()
```




    [('bem', 11),
     ('rua', 9),
     ('nome', 9),
     ('artéria', 3),
     ('praça', 3),
     ('ruas', 3),
     ('centro', 3),
     ('escola', 3),
     ('logradouros', 2),
     ('estádio', 2),
     ('rodovia', 2),
     ('biblioteca', 2),
     ('travessa', 2),
     ('quadra', 1),
     ('banda', 1),
     ('avenida', 1),
     ('hospital', 1),
     ("'conjunto", 1),
     ('"denomina', 1),
     ("'praça", 1),
     ('av.', 1),
     ('anexos', 1),
     ('ruas,', 1),
     ('creche', 1),
     ('por', 1),
     ('adnilson', 1),
     ('professora', 1),
     ('"rodovia', 1),
     ('amaro', 1),
     ('misael', 1),
     ('jonas', 1),
     ('josé', 1),
     ('unidade', 1),
     ('jorge', 1)]




```python
categorias_denomina = [x for x,cnt in sub_categoria_denomina.most_common() if cnt > 2 ]

def get_sub_categoria_denomina(ementa):
    ementa = ementa.split(" ")
    second_word = ementa[1]
    fifth_word = ementa[2]
    if (second_word in categorias_denomina):
        return second_word
    else:
        if(second_word == 'designação' or second_word == 'denominação'):
            word = ementa[3]
            return word
        else:
            return 'outra'


def change_sub_category_label_denomina(to_change, category,old_sub_category, ementa):
    if (category == to_change):
        return get_sub_categoria_denomina(ementa)
    else:
        return old_sub_category

leis['sub_categoria'] = leis.apply(lambda row: change_sub_category_label_denomina('denomina',row['categoria_geral'],row['sub_categoria'],row['ementa']), axis=1)
```


```python
leis[leis['categoria_geral'] == 'denomina'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>24</th>
      <td>1036</td>
      <td>1993</td>
      <td>dá designação a logradouros públicos - rua hil...</td>
      <td>denomina</td>
      <td>logradouros</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1041</td>
      <td>1993</td>
      <td>dá designação a logradouros av. gilvan leonico...</td>
      <td>denomina</td>
      <td>logradouros</td>
    </tr>
    <tr>
      <th>30</th>
      <td>1042</td>
      <td>1993</td>
      <td>dá designação ao estádio municipal de nossa se...</td>
      <td>denomina</td>
      <td>estádio</td>
    </tr>
    <tr>
      <th>31</th>
      <td>1043</td>
      <td>1993</td>
      <td>dá designação a bem público centro de saúde ve...</td>
      <td>denomina</td>
      <td>bem</td>
    </tr>
    <tr>
      <th>77</th>
      <td>1090</td>
      <td>1995</td>
      <td>atribui denominação a bem público municipal e ...</td>
      <td>denomina</td>
      <td>bem</td>
    </tr>
  </tbody>
</table>
</div>




```python
p = Bar(leis, 'categoria_geral', values='index', agg='count', title="Quantidade de Leis por categoria")
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="dc7f0dd4-0fb7-42bf-94c2-9ab13764aca2"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("dc7f0dd4-0fb7-42bf-94c2-9ab13764aca2").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("dc7f0dd4-0fb7-42bf-94c2-9ab13764aca2");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid 'dc7f0dd4-0fb7-42bf-94c2-9ab13764aca2' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"94d29d15-20f9-4fee-91c1-804f253de8e7":{"roots":{"references":[{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["reajusta"],"chart_index":[{"categoria_geral":"reajusta"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[25.0],"label":[{"categoria_geral":"reajusta"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["reajusta"],"y":[12.5]}},"id":"39c614ab-8eff-4a02-afe3-b86b0a57d2c6","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["outra"],"chart_index":[{"categoria_geral":"outra"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[85.0],"label":[{"categoria_geral":"outra"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["outra"],"y":[42.5]}},"id":"fe0d44cc-eadc-43c3-9c28-f6c97525bbec","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"1cb3dafa-7664-4962-baa6-66e7a766ca20","type":"ColumnDataSource"},"glyph":{"id":"37c6b8bb-73e2-4aaa-90c3-46b8fcecabc5","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"e0da1894-c84c-4cb8-a0eb-2cb11a1b449a","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"94949249-5e16-4406-8d29-1974641ea4ad","type":"Rect"},{"attributes":{"data_source":{"id":"39c614ab-8eff-4a02-afe3-b86b0a57d2c6","type":"ColumnDataSource"},"glyph":{"id":"94949249-5e16-4406-8d29-1974641ea4ad","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"cf1cfe9b-fc91-490c-a8e0-c43b0ea4c948","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"291acae6-f5fe-41f0-a9c1-152ebb41943f","type":"ColumnDataSource"},"glyph":{"id":"8b2b3a91-355e-4a55-b0de-bdee1059e5c1","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8b4f9443-e9e3-4ec1-a67f-774c1fb76b6a","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"bd7f92fb-6b07-436f-9441-29715dc2de73","type":"ColumnDataSource"},"glyph":{"id":"c45d29a9-6553-4df2-9e3b-d167513c3013","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"20943e7b-6180-4222-81ba-8a1bb9568524","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["autoriza poder executivo"],"chart_index":[{"categoria_geral":"autoriza poder executivo"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[93.0],"label":[{"categoria_geral":"autoriza poder executivo"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["autoriza poder executivo"],"y":[46.5]}},"id":"1cb3dafa-7664-4962-baa6-66e7a766ca20","type":"ColumnDataSource"},{"attributes":{"label":{"value":"concede"},"renderers":[{"id":"4a071c97-34ea-4daf-a10d-046a1d077806","type":"GlyphRenderer"}]},"id":"b6323c1a-b385-4514-a48e-1c5c6957ad2e","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"fa06d841-f5cd-4d3c-a461-2302d85a6feb","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["concede"],"chart_index":[{"categoria_geral":"concede"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[31.0],"label":[{"categoria_geral":"concede"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["concede"],"y":[15.5]}},"id":"b3f4a4b8-629c-4cb6-ad78-240431f18675","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"b3f4a4b8-629c-4cb6-ad78-240431f18675","type":"ColumnDataSource"},"glyph":{"id":"29f7adbb-702f-4af5-a0eb-2761f33b2095","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"4a071c97-34ea-4daf-a10d-046a1d077806","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"37c6b8bb-73e2-4aaa-90c3-46b8fcecabc5","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["altera lei"],"chart_index":[{"categoria_geral":"altera lei"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[124.0],"label":[{"categoria_geral":"altera lei"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["altera lei"],"y":[62.0]}},"id":"291acae6-f5fe-41f0-a9c1-152ebb41943f","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"f0014895-e313-4505-97f1-b0502a18d003","type":"HelpTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["cria"],"chart_index":[{"categoria_geral":"cria"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[77.0],"label":[{"categoria_geral":"cria"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["cria"],"y":[38.5]}},"id":"bd7f92fb-6b07-436f-9441-29715dc2de73","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"29f7adbb-702f-4af5-a0eb-2761f33b2095","type":"Rect"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"0224754a-283a-4f7f-a53b-dfad60eb336a","type":"BoxAnnotation"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"c45d29a9-6553-4df2-9e3b-d167513c3013","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8b2b3a91-355e-4a55-b0de-bdee1059e5c1","type":"Rect"},{"attributes":{"callback":null,"end":174.3},"id":"f8014d15-5e91-4a4d-a39c-af5b364b3081","type":"Range1d"},{"attributes":{"data_source":{"id":"0eb4dcc9-47b6-4d64-bf16-2037bc3c7d16","type":"ColumnDataSource"},"glyph":{"id":"6fc241f5-3d87-40f6-8fd8-b342a6ced76c","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"2ba5baf6-9b18-49d0-bb5b-04d75bee254e","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["disp\u00f5e"],"chart_index":[{"categoria_geral":"disp\u00f5e"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[166.0],"label":[{"categoria_geral":"disp\u00f5e"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["disp\u00f5e"],"y":[83.0]}},"id":"ca59bf91-2f07-4943-9b56-7eafe9a9e835","type":"ColumnDataSource"},{"attributes":{"callback":null,"factors":["?","altera lei","autoriza poder executivo","concede","cria","denomina","desafeta","disp\u00f5e","institui","outra","reajusta","receitas e despesas","revoga"]},"id":"3ebcdea6-acff-47cf-a368-f39d1ee2c57b","type":"FactorRange"},{"attributes":{},"id":"0132ca43-072c-44ea-bc42-d269085af7f2","type":"CategoricalTicker"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"103acc40-c69e-411e-9edd-d32a60898e33","type":"Rect"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"86cd22f9-20c5-462d-a720-09ef3b9e92d1","type":"Rect"},{"attributes":{"data_source":{"id":"fe0d44cc-eadc-43c3-9c28-f6c97525bbec","type":"ColumnDataSource"},"glyph":{"id":"103acc40-c69e-411e-9edd-d32a60898e33","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"60524c8f-3eb4-434e-b3bd-91da75740c12","type":"GlyphRenderer"},{"attributes":{"label":{"value":"cria"},"renderers":[{"id":"20943e7b-6180-4222-81ba-8a1bb9568524","type":"GlyphRenderer"}]},"id":"0f53a58c-d639-451e-8a60-5defc296156e","type":"LegendItem"},{"attributes":{"label":{"value":"reajusta"},"renderers":[{"id":"cf1cfe9b-fc91-490c-a8e0-c43b0ea4c948","type":"GlyphRenderer"}]},"id":"9cbf39fb-b0f8-4360-b9c2-6b5e3cf7eaa0","type":"LegendItem"},{"attributes":{"label":{"value":"autoriza poder executivo"},"renderers":[{"id":"e0da1894-c84c-4cb8-a0eb-2cb11a1b449a","type":"GlyphRenderer"}]},"id":"f22a3ae7-d3ec-4458-ad86-516af94e1810","type":"LegendItem"},{"attributes":{"items":[{"id":"9cbf39fb-b0f8-4360-b9c2-6b5e3cf7eaa0","type":"LegendItem"},{"id":"f22a3ae7-d3ec-4458-ad86-516af94e1810","type":"LegendItem"},{"id":"b6323c1a-b385-4514-a48e-1c5c6957ad2e","type":"LegendItem"},{"id":"0f53a58c-d639-451e-8a60-5defc296156e","type":"LegendItem"},{"id":"55ae319a-5a5a-4dcd-abde-a582ed3d1076","type":"LegendItem"},{"id":"7cc785fe-719e-48d7-9b03-9be0f5c435a6","type":"LegendItem"},{"id":"1293e332-497f-4e7c-8b1d-577cc5b2958c","type":"LegendItem"},{"id":"fefb1c90-f72b-42f1-9cbe-61e5f92c921c","type":"LegendItem"},{"id":"76cdd036-f070-43c7-8595-30ebe11742b0","type":"LegendItem"},{"id":"6f4b287b-5370-4d37-9563-ba564ef3b748","type":"LegendItem"},{"id":"c7edf73a-b1b0-4c23-a9bf-f03b69a52758","type":"LegendItem"},{"id":"4c1078ef-85c7-4897-9644-72f4108cf69f","type":"LegendItem"},{"id":"d6faa155-b185-423b-9dfc-0cc173b12752","type":"LegendItem"}],"location":"top_left","plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"0bf0c0a2-0eba-4f47-8565-e2c1b67cf565","type":"Legend"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8a305415-95de-4940-b255-110cf284e142","type":"Rect"},{"attributes":{},"id":"595831f4-5caf-4dbd-95c5-dfd9454f15ed","type":"ToolEvents"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["denomina"],"chart_index":[{"categoria_geral":"denomina"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[75.0],"label":[{"categoria_geral":"denomina"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["denomina"],"y":[37.5]}},"id":"0eb4dcc9-47b6-4d64-bf16-2037bc3c7d16","type":"ColumnDataSource"},{"attributes":{"below":[{"id":"89a21f31-54b6-499c-afd8-db9da8ba807e","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"b689c887-ce8e-41ca-98bc-87c51fee7a18","type":"LinearAxis"}],"renderers":[{"id":"0224754a-283a-4f7f-a53b-dfad60eb336a","type":"BoxAnnotation"},{"id":"cf1cfe9b-fc91-490c-a8e0-c43b0ea4c948","type":"GlyphRenderer"},{"id":"e0da1894-c84c-4cb8-a0eb-2cb11a1b449a","type":"GlyphRenderer"},{"id":"4a071c97-34ea-4daf-a10d-046a1d077806","type":"GlyphRenderer"},{"id":"20943e7b-6180-4222-81ba-8a1bb9568524","type":"GlyphRenderer"},{"id":"8b4f9443-e9e3-4ec1-a67f-774c1fb76b6a","type":"GlyphRenderer"},{"id":"60524c8f-3eb4-434e-b3bd-91da75740c12","type":"GlyphRenderer"},{"id":"f03fea06-f6b3-4d82-a017-f0d4838729c0","type":"GlyphRenderer"},{"id":"15187d6c-12a1-4fae-8075-5fdffffc834e","type":"GlyphRenderer"},{"id":"2ba5baf6-9b18-49d0-bb5b-04d75bee254e","type":"GlyphRenderer"},{"id":"b72f3777-2b80-4164-83b7-a1799469316e","type":"GlyphRenderer"},{"id":"412ff388-b4d7-44c7-b9d4-4c31748fce78","type":"GlyphRenderer"},{"id":"a918fec6-583b-48c7-ad2f-99e030d7ec21","type":"GlyphRenderer"},{"id":"8c9060bb-d651-4f43-a963-663b17275234","type":"GlyphRenderer"},{"id":"0bf0c0a2-0eba-4f47-8565-e2c1b67cf565","type":"Legend"},{"id":"89a21f31-54b6-499c-afd8-db9da8ba807e","type":"CategoricalAxis"},{"id":"b689c887-ce8e-41ca-98bc-87c51fee7a18","type":"LinearAxis"},{"id":"08e2447f-6175-4329-912a-2145c4d027cc","type":"Grid"}],"title":{"id":"d21d536e-b703-4fc5-b7c2-1900082a314b","type":"Title"},"tool_events":{"id":"595831f4-5caf-4dbd-95c5-dfd9454f15ed","type":"ToolEvents"},"toolbar":{"id":"075626d5-b66e-4b33-933f-d96f7186e1fa","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"3ebcdea6-acff-47cf-a368-f39d1ee2c57b","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"f8014d15-5e91-4a4d-a39c-af5b364b3081","type":"Range1d"}},"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"},{"attributes":{"data_source":{"id":"ca59bf91-2f07-4943-9b56-7eafe9a9e835","type":"ColumnDataSource"},"glyph":{"id":"8a305415-95de-4940-b255-110cf284e142","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"f03fea06-f6b3-4d82-a017-f0d4838729c0","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["desafeta"],"chart_index":[{"categoria_geral":"desafeta"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[7.0],"label":[{"categoria_geral":"desafeta"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["desafeta"],"y":[3.5]}},"id":"617a7dee-1c6f-4675-93ad-d694cd72f44e","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6fc241f5-3d87-40f6-8fd8-b342a6ced76c","type":"Rect"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"dce1420d-6bb5-4537-9645-9e4808026b71","type":"PanTool"},{"id":"b15e7636-a162-44ed-b8b1-09aadba81b96","type":"WheelZoomTool"},{"id":"e56c0939-cc3c-4c6b-a902-887f348a27c2","type":"BoxZoomTool"},{"id":"01039aa6-96f4-4aec-a175-3f804b6b06fc","type":"SaveTool"},{"id":"ff7a3b3b-ccc1-484f-8388-fe38859f96fd","type":"ResetTool"},{"id":"f0014895-e313-4505-97f1-b0502a18d003","type":"HelpTool"}]},"id":"075626d5-b66e-4b33-933f-d96f7186e1fa","type":"Toolbar"},{"attributes":{"data_source":{"id":"617a7dee-1c6f-4675-93ad-d694cd72f44e","type":"ColumnDataSource"},"glyph":{"id":"86cd22f9-20c5-462d-a720-09ef3b9e92d1","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"15187d6c-12a1-4fae-8075-5fdffffc834e","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["institui"],"chart_index":[{"categoria_geral":"institui"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[61.0],"label":[{"categoria_geral":"institui"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["institui"],"y":[30.5]}},"id":"af2adc9b-2731-4aa6-9554-9fd50c555506","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["revoga"],"chart_index":[{"categoria_geral":"revoga"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[9.0],"label":[{"categoria_geral":"revoga"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["revoga"],"y":[4.5]}},"id":"fb4f44f7-0c70-4efb-8c63-8508b2eb6b15","type":"ColumnDataSource"},{"attributes":{"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"01039aa6-96f4-4aec-a175-3f804b6b06fc","type":"SaveTool"},{"attributes":{"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"dce1420d-6bb5-4537-9645-9e4808026b71","type":"PanTool"},{"attributes":{"data_source":{"id":"fb4f44f7-0c70-4efb-8c63-8508b2eb6b15","type":"ColumnDataSource"},"glyph":{"id":"fa06d841-f5cd-4d3c-a461-2302d85a6feb","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"b72f3777-2b80-4164-83b7-a1799469316e","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["receitas e despesas"],"chart_index":[{"categoria_geral":"receitas e despesas"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[58.0],"label":[{"categoria_geral":"receitas e despesas"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["receitas e despesas"],"y":[29.0]}},"id":"ee70ba65-fc47-4ae4-aefb-c2e5dd9841df","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"categoria_geral":["?"],"chart_index":[{"categoria_geral":"?"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[10.0],"label":[{"categoria_geral":"?"}],"line_alpha":[1.0],"line_color":["white"],"width":[0.8],"x":["?"],"y":[5.0]}},"id":"b4d41358-8823-4829-bddf-d310817e2939","type":"ColumnDataSource"},{"attributes":{"data_source":{"id":"ee70ba65-fc47-4ae4-aefb-c2e5dd9841df","type":"ColumnDataSource"},"glyph":{"id":"8509c379-bf7c-4c4f-870c-3233559aa2d4","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"412ff388-b4d7-44c7-b9d4-4c31748fce78","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8509c379-bf7c-4c4f-870c-3233559aa2d4","type":"Rect"},{"attributes":{"label":{"value":"altera lei"},"renderers":[{"id":"8b4f9443-e9e3-4ec1-a67f-774c1fb76b6a","type":"GlyphRenderer"}]},"id":"55ae319a-5a5a-4dcd-abde-a582ed3d1076","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"a69ed1d0-8b35-4f42-9ea2-f73dbd67753f","type":"Rect"},{"attributes":{},"id":"5a9e605f-3a42-4e07-818b-dc8f1172a360","type":"BasicTicker"},{"attributes":{"label":{"value":"disp\u00f5e"},"renderers":[{"id":"f03fea06-f6b3-4d82-a017-f0d4838729c0","type":"GlyphRenderer"}]},"id":"1293e332-497f-4e7c-8b1d-577cc5b2958c","type":"LegendItem"},{"attributes":{"data_source":{"id":"af2adc9b-2731-4aa6-9554-9fd50c555506","type":"ColumnDataSource"},"glyph":{"id":"a69ed1d0-8b35-4f42-9ea2-f73dbd67753f","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"a918fec6-583b-48c7-ad2f-99e030d7ec21","type":"GlyphRenderer"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"fba9cd69-92d7-4352-a8a2-80dcd287bfb4","type":"BasicTickFormatter"},"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"},"ticker":{"id":"5a9e605f-3a42-4e07-818b-dc8f1172a360","type":"BasicTicker"}},"id":"b689c887-ce8e-41ca-98bc-87c51fee7a18","type":"LinearAxis"},{"attributes":{"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"b15e7636-a162-44ed-b8b1-09aadba81b96","type":"WheelZoomTool"},{"attributes":{},"id":"8608c0b1-9f6a-406d-bd5f-ad2fc0ddb5cb","type":"CategoricalTickFormatter"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"60524c8f-3eb4-434e-b3bd-91da75740c12","type":"GlyphRenderer"}]},"id":"7cc785fe-719e-48d7-9b03-9be0f5c435a6","type":"LegendItem"},{"attributes":{"label":{"value":"revoga"},"renderers":[{"id":"b72f3777-2b80-4164-83b7-a1799469316e","type":"GlyphRenderer"}]},"id":"6f4b287b-5370-4d37-9563-ba564ef3b748","type":"LegendItem"},{"attributes":{"data_source":{"id":"b4d41358-8823-4829-bddf-d310817e2939","type":"ColumnDataSource"},"glyph":{"id":"8f1092f1-16be-4520-b6a9-4b53a9c56108","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"8c9060bb-d651-4f43-a963-663b17275234","type":"GlyphRenderer"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"8f1092f1-16be-4520-b6a9-4b53a9c56108","type":"Rect"},{"attributes":{"label":{"value":"desafeta"},"renderers":[{"id":"15187d6c-12a1-4fae-8075-5fdffffc834e","type":"GlyphRenderer"}]},"id":"fefb1c90-f72b-42f1-9cbe-61e5f92c921c","type":"LegendItem"},{"attributes":{"label":{"value":"?"},"renderers":[{"id":"8c9060bb-d651-4f43-a963-663b17275234","type":"GlyphRenderer"}]},"id":"d6faa155-b185-423b-9dfc-0cc173b12752","type":"LegendItem"},{"attributes":{"label":{"value":"institui"},"renderers":[{"id":"a918fec6-583b-48c7-ad2f-99e030d7ec21","type":"GlyphRenderer"}]},"id":"4c1078ef-85c7-4897-9644-72f4108cf69f","type":"LegendItem"},{"attributes":{"label":{"value":"denomina"},"renderers":[{"id":"2ba5baf6-9b18-49d0-bb5b-04d75bee254e","type":"GlyphRenderer"}]},"id":"76cdd036-f070-43c7-8595-30ebe11742b0","type":"LegendItem"},{"attributes":{"dimension":1,"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"},"ticker":{"id":"5a9e605f-3a42-4e07-818b-dc8f1172a360","type":"BasicTicker"}},"id":"08e2447f-6175-4329-912a-2145c4d027cc","type":"Grid"},{"attributes":{"axis_label":"Categoria_Geral","formatter":{"id":"8608c0b1-9f6a-406d-bd5f-ad2fc0ddb5cb","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"},"ticker":{"id":"0132ca43-072c-44ea-bc42-d269085af7f2","type":"CategoricalTicker"}},"id":"89a21f31-54b6-499c-afd8-db9da8ba807e","type":"CategoricalAxis"},{"attributes":{"label":{"value":"receitas e despesas"},"renderers":[{"id":"412ff388-b4d7-44c7-b9d4-4c31748fce78","type":"GlyphRenderer"}]},"id":"c7edf73a-b1b0-4c23-a9bf-f03b69a52758","type":"LegendItem"},{"attributes":{"plot":null,"text":"Quantidade de Leis por categoria"},"id":"d21d536e-b703-4fc5-b7c2-1900082a314b","type":"Title"},{"attributes":{},"id":"fba9cd69-92d7-4352-a8a2-80dcd287bfb4","type":"BasicTickFormatter"},{"attributes":{"overlay":{"id":"0224754a-283a-4f7f-a53b-dfad60eb336a","type":"BoxAnnotation"},"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"e56c0939-cc3c-4c6b-a902-887f348a27c2","type":"BoxZoomTool"},{"attributes":{"plot":{"id":"f39fcc8c-c1c3-4912-b1b2-042573a98177","subtype":"Chart","type":"Plot"}},"id":"ff7a3b3b-ccc1-484f-8388-fe38859f96fd","type":"ResetTool"}],"root_ids":["f39fcc8c-c1c3-4912-b1b2-042573a98177"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"94d29d15-20f9-4fee-91c1-804f253de8e7","elementid":"dc7f0dd4-0fb7-42bf-94c2-9ab13764aca2","modelid":"f39fcc8c-c1c3-4912-b1b2-042573a98177"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("dc7f0dd4-0fb7-42bf-94c2-9ab13764aca2")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>



```python
p = Bar(leis[leis['categoria_geral']=='cria'], 'sub_categoria', values='index', agg='count', title="Leis da categoria 'cria'", stack='sub_categoria')
show(p)
```




    <div class="bk-root">
        <div class="bk-plotdiv" id="69182cd7-c414-4bfd-977a-e3a747ae139e"></div>
    </div>
<script type="text/javascript">

  (function(global) {
    function now() {
      return new Date();
    }

    var force = false;

    if (typeof (window._bokeh_onload_callbacks) === "undefined" || force === true) {
      window._bokeh_onload_callbacks = [];
      window._bokeh_is_loading = undefined;
    }



    if (typeof (window._bokeh_timeout) === "undefined" || force === true) {
      window._bokeh_timeout = Date.now() + 0;
      window._bokeh_failed_load = false;
    }

    var NB_LOAD_WARNING = {'data': {'text/html':
       "<div style='background-color: #fdd'>\n"+
       "<p>\n"+
       "BokehJS does not appear to have successfully loaded. If loading BokehJS from CDN, this \n"+
       "may be due to a slow or bad network connection. Possible fixes:\n"+
       "</p>\n"+
       "<ul>\n"+
       "<li>re-rerun `output_notebook()` to attempt to load from CDN again, or</li>\n"+
       "<li>use INLINE resources instead, as so:</li>\n"+
       "</ul>\n"+
       "<code>\n"+
       "from bokeh.resources import INLINE\n"+
       "output_notebook(resources=INLINE)\n"+
       "</code>\n"+
       "</div>"}};

    function display_loaded() {
      if (window.Bokeh !== undefined) {
        document.getElementById("69182cd7-c414-4bfd-977a-e3a747ae139e").textContent = "BokehJS successfully loaded.";
      } else if (Date.now() < window._bokeh_timeout) {
        setTimeout(display_loaded, 100)
      }
    }

    function run_callbacks() {
      window._bokeh_onload_callbacks.forEach(function(callback) { callback() });
      delete window._bokeh_onload_callbacks
      console.info("Bokeh: all callbacks have finished");
    }

    function load_libs(js_urls, callback) {
      window._bokeh_onload_callbacks.push(callback);
      if (window._bokeh_is_loading > 0) {
        console.log("Bokeh: BokehJS is being loaded, scheduling callback at", now());
        return null;
      }
      if (js_urls == null || js_urls.length === 0) {
        run_callbacks();
        return null;
      }
      console.log("Bokeh: BokehJS not loaded, scheduling load and callback at", now());
      window._bokeh_is_loading = js_urls.length;
      for (var i = 0; i < js_urls.length; i++) {
        var url = js_urls[i];
        var s = document.createElement('script');
        s.src = url;
        s.async = false;
        s.onreadystatechange = s.onload = function() {
          window._bokeh_is_loading--;
          if (window._bokeh_is_loading === 0) {
            console.log("Bokeh: all BokehJS libraries loaded");
            run_callbacks()
          }
        };
        s.onerror = function() {
          console.warn("failed to load library " + url);
        };
        console.log("Bokeh: injecting script tag for BokehJS library: ", url);
        document.getElementsByTagName("head")[0].appendChild(s);
      }
    };var element = document.getElementById("69182cd7-c414-4bfd-977a-e3a747ae139e");
    if (element == null) {
      console.log("Bokeh: ERROR: autoload.js configured with elementid '69182cd7-c414-4bfd-977a-e3a747ae139e' but no matching script tag was found. ")
      return false;
    }

    var js_urls = [];

    var inline_js = [
      function(Bokeh) {
        (function() {
          var fn = function() {
            var docs_json = {"b2576041-c9d7-42f8-89d5-78d65d73f702":{"roots":{"references":[{"attributes":{},"id":"f8b7373e-5f80-425b-b3c7-b843c9aed858","type":"BasicTicker"},{"attributes":{"data_source":{"id":"a956fbd5-d0c1-4d85-94b7-01ca8c6e117d","type":"ColumnDataSource"},"glyph":{"id":"061808ec-5bcc-422a-8eb0-d9ead4d9b7cf","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"267e78c8-f7a1-4efa-a45d-537251931781","type":"GlyphRenderer"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"chart_index":[{"sub_categoria":"secretaria"}],"color":["#00ad9c"],"fill_alpha":[0.8],"height":[4.0],"label":[{"sub_categoria":"secretaria"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["secretaria"],"width":[0.8],"x":["secretaria"],"y":[2.0]}},"id":"ce485a8b-895c-43e8-95eb-eee6850350af","type":"ColumnDataSource"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"chart_index":[{"sub_categoria":"conselho"}],"color":["#5ab738"],"fill_alpha":[0.8],"height":[9.0],"label":[{"sub_categoria":"conselho"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["conselho"],"width":[0.8],"x":["conselho"],"y":[4.5]}},"id":"c5c5b400-d6ee-4ac1-b77e-60115007f40d","type":"ColumnDataSource"},{"attributes":{},"id":"ef80aac8-6f6f-4362-a54d-8963d832524a","type":"BasicTickFormatter"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"d1f6fc87-f9c6-40c7-b69f-83c08a6e2021","type":"Rect"},{"attributes":{},"id":"1a1d087f-afc8-4dc9-86d5-32a785ea0e9c","type":"CategoricalTickFormatter"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"chart_index":[{"sub_categoria":"outra"}],"color":["#df5320"],"fill_alpha":[0.8],"height":[22.0],"label":[{"sub_categoria":"outra"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["outra"],"width":[0.8],"x":["outra"],"y":[11.0]}},"id":"390ad096-c8c3-488a-b231-43c0d0adf750","type":"ColumnDataSource"},{"attributes":{"axis_label":"Count( Index )","formatter":{"id":"ef80aac8-6f6f-4362-a54d-8963d832524a","type":"BasicTickFormatter"},"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"},"ticker":{"id":"f8b7373e-5f80-425b-b3c7-b843c9aed858","type":"BasicTicker"}},"id":"412254c3-0625-4673-a8b6-737398a574ca","type":"LinearAxis"},{"attributes":{"dimension":1,"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"},"ticker":{"id":"f8b7373e-5f80-425b-b3c7-b843c9aed858","type":"BasicTicker"}},"id":"dbeb26b3-e2db-4eab-94fd-39aa8a1c84df","type":"Grid"},{"attributes":{"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"aba2f459-cb7d-4316-a0e9-8cf00f534e9e","type":"PanTool"},{"attributes":{},"id":"b9f84b35-ed6a-4bb7-8f1b-939b2492df53","type":"ToolEvents"},{"attributes":{"bottom_units":"screen","fill_alpha":{"value":0.5},"fill_color":{"value":"lightgrey"},"left_units":"screen","level":"overlay","line_alpha":{"value":1.0},"line_color":{"value":"black"},"line_dash":[4,4],"line_width":{"value":2},"plot":null,"render_mode":"css","right_units":"screen","top_units":"screen"},"id":"1d57b333-1271-4b65-9ef5-d48671bd2216","type":"BoxAnnotation"},{"attributes":{"data_source":{"id":"390ad096-c8c3-488a-b231-43c0d0adf750","type":"ColumnDataSource"},"glyph":{"id":"b38d10ea-1a8b-466d-846b-1c9b58d00b7b","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"cbf1236f-2bc6-4276-93ea-40b679d5136d","type":"GlyphRenderer"},{"attributes":{"label":{"value":"conselho"},"renderers":[{"id":"792774c6-3f42-41fd-a72c-30585fc97da1","type":"GlyphRenderer"}]},"id":"ae6cd8f3-26c0-4e4b-b6ad-174cb5777439","type":"LegendItem"},{"attributes":{"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"830d5009-db7c-4ad5-a596-52e988bdde5d","type":"ResetTool"},{"attributes":{},"id":"21b51f20-655b-407b-9476-a97aed01a726","type":"CategoricalTicker"},{"attributes":{"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"0080ac9a-6681-4ce4-a711-886f75346fdc","type":"WheelZoomTool"},{"attributes":{"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"4a20e507-2e62-4c80-a9ca-f95daecd276e","type":"HelpTool"},{"attributes":{"label":{"value":"secretaria"},"renderers":[{"id":"41e5fb45-34c4-4cab-bd87-3dc30ec74278","type":"GlyphRenderer"}]},"id":"32e73b6b-8c29-4996-ac71-eee2ce7c7509","type":"LegendItem"},{"attributes":{"label":{"value":"nome"},"renderers":[{"id":"267e78c8-f7a1-4efa-a45d-537251931781","type":"GlyphRenderer"}]},"id":"b9b1c995-6d13-4329-84c7-e6ca05a87a70","type":"LegendItem"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"4769fcc0-4c29-4335-9827-12021e5ee8e2","type":"Rect"},{"attributes":{"plot":null,"text":"Leis da categoria 'cria'"},"id":"4dd65458-8347-4c3b-943e-04a59cfe0f36","type":"Title"},{"attributes":{"label":{"value":"cargos"},"renderers":[{"id":"3053d7d4-2a82-497b-8be1-43340d4f5a77","type":"GlyphRenderer"}]},"id":"6fef32a3-233e-4f46-9435-d715a21ba5da","type":"LegendItem"},{"attributes":{"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"80fcd317-29e5-4e02-a1bf-fd04d172fae4","type":"SaveTool"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"061808ec-5bcc-422a-8eb0-d9ead4d9b7cf","type":"Rect"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"chart_index":[{"sub_categoria":"cargos"}],"color":["#f22c40"],"fill_alpha":[0.8],"height":[25.0],"label":[{"sub_categoria":"cargos"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["cargos"],"width":[0.8],"x":["cargos"],"y":[12.5]}},"id":"072cc80d-37cc-43e6-8f2c-2ae82dba1144","type":"ColumnDataSource"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"6eef8ca9-12de-44c1-be34-bb59728cc673","type":"Rect"},{"attributes":{"callback":null,"factors":["cargos","conselho","nome","outra","secretaria"]},"id":"4813f957-93df-421a-b0dc-350c85824a21","type":"FactorRange"},{"attributes":{"fill_alpha":{"field":"fill_alpha"},"fill_color":{"field":"color"},"height":{"field":"height","units":"data"},"line_color":{"field":"line_color"},"width":{"field":"width","units":"data"},"x":{"field":"x"},"y":{"field":"y"}},"id":"b38d10ea-1a8b-466d-846b-1c9b58d00b7b","type":"Rect"},{"attributes":{"items":[{"id":"6fef32a3-233e-4f46-9435-d715a21ba5da","type":"LegendItem"},{"id":"19ee73e5-fb56-4e12-84b3-fea625c9671e","type":"LegendItem"},{"id":"ae6cd8f3-26c0-4e4b-b6ad-174cb5777439","type":"LegendItem"},{"id":"32e73b6b-8c29-4996-ac71-eee2ce7c7509","type":"LegendItem"},{"id":"b9b1c995-6d13-4329-84c7-e6ca05a87a70","type":"LegendItem"}],"location":"top_left","plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"7b2b9350-145b-4c03-b1bc-18c97a516698","type":"Legend"},{"attributes":{"active_drag":"auto","active_scroll":"auto","active_tap":"auto","tools":[{"id":"aba2f459-cb7d-4316-a0e9-8cf00f534e9e","type":"PanTool"},{"id":"0080ac9a-6681-4ce4-a711-886f75346fdc","type":"WheelZoomTool"},{"id":"d38072b7-0b06-44de-ad8f-fcc8d4241ed7","type":"BoxZoomTool"},{"id":"80fcd317-29e5-4e02-a1bf-fd04d172fae4","type":"SaveTool"},{"id":"830d5009-db7c-4ad5-a596-52e988bdde5d","type":"ResetTool"},{"id":"4a20e507-2e62-4c80-a9ca-f95daecd276e","type":"HelpTool"}]},"id":"50419f46-8d3a-493b-a3d6-3a31ed9c18eb","type":"Toolbar"},{"attributes":{"data_source":{"id":"c5c5b400-d6ee-4ac1-b77e-60115007f40d","type":"ColumnDataSource"},"glyph":{"id":"6eef8ca9-12de-44c1-be34-bb59728cc673","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"792774c6-3f42-41fd-a72c-30585fc97da1","type":"GlyphRenderer"},{"attributes":{"data_source":{"id":"ce485a8b-895c-43e8-95eb-eee6850350af","type":"ColumnDataSource"},"glyph":{"id":"d1f6fc87-f9c6-40c7-b69f-83c08a6e2021","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"41e5fb45-34c4-4cab-bd87-3dc30ec74278","type":"GlyphRenderer"},{"attributes":{"callback":null,"end":26.25},"id":"ed0d5fb6-71c4-47d8-9f90-c050d0c9a5f7","type":"Range1d"},{"attributes":{"data_source":{"id":"072cc80d-37cc-43e6-8f2c-2ae82dba1144","type":"ColumnDataSource"},"glyph":{"id":"4769fcc0-4c29-4335-9827-12021e5ee8e2","type":"Rect"},"hover_glyph":null,"nonselection_glyph":null,"selection_glyph":null},"id":"3053d7d4-2a82-497b-8be1-43340d4f5a77","type":"GlyphRenderer"},{"attributes":{"label":{"value":"outra"},"renderers":[{"id":"cbf1236f-2bc6-4276-93ea-40b679d5136d","type":"GlyphRenderer"}]},"id":"19ee73e5-fb56-4e12-84b3-fea625c9671e","type":"LegendItem"},{"attributes":{"below":[{"id":"c70cdf89-1893-47ec-a81c-2aaa63a67b9b","type":"CategoricalAxis"}],"css_classes":null,"left":[{"id":"412254c3-0625-4673-a8b6-737398a574ca","type":"LinearAxis"}],"renderers":[{"id":"1d57b333-1271-4b65-9ef5-d48671bd2216","type":"BoxAnnotation"},{"id":"3053d7d4-2a82-497b-8be1-43340d4f5a77","type":"GlyphRenderer"},{"id":"cbf1236f-2bc6-4276-93ea-40b679d5136d","type":"GlyphRenderer"},{"id":"792774c6-3f42-41fd-a72c-30585fc97da1","type":"GlyphRenderer"},{"id":"41e5fb45-34c4-4cab-bd87-3dc30ec74278","type":"GlyphRenderer"},{"id":"267e78c8-f7a1-4efa-a45d-537251931781","type":"GlyphRenderer"},{"id":"7b2b9350-145b-4c03-b1bc-18c97a516698","type":"Legend"},{"id":"c70cdf89-1893-47ec-a81c-2aaa63a67b9b","type":"CategoricalAxis"},{"id":"412254c3-0625-4673-a8b6-737398a574ca","type":"LinearAxis"},{"id":"dbeb26b3-e2db-4eab-94fd-39aa8a1c84df","type":"Grid"}],"title":{"id":"4dd65458-8347-4c3b-943e-04a59cfe0f36","type":"Title"},"tool_events":{"id":"b9f84b35-ed6a-4bb7-8f1b-939b2492df53","type":"ToolEvents"},"toolbar":{"id":"50419f46-8d3a-493b-a3d6-3a31ed9c18eb","type":"Toolbar"},"x_mapper_type":"auto","x_range":{"id":"4813f957-93df-421a-b0dc-350c85824a21","type":"FactorRange"},"y_mapper_type":"auto","y_range":{"id":"ed0d5fb6-71c4-47d8-9f90-c050d0c9a5f7","type":"Range1d"}},"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"},{"attributes":{"overlay":{"id":"1d57b333-1271-4b65-9ef5-d48671bd2216","type":"BoxAnnotation"},"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"}},"id":"d38072b7-0b06-44de-ad8f-fcc8d4241ed7","type":"BoxZoomTool"},{"attributes":{"callback":null,"column_names":["x","y","width","height","color","fill_alpha","line_color","line_alpha","label"],"data":{"chart_index":[{"sub_categoria":"nome"}],"color":["#407ee7"],"fill_alpha":[0.8],"height":[17.0],"label":[{"sub_categoria":"nome"}],"line_alpha":[1.0],"line_color":["white"],"sub_categoria":["nome"],"width":[0.8],"x":["nome"],"y":[8.5]}},"id":"a956fbd5-d0c1-4d85-94b7-01ca8c6e117d","type":"ColumnDataSource"},{"attributes":{"axis_label":"Sub_Categoria","formatter":{"id":"1a1d087f-afc8-4dc9-86d5-32a785ea0e9c","type":"CategoricalTickFormatter"},"major_label_orientation":0.7853981633974483,"plot":{"id":"0c91d077-052b-4850-9b0b-ab3e6cb810e8","subtype":"Chart","type":"Plot"},"ticker":{"id":"21b51f20-655b-407b-9476-a97aed01a726","type":"CategoricalTicker"}},"id":"c70cdf89-1893-47ec-a81c-2aaa63a67b9b","type":"CategoricalAxis"}],"root_ids":["0c91d077-052b-4850-9b0b-ab3e6cb810e8"]},"title":"Bokeh Application","version":"0.12.4"}};
            var render_items = [{"docid":"b2576041-c9d7-42f8-89d5-78d65d73f702","elementid":"69182cd7-c414-4bfd-977a-e3a747ae139e","modelid":"0c91d077-052b-4850-9b0b-ab3e6cb810e8"}];

            Bokeh.embed.embed_items(docs_json, render_items);
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
      },
      function(Bokeh) {
      }
    ];

    function run_inline_js() {

      if ((window.Bokeh !== undefined) || (force === true)) {
        for (var i = 0; i < inline_js.length; i++) {
          inline_js[i](window.Bokeh);
        }if (force === true) {
          display_loaded();
        }} else if (Date.now() < window._bokeh_timeout) {
        setTimeout(run_inline_js, 100);
      } else if (!window._bokeh_failed_load) {
        console.log("Bokeh: BokehJS failed to load within specified timeout.");
        window._bokeh_failed_load = true;
      } else if (force !== true) {
        var cell = $(document.getElementById("69182cd7-c414-4bfd-977a-e3a747ae139e")).parents('.cell').data().cell;
        cell.output_area.append_execute_result(NB_LOAD_WARNING)
      }

    }

    if (window._bokeh_is_loading === 0) {
      console.log("Bokeh: BokehJS loaded, going straight to plotting");
      run_inline_js();
    } else {
      load_libs(js_urls, function() {
        console.log("Bokeh: BokehJS plotting callback run at", now());
        run_inline_js();
      });
    }
  }(this));
</script>


## Categoria 'outra'
Esta categoria pode ser refinada, mas exige uma análise mais elaborda, então só vamos fazer isso no futuro


```python
leis[leis['categoria_geral'] == 'outra'].head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>numero</th>
      <th>ano</th>
      <th>ementa</th>
      <th>categoria_geral</th>
      <th>sub_categoria</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>1017</td>
      <td>1992</td>
      <td>insere parágrafo único do art. 1º da lei n. 97...</td>
      <td>outra</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1018</td>
      <td>1992</td>
      <td>autorizo o poder executivo a firmar acordo de ...</td>
      <td>outra</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1027</td>
      <td>1992</td>
      <td>oferece incentivos sobre iptu e dá outras prov...</td>
      <td>outra</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1028</td>
      <td>1992</td>
      <td>da denominação a prédios e logradouros públicos.</td>
      <td>outra</td>
      <td>outra</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1030</td>
      <td>1992</td>
      <td>orça a receita e fixa a despesa do município p...</td>
      <td>outra</td>
      <td>outra</td>
    </tr>
  </tbody>
</table>
</div>



## Considerações finais
Os resultados foram satisfatórios para uma classificação simples, agora com este dataset com labels já é possível treinar um modelo para classificar as Leis de 2017

Em um segundo momento vamos trabalhar outros aspectos do processamento de linguagem natural para classificar as Leis da categoria 'outra'


```python
leis['ementa'] = leis.apply(lambda row: (row['ementa']).capitalize(), axis=1)
leis.to_csv('Leis_2016_1992_categorizadas.csv')
```
