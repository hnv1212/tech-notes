# CORS
A JavaScript application running in the browser can usually only access HTTP resources on the same domain (origin) that serves it. <br>
Loading images or scripts/styles always works, but XHR and Fetch calls to another server will fail, unless that server implements a way to allow that connection.

This way is called CORS, **Cross-Origin Resource Sharing**

One very important thing that needs CORS is ES Modules, recently introduced in modern browsers.

If you don't set up a CORS policy on the server that allows to serve 3rd party origins, the request will fail.

A Cross-Origin resource fails if it's:
- to a different **domain**
- to a different **subdomain**
- to a different **port**
- to a different **protocol**

