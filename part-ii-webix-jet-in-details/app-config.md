# App Config

An app module is defined as an extension of the JetApp class (OOP inheritance).

In the app file, you will typically include:

* stylesheets \(any CSS or LESS\)
* custom widgets
* app-level plugins
* app-level Webix settings

You can include various parameters into the app configuration. The details are below.

| Parameter | What For |
| :--- | :--- |
| debug | to enable debugging |
| router | to change a router |
| routes | to shorten the app URL |
| start | to set the start app URL |
| views | to change view modules names |
| arbitrary parameters | e.g. access mode, screen size, etc. |

## Setting the Start URL

The app UI will be rendered from the URL elements from **start** when you open the app for the first time. The app splits the URL into parts, finds the corresponding files in the **views** folder and creates an interface by combining UI modules from those files.

```javascript
// myapp.js
import "./styles/app.css";
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router: BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug: !PRODUCTION,
            start: "/top/layout" // !
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

## Debugging

You really should enable the **debug** mode during the development stage. When _debug_ is enabled, error messages are logged into console and a debugger will pause the app on errors. Note that `webix.debug({ events: true });` also should be switched on.

```javascript
// myapp.js
import "./styles/app.css";
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router: BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug: !PRODUCTION,  // !
            start: "/top/layout"
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

[You can read more about debugging with Webix in this blog post](https://blog.webix.com/ui-development-and-debug-with-webix-js/).

## Choosing a Router

Webix Jet has four types of routers. The default router is _HashRouter_. If you don't want to display the hashbang in the URL, you can change the router to _UrlRouter_:

```javascript
// myapp.js
import "./styles/app.css";
import { JetApp, EmptyRouter, UrlRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router: BUILD_AS_MODULE ? EmptyRouter : UrlRouter,    // !
            debug: !PRODUCTION,
            start: "/top/layout"
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

## Changing View Creation Logic

Use the **views** parameter to change the names of view modules inside your code.

For example, if the module you want to show is in a subfolder and you want to shorten the URL of the module, you can do it in the **views** parameter:

```javascript
// myapp.js
import "./styles/app.css";
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router     : BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug     : !PRODUCTION,
            start     : "/top/layout",
            views: {
                "start" : "area.list" // load /views/area/list.js
            }
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

In this example, **list** module is stored in the **area** subfolder in _/views_ \(_/views/area/list.js_\). Later, you can show the view by the new name, e.g.:

```javascript
// views/top
import {JetView} from "webix-jet";

export default class TopView extends JetView {
    config(){
        return {
            cols:[
                { view:"button", value:"start",
                  click:() => {
                    this.show("start");
                }},
                { $subview: true }
            ]
        };
    }
}
```

**this** in the button click handler refers to the current instance of a Jet view class [\[1\]](app-config.md#1).

[Check out the demo &gt;&gt;](https://github.com/webix-hub/jet-demos/blob/master/sources/viewresolve.js)

### Code Splitting

You can [split your code into bundles](https://webpack.js.org/guides/code-splitting/) and load them on demand \(lazy loading\), which can greatly influence the initial loading time of the application. Lazy loading of app code is possible in Webix Jet with the help of a custom **app.views** handler. Where you can import the bundle on demand [\[2\]](app-config.md#2).

```javascript
// myapp.js
import "./styles/app.css";
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router: BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug: !PRODUCTION,
            start: "/top/layout",
            views: (name) => {
                if (name === "modules.clients") //sources/modules/
                    return import("modules/clients");

                // load all other modules with default strategy
                return name;
            }
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

In **webpack.config.js**, you can define the chunk naming scheme \(the _chunkFilename_ property\) in **output**:

```javascript
// webpack.config.js
...
output: {
    ...
    filename: "[name].js",
    chunkFilename: "[name].bundle.js"
    // will be 'clients.bundle.js' in this case
}
```

[Check out the demo &gt;&gt;](https://github.com/webix-hub/jet-demos/blob/master/sources/bundles.js)

### Custom Logic of Creating Views

You can also implement your own logic of view creating. Define **views** as a function for that:

```javascript
// myapp.js
...
const defaults = {
    ...
    views: function(url){
        //implement your own logic here
        url = url.replace(/\./g, "/");
        var view = require("jet-views/"+url);
        if (view.__esModule) {
            view = view.default;
        }
        return view;
    }
};
...
```

## Beautifying the URL

If you do not want to display some paths, you can replace them with shorter URLs with the help of **routes** in the app configuration. For instance, you might want to display only the names of subviews in the URL:

```javascript
// myapp.js
import "./styles/app.css";
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router	: BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug	: !PRODUCTION,
            start	: "/top/layout",
            routes: {
                "/hi"	: "/top/about",
                "/form"	: "/top/area.left.form",
                "/list"	: "/top/area.list",
            }
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

Instead of a long URL with subdirectories and parent views, e.g. _"/top/area.left.form"_, the app URL will be displayed as _/form_.

{% hint style="warning" %}
Note that **routes** can be used to replaced the whole URL, not segments of the URL.
{% endhint %}

[Check out the demo &gt;&gt;](https://github.com/webix-hub/jet-demos/blob/master/sources/routes.js)

## Various App Parameters

In the app config, for example, you can set the mode in which the app will work:

```javascript
// app.js
import "./styles/app.css";
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            mode:"readonly",  //application wide configuration
            router: BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug: !PRODUCTION,
            start: "/top/layout"
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );   // mandatory!
}
```

Later in the code, you can do some actions according to the mode:

```javascript
// views/games.js
...
if (this.app.config.mode === "readonly"){
    this.show("limited");
}
...
```

**this** refers to the current instance of a Jet view class \[1\].

## Adding Stylesheets

You can include several stylesheets. When the app will be built, all stylesheets will be compiled into _app.css_. _app.css_ will be added to _/codebase_.

```javascript
//app.js
import "./styles/app.css";  // !
import {JetApp, EmptyRouter, HashRouter } from "webix-jet";

export default class MyApp extends JetApp{
    constructor(config){
        const defaults = {
            router: BUILD_AS_MODULE ? EmptyRouter : HashRouter,
            debug: !PRODUCTION,
            start: "/top/layout"
        };

        super({ ...defaults, ...config });
    }
}

if (!BUILD_AS_MODULE){
    webix.ready(() => new MyApp().render() );
}
```

## Footnotes

#### \[1\]:

To read more about how to reference apps and view classes, go to ["Referencing views"](referencing-views.md).

#### \[2\]:

Beginning from Webix Jet 1.6.

