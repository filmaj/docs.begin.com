## Overview

Pretty much every app requires access to hosted static assets of some kind or another. Images are probably the most obvious example.

And while Begin can deliver your app's [JS and CSS dynamically with an HTTP Function](/en/functions/http/), it's still super common for workflows to generate build artifacts best delivered from cloud services optimized for static hosting.

Fortunately, that's why Begin has the ability to build (if necessary) and deploy static assets to S3 (origin) and CloudFront (CDN) out of the box.


## The `public/` directory

Get started working with static assets by using the `public/` directory in your project's root directory.

Anything in your `public/` folder will be deployed to your app's S3 bucket, retaining its file name and any subdirectories you've specified.

You can also work locally with static assets, by calling them from the Begin sandbox's root directory. For example, when working locally:
- `public/fluffypupper.jpg` resolves to:
- `http://localhost:3333/fluffypupper.jpg`

Just remember to point your app at the correct root path in your code. More below on [accessing static assets](#accessing-static-assets).



## `build`ing and deploying assets

A common pattern for generating frontend build artifacts is to compile JS and CSS (among other things).

`public/` works great with [Webpack](https://webpack.js.org/comparison/), [Parcel](https://parceljs.org/), [Rollup](https://rollupjs.org/guide/en), and other modern bundlers.

Simply create and specify a [build script or build command in your root `package.json`](https://docs.begin.com/en/getting-started/builds-deploys/#build), with your compilation target set to `public/`.

Your build steps will run with each deploy, and anything present in `public/` will be pushed to your app's S3 bucket.

As with the rest of your app, any push to `master` deploys assets to your `staging` S3 bucket, and `production` releases deploy assets to, you guessed it, your `production` S3 bucket.

By default, your app is also configured with `staging` and `production` CDN (CloudFront) paths pointing to your respective S3 buckets as its origin. Thus, to access your assets via CDN, simply use your app's CDN path when appropriate.


## Accessing static assets

Begin supports accessing static assets locally before they've been promoted to S3. Depending on the stage and service you'd like to access asset from, just swap out the paths as necessary.

Here is an example environment-aware helper for accessing your static assets. (You may want to add additional customization to better take advantage of Begin's staging and production asset pipelines, too.)


```javascript
function staticAsset(filename) {
  let origin
  // these variables are always available to all lambdas
  let env = process.env.NODE_ENV
  if (env === 'production') {
    origin = process.env.BEGIN_STATIC_EDGE // CDN
  } else if (env === 'staging') {
    origin = process.env.BEGIN_STATIC_ORIGIN // Preview
  } else {
    origin = '/_static' // Handles local use cases
  }
  return `${origin}/${filename}`
}
```
