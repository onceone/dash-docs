# ![](/assets/images/language_icons/python_50px.svg) Long Callbacks

> We recommend using [background callbacks with @dash.callback(...,
> background=True)](/background-callbacks) instead of `long_callback`. The
> `background=True` argument was introduced in Dash 2.6.0 and addresses
> several limitations with `long_callback`, including its incompatibility with
> dynamically added components and with [Pattern Matching Callbacks](/pattern-
> matching-callbacks). `long_callback` is still safe to use but no longer
> recommended for versions of Dash later than 2.6.0.

Most web servers have a 30 second timeout by default, which is an issue for
callbacks that take longer to complete. While you can increase the timeout on
the web server, you risk allowing long-running callbacks to use all of your
app's workers, preventing other requests from going through. Long callbacks
offer a scalable solution for using long-running callbacks.

Long callbacks can be added to your apps with the decorator
`@app.long_callback`. Callbacks with this decorator use a backend configured
by you to run the callback logic. There are currently two options:

  * A [DiskCache](http://www.grantjenks.com/docs/diskcache/index.html) backend that runs callback logic in a separate process and stores the results to disk using the `diskcache` library. This is the easiest backend to use for local development, but is **not recommended for production**.

  * A [Celery](https://docs.celeryproject.org/en/stable/getting-started/introduction.html) backend that runs callback logic in a Celery worker and returns results to the Dash app through a Celery broker like [RabbitMQ](https://www.rabbitmq.com/) or [Redis](https://redis.io/). This is recommended for production as, unlike Disk Cache, it uses a job queue and won't accidentally consume all your server's resources. For further information on the benefits of job queues, see the **Why Job Queues?** section below.

> [Dash Enterprise](http://plotly.com/dash) makes it easy to deploy Celery and
> Redis for using long callbacks in production. [Get
> Pricing](https://plotly.com/get-
> pricing/?utm_medium=dash_docs&utm_content=long-callbacks) or see Dash in
> action at our next [demo session](https://plotly.com/get-
> demo/?utm_medium=dash_docs&utm_content=long-callbacks).

## Getting Started

The examples below use the `diskcache` manager, which requires `diskcache`,
`multiprocess`, and `psutil` libraries:

`$ pip install diskcache multiprocess psutil`

## Basic Steps

The first step to use long callbacks is to configure a long callback manager
instance using your chosen backend.

The `@app.long_callback` decorator requires this long callback manager
instance. You can provide the manager instance to the dash.Dash app
constructor as the `long_callback_manager` keyword argument, or as the
`manager` argument to the `@app.long_callback` decorator itself.

The `@app.long_callback` decorator supports the same arguments as the normal
`@app.callback` decorator, but also includes support for several additional
optional arguments explained further below: `manager`, `running`, `cancel`,
`progress`, and `progress_default`.

In the next five examples, we'll discuss in more detail how to implement long
callbacks.

## Example 1: Simple Example

Here is a simple example of using the `@app.long_callback` decorator to
register a callback function that updates an `html.P` element with the number
of times that a button has been clicked. The callback uses `time.sleep` to
simulate a long-running operation.

    
    
    import time
    import dash
    from dash import html
    from dash.long_callback import DiskcacheLongCallbackManager
    from dash.dependencies import Input, Output
    
    ## Diskcache
    import diskcache
    cache = diskcache.Cache("./cache")
    long_callback_manager = DiskcacheLongCallbackManager(cache)
    
    app = dash.Dash(__name__)
    
    app.layout = html.Div(
        [
            html.Div([html.P(id="paragraph_id", children=["Button not clicked"])]),
            html.Button(id="button_id", children="Run Job!"),
        ]
    )
    
    @app.long_callback(
        output=Output("paragraph_id", "children"),
        inputs=Input("button_id", "n_clicks"),
        manager=long_callback_manager,
    )
    def callback(n_clicks):
        time.sleep(2.0)
        return [f"Clicked {n_clicks} times"]
    
    
    if __name__ == "__main__":
        app.run(debug=True)

![Simple example](./assets/images/long-callbacks/long-callbacks-example-1.gif)

## Example 2: Disable Button While Callback Is Running

Notice how in the previous example, there is no visual indication that the
long callback is running. A user might click the "Run Job!" button multiple
times before the original job can complete. We can address these shortcomings
by disabling the button while the callback is running, and re-enabling it when
the callback completes.

This is accomplished using the `running` argument to `@app.long_callback`.
This argument accepts a list of 3-element tuples. The first element of each
tuple must be an `Output` dependency object referencing a property of a
component in the app layout. The second element is the value that the property
should be set to while the callback is running, and the third element is the
value the property should be set to when the callback completes.

This example uses `running` to set the `disabled` property of the button to
`True` while the callback is running, and `False` when it completes.

In this example, the long callback manager is provided to the `dash.Dash` app
constructor instead of the `@app.long_callback` decorator.

    
    
    import time
    import dash
    from dash import html
    from dash.long_callback import DiskcacheLongCallbackManager
    from dash.dependencies import Input, Output
    
    ## Diskcache
    import diskcache
    
    cache = diskcache.Cache("./cache")
    long_callback_manager = DiskcacheLongCallbackManager(cache)
    
    app = dash.Dash(__name__, long_callback_manager=long_callback_manager)
    
    app.layout = html.Div(
        [
            html.Div([html.P(id="paragraph_id", children=["Button not clicked"])]),
            html.Button(id="button_id", children="Run Job!"),
        ]
    )
    
    @app.long_callback(
        output=Output("paragraph_id", "children"),
        inputs=Input("button_id", "n_clicks"),
        running=[
            (Output("button_id", "disabled"), True, False),
        ],
    )
    def callback(n_clicks):
        time.sleep(2.0)
        return [f"Clicked {n_clicks} times"]
    
    
    if __name__ == "__main__":
        app.run(debug=True)

![Disable button while callback is running example](./assets/images/long-
callbacks/long-callbacks-example-2.gif)

## Example 3: Cancelable Callback

This example builds on the previous example, adding support for canceling a
long-running callback using the `cancel` argument to the `@app.long_callback`
decorator. We set the `cancel` argument to a list of `Input` dependency
objects that reference a property of a component in the app's layout. When the
value of this property changes while a callback is running, the callback is
canceled. Note that the value of the property is not significant — any change
in value cancels the running job (if any).

    
    
    import time
    import dash
    from dash import html
    from dash.long_callback import DiskcacheLongCallbackManager
    from dash.dependencies import Input, Output
    
    ## Diskcache
    import diskcache
    
    cache = diskcache.Cache("./cache")
    long_callback_manager = DiskcacheLongCallbackManager(cache)
    
    app = dash.Dash(__name__, long_callback_manager=long_callback_manager)
    
    app.layout = html.Div(
        [
            html.Div([html.P(id="paragraph_id", children=["Button not clicked"])]),
            html.Button(id="button_id", children="Run Job!"),
            html.Button(id="cancel_button_id", children="Cancel Running Job!"),
        ]
    )
    
    @app.long_callback(
        output=Output("paragraph_id", "children"),
        inputs=Input("button_id", "n_clicks"),
        running=[
            (Output("button_id", "disabled"), True, False),
            (Output("cancel_button_id", "disabled"), False, True),
        ],
        cancel=[Input("cancel_button_id", "n_clicks")],
    )
    def callback(n_clicks):
        time.sleep(2.0)
        return [f"Clicked {n_clicks} times"]
    
    
    if __name__ == "__main__":
        app.run(debug=True)

![Cancelable callback example](./assets/images/long-callbacks/long-callbacks-
example-3.gif)

## Example 4: Progress Bar

This example uses the `progress` argument to the `@app.long_callback`
decorator to update a progress bar while the callback is running. We set the
`progress` argument to an `Output` dependency grouping that references
properties of components in the app's layout.

When a dependency grouping is assigned to the `progress` argument of
`@app.long_callback`, the decorated function is called with a new special
argument as the first argument to the function. This special argument, named
`set_progress` in the example below, is a function handle that the decorated
function calls in order to provide updates to the app on its current progress.
The `set_progress` function accepts a single argument, which corresponds to
the grouping of properties specified in the `Output` dependency grouping
passed to the `progress` argument of `@app.long_callback`.

    
    
    import time
    import dash
    from dash import html
    from dash.long_callback import DiskcacheLongCallbackManager
    from dash.dependencies import Input, Output
    
    ## Diskcache
    import diskcache
    cache = diskcache.Cache("./cache")
    long_callback_manager = DiskcacheLongCallbackManager(cache)
    
    app = dash.Dash(__name__, long_callback_manager=long_callback_manager)
    
    app.layout = html.Div(
        [
            html.Div(
                [
                    html.P(id="paragraph_id", children=["Button not clicked"]),
                    html.Progress(id="progress_bar"),
                ]
            ),
            html.Button(id="button_id", children="Run Job!"),
            html.Button(id="cancel_button_id", children="Cancel Running Job!"),
        ]
    )
    
    @app.long_callback(
        output=Output("paragraph_id", "children"),
        inputs=Input("button_id", "n_clicks"),
        running=[
            (Output("button_id", "disabled"), True, False),
            (Output("cancel_button_id", "disabled"), False, True),
            (
                Output("paragraph_id", "style"),
                {"visibility": "hidden"},
                {"visibility": "visible"},
            ),
            (
                Output("progress_bar", "style"),
                {"visibility": "visible"},
                {"visibility": "hidden"},
            ),
        ],
        cancel=[Input("cancel_button_id", "n_clicks")],
        progress=[Output("progress_bar", "value"), Output("progress_bar", "max")],
    )
    def callback(set_progress, n_clicks):
        total = 10
        for i in range(total):
            time.sleep(0.5)
            set_progress((str(i + 1), str(total)))
        return [f"Clicked {n_clicks} times"]
    
    
    if __name__ == "__main__":
        app.run(debug=True,port=8054)

![Progress bar example](./assets/images/long-callbacks/long-callbacks-
example-4.gif)

## Example 5: Progress Bar Chart Graph

The `progress` argument to the `@app.long_callback` decorator can be used to
update arbitrary component properties. This example creates and updates a
Plotly bar graph to display the current calculation status.

This example also uses the `progress_default` argument to `long_callback` to
specify a grouping of values that should be assigned to the components
specified by the `progress` argument when the callback is not in progress. If
`progress_default` is not provided, all the dependency properties specified in
`progress` are set to `None` when the callback is not running. In this case,
`progress_default` is set to a figure with a zero width bar.

    
    
    import time
    import dash
    from dash import html, dcc
    from dash.long_callback import DiskcacheLongCallbackManager
    from dash.dependencies import Input, Output
    import plotly.graph_objects as go
    
    ## Diskcache
    import diskcache
    cache = diskcache.Cache("./cache")
    long_callback_manager = DiskcacheLongCallbackManager(cache)
    
    def make_progress_graph(progress, total):
        progress_graph = (
            go.Figure(data=[go.Bar(x=[progress])])
            .update_xaxes(range=[0, total])
            .update_yaxes(
                showticklabels=False,
            )
            .update_layout(height=100, margin=dict(t=20, b=40))
        )
        return progress_graph
    
    
    app = dash.Dash(__name__, long_callback_manager=long_callback_manager)
    
    app.layout = html.Div(
        [
            html.Div(
                [
                    html.P(id="paragraph_id", children=["Button not clicked"]),
                    dcc.Graph(id="progress_bar_graph", figure=make_progress_graph(0, 10)),
                ]
            ),
            html.Button(id="button_id", children="Run Job!"),
            html.Button(id="cancel_button_id", children="Cancel Running Job!"),
        ]
    )
    
    @app.long_callback(
        output=Output("paragraph_id", "children"),
        inputs=Input("button_id", "n_clicks"),
        running=[
            (Output("button_id", "disabled"), True, False),
            (Output("cancel_button_id", "disabled"), False, True),
            (
                Output("paragraph_id", "style"),
                {"visibility": "hidden"},
                {"visibility": "visible"},
            ),
            (
                Output("progress_bar_graph", "style"),
                {"visibility": "visible"},
                {"visibility": "hidden"},
            ),
        ],
        cancel=[Input("cancel_button_id", "n_clicks")],
        progress=Output("progress_bar_graph", "figure"),
        progress_default=make_progress_graph(0, 10),
        interval=1000,
    )
    def callback(set_progress, n_clicks):
        total = 10
        for i in range(total):
            time.sleep(0.5)
            set_progress(make_progress_graph(i, 10))
    
        return [f"Clicked {n_clicks} times"]
    
    
    if __name__ == "__main__":
        app.run(debug=True)

![Progress bar chart graph example](./assets/images/long-callbacks/long-
callbacks-example-5.gif)

## Example with Celery/Redis

We recommend using a Celery/Redis backend for production environments.

With Celery and Redis, example 3 looks like this:

    
    
    import time
    import dash
    from dash import html
    from dash.long_callback import CeleryLongCallbackManager
    from dash.dependencies import Input, Output
    from celery import Celery
    
    celery_app = Celery(
        __name__, broker="redis://localhost:6379/0", backend="redis://localhost:6379/1"
    )
    long_callback_manager = CeleryLongCallbackManager(celery_app)
    
    app = dash.Dash(__name__, long_callback_manager=long_callback_manager)
    
    app.layout = html.Div(
        [
            html.Div([html.P(id="paragraph_id", children=["Button not clicked"])]),
            html.Button(id="button_id", children="Run Job!"),
            html.Button(id="cancel_button_id", children="Cancel Running Job!"),
        ]
    )
    
    @app.long_callback(
        output=Output("paragraph_id", "children"),
        inputs=Input("button_id", "n_clicks"),
        running=[
            (Output("button_id", "disabled"), True, False),
            (Output("cancel_button_id", "disabled"), False, True),
        ],
        cancel=[Input("cancel_button_id", "n_clicks")],
    )
    def callback(n_clicks):
        time.sleep(2.0)
        return [f"Clicked {n_clicks} times"]
    
    
    if __name__ == "__main__":
        app.run(debug=True)

In place of the `DiskcacheLongCallbackManager`, we use
`CeleryLongCallbackManager` and import Celery using the line: `from celery
import Celery`

We configure a Celery app (`celery_app`) and pass it to the
`CeleryLongCallbackManager`. This is stored in the variable
`long_callback_manager` and then passed to the app:

`app = dash.Dash(__name__, long_callback_manager=long_callback_manager)`

## Caching Results

The `@app.long_callback` decorator can optionally
[memoize](https://en.wikipedia.org/wiki/Memoization) callback function results
through caching, and it provides a flexible API for configuring when cached
results may be reused.

> Caching with long callbacks can help improve your application's response
> time, but if you want users to be able to save and access views of the
> application at a particular point in time, and generate PDF reports from
> these views, use [Dash Enterprise Snapshot
> Engine](https://plotly.com/dash/snapshot-
> engine/?utm_medium=dash_docs&utm_content=long-callbacks). As Dash Enterprise
> Snapshot Engine stores a full record of results, you can track how the
> result for a specific set of parameters changes over time. For long-running
> callbacks where you don't need to have access to past results, use long
> callbacks.

### How It Works

Here is a high-level description of how caching works in `long_callback`.
Conceptually, you can imagine a dictionary is associated with each decorated
callback function. Each time the decorated function is called, the input
arguments to the function (and potentially other information about the
environment) are hashed to generate a key. The `long_callback` decorator then
checks the dictionary to see if there is already a value stored using this
key. If so, the decorated function is not called, and the cached result is
returned. If not, the function is called and the result is stored in the
dictionary using the associated key.

The built-in `functools.lru_cache` decorator uses a Python `dict` just like
this. The situation is slightly more complicated with Dash for two reasons:

  * We might want the cache to persist across server restarts.
  * When an app is served using multiple processes (e.g. multiple `gunicorn` workers on a single server, or multiple servers behind a load balancer), we might want to share cached values across all of these processes.

For these reasons, a simple Python `dict` is not a suitable storage container
for caching Dash callbacks. Instead, `long_callback` uses the current
DiskCache or Celery callback manager to store cached results.

### Caching Flexibility Requirements

To support caching in a variety of development and production use cases,
`long_callback` may be configured by one or more zero-argument functions,
where the return values of these functions are combined with the function
input arguments when generating the cache key. Several common use-cases are
described below.

### Enabling Caching

Caching is enabled by providing one or more zero-argument functions to the
`cache_by` argument of `long_callback`. These functions are called each time
the status of a `long_callback` function is checked, and their return values
are hashed as part of the cache key.

Here is an example using the DiskCache callback manager. In this example, the
`cache_by` argument is set to a `lambda` function that returns a fixed UUID
that is randomly generated during app initialization. The implication of this
`cache_by` function is that the cache is shared across all invocations of the
callback across all user sessions that are handled by a single server
instance. Each time a server process is restarted, the cache is cleared and a
new UUID is generated.

    
    
    import time
    from uuid import uuid4
    import dash
    from dash import html
    from dash.long_callback import DiskcacheLongCallbackManager
    from dash.dependencies import Input, Output
    
    ## Diskcache
    import diskcache
    launch_uid = uuid4()
    cache = diskcache.Cache("./cache")
    long_callback_manager = DiskcacheLongCallbackManager(
        cache, cache_by=[lambda: launch_uid], expire=60,
    )
    
    app = dash.Dash(__name__, long_callback_manager=long_callback_manager)
    app.layout = html.Div(
        [
            html.Div([html.P(id="paragraph_id", children=["Button not clicked"])]),
            html.Button(id="button_id", children="Run Job!"),
            html.Button(id="cancel_button_id", children="Cancel Running Job!"),
        ]
    )
    
    
    @app.long_callback(
        output=(Output("paragraph_id", "children"), Output("button_id", "n_clicks")),
        inputs=Input("button_id", "n_clicks"),
        running=[
            (Output("button_id", "disabled"), True, False),
            (Output("cancel_button_id", "disabled"), False, True),
        ],
        cancel=[Input("cancel_button_id", "n_clicks")],
    )
    def callback(n_clicks):
        time.sleep(2.0)
        return [f"Clicked {n_clicks} times"], (n_clicks or 0) % 4
    
    
    if __name__ == "__main__":
        app.run(debug=True)

![Caching example](./assets/images/long-callbacks/long-callbacks-
example-6.gif)

Here you can see that it takes a few seconds to run the callback function, but
the cached results are used after `n_clicks` cycles back around to 0. By
interacting with the app in a separate tab, you can see that the cached
results are shared across user sessions.

### Omitting Properties from Cache Key Calculation

The `@app.long_callback` decorator has an argument `cache_args_to_skip` that
can be used to omit properties you specify from the cache key calculation. For
example, you likely won't want to include a button's `n_clicks` property in a
cache key because it will have a new value each time it's clicked, and in this
case you could use `cache_args_to_skip`.

If the callback is configured with keyword arguments (`Input/State` provided
in a dict), `cache_args_to_skip` should be a list of argument names as
strings. Otherwise, it should be a list of argument indices as integers. See
the [Flexible Callback Signatures chapter](/flexible-callback-
signatures#keyword-arguments) for more information on keyword arguments.

### Cache_by Function Workflows

You can use `cache_by` functions to implement a variety of caching policies.
Here are a few examples:

  * The `cache_by` function could return the file modification time of a dataset to automatically invalidate the cache when an input dataset changes.
  * In a Heroku or [Dash Enterprise](https://plotly.com/dash/?utm_medium=dash_docs&utm_content=long-callbacks) deployment setting, the `cache_by` function could return the git hash of the app, making it possible to persist the cache across redeploys, but invalidate it when the app's source changes.
  * In a [Dash Enterprise](https://plotly.com/dash/?utm_medium=dash_docs&utm_content=long-callbacks) setting, the `cache_by` function could return user meta-data to prevent cached values from being shared across authenticated users.

### Setting an Expiration Time for Cache Entries

With both `CeleryLongCallbackManager` and `DiskcacheLongCallbackManager`, you
can use the `expire` argument to limit how long a cache entry is retained for
in the database. This is the number of seconds to keep a cache entry for after
its last use. When an entry is accessed, the timer restarts.

    
    
    celery_app = Celery(
        __name__, broker="redis://localhost:6379/0", backend="redis://localhost:6379/1"
    )
    long_callback_manager = CeleryLongCallbackManager(celery_app, expire=100)

## Limitations

  * Long callbacks support Disk Cache/multiprocessing as a backend, which is ideal for development environments and can be easier to set up if working on Windows. For production environments, however, we recommend Celery/Redis.
  * You can't access previous results returned by a long callback. This functionality is available in Dash Enterprise's [Snapshot Engine](https://plotly.com/dash/snapshot-engine/?utm_medium=dash_docs&utm_content=long-callbacks).
  * [Pattern-matching dependencies](/pattern-matching-callbacks) are not supported with long callbacks. An error is raised if a callback ID contains `ALL`, `MATCH`, or `ALL_SMALLER`.
  * [`dash.callback_context`](https://dash.plotly.com/advanced-callbacks) is not supported.
  * If the `Input/State/Output` dependencies do not exist when the app starts (if they reference components that are generated by another callback), `suppress_callback_exceptions=True` does not prevent Dash from raising callback validation errors. We recommend the following approach:

    1. The app must provide a [validation_layout](/urls) that contains all of the components referenced by callbacks in the app.
    2. The `prevent_initial_call` argument to `app.long_callback` must be set to `True`.

## Why Job Queues?

When your app is deployed in production, a finite number of CPUs serve
requests for that app. Callbacks that take longer than 30 seconds often
experience timeouts when deployed in production. And even callbacks that take
less than 30 seconds can tie up all available server resources when multiple
users access your app at the same time. When all CPUs are processing
callbacks, new visitors to your app see a blank screen and eventually a
"Server Timed Out" message.

![Example with no job queue](./assets/images/long-
callbacks/long_callbacks_scenario_1.png)

Job queues are a solution to these timeout issues. Like the web processes
serving your Dash app, job queues run with a dedicated number of CPU workers.
These workers go through the jobs one at a time and aren’t subject to
timeouts. While the job queue workers are processing the data, the web
processes serving the Dash app and the regular callbacks display informative
loading screens, progress bars, and the results of the job queues. End users
never see a timeout and always see a responsive app.

![Example with no job queue](./assets/images/long-
callbacks/long_callbacks_scenario_2.png)

## Additional Resources

  * [Introduction to Celery](https://docs.celeryproject.org/en/stable/getting-started/introduction.html)

  * [Redis documentation](https://redis.io/documentation)

  * [DiskCache tutorial](http://www.grantjenks.com/docs/diskcache/tutorial.html)

Long callbacks were developed through Dash Labs. We received lots of feedback
from users throughout the development. You can see the original community
discussions around the feature here:

  * [dash-labs-0-3-0-app-long-callback-support](https://community.plotly.com/t/dash-labs-0-3-0-app-long-callback-support/53719)
  * [dash-labs-0-4-0-long-callback-caching-and-windows-support](https://community.plotly.com/t/dash-labs-0-4-0-long-callback-caching-and-windows-support/54542)

## Reference

[API Reference for long callbacks](/reference)

