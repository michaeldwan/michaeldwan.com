This is the source for my website running at [michaeldwan.com](http://michaeldwan.com). All content is written by me, Michael Dwan. The site is built atop [Jekyll](http://jekyllrb.com) and hosted on [S3](http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html).

# Getting Started

```
bundle install
bundle exec rake serve
```

This site uses mostly vanilla Jekyll, so [read the docs](http://jekyllrb.com/docs/home/) for more info. 
There are a few custom things going on though:

* The actual source files are under the `source` directory instead of the repo root. Jekyll compiles the source to the build directory.
* [jekyll-assets](https://github.com/ixti/jekyll-assets) is used to compile [SCSS](http://sass-lang.com) files, located in `source/_assets/stylesheets`, into CSS. Jekyll 2.0 has builtin support for SCSS, but jekyll-assets appends a hash to the generated asset files which ensures updated assets aren't cached downstream. It also plays nicely with images.
* The [alias generator](http://github.com/tsmango/jekyll_alias_generator) plugin is being used to generate redirect pages.

# Configuration

I love when people fork and redeploy this code, but a few times it's been done while still using my site's Google Anaytics or Disqus configs. To stop that from happening, I've removed the hardcoded ids and split out configuration into global and personal config files.

Standard Jekyll settings are in the `_config.yml` file and config specific to my site is in an uncommitted file called `_local_config.yml`. An example of what my personal config looks is in `_local_config.yml.example`.

When calling Jekyll, just specify both config files and the values will be merged.

`bundle exec jekyll serve --config _config.yml,_local_config.yml`

A `rake serve` task wraps that call up so you don't have to remember the `--config` flag each time:

`bundle exec rake serve`

# Deployment

My site is hosted on S3 and served with CloudFront. I've created a small rake task that builds the site and deploys it to an S3 bucket. It only uploads new or changed files, so deploys are fast even for large sites. It'll also remove any stale files left from previous deploys.

Your AWS keys and bucket are through `ENV` variables. I'm using the [dotenv](https://github.com/bkeepers/dotenv) gem to load variables from a `.env` file before deploying. Since AWS keys are sensitive, the `.env` file isn't committed to source. Take a look at the `.env.example` file to see what your `.env` file should look like.

To deploy, just run:

`rake deploy`

You can probably also deploy this to Github pages, though I haven't tried it in a few years.

# License

Feel free to use this code however you want, just attribute me somewhere on your site with a link back to [michaeldwan.com](http://michaeldwan.com). Enjoy.
