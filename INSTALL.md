# Updating dependencies

GitHub will show a warning on the repository’s home page if outdated dependencies are a security vulnerability.

To update dependencies, run `bundle update`.


# Troubleshooting local Jekyll installation

## macOS

If `sudo gem install bundler` fails with:

> ERROR:  While executing gem ... (TypeError)
> no implicit conversion of nil into String

… then `sudo gem update --system`

If `bundle install'` fails with:

> An error occurred while installing nokogiri (1.8.3), and Bundler cannot continue.
> Make sure that `gem install nokogiri -v '1.8.3'` succeeds before bundling.

… then `bundle config build.nokogiri --use-system-libraries` and retry.

If installing nokogiri fails with:

> ERROR: cannot discover where libxml2 is located on your system. please make sure `pkg-config` is installed.

… then: `sudo gem install nokogiri -v '1.8.3' -- --with-xml2-include=/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/libxml2 --use-system-libraries`


## Ubuntu

1. Run `bundle update` to check that the ruby bundler is installed
    1. If it is not installed do ```sudo apt install ruby-bundler```
2. Run `bundle install`
  * Problem running `bundle install`: `Gem::Ext::BuildError: ERROR: Failed to build gem native extension.`
     * Run `sudo gem install json -v '1.8.6' -- --use-system-libraries`
   * Problem with `sudo gem install json...`: `ERROR:  Error installing json:`
     * Run `sudo apt-get install ruby-dev`
3. Run `bundle exec jekyll build`
4. Run `bundle exec jekyll serve` to start up a local instance.
5. Now you can browse the content in your favorite web browser
