# ![](/assets/images/language_icons/python_50px.svg) Duplicate Callback
Outputs

> New in Dash 2.9: Dash supports the `allow_duplicate=True` argument to allow
> multiple callbacks to target the same output. See the "Setting
> `allow_duplicate` on Duplicate Outputs" example below.

A duplicate callback output is when the same component-property pair is an
`Output` on more than one callback. A component-property pair means the `id`
of the component and the `property`. In this `html.H1` component, the
component-property pair is `'app-heading', 'children'`:

    
    
    html.H1(children='Analytics app', id='app-heading')

If you add `'app-heading', 'children'` as an `Output` on two callbacks in your
app you'll get a "Duplicate callback outputs" error when running in debug mode
as the the default behavior of callbacks is that a component/property pair can
only be the `Output` of one callback.

![Duplicate callback outputs](./..//assets/images/input-
triggered/duplicate_outputs.jpg)

## Duplicate Callback Outputs Example

This is a complete example demonstrating an app that throws a "Duplicate
callback outputs" error. Here we have two `html.Button` components and a
`dcc.Graph` component in our `app.layout`:

    
    
    app.layout = html.Div([
        html.Button('Draw Graph', id='draw'),
        html.Button('Reset Graph', id='reset'),
        dcc.Graph(id='our-graph')
    ])

If we try to update the graph component's (`'our-graph'`) `'fig'` property
differently depending on which button is clicked, we _cannot_ do this by
adding `'our-graph'`, `'fig'` as an `Output` on two callbacks:

    
    
    @callback(
        Output('our-graph', 'figure'),
        Input('draw', 'n_clicks'),
        prevent_initial_call=True
    )
    def draw_graph(_):
        df = px.data.iris()
        return px.scatter(df, x=df.columns[0], y=df.columns[1])
    
    @callback(
        Output('our-graph', 'figure'),
        Input('reset', 'n_clicks'),
        prevent_initial_call=True
    )
    def reset_graph(_):
        return go.Figure()

## Updating the Same Output From Different Inputs

We can update our graph in the above example differently based on the inputs
by combining the inputs into one callback and using `dash.callback_context` to
determine which input triggered the callback.

Here is the same example rewritten. Our new callback has the two inputs. It
gets the ID of the component that triggered the callback using
`ctx.triggered_id`. This will return either `draw` or `reset`. If it is
`draw`, the callback returns `draw_graph()`. If it is `reset` it returns
`reset_graph()`:

 __

    
    
     from dash import Dash, Input, Output, ctx, html, dcc, callback
    import plotly.express as px
    import plotly.graph_objects as go
    
    app = Dash(__name__)
    
    app.layout = html.Div([
        html.Button('Draw Graph', id='draw'),
        html.Button('Reset Graph', id='reset'),
        dcc.Graph(id='graph')
    ])
    
    @callback(
        Output('graph', 'figure'),
        Input('reset', 'n_clicks'),
        Input('draw', 'n_clicks'),
        prevent_initial_call=True
    )
    def update_graph(b1, b2):
        triggered_id = ctx.triggered_id
        print(triggered_id)
        if triggered_id == 'reset':
             return reset_graph()
        elif triggered_id == 'draw':
             return draw_graph()
    
    def draw_graph():
        df = px.data.iris()
        return px.scatter(df, x=df.columns[0], y=df.columns[1])
    
    def reset_graph():
        return go.Figure()
    
    app.run(debug=True)

Draw GraphReset Graph

−10123456−101234

[ plotly-logomark ](https://plotly.com/)

See [Determining Which Callback Input Changed](/determining-which-callback-
input-changed) for more on `dash.callback_context` and understanding which
input triggered a callback.

## Setting `allow_duplicate` on Duplicate Outputs

In Dash 2.9 and later, you can set `allow_duplicate=True` on a callback output
if it is already used as an output on another callback. This will allow you to
target the same component-property from different callbacks. Here is the
earlier example rewritten to allow duplicates.

> When setting `allow_duplicate=True` on a callback output, you'll need to
> either set `prevent_initial_call=True` on the callback, or set `app =
> Dash(prevent_initial_callbacks="initial_duplicate")` on your app. This
> prevents callbacks that target the same output running at the same time when
> the page initially loads. `allow_duplicate` is common when using the [Patch
> object for Partial Property Updates](/partial-properties).

 __

    
    
     from dash import Dash, Input, Output, html, dcc, callback
    import plotly.express as px
    import plotly.graph_objects as go
    
    app = Dash(__name__)
    
    app.layout = html.Div([
        html.Button('Draw Graph', id='draw-2'),
        html.Button('Reset Graph', id='reset-2'),
        dcc.Graph(id='duplicate-output-graph')
    ])
    
    @callback(
        Output('duplicate-output-graph', 'figure', allow_duplicate=True),
        Input('draw-2', 'n_clicks'),
        prevent_initial_call=True
    )
    def draw_graph(n_clicks):
        df = px.data.iris()
        return px.scatter(df, x=df.columns[0], y=df.columns[1])
    
    @callback(
        Output('duplicate-output-graph', 'figure'),
        Input('reset-2', 'n_clicks'),
    )
    def reset_graph(input):
        return go.Figure()
    
    app.run(debug=True)

Draw GraphReset Graph

−10123456−101234

[ plotly-logomark ](https://plotly.com/)

Where you have multiple callbacks targeting the same output, and they both run
at the same time, the order in which updates happen is not guaranteed. This
may be an issue if you are updating the same part of the property from each
callback. For example, a callback output that updates the entire figure and
another one that updates the figure layout.

