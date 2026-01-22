# Web Platform

## JavaScript Modules


**.MJS file**

- mjs an extension for EcmaScript modules
    
- An MJS file is a source code file containing an ES Module (ECMAScript Module) for use with a Node.js application.
    
- MJS files are written in JavaScript, and may also use the .JS extension outside of the Node.js context.
    
- ES Modules allow web and application developers to organize code into smaller reusable components.

- ECMAScript 6 (ES6) introduced the specification for ES Modules, providing a standard for implementing modules in JavaScript. As of 2018, all major web browsers support ES Modules.

However, the popularity of modularized JavaScript pre-dates ES6. Node.js, a JavaScript runtime environment, used CommonJS as the specification for modules. Because so many existing applications were built with CommonJS, when Node.js added support for native ES modules, it controversially introduced the MJS file extension to differentiate the two and prevent applications from breaking.

NOTE: Some developers informally refer to MJS files as "Michael Jackson Script" files.

![](https://i.sstatic.net/bb9vsqUr.png)

### A note on file extensions

You may have noticed we’re using the `.mjs` file extension for modules. On the Web, the file extension doesn’t really matter, as long as the file is served with [the JavaScript MIME type `text/javascript`](https://html.spec.whatwg.org/multipage/scripting.html#scriptingLanguages:javascript-mime-type). The browser knows it’s a module because of the `type` attribute on the script element.

Still, we recommend using the `.mjs` extension for modules, for two reasons:

1. During development, the `.mjs` extension makes it crystal clear to you and anyone else looking at your project that the file is a module as opposed to a classic script. (It’s not always possible to tell just by looking at the code.) As mentioned before, modules are treated differently than classic scripts, so the difference is hugely important!
2. It ensures that your file is parsed as a module by runtimes such as [Node.js](https://nodejs.org/api/esm.html#enabling) and [`d8`](https://v8.dev/docs/d8), and build tools such as [Babel](https://babeljs.io/docs/en/options#sourcetype). While these environments and tools each have proprietary ways via configuration to interpret files with other extensions as modules, the `.mjs` extension is the cross-compatible way to ensure that files are treated as modules.

**Note:** To deploy `.mjs` on the web, your web server needs to be configured to serve files with this extension using the appropriate `Content-Type: text/javascript` header, as mentioned above. Additionally, you may want to configure your editor to treat `.mjs` files as `.js` files to get syntax highlighting. Most modern editors already do this by default.

## Polyfills

a polyfill is code that implements a new standard feature of a deployment environment within an old version of that environment that does not natively support the feature

- coined by Remy Sharp, based on multipurpose filling paste brand, but it mainly means to fill in holes in functionality in many ways.

## HTML Spec. And Stuff.

##### Reading

- [HTML5 Specification](https://html.spec.whatwg.org/)
- [html dog](https://www.htmldog.com/)


- The WHATWG works on a number of technologies that are fundamental parts of the web platform. They are organised somewhat arbitrarily based on the preferences of those editing the standard for those technologies.
- This specification defines a big part of the web platform, in lots of detail. Its place in the web platform specification stack relative to other specifications can be best summed up as follows.

![](../assets/images/kitchensink.png)

### EMBEDDED CONTENT


#### iframe element

- An inline frame, or nested browsing context -- essentially, a web page embedded within a web page.