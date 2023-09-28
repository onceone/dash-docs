# Dash 速览

在本教程结束时，您将了解Dash的基本构建块，您将知道如何构建此应用程序

```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
import pandas as pd
import plotly.express as px

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app
app = Dash(__name__)

# App layout
app.layout = html.Div([
    html.Div(children='My First App with Data, Graph, and Controls'),
    html.Hr(),
    dcc.RadioItems(options=['pop', 'lifeExp', 'gdpPercap'], value='lifeExp', id='my-final-radio-item-example'),
    dash_table.DataTable(data=df.to_dict('records'), page_size=6),
    dcc.Graph(figure={}, id='my-final-graph-example')
])

# Add controls to build the interaction
@callback(
    Output(component_id='my-final-graph-example', component_property='figure'),
    Input(component_id='my-final-radio-item-example', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig

# Run the app
if __name__ == '__main__':
app.run(debug=True)

```
My First App with Data, Graph, and Controls
我的第一个应用程序与数据，图形和控件

* * *

[ plotly-logomark ](https://plotly.com/)

## Hello World
使用Dash构建和启动应用程序只需7行代码即可完成

在计算机上打开Python IDE，用下面的代码创建一个“app.py”文件，如果还没有安装Dash的话，安装Dash。要启动应用程序，在终端输入命令' python app.py '。然后，进入http链接

或者，在Dash 2.11或更高版本中，您可以运行此应用程序和本文档中的其他示例

下面的代码创建了一个非常小的“Hello World”Dash应用

```python
from dash import Dash, html

app = Dash(__name__)

app.layout = html.Div([
    html.Div(children='Hello World')
])

if __name__ == '__main__':
    app.run(debug=True)
```
Hello World

Follow this example gif (using VS Code) if you are not sure how to set up the
app:



![img](/home/v/Project/workproject/test/test_dash_tran/mynewbook/content/0%E5%BF%AB%E9%80%9F%E5%85%A5%E9%97%A8/2tutorial.assets/dash-in-20-tutorial.gif)





https://dash.plotly.com/assets/images/dash_in_20/dash-in-20-tutorial.gif

### Hello World: 代码分解


```python
# Import packages
from dash import Dash, html
```
在创建Dash应用程序时，您几乎总是会使用上面的import语句。当您创建更高级的Dash应用程序时，您将导入更多的包
```python
# Initialize the app
app = Dash(__name__)
```
这一行被称为Dash构造函数，负责初始化您的应用程序。对于您创建的任何Dash应用程序，它几乎总是相同的

```python
# App layout
app.layout = html.Div([
    html.Div(children='Hello World')
])
```
应用程序布局表示将在web浏览器中显示的应用程序组件，通常包含在' html.Div '中。在这个例子中，只添加了一个组件:另一个' html.Div '。Div有一些属性，比如' children '，我们用它来向页面添加文本内容:"Hello World "

```python
# Run the app
if __name__ == '__main__':
    app.run(debug=True)
```
这些行是用于运行应用程序的，对于你创建的任何Dash应用程序，它们几乎总是相同的

## 连接数据

有很多方法可以将数据添加到应用程序中:api、外部数据库、本地“。txt”文件、JSON文件等等。在本例中，我们将重点介绍从CSV工作表中合并数据的最常用方法之一

用下面的代码替换上一节中的“app.py”代码。然后，安装pandas (' pip install pandas ')并启动应用程序
    
```python
# Import packages
from dash import Dash, html, dash_table
import pandas as pd

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app
app = Dash(__name__)

# App layout
app.layout = html.Div([
    html.Div(children='My First App with Data'),
    dash_table.DataTable(data=df.to_dict('records'), page_size=10)
])

# Run the app
if __name__ == '__main__':
    app.run(debug=True)
```

My First App with Data


### 连接数据: 代码分解

```python
# Import packages
from dash import Dash, html, dash_table
import pandas as pd
```
我们导入' dash_table '模块来在dash DataTable中显示数据。我们还导入pandas库以读取CSV工作表数据


```python
# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')
```
* 这里我们将CSV表读入pandas dataframe。这将使切片、过滤和检查数据变得更容易
* 如果您更喜欢使用计算机上(而不是在线)的CSV表，请确保将其保存在包含“app.py”文件的同一文件夹中。然后，将这行代码更新为:' df = pd.read_csv(“NameOfYourFile.csv”)
* 如果你使用的是Excel表格，请确保“pip install openpyxl”。然后，将这行代码更新为:' df = pd.read_excel('NameOfYourFile.xlsx'，工作表名称='Sheet1')



>  **Tip** : You can read the [pandas docs](https://pandas.pydata.org/pandas-
> docs/stable/reference/io.html) on reading data if your data is in a
> different format, or consider using another Python library if you are
> connecting to a specific database type or file format. For example, if
> you're considering using Databricks as a backend for your Dash app, you may
> review their Python documentation for recommendations on how to connect.


```python
# App layout
app.layout = html.Div([
    html.Div(children='My First App with Data'),
    dash_table.DataTable(data=df.to_dict('records'), page_size=10)
])
```
* 除了应用标题之外，我们还添加了DataTable组件，并将pandas数据框读入表中


## 可视化数据

Plotly绘图库有超过50种图表类型可供选择(https://plotly.com/python/)。在这个例子中，我们将使用直方图

用下面的代码替换上一节中的“app.py”代码

```python
# Import packages
from dash import Dash, html, dash_table, dcc
import pandas as pd
import plotly.express as px

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app
app = Dash(__name__)

# App layout
app.layout = html.Div([
    html.Div(children='My First App with Data and a Graph'),
    dash_table.DataTable(data=df.to_dict('records'), page_size=10),
    dcc.Graph(figure=px.histogram(df, x='continent', y='lifeExp', histfunc='avg'))
])

# Run the app
if __name__ == '__main__':
    app.run(debug=True)
```

My First App with Data and a Graph



AsiaEuropeAfricaAmericasOceania01020304050607080

continentavg of lifeExp

[ plotly-logomark ](https://plotly.com/)

### 可视化数据: 代码分解

```python
# Import packages
from dash import Dash, html, dash_table, dcc
import pandas as pd
import plotly.express as px
```
* 我们导入“dcc”模块(dcc代表Dash Core Components)。该模块包括一个名为“dcc”的图形组件。，用于呈现交互式图形
* 我们还导入了“plot”。Express的库来构建交互图形

```python
    # App layout
    app.layout = html.Div([
        html.Div(children='My First App with Data and a Graph'),
        dash_table.DataTable(data=df.to_dict('records'), page_size=10),
        dcc.Graph(figure=px.histogram(df, x='continent', y='lifeExp', histfunc='avg'))
    ])
```
* 使用“plot”。express库中，我们构建了直方图，并将其赋值给dcc.Graph的“figure”属性。这将在我们的应用程序中显示直方图
  

## 控件和回调
到目前为止，您已经构建了一个显示表格数据和图形的静态应用程序。然而，当你开发更复杂的Dash应用程序时，你可能会希望给应用程序用户更多的自由来与应用程序**交互**，并更深入地探索数据。要实现这一点，你需要**使用回调函数向应用程序添加控件**

在这个例子中，我们将添加**单选按钮**到应用程序布局。然后，我们将构建回调来创建单选按钮和直方图之间的交互

```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
import pandas as pd
import plotly.express as px

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app
app = Dash(__name__)

# App layout
app.layout = html.Div([
    html.Div(children='My First App with Data, Graph, and Controls'),
    html.Hr(),
    dcc.RadioItems(options=['pop', 'lifeExp', 'gdpPercap'], value='lifeExp', id='controls-and-radio-item'),
    dash_table.DataTable(data=df.to_dict('records'), page_size=6),
    dcc.Graph(figure={}, id='controls-and-graph')
])

# Add controls to build the interaction
@callback(
    Output(component_id='controls-and-graph', component_property='figure'),
    Input(component_id='controls-and-radio-item', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig

# Run the app
if __name__ == '__main__':
    app.run(debug=True)

```
My First App with Data, Graph, and Controls

* * *



AsiaEuropeAfricaAmericasOceania01020304050607080

continentavg of lifeExp

[ plotly-logomark ](https://plotly.com/)

### 控制:代码分解


```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
```
* 我们像在前一节中那样导入' dcc '，以使用' dcc. graph '。在这个例子中，我们需要' dcc '代替' dcc '。图形'以及单选按钮组件' dcc。RadioItems
* 为了在Dash应用中使用回调，我们导入了' callback '模块和回调中常用的两个参数:' Output '和' Input '

```python 
# App layout
app.layout = html.Div([
    html.Div(children='My First App with Data, Graph, and Controls'),
    html.Hr(),
    dcc.RadioItems(options=['pop', 'lifeExp', 'gdpPercap'], value='lifeExp', id='controls-and-radio-item'),
    dash_table.DataTable(data=df.to_dict('records'), page_size=6),
    dcc.Graph(figure={}, id='controls-and-graph')
])
```

* 注意，我们将' RadioItems '组件添加到布局中，就在DataTable的正上方。有三个选项，每个单选按钮一个。' lifeExp '选项被分配给' value '属性，使其成为当前选择的值
* “RadioItems”和“Graph”组件都被赋予了“id”名称:这些名称将被回调函数用来标识组件
  
```python
# Add controls to build the interaction
@callback(
    Output(component_id='controls-and-graph', component_property='figure'),
    Input(component_id='controls-and-radio-item', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig
```

* 我们应用程序的输入和输出是特定组件的属性。在这个例子中，我们的输入是ID为“controls-and-radio-item”的组件的“value”属性。如果你回头看看布局，你会发现这是当前的“lifeExp”。我们的输出是ID为“controls-and-graph”的组件的“figure”属性，它目前是一个空字典(空图)
* 回调函数的参数' colchosen '引用了输入' lifeExp '的组件属性。我们在回调函数中构建直方图，将选择的单选项分配给直方图的y轴属性。这意味着每次用户选择一个新的单选项时，都会重建图形并更新图形的y轴
* 最后，我们在函数的末尾返回直方图。这将直方图分配给“dcc”的“figure”属性。，从而在应用程序中显示该图形


## 设计你的应用
上一节中的示例使用Dash HTML组件构建了一个简单的应用程序布局，但是你可以将应用程序设计得更专业。本节将简要介绍可以用来增强Dash应用程序布局风格的多种工具

* HTML and CSS
* Dash Design Kit (DDK)
* Dash Bootstrap Components
* Dash Mantine Components

### HTML and CSS

HTML和CSS是在web上呈现内容的最低级别的接口。HTML是一组组件，CSS是应用于这些组件的一组样式。CSS样式可以通过' style '属性在组件中应用，也可以通过' className '属性将其定义为单独的CSS文件，如下面的例子所示

```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
import pandas as pd
import plotly.express as px

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app - incorporate css
external_stylesheets = ['https://codepen.io/chriddyp/pen/bWLwgP.css']
app = Dash(__name__, external_stylesheets=external_stylesheets)

# App layout
app.layout = html.Div([
    html.Div(className='row', children='My First App with Data, Graph, and Controls',
                style={'textAlign': 'center', 'color': 'blue', 'fontSize': 30}),

    html.Div(className='row', children=[
        dcc.RadioItems(options=['pop', 'lifeExp', 'gdpPercap'],
                        value='lifeExp',
                        inline=True,
                        id='my-radio-buttons-final')
    ]),

    html.Div(className='row', children=[
        html.Div(className='six columns', children=[
            dash_table.DataTable(data=df.to_dict('records'), page_size=11, style_table={'overflowX': 'auto'})
        ]),
        html.Div(className='six columns', children=[
            dcc.Graph(figure={}, id='histo-chart-final')
        ])
    ])
])

# Add controls to build the interaction
@callback(
    Output(component_id='histo-chart-final', component_property='figure'),
    Input(component_id='my-radio-buttons-final', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig

# Run the app
if __name__ == '__main__':
    app.run(debug=True)
```

My First App with Data, Graph, and Controls



AsiaEuropeAfricaAmericasOceania01020304050607080

continentavg of lifeExp

[ plotly-logomark ](https://plotly.com/)

### Dash Design Kit (DDK)
Dash设计工具包是我们的高级UI框架，是专门为Dash构建的。使用Dash Design Kit，您不需要使用HTML或CSS。默认情况下，应用程序是移动响应的，所有内容都是主题化的。Dash Design Kit被授权为Dash Enterprise的一部分，并得到Plotly的官方支持

下面是一个使用Dash Design Kit的示例(请注意，如果没有Dash Enterprise许可证，您将无法运行此示例)

```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
import pandas as pd
import plotly.express as px
import dash_design_kit as ddk

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app
app = Dash(__name__)

# App layout
app.layout = ddk.App([
    ddk.Header(ddk.Title('My First App with Data, Graph, and Controls')),
    dcc.RadioItems(options=['pop', 'lifeExp', 'gdpPercap'],
                    value='lifeExp',
                    inline=True,
                    id='my-ddk-radio-items-final'),
    ddk.Row([
        ddk.Card([
            dash_table.DataTable(data=df.to_dict('records'), page_size=12, style_table={'overflowX': 'auto'})
        ], width=50),
        ddk.Card([
            ddk.Graph(figure={}, id='graph-placeholder-ddk-final')
        ], width=50),
    ]),

])

# Add controls to build the interaction
@callback(
    Output(component_id='graph-placeholder-ddk-final', component_property='figure'),
    Input(component_id='my-ddk-radio-items-final', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig

# Run the app
if __name__ == '__main__':
    app.run(debug=True)
```

My First App with Data, Graph, and Controls

p

AsiaEuropeAfricaAmericasOceania01020304050607080

continentavg of lifeExp

[ plotly-logomark ](https://plotly.com/)

### Dash Bootstrap Components

Dash Bootstrap是一个社区维护的库，建立在Bootstrap组件系统之上。虽然它不是由Plotly官方维护或支持，但Dash Bootstrap是构建优雅应用程序布局的强大方式。注意，我们首先定义一行，然后使用' dbc定义行内列的宽度。行'和' dbc。上校的组件

要使下面的应用程序成功运行，请确保安装Dash Bootstrap Components库

> Read more about the Dash Bootstrap Components in the [third-party
> documentation](https://dash-bootstrap-
> components.opensource.faculty.ai/docs/components/).

```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
import pandas as pd
import plotly.express as px
import dash_bootstrap_components as dbc

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app - incorporate a Dash Bootstrap theme
external_stylesheets = [dbc.themes.CERULEAN]
app = Dash(__name__, external_stylesheets=external_stylesheets)

# App layout
app.layout = dbc.Container([
    dbc.Row([
        html.Div('My First App with Data, Graph, and Controls', className="text-primary text-center fs-3")
    ]),

    dbc.Row([
        dbc.RadioItems(options=[{"label": x, "value": x} for x in ['pop', 'lifeExp', 'gdpPercap']],
                        value='lifeExp',
                        inline=True,
                        id='radio-buttons-final')
    ]),

    dbc.Row([
        dbc.Col([
            dash_table.DataTable(data=df.to_dict('records'), page_size=12, style_table={'overflowX': 'auto'})
        ], width=6),

        dbc.Col([
            dcc.Graph(figure={}, id='my-first-graph-final')
        ], width=6),
    ]),

], fluid=True)

# Add controls to build the interaction
@callback(
    Output(component_id='my-first-graph-final', component_property='figure'),
    Input(component_id='radio-buttons-final', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig

# Run the app
if __name__ == '__main__':
    app.run(debug=True)
```

My First App with Data, Graph, and Controls

p

AsiaEuropeAfricaAmericasOceania01020304050607080

continentavg of lifeExp

[ plotly-logomark ](https://plotly.com/)

### Dash Mantine Components
Dash Mantine是一个社区维护的库，建立在Mantine组件系统之上。虽然它不是由Plotly团队正式维护或支持，但Dash Mantine是另一种自定义应用程序布局的强大方式。Dash Mantine Components使用Grid模块来构建布局。我们不定义行，而是定义dmc。Grid '，我们在其中插入' dmc。并通过给' span '属性赋值一个数字来定义它们的宽度

要使下面的应用程序成功运行，请确保安装Dash Mantine Components库
Components library: `pip install dash-mantine-components`

> Read more about the Dash Mantine Components in the [third-party
> documentation](https://www.dash-mantine-components.com/).

```python
# Import packages
from dash import Dash, html, dash_table, dcc, callback, Output, Input
import pandas as pd
import plotly.express as px
import dash_mantine_components as dmc

# Incorporate data
df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/gapminder2007.csv')

# Initialize the app - incorporate a Dash Mantine theme
external_stylesheets = [dmc.theme.DEFAULT_COLORS]
app = Dash(__name__, external_stylesheets=external_stylesheets)

# App layout
app.layout = dmc.Container([
    dmc.Title('My First App with Data, Graph, and Controls', color="blue", size="h3"),
    dmc.RadioGroup(
            [dmc.Radio(i, value=i) for i in  ['pop', 'lifeExp', 'gdpPercap']],
            id='my-dmc-radio-item',
            value='lifeExp',
            size="sm"
        ),
    dmc.Grid([
        dmc.Col([
            dash_table.DataTable(data=df.to_dict('records'), page_size=12, style_table={'overflowX': 'auto'})
        ], span=6),
        dmc.Col([
            dcc.Graph(figure={}, id='graph-placeholder')
        ], span=6),
    ]),

], fluid=True)

# Add controls to build the interaction
@callback(
    Output(component_id='graph-placeholder', component_property='figure'),
    Input(component_id='my-dmc-radio-item', component_property='value')
)
def update_graph(col_chosen):
    fig = px.histogram(df, x='continent', y=col_chosen, histfunc='avg')
    return fig

# Run the App
if __name__ == '__main__':
    app.run(debug=True)
```

![Dash Mantine app](./assets/images/dash_in_20/dash_mantine_images.png)

## 下一步
恭喜你!您已经学会了构建和设计您的第一个Dash应用程序。这是令人兴奋的Dash学习路径的第一步，它将为您提供构建复杂的Python分析应用程序的技能和工具

为了继续您的学习之旅，我们建议您查看以下资源


* [100 simple examples](https://dash-example-index.herokuapp.com) of Dash components interacting with Plotly graphs
* [Dash Core Components](/dash-core-components)

> These docs are a Dash app running on Dash Enterprise on Azure Kubernetes
> Service.
>
> Write, deploy, and scale Dash apps on a Dash Enterprise Kubernetes cluster.
>
> [Learn
> More](https://plotly.com/dash/?utm_medium=dash_docs&utm_content=tutorial) |
> [Pricing](https://plotly.com/get-
> pricing/?utm_medium=dash_docs&utm_content=tutorial) | [Dash Enterprise
> Demo](https://plotly.com/get-
> demo/?utm_medium=dash_docs&utm_content=tutorial) | [Dash Enterprise
> Overview](https://plotly.com/dash/?utm_medium=dash_docs&utm_content=tutorial)

