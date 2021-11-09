# Welcome to React Router .sh&middot; [![build][build-badge]][build]

[build-badge]: https://img.shields.io/github/workflow/status/remix-run/react-router/test/dev?style=flat-square
[build]: https://github.com/remix-run/react-router/actions/workflows/test.yml

React Router is a lightweight, fully-featured routing library for the [React](https://reactjs.org) JavaScript library. React Router runs everywhere that React runs; on the web, on the server (using node.js), and on React Native.

If you're new to React Router, we recommend you start with [the getting started guide](/docs/getting-started/installation.md).

If you're migrating to v6 from v5 (or v4, which is the same as v5), check out [the migration guide](/docs/upgrading/v5.md). If you're migrating from Reach Router, check out [the migration guide for Reach Router](/docs/upgrading/reach.md). If you need to find the code for v5, [it is on the `v5` branch](https://github.com/remix-run/react-router/tree/v5).

When v6 is stable we will publish the docs on our website.

## Contributing

There are many different ways to contribute to React Router's development. If you're interested, check out [our contributing guidelines](CONTRIBUTING.md) to learn how you can get involved.

## Packages

This repository is a monorepo containing the following packages:

- [`react-router`](/packages/react-router)
- [`react-router-dom`](/packages/react-router-dom)
- [`react-router-native`](/packages/react-router-native)

## Changes

Detailed release notes for a given version can be found [on our releases page](https://github.com/remix-run/react-router/releases).

## Funding

You may provide financial support for this project by donating [via Open Collective](https://opencollective.com/react-router). Thank you for your support!

## About

React Router is developed and maintained by [Remix Software](https://remix.run) and many [amazing contributors](https://github.com/remix-run/react-router/graphs/contributors).

# Porting REPL code to reactapp and deploying with surge

## the differences

This is an attempt to figure out the absolute simplest way to deploy a React app from your desktop with as little time and effort as possible. In order to do this there are a few step that need to be understood.

* most of the install procedures above will suffice for getting the main code working. 
  * Using the react install starter guide above will get all of the dependencies loaded on your local machine. 
  * Once this is done you can modify/add the required serviceWorker.js code (it threw an error when deploying my app), modify your index.js file to point to the local files and dependencies, and then get everything working locally. 
  * After this we dive into pushing everything directly from your desktop to surge (also a simple install). 
My hope is to avoid going through some service provider or some crazy setup in order to get a react app pushed to the web without the cost or headaches associated. Lets dive in.

 
* In order to make a react/js app easily you can work it all out in [repl](https://replit.com) and then simply download the zipped code to your desktop
* inside of your repl you may have to add the serviceWorker.js file so that your app will run properly. I found this [boilerplate](https://stackoverflow.com/questions/65255315/module-not-found-cant-resolve-serviceworker-in-myapp-src) to use:

```
// This optional code is used to register a service worker.
// register() is not called by default.

// This lets the app load faster on subsequent visits in production, and gives
// it offline capabilities. However, it also means that developers (and users)
// will only see deployed updates on subsequent visits to a page, after all the
// existing tabs open on the page have been closed, since previously cached
// resources are updated in the background.

// To learn more about the benefits of this model and instructions on how to

const isLocalhost = Boolean(
    window.location.hostname === 'localhost' ||
    // [::1] is the IPv6 localhost address.
    window.location.hostname === '[::1]' ||
    // 127.0.0.0/8 are considered localhost for IPv4.
    window.location.hostname.match(
        /^127(?:\.(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)){3}$/
    )
);

export function register(config) {
    if (process.env.NODE_ENV === 'production' && 'serviceWorker' in navigator) {
        // The URL constructor is available in all browsers that support SW.
        const publicUrl = new URL(process.env.PUBLIC_URL, window.location.href);
        if (publicUrl.origin !== window.location.origin) {
            // Our service worker won't work if PUBLIC_URL is on a different origin
            // from what our page is served on. This might happen if a CDN is used to
            // serve assets; see https://github.com/facebook/create-react-app/issues/2374
            return;
        }

        window.addEventListener('load', () => {
            const swUrl = `${process.env.PUBLIC_URL}/service-worker.js`;

            if (isLocalhost) {
                // This is running on localhost. Let's check if a service worker still exists or not.
                checkValidServiceWorker(swUrl, config);

                // Add some additional logging to localhost, pointing developers to the
                // service worker/PWA documentation.
                navigator.serviceWorker.ready.then(() => {
                    console.log(
                        'This web app is being served cache-first by a service '
                    );
                });
            } else {
                // Is not localhost. Just register service worker
                registerValidSW(swUrl, config);
            }
        });
    }
}

function registerValidSW(swUrl, config) {
    navigator.serviceWorker
        .register(swUrl)
        .then(registration => {
            registration.onupdatefound = () => {
                const installingWorker = registration.installing;
                if (installingWorker == null) {
                    return;
                }
                installingWorker.onstatechange = () => {
                    if (installingWorker.state === 'installed') {
                        if (navigator.serviceWorker.controller) {
                            // At this point, the updated precached content has been fetched,
                            // but the previous service worker will still serve the older
                            // content until all client tabs are closed.
                            console.log(
                                'New content is available and will be used when all '
                            );

                            // Execute callback
                            if (config && config.onUpdate) {
                                config.onUpdate(registration);
                            }
                        } else {
                            // At this point, everything has been precached.
                            // It's the perfect time to display a
                            // "Content is cached for offline use." message.
                            console.log('Content is cached for offline use.');

                            // Execute callback
                            if (config && config.onSuccess) {
                                config.onSuccess(registration);
                            }
                        }
                    }
                };
            };
        })
        .catch(error => {
            console.error('Error during service worker registration:', error);
        });
}

function checkValidServiceWorker(swUrl, config) {
    // Check if the service worker can be found. If it can't reload the page.
    fetch(swUrl, {
            headers: {
                'Service-Worker': 'script'
            },
        })
        .then(response => {
            // Ensure service worker exists, and that we really are getting a JS file.
            const contentType = response.headers.get('content-type');
            if (
                response.status === 404 ||
                (contentType != null && contentType.indexOf('javascript') === -1)
            ) {
                // No service worker found. Probably a different app. Reload the page.
                navigator.serviceWorker.ready.then(registration => {
                    registration.unregister().then(() => {
                        window.location.reload();
                    });
                });
            } else {
                // Service worker found. Proceed as normal.
                registerValidSW(swUrl, config);
            }
        })
        .catch(() => {
            console.log(
                'No internet connection found. App is running in offline mode.'
            );
        });
}

export function unregister() {
    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.ready
            .then(registration => {
                registration.unregister();
            })
            .catch(error => {
                console.error(error.message);
            });
    }
}
```

* once all of this is done and you have done the ```npm install history@5 react-router-dom@6``` you can continue to test to see if everything is working with ```npm start``` as stated above
* once you see the app working in localhost:3000 you know the app works - now it is time to push it to CDN with some simple steps

* goto [surge](https://surge.sh) and use the command line ```npm install --global surge``` to set up surge on your CLI.
* there are a couple things you have to set up in advance before you push:
  * you will need a CNAME file to hold your URL so you dont have to update it every single time ```echo <yourURL>.surge.sh > CNAME``` 
  * set up a html.200 file to launch your application with <script> tags
    * ...needs to be updated... working on this now
* then just type surge --help to see some of the commands. for my push install I am simply using ```surge <directory of application files/folders> <yourURL>.surge.sh```
