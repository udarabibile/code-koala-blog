+++
author = "Udara Bibile"
authorImage = "/img/udarabibile-thumbnail.png"
title = "Introduction to React Context API"
date = "2019-07-20"
description = "Avoid props drilling to child components"
tags = ["reactjs", "syntax"]
categories = ["javascript", "reactjs"]
images  = ["img/2019/react-context-api.png"]
type = "post"
aliases = []
draft = false
+++

From **React’s version 16.3.0**, Context API is officially released to avoid props drilling to child components.

<hr/>

In order to understand use of React context and its application, lets look into what complication its trying to fix.

```
import React from 'react';

const App = () => {
    return <Menu colour="blue" language="en" />;
}

function Menu(props) {
    return(
        <MenuItem 
            colour={props.colour}
            language={props.language}
        />
    )
}

function MenuItem(props) {
    return(
        <div>
            <p>Theme colour: {props.colour}</p>
            <p>Locale: {props.language}</p>
        </div>
    );
}

export default App;
```

In our React application, there can be global properties such as **language**, or **theme colour** that will configured at top most component in `<App />` . These properties will be used by child components & also grand child components. Here in above example `<MenuItem>` element may need value of colour for styling, and language for localization. However these properties had to be passed through `<Menu />` component, even though its not used in `<Menu />`.

```
<App /> → <Menu /> → <MenuItem />
```

Thereby each of these global properties, have to be **mapped as props from intermediate components** even though it may not be used in-between. This is called **props drilling** and may introduce **redundant mapping of props** for complex application for multiple properties.

<hr/>

### Context API

> “Context provides a way to pass data through the component tree without having to pass props down manually at every level.”</p>
> → React Docs

React’s Context API is introduced to address issue of props drilling, where **higher order component’s context can be accessed by child component without mapping props in intermediate components**.

<hr/>

#### Create context by React Context

Lets create a context, where default value should be passed to initiate.

```
const AppContext = React.createContext({ lang: 'en' });
```

This context object, has `Provider` and `Consumer` with it. Or can extract these properties separately.

```
--- Can be used in AppContext.js ---
export const AppProvider = AppContext.Provider;
export const AppConsumer = AppContext.Consumer;

--- OR ---
const { Provider, Consumer } = React.createContext();
```

<hr/>

#### Set context using Context Provider

In order for context value to be addressed at lower levels, context should be attached to higher level component intended. Here this component will be wrapped by `Context.Provider`, and current value is attached to it as `value`.

```
const App = () =>
   <AppContext.Provider value={{ lang: 'fr' }}>
         <Menu>
   <AppContext.Provider />
```

**Note**, that in here value is **not passed as props to `<Menu>`** component, avoiding props drilling. Moreover instead of using `<AppContext.Provider>`, we can also use `<AppProvider value="">` as above if imported from external file.

<hr/>

#### Get context by Context Consumer

To access context, which was provided from above **Consumer** can be used.

```
const MenuItem = () =>
   <AppContext.Consumer>
      { value =>
         <p>Locale: {value.lang}</p>
      }
   </AppContext.Consumer>
```

As in providing context example, consuming component is also wrapped in `<AppContext.Consumer>`. And `value` passed to `<AppContext.Provider>` can be accessed as above.

<hr/>

#### Get context by Class contextType

If using class components in React application, its `contextType` property can be utilized to access context data. In can be accessed in class as `this.context`.

```
class MenuItem extends React.Component {
   static contextType = AppContext;

   render() {
      const appContext = this.context;
      return (<p>{appContext.lang}</p>);
   }
}
```

Here `contextType` is set to required context object using **static** value. Another approach is attaching to context to `contextType` of class itself

```
class MenuItem extends React.Component {
   ...
}
MenuItem.contextType = AppContext;
```

<hr/>

Lets recreate above props drilling example using React Context here:

```
import React from 'react';

const AppContext = React.createContext({ colour: 'blue', lang: 'en' });

const App = () => 
    <AppContext.Provider value={{ colour: 'blue', lang: 'fr' }}>
        <Menu />
    </AppContext.Provider>;

function Menu() {
    return <MenuItem />
}

const MenuItem = () =>
    <AppContext.Consumer>
        { value =>
            <div>
                <p>Theme colour: {value.colour}</p>
                <p>Locale: {value.lang}</p>
            </div>
        }
    </AppContext.Consumer>

export default App;
```

In above implementation, theme related information are passed from `<App>` component directly into `<MenuItem>` component, without being mapped in intermediate `<Menu>` component.

#### Changing value of React Context

In these examples, `value` passed to `React.Context` is constant. Therefore there is not that much advantage of passing through context, but rather fetch just from configuration file. But in real application, this `value` is to change according to user interaction. As example application colour could be changed by user preference, or language can be switched by geographical area.

```
import React, { useState } from 'react';

const languages = { en: 'en', fr: 'fr' };
const initialTheme = { colour: 'blue', lang: languages.en };
const { Provider: AppProvider, Consumer: AppConsumer } = React.createContext(initialTheme);

const App = () => {

    const [theme, setTheme] = useState(initialTheme);

    const setLanguage = (lang) => {
        setTheme({ ...theme, lang: lang });
    }

    return (
        <AppProvider value={theme}>
            <button onClick={() => setLanguage(languages.en)}>English</button>
            <button onClick={() => setLanguage(languages.fr)}>French</button>
            <Menu />
        </AppProvider>
    );
}

const Menu = () => <MenuItem />;

const MenuItem = () =>
    <AppConsumer>
        { value =>
            <div>
                <p>Theme colour: {value.colour}</p>
                <p>Locale: {value.lang}</p>
            </div>
        }
    </AppConsumer>

export default App;
```

In `<App>` component, theme values are kept in the state. (Note: here React hook is used for state management. If unfamiliar, its just `this.state` and `this.setState` in class perspective and refer to my **React Hooks article here**) This `state` value is passed as `value` for `Context.Provider`. Moreover theme related `state` value can be changed by other component as given by buttons. When `state` changes in `<App>`, it’ll also be reflected in value in context. Thereby any `Context.Consumer` accessing `value` will get **current** updated changes.

However, in here `context` was changed by highest level, where it has access to `state` variable to manipulate. However this might not be the case and child components need to invoke functions to make context changes. For this context changing function can be attached to `context` itself and passed as `value` for child components. Here `setLanguage` is passed as callback function to `context` object to make state change and then context `value` change.

```
{
  lang: 'en',
  setLanguage: (lang) => { ...callback function here }
}
```

```
import React, { useState } from 'react';

const languages = { en: 'en', fr: 'fr' };
const initialTheme = { colour: 'blue', lang: languages.en, setLanguage: () => {} };
const { Provider: AppProvider, Consumer: AppConsumer } = React.createContext(initialTheme);

const App = () => {

    const setLanguage = (lang) => {
        setTheme({ ...theme, lang: lang });
    }

    const [theme, setTheme] = useState({...initialTheme, setLanguage: setLanguage});

    return (
        <AppProvider value={theme}>
            <LanguagePicker />
            <Menu />
        </AppProvider>
    );
}

const Menu = () => <MenuItem />;

const LanguagePicker = () =>
    <AppConsumer>
        {context =>
            <div>
                <button onClick={() => context.setLanguage(languages.en)}>English</button>
                <button onClick={() => context.setLanguage(languages.fr)}>French</button>
            </div>
        }
    </AppConsumer>

const MenuItem = () =>
    <AppConsumer>
        { value =>
            <div>
                <p>Theme colour: {value.colour}</p>
                <p>Locale: {value.lang}</p>
            </div>
        }
    </AppConsumer>

export default App;
```

<hr/>

#### Use of multiple contexts

According to application, there might be requirement of using multiple context objects. React supports this, but implementation can be bit messy as contexts will be nested here and nesting keeps adding up if more that 2 contexts needed.

```
<LanguageContext.Provider value={lang}>
   <ColourContext.Provider value={colour}>
      <Menu />
   </ColourContext.Provider>
</LanguageContext.Provider>
```

Accessing context will also be nested:

```
<LanguageContext.Consumer>
   {lang => (
   <ColourContext.Consumer>
      {colour => (
         <p>Lang: {lang} and colour: {colour}</p>
      )}
   </ColourContext.Consumer>
   )}
</LanguageContext.Consumer>
```

<hr/>

It should be noted that when **context changes in parent component, its child components dependent on the context value should also be changed and re-rendered**. Change in context value is determined by **reference identity**, where React keeps track whether `value` is changed by reference. Thereby unnecessary re-renders of consumer components needs to be handled when provider re-render for unrelated change. Refer to react docs here for more information on this, and avoiding such re-renders.

<hr/>

Just to wrap it up, React Context API is introduced to avoid props drilling in React applications.

> Context is designed to share data that can be considered “global” for a tree of React components, such as the current authenticated user, theme, or preferred language. →React Context Docs

<br/>

<mark>TODO: LINK TO HOOKS ARTICLE</mark>

<br/><br/>
