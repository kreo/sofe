# Sofe Configuration and API
Most of the API is for telling sofe how to resolve a service name to an actual file location. Because we don't want to favor any specific back-end technology, sofe tries to favor configuration over convention. As a result, there are a variety of approaches to setting up your project.

### Approach #1
**npmcdn to find urls and host files**

*Any* npm package can be coerced into being a sofe service, since npmcdn.com hosts all files for all npm packages. This is automatically done by sofe whenever the package.json for an npm package does not have a valid `sofe` property.

### Approach #2
**npmcdn to find urls, your own CDN for files**

Simply put a distributable file on a CDN, modify the package.json, and publish the service to npm -- then any application can automatically resolve and load the service.

For example, a `sofe-hello-world` service could automatically be resolved if `sofe-hello-world` is published to npm with the following
`package.json`:
```json
{
  "name": "sofe-hello-world",
  "version": "1.0.0",
  "sofe": {
    "url": "https://npmcdn.com/sofe-hello-world@1.0.0/hello.js"
  }
}
```

### Approach #3
**Put urls to all the services into the `System.config`**

Instead of automatically resolving services, provide a manifest of services with associated service deployable locations:
```javascript
System.config({
  sofe: {
    manifest: {
      "sofe-hello-world": "https://npmcdn.com/sofe-hello-world@1.0.0/hello.js"
    }
  }
});
```

### Approach #4
**Put a url to a manifest file (which, in turn points to the source files) into the `System.config`.**
```javascript
System.config({
  sofe: {
    manifestUrl: 'https://cdn.canopytax.com/canopy-services.json'
  }
});
```
**Manifest file format:**
```json
{
  "sofe": {
    "manifest": {
      "sofe-hello-world": "https://npmcdn.com/sofe-hello-world@1.0.0/hello.js"
    }
  }
}
```

### Approach #5
**Browser storage**

In addition to automatic resolution or manifest resolution, the urls to individual sofe services can be overridden with sessionStorage and/or localStorage. This is meant for times when you want to test out new changes to a service on an application where you can't easily change the `System.config` (i.e., you don't own the code to the application). An override is a sessionStorage/localStorage item whose key is `sofe:service-name` and whose value is a url.

Example:
```js
window.localStorage.setItem('sofe-hello-world', 'https://npmcdn.com/sofe-hello-world@1.0.0/hello.js');
// OR
window.sessionStorage.setItem('sofe-hello-world', 'https://npmcdn.com/sofe-hello-world@1.0.0/hello.js');
```

## Resolution precedence
If there are multiple urls that a service could be resolved to, sofe will resolve the service url in the following order (highest precedence to lowest precedence):

1. Session storage
2. Local storage
3. The `manifest` property inside of the `sofe` attribute of the `System.config` or manifest file
4. The `manifestUrl` property inside of `sofe` attribute of the `System.config` or manifest file
5. The `url` property inside of the `sofe` attribute of the NPM package's package.json file
6. The `main` file inside of the NPM package's package.json file, at the `latest` version. The files themselves are retrieved from npmcdn.com.

## When to use `System.import` instead of just `import`
Because services are loaded at run-time, they cannot be bundled inside the application. Avoid bundling by using `System.import` syntax instead of `import`. The problem is `System.import` can be cumbersome to use for all imports. If you bundle a JSPM project with sofe `import` statements, you are likely to get errors that say: `Uncaught (in promise) Error: This sofe service was bundled and needs to be removed from System.register`. Removing the service from System.register will allow it to be loaded at run-time, even if the project is bundled.

For example:
```javascript
// main file to be bundled
import hello from 'sofe-hello-world!sofe';
hello();
```

```html
<script src="dist/app-bundle.js"></script>
<script>
  System.delete(System.normalizeSync('sofe-hello-world!sofe'));
</script>
```
Alternatively, if your project uses webpack, use the [sofe-babel-plugin](https://github.com/CanopyTax/sofe-babel-plugin) which allows the use of `import` in bundled projects.

## Full API
Sofe's configuration API exists within a `System.config` property:
```javascript
System.config({
    sofe: {
       manifest: Object,    // Map of services with their distributable urls   
       manifestUrl: String, // Url for a manifest of available services
       registry: String     // Provide a custom registry defaults to "https://npmcdn.com"
    }
});
```

**Manifest file format:**
```javascript
{
  "sofe": {
    "manifest": Object // Map of services with their distrubable urls
  }
}
```
