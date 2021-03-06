CloudBoxAPI Server (Ruby)
============

### Note: This project is no longer maintained for production use, this is for reference and demonstration purpose only.

CloudBoxAPI is a framework for over-the-air, asynchronous, in-the-background, resource syncing between iOS/Mac OS X apps and a server. Let's say your app depends on a javascript resource file called `MyResource.js`, but you want to be able to change it often without resubmitting your entire app to the App Store. CloudBoxAPI allows you to ship a bundled version of the resource inside your app, publish and distribute your app, and then once the app is out in the wild push updated versions of your resource to the cloud and have your apps in the wild automatically sync the resource as soon as the new one becomes available.

This is a server implementation for the CloudBoxAPI, and is configured for 1 click deployment to Heroku. It is implemented using Ruby with Async Sinatra (Eventmachine) and is deployed with the Rainbows server. It consumes about 35MB/process. It's currently configured to spawn 12 worker processes on Heroku, corresponding to 3 per core. Has been thoroughly load tested and can sustain a peak performance with 0% error rate of 1000 concurrent requests with an end-to-end response time of ~750ms on a single dyno (i.e. for free!); this corresponds to about 1300 req/s with a concurrency of 1000 simulatenously connected users. The server features graceful degradation at overload capacity: tested at 4000 concurrent users, the server will maintain a ~800ms response rate with 38% dropped requests for a throughput of 3100 successful req/s. This is all on a single free dyno. App is stateless so you can scale your dynos and multiply performance linearly for the price of additional dynos. Or you can create several single-dyno free Heroku apps, and load balance on the client for free (the client library has a feature to sync with multiple servers).

Dependency management using bundler.  Includes NewRelic monitoring.

CloudBox Clients
------------
Android: https://github.com/duriana/Duriana-Cloudbox-Android (Deleted)

iOS: https://github.com/duriana/CloudBox-iOS (Deleted)

Usage
------------

1- List your files in the resources folder or update old ones:

```
resources
│   currencies.json
│   options.zip
│   ......

```

2- just run this command to commit the changed that you made it and update resources_manifest.yml file

```
ruby commit_changes
```

or give authorize to run the script by  ``` chmod +x commit_changes ```, and then

```
./commit_changes

```

That's it. Client library will take care of the rest.

Notes:

- You can configure the server API paths and other configrations if you need (make sure to set the same thing in client side):

```ruby
RESOURCES_META_PATH = "CloudBoxAPIResourcesMeta"
RESOURCES_DATA_PATH = "CloudBoxAPIResourcesData"
```

- You can edit resources_manifest.yml file manually to remove or add new files with specfic versions, but be carful resources_manifest.yml file will change every time after run commit_changes script.

Local testing
------------

First update dependencies (make sure you have bundler installed, if not `gem install bundler` first):

```sh
bundle update
```

Then launch using foreman (if you don't have foreman installed: first do `gem install foreman`):

```sh
foreman start
```

Test
------------
to run the test
```
rspec
```

Implementing your own server
------------

You can always create your own implementation of the server, the protocol is very simple... you need to implement a `meta` GET url which returns a JSON string like this:
```json
{
  "v": "3",
  "url": "http://www.my-company.com/path/to/the/actual/resource.zip"
}
```

The path to this JSON can be set in the client library, it defaults to `/CloudBoxAPIResourcesMeta/<resource>`. e.g. if your server is at `files.my-company.com` and the resource is called `MyResource.zip`, it will be `https://files.my-company.com/CloudBoxAPIResourcesMeta/MyResource.zip`.

That basically tells the client what the latest version is and where to find it. Then just make sure that the resouce (in this case `resource.zip`) is actually available at the url you claim it's at. The client will check the meta path to see if there's a newer version out, and if there is it will get it from the url your server specifies. You can serve the actual file from something like Amazon S3, a CDN, or your own server.

Updating resources
------------
- increase version in bower file

TODO & issues:
------------
support external files, https://github.com/duriana/CloudBox-API/issues/2
