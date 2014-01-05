# rebund

Makes your CI build much faster by caching your bundle.

## Rationale

Running `bundle install` to get all gems ready is often the longest part of a
build. Any Rails application will depend on tens of gems; and any gem, besides
dependency, may have a fairly large build matrix.

Rebund can easily cut your build times by 75%, saving you time and saving the
good folks at [Travis CI](https://travis-ci.org/) some money.

Example for an application, [appfab.io](http://github.com/mezis/appfab) (5.3x
speedup):

<img src="http://f.cl.ly/items/0y1M3K100J0e222y2T2L/rebund-appfab.png"/>

Example for a gem, [fuzzily](http://github.com/mezis/fuzzily) (2.7x speedup):

<img src="http://f.cl.ly/items/2a1c2O3M0w3i2G1D2d3M/rebund-fuzzily.png"/>

Rebund is an alernative to
[bundle_cache](https://github.com/data-axle/bundle_cache), which didn't work
well enough for us on Travis because it relies on gems to be installed (which
partly defeats the purpose).


## How it works

Before your build, Rebund checks with a file server if there's a bundle already
available for your current Ruby VM and version of the `Gemfile.lock`.
Technically, it hashes the output of `ruby --version` and the lockfile.

If there is one, it downloads and unpacks it, then making `bundle install`
blazing fast.

After the build, Rebund packages the bundle (if changed) and uploads it for
future uses.


## Prerequisites

Rebund needs an HTTP endpoint to store and retrieve bundle files.
It's tested with [keyfile](http://github.com/mezis/keyfile), a no-frills alternative to S3 with more standard
authentication, and defaults to the version I host.

If you don't have credentials to use that Keyfile server, you can

- deploy your own (read [the docs](http://github.com/mezis/keyfile#installation))
- ask [me](mailto:julien.letessier@gmail.com) for credentials on my hosted
  version.


## Installation

Add rebund to your repository as a submodule:

    $ git submodule add https://github.com/mezis/rebund.git

While submodules are not necessarily a great idea in general, we can't rely on
Rubygems to install rebund as it needs to run before running Bundler.

The next step is to instruct Travis to attempt to use a cached bundle before the
build, and upload a bundle if needed after the build.  Add the following lines
to your `.travis.yml` config file.

    install:
      - ./rebund/run download
      - bundle install --path vendor/bundle
    after_script:
      - ./rebund/run upload

(Optional) if you're using your own Keyfile server, specify it in your Travis
config:

    env:
      global:
        - REBUND_ENDPOINT=http://my-rebund.io/

Finally, add your rebund cretentials to the configuration:

    $ travis encrypt REBUND_CREDENTIALS=username:secret --add    

Push your changes, and watch the build work. Of course you'll only get the
benefits from the second build onwards!


# Caveats

### Gemfiles in subdirectories

Rebund assumes that your bundle goes to `vendor/bundle`, relative to the root of
your project. In some cases (e.g. when using
[Appraisal](https://github.com/thoughtbot/appraisal)), the Gemfile resides in a
subdirectory.

Given the `--path` option to `bundle install` is relative to the Gemfile, you
will need to change the Travis configuration to

    bundle install --path vendor/bundle


## License

Released under the MIT licence.
Copyright (c) 2014 HouseTrip Ltd.

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request
