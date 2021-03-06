bower-rails
===========

[![Gem Version](https://badge.fury.io/rb/bower-rails.png)](http://badge.fury.io/rb/bower-rails)
[![Code Climate](https://codeclimate.com/github/42dev/bower-rails.png)](https://codeclimate.com/github/42dev/bower-rails)
[![Dependency Status](https://gemnasium.com/SergeyKishenin/bower-rails.png)](https://gemnasium.com/SergeyKishenin/bower-rails)
[![Build Status](https://travis-ci.org/42dev/bower-rails.png?branch=master)](https://travis-ci.org/42dev/bower-rails)
[![Coverage Status](https://coveralls.io/repos/42dev/bower-rails/badge.png)](https://coveralls.io/r/42dev/bower-rails)

Bower support for Rails projects. Dependency file is bower.json in Rails root dir or Bowerfile if you use DSL.
Check out [changelog][] for the latest changes and releases.

[changelog]: https://github.com/42dev/bower-rails/blob/master/CHANGELOG.md

**Requirements**

* [node](http://nodejs.org) ([on github](https://github.com/joyent/node))
* [bower](https://github.com/bower/bower) (>= 0.10.0) installed with npm

**Install**

in Gemfile

``` Ruby
  gem "bower-rails", "~> 0.7.1"
```

##JSON configuration

Bower-rails now supports the standard [bower package](https://github.com/bower/bower#defining-a-package) format out-of-the-box. Simply place your bower.json file the Rails root directory to start. Using the standard format will default all bower components to be installed under the `vendor` directory.

To install dependencies into both `lib` and `vendor` directories, run the initializer to generate a custom bower.json:

``` Bash
  rails g bower_rails:initialize json
```

This will generate a `config/initializers/bower_rails.rb` config file and a special bower.json that combines two standard bower packages into one. Simply specify your dependencies under each folder name to install them into the corresponding directories.

**example bower.json file**

``` javascript
{
   "lib": {
    "name": "bower-rails generated lib assets",
    "dependencies": {
      "threex"      : "git@github.com:rharriso/threex.git",
      "gsvpano.js"  : "https://github.com/rharriso/GSVPano.js/blob/master/src/GSVPano.js"
    }
  },
  "vendor": {
    "name": "bower-rails generated vendor assets",
    "dependencies": {
      "three.js"    : "https://raw.github.com/mrdoob/three.js/master/build/three.js"
    }
  }
}
```

##Ruby DSL configuration

The Ruby DSL configuration is a Bowerfile at the project's root with DSL syntax similar to Bundler.

Run the initializer to generate a sample Bowerfile inside the Rails root and a `config/initializers/bower_rails.rb` config file:

``` Bash
  rails g bower_rails:initialize
```

**Example Bowerfile**

By default assets are put to `./vendor/assets/bower_components` directory:

``` ruby

# Puts to ./vendor/assets/bower_components
asset "backbone"
asset "moment", "2.0.0" # get exactly version 2.0.0
asset "secret_styles", "git@github.com:initech/secret_styles" # get from a git repo
# get from a git repo using the tag 1.0.0
asset "secret_logic", "1.0.0", git: "git@github.com:initech/secret_logic"
# short-hand for
# asset "secret_logic", "git@github.com:initech/secret_logic#1.0.0"
```

But the default value can be overridden by `assets_path` method:

``` ruby
assets_path "assets/my_javascripts"

# Puts to ./vendor/assets/my_javascripts/bower_components
asset "backbone"
asset "moment"
```

And finally, the `assets_path` method can be overridden by an option in a `group` call:

``` ruby
assets_path "assets/javascript"

# Puts files under ./vendor/assets/js/bower_components
group :vendor, :assets_path => "assets/js"  do
  asset "jquery"            # Assummes it's latests
  asset "backbone", "1.1.1"
end

# Puts files under ./lib/assets/javascript/bower_components
group :lib do
  asset "jquery"
  asset "backbone", "1.1.1"
end
```
NOTE: Available groups are `:lib` and `:vendor`. Others are not allowed according to the Rails convention.
NOTE: All the assets should be stored in `/assets` subdirectory so putting it under `./vendor/js` directory is unavailable

##Configuration

Change options in your `config/initializers/bower_rails.rb`:

``` ruby
BowerRails.configure do |bower_rails|
  # By default options are false
  bower_rails.resolve_before_precompile = true # invokes rake bower:resolve before precompilation
  bower_rails.clean_before_precompile = true   # invokes rake bower:clean before precompilation
end
```

##Rake tasks

Once you are done with `bower.json` or `Bowerfile` you can run

* `rake bower:install` to install js components
* `rake bower:update` to update js components
* `rake bower:update:prune` to update components and uninstall extraneous packages
* `rake bower:list` to list all packages
* `rake bower:clean` to remove all files not listed as [main files](#bower-main-files) (if specified)
* `rake bower:resolve` to resolve [relative asset paths](#relative-asset-paths) in components

If you'd like to pass any bower CLI options to a rake task, like `-f`, `-j`, you can simply do:

```bash
rake bower:install['-f']
```

##Bower Configuration

If you provide a `.bowerrc` in the rails project root, bower-rails will use it for bower configuration.
Some .bowerrc options are not supported: `directory`, `cwd`, and `interactive`. Bower-rails
will ignore the `directory` property and instead will use the automatically generated asset path.

###Bower Installation

[Bower](https://github.com/bower/bower) should be installed using npm. Bower can be installed globally (with `$ npm install -g bower`) or in `node_modules` in the root directory of your project.

##Relative asset paths

Some bower components (eg. [Bootstrap](https://github.com/twbs/bootstrap/blob/0016c17f9307bc71fc96d8d4680a9c861f137cae/dist/css/bootstrap.css#L2263)) have relative urls in the CSS files for imports, images, etc. Rails prefers using [helper methods](http://guides.rubyonrails.org/asset_pipeline.html#coding-links-to-assets) for linking to assets within CSS. Relative paths can cause issues when assets are precompiled for production.

Remember that you should have [bower installed](#bower-installation) either locally in your project or on a remote server.

##Bower Main Files

Each bower component should follow the [bower.json spec](https://github.com/bower/bower.json-spec) which designates a recommended `main` directive that lists the primary files of that component. You may choose to reference these files if you are using the asset pipeline, in which case other extraneous includes of the bower component are not needed. The `rake bower:clean` task removes every file that isn't listed in the `main` directive, if the component specifies a `main` directive. Otherwise, the library will remain as bower installed it.
