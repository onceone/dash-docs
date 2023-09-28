# ![](/assets/images/language_icons/python_50px.svg) A Minimal Dash App

A minimal Dash app will typically look like this:



```python
from dash import Dash, html, dcc, callback, Output, Input
import plotly.express as px
import pandas as pd

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder_unfiltered.csv')

app = Dash(__name__)

app.layout = html.Div([
    html.H1(children='Title of Dash App', style={'textAlign':'center'}),
    dcc.Dropdown(df.country.unique(), 'Canada', id='dropdown-selection'),
    dcc.Graph(id='graph-content')
])

@callback(
    Output('graph-content', 'figure'),
    Input('dropdown-selection', 'value')
)
def update_graph(value):
    dff = df[df.country==value]
    return px.line(dff, x='year', y='pop')

if __name__ == '__main__':
    app.run(debug=True)

``` 
# Title of Dash App

Canada

×

19501960197019801990200015M20M25M30M

yearpop

[ plotly-logomark ](https://plotly.com/)

复制上述代码到一个新的Py文件`app.py`,在终端输入`python app.py`,得到一个http链接。

```
Dash is running on http://127.0.0.1:8050/

* Serving Flask app 'app' (lazy loading)
* Environment: production
  WARNING: This is a development server. Do not use it in a production deployment Use a production WSGI server instead.
* Debug mode: on
* Running on http://127.0.0.1:8050 (Press CTRL+C to quit)
```

也可以在Jupyter Notebook中运行上述代码。

下一小节将覆盖Dash的主要内容。

The next section will cover the main elements of a Dash app. [Dash in 20
minutes tutorial](/tutorial)!

> These docs are a Dash app running on Dash Enterprise on Azure Kubernetes
> Service.
>
> Write, deploy, and scale Dash apps on a Dash Enterprise Kubernetes cluster.
>
> [Learn
> More](https://plotly.com/dash/?utm_medium=dash_docs&utm_content=minimal-app)
> | [Pricing](https://plotly.com/get-
> pricing/?utm_medium=dash_docs&utm_content=minimal-app) | [Dash Enterprise
> Demo](https://plotly.com/get-demo/?utm_medium=dash_docs&utm_content=minimal-
> app) | [Dash Enterprise
> Overview](https://plotly.com/dash/?utm_medium=dash_docs&utm_content=minimal-
> app)

