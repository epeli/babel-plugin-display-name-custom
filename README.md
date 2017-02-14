# babel-plugin-display-name-custom

display name inference for your custom react component creators

so that instead of

![unknown](https://raw.githubusercontent.com/epeli/babel-plugin-display-name-custom/master/assets/unknown.png)

you could see 

![unknown](https://raw.githubusercontent.com/epeli/babel-plugin-display-name-custom/master/assets/displayname.png)

in [React Developer Tools](https://github.com/facebook/react-devtools)

for code like

```js
import {createComponent} from "./create";

const Container = createComponent("div", {
    border: "1px solid black",
});

const Red = createComponent("div", {
    color: "red",
});

const Green = createComponent("div", {
    color: "green",
});

const Blue = createComponent("div", {
    color: "blue",
});

const Root = () => (
    <Container>
        <Red>red</Red>
        <Green>green</Green>
        <Blue>blue</Blue>
    </Container>
);
```



# Usage

install with yarn

    yarn add babel-plugin-display-name-custom

add `display-name-custom` to your `.babelrc`

```json
{
    "presets": ["es2015", "react"],
    "plugins": [
        "syntax-object-rest-spread",
        "transform-object-rest-spread",
        ["display-name-custom", {
            "modules": {
                "./create": {
                    "createComponent": true
                }
            }
        }]
    ]
}
```

The `modules` object is a mapping of module names to the exported
functions which return new React components.

In the above example `./create` is

```js
export const createComponent = (Component, styles) => {
    return props => <Component {...props} style={styles} />;
};
```

You can also add inference to 3rd party modules

```json
{
    "modules": {
        "react-redux": {
            "connect": true
        }
    }
}
```

The default export can be added using the `default` keyword

```json
{
    "modules": {
        "create-component": {
            "default": true
        }
    }
}
```

# How it works

After the plugin is configured it knows which functions return new React components.
Using that information it scans your source code for variable declarations which are initialized
using those. When a declaration is found it sets the `displayName` property
of the component to the variable name.

In practice it transpiles

```js
const Red = createComponent("div", {
    color: "red",
});
```

to

```js
const Red = createComponent("div", {
    color: "red",
});
Red.displayName = "Red";
```

# Library author?

For library authors the plugin also exports a `createPlugin` factory
for generating preconfigured library specific display name plugins.

Just add `babel.js` to the root of your npm module with contents like

```js
module.exports = require("babel-plugin-display-name-custom").createPlugin({
    modules: {
        "mylib": {createComponent: true},
    },
});
```

and your users then can use it with just

```json
{
    "presets": ["es2015", "react"],
    "plugins": [
        "mylib/babel"
    ]
}
```

Checkout [react-simple][] for a real world example

[react-simple]: https://github.com/epeli/react-simple
