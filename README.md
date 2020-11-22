GitHub Markdown on GitHub Pages
===============================

GitHub Pages are great but wouldn't you like to be able to use .md files, instead of having to provide HTML, and have those files render like they do in GitHub README's etc?

The following covers how to do this.

Install RVM
-----------

Install [RVM](https://rvm.io/) as per their instructions:

    $ sudo gpg2 --keyserver hkp://pool.sks-keyservers.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    $ curl -sSL https://get.rvm.io | bash -s stable --ruby --ignore-dotfiles

I added `--ruby` for the default version or Ruby and `--ignore-dotfiles` as otherwise it completely spams a surprising array of dot file.

Then all you need to add to your `~/.bashrc` is:

    # Add RVM to PATH for scripting.
    export PATH=$HOME/.rvm/bin:$PATH

    # Load RVM into a shell session *as a function*.
    source $HOME/.rvm/scripts/rvm

Open a new terminal to pull in these changes.

Install Jekyll
--------------

Install Jekyll (as per the Jekyll [installation instructions](https://jekyllrb.com/docs/installation/ubuntu/):

    $ gem install jekyll bundler

Create your site
----------------

Then make a directory for your experiments, this will _not_ be the root for your GitHub pages site but rather its parent:

    $ mdkir experiment
    $ cd experiment

Then roughly following the [details](https://docs.github.com/en/free-pro-team@latest/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll) from GitHub docs:

    $ bundle init
    $ echo '' >> Gemfile
    $ echo 'gem "jekyll", "~> 3.9.0"' >> Gemfile

Adjust `3.9.0` to match the version currently specified in <https://pages.github.com/versions/> and then:

    $ bundle install

Now we'll create the subdirectory containing the actual GitHub pages site:

    $ bundle exec jekyll new my-gh-pages-site
    $ cd my-gh-pages-site

Now edit the new `Gemfile` that's been created here:

    $ vi Gemfile

Delete the `gem "jekyll"` line and uncomment the `gem "github-pages"` line but add in a version so you end up with something like this:

    gem "github-pages", "~> 209", group: :jekyll_plugins

The `, "~> 209"` is the added-in bit and the version, here `209`, again comes from <https://pages.github.com/versions/>

Now install everything and serve the canned initial content:

    $ bundle install
    $ bundle exec jekyll serve

Setup code highlighting
-----------------------

Code highlighting is on by default but there's no default CSS, to create it:

    $ mkdir css
    $ bundle exec rougify style github > css/syntax.css

Setup a layout using GitHub Markdown style
------------------------------------------

First delete the canned initial content:

    $ rm -r about.md index.md _posts

And create some Jekyll specific subdirectories:

    $ mkdir _layouts _includes

And copy [`_layouts/default.html`](_layouts/default.html) into `_layouts` and [`_includes/anchor_links.html`](_includes/anchor_links.html) into `_includes`.

Note that `_layouts/default.html` depends on Sindre Sorhus's [github-markdown-css](https://github.com/sindresorhus/github-markdown-css) to achieve the GitHub Markdown style.

By default smart-quotes are enable - which isn't the default on GitHub. So to disable this:

    $ echo '' >> _config.yml 
    $ echo 'kramdown:' >> _config.yml 
    $ echo '  smart_quotes: ["apos", "apos", "quot", "quot"]' >> _config.yml

The `Site settings` block in `_config.yml` contains various values:

* `twitter_username` and `github_username` can be removed
* `title`, `email` and `description` appear to be mandatory but can be left with their default canned values.
* Only the `baseurl` and `url` entries are really important - we'll come back to them later.

Creating the first page
-----------------------

Now you're ready to go - just take any GitHub markdown page and save it as `index.md`, e.g. take the raw markdown for this page.

Now all you need to change is to add what's called a _front matter_ section at the top of the file:

    ---
    layout: default
    title: My example page
    ---

The `layout` line tells Jekyll to use the `default.html` layout file that we set up above. And the `title` line sets the value used for the page's `<title>` tag.

Now we're ready to serve it locally:

    $ bundle exec jekyll serve

Open the link <http://127.0.0.1:4000/> in your browser to see the result.

Getting the result served by GitHub pages
-----------------------------------------



Things to check
---------------

The following are examples that test that everything renders as it does in GitHub markdown.

The `_` character within words shouldn't turn on emphasis, i.e. foo_bar shouldn't render as foo<i>bar</i>, i.e. without the underscore and with the "bar" part in italics.

Tables should render properly:

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

Standard Markdown supports code blocks, with no special highlighting, that are created by indenting each line by four spaces.

With `fenced_code_blocks` code blocks can be fenced by three backticks with an optional language name from [languages.yml](https://github.com/github/linguist/blob/master/lib/linguist/languages.yml).

The following fenced code block should have syntax highlighting:

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```

GitHub automatically renders things like the following as links:

foo@bar.com  
www.example.com  
http://www.example.com/

This isn't supported, if you want them rendered as links you must surround them with `<` and `>`, e.g. `<foo@bar.com>` renders as <foo@bar.com>.

You can strikethrough text with two `~` characters at the start and end, e.g. `~~strikethrough~~` renders as ~~strikethrough~~.

In traditional Markdown any HTML has to be bounded by blank lines in order to be interpreted as HTML. But this shouldn't be necessary here - the following should render as a single cell HTML table despite not being surrounded by blank lines:

A non-empty line before the HTML.
<table>
    <tr>
        <td>Single cell HTML table</td>
    </tr>
</table>
A non-empty line after the HTML.

GitHub used to support you being sloppy and not having to add a space after the hashes used to introduce a heading, but no more. The example here without the space should not render as a heading:

##### With a space h5-header

#####Without a space h5-header

GitHub disables using `==` to highlight text, if it were enabled then ==highlighted== would appear as <mark>highlighted</mark>.

Smart quotes are disabled by GitHub, if they weren't then then "quote" would appear as <q>quote</q>.
