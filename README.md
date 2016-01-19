# Getting Started

This guide is intended to introduce programming beginners to making
websites, in particular static websites. It aims to assume little prior
knowledge except for the very basics of programming with Clojure.

> If you have any questions while following this guide please open an
> issue or join the `#perun` channel in the http://clojurians.net
> Slack group.

#### Step 1: Install Boot

[Boot][boot] is a tool to help authoring Clojure projects.
It downloads dependencies and can build your project.

Installation instructions can be found in Boot's [Readme][boot-install].
If you already have Boot installed make sure you're using the latest version (`boot -u`).

[boot]: https://github.com/boot-clj/boot
[boot-install]: https://github.com/boot-clj/boot#install

#### Step 2: Create a new directory & add a `build.boot` file

> Note: This tutorial will assume a UNIX-like system (i.e. Linux/Mac), if you're using
> windows you can either look up the respective commands or try using the Explorer for basic tasks.
> Here are some basic intro links to working with a terminal:
> [Mac][terminal-basics-mac], [Linux][terminal-basics-linux], [Windows][terminal-basics-windows]

Create a directory:
```
mkdir my-website                         # creates the directory
cd my-website                            # moves you into the directory
```

and use your favorite text editor to add a file `build.boot` as below:

```clojure
(set-env!
 :source-paths #{"src" "content"}
 :dependencies '[[perun "0.3.0" :scope "test"]])

(require '[io.perun :refer :all])
```

also add a file `boot.properties` (dont worry about it for now):

```sh
#http://boot-clj.com
#Mon Jan 18 23:19:36 CET 2016
BOOT_CLOJURE_NAME=org.clojure/clojure
BOOT_CLOJURE_VERSION=1.7.0
BOOT_VERSION=2.5.5
BOOT_EMIT_TARGET=no
```

#### Step 3: The Simplest Thing Possible &trade;

Now let's create the simplest possible website possible: a single page with some text on it.

Create the directory `content` and place a file `index.markdown` inside it:

```markdown
# Hello World
We are making a website!
```

> Note: [Markdown][markdown] is a lightweight markup language with
> plain text formatting syntax designed so that it can be converted to
> [HTML][html] (among other formats).

Now Boot allows us to run tasks from the terminal. To see what tasks
we have at our disposal we can run `boot --help` from a terminal
session.

The printed output will have a section similar to this:
```
Tasks:   add-repo                    Add all files in project git repo to fileset.
         aot                         Perform AOT compilation of Clojure namespaces.
         checkout                    Checkout dependencies task.
         help                        Print usage info and list available tasks.
         install                     Install project jar to local Maven repository.
         jar                         Build a jar file for the project.
         [...]
         atom-feed                   Generate Atom feed
         base                        Adds some basic information to the perun metadata and
         build-date                  Add :date-build attribute to each file metadata and also to the global meta
         canonical-url               Adds :canonical-url key to files metadata.
         collection                  Render collection files
         draft                       Exclude draft files
         global-metadata             Read global metadata from `perun.base.edn` or configured file.
         gravatar                    Find gravatar urls using emails
         images-dimensions           Adds images' dimensions to the file metadata:
         images-resize               Resize images to the provided resolutions.
         inject-scripts              Inject JavaScript scripts into html files.
         markdown                    Parse markdown files
         permalink                   Adds :permalink key to files metadata. Value of key will determine target path.
         print-meta                  Utility task to print perun metadata
         render                      Render pages.
         rss                         Generate RSS feed
         sitemap                     Generate sitemap
         slug                        Adds :slug key to files metadata. Slug is derived from filename.
         ttr                         Calculate time to read for each file
         word-count                  Count words in each file
```

The part below the `[...]` has been added to the set of available
tasks by the code we put into our `build.boot` file and are part of
[Perun][perun], the static site generator we will use in this guide.

> Perun works, similarily to (and on top of) Boot, by tasks operating on
> a value and producing a new value. By ensuring that the newly
> produced value looks similarily to the received value you can mix
> and match tasks as you wish (this is often called composition).

Since we've just created a `index.markdown` file, let's try the `markdown` task:

```sh
$ boot markdown
[markdown] - parsed 1 markdown files
```

Great! We parsed a Markdown file. But where did the resulting
[HTML][html] go?  As mentioned earlier Perun works by passing values
from task to task. Most of Perun's tasks don't write files directly
but instead add information to this value so that it can all be
written to files at a later stage.

```
┌───────┐            ┌───────┐            ┌───────┐            ┌───────┐
│ Value │   Task 1   │ Value │   Task 2   │ Value │   Task 3   │ Value │
│ V0    │───────────▶│ V1    │───────────▶│ V2    │───────────▶│ V3    │
│       │            │       │            │       │            │       │
└───────┘            └───────┘            └───────┘            └───────┘
```

To inspect the value that is passed from task to task we can use the
`print-meta` task. Below you can see how `print-meta` first prints an
empty list `()` and after the markdown task ran prints more information
including the rendered markdown.

```sh
$ boot print-meta markdown print-meta
()
[markdown] - parsed 1 markdown files
({:content "<h1><a href=\"#hello-world\" name=\"hello-world\"></a>hello world</h1>",
  :extension "md",
  :filename "index.md",
  :full-path "/Users/martin/etc/boot/cache/tmp/Users/martin/code/perun-guide/k8y/-grrwi1/index.md",
  :original true,
  :parent-path "",
  :path "index.md",
  :short-filename "index"})
```

Let's do something with that information, let's render it to a
file. Perun provides a `render` task. Let's use `boot` to figure out
what it does:

```sh
TODO fix this
$ boot render --help
Render individual pages for entries in perun data.

The symbol supplied as `renderer` should resolve to a function
which will be called with a map containing the following keys:
 - `:meta`, global perun metadata
 - `:entries`, all entries
 - `:entry`, the entry to be rendered

Entries can optionally be filtered by supplying a function
to the `filterer` option.

Filename is determined as follows:
If permalink is set for the file, it is used as the filepath.
If permalink ends in slash, index.html is used as filename.
If permalink is not set, the original filename is used with file extension set to html.

Options:
  -h, --help               Print this help info.
  -o, --out-dir OUTDIR     Set the output directory to OUTDIR.
      --filterer FILTER    Set filter function to FILTER.
  -r, --renderer RENDERER  Set page renderer (fully qualified symbol which resolves to a function) to RENDERER.
```

Ok, so rendering individual pages for entries. Given that we only have
a single entry right now that sounds like what we want. Let's try just calling
the `render` task after the `markdown` task:

```
$ boot markdown render
[markdown] - parsed 1 markdown files
             clojure.lang.ExceptionInfo: java.lang.AssertionError: Assert failed: Renderer must be a fully qualified symbol, i.e. 'my.ns/fun
                                         (and (symbol? sym) (namespace sym))
    data: {:file
           "/var/folders/ss/4qg3hk1d4nv40phg1360ng5w0000gn/T/boot.user1104049499782741856.clj",
           :line 19}
java.util.concurrent.ExecutionException: java.lang.AssertionError: Assert failed: Renderer must be a fully qualified symbol, i.e. 'my.ns/fun
                                         (and (symbol? sym) (namespace sym))
               java.lang.AssertionError: Assert failed: Renderer must be a fully qualified symbol, i.e. 'my.ns/fun
                                         (and (symbol? sym) (namespace sym))
          io.perun/assert-renderer!  perun.clj:  375
             io.perun/render-in-pod  perun.clj:  379
         io.perun/eval2098/fn/fn/fn  perun.clj:  417
         io.perun/eval1600/fn/fn/fn  perun.clj:  116
                boot.core/run-tasks   core.clj:  794
                  boot.core/boot/fn   core.clj:  804
clojure.core/binding-conveyor-fn/fn   core.clj: 1916
                                ...
```

Duh, that didn't work. The error tells us we need to supply a
(fully-qualified) symbol to the `renderer` option pointing to a
function. Let's create a namespace with a renderer function for our
page. First create the required directory structure:

```sh
mkdir -p src/site
```

Now add a file at `src/site/core.clj` containing the following:

```clojure
(ns site.core
  (:require [hiccup.page :as hp]))

(defn page [data]
  (hp/html5
   [:div {:style "max-width: 900px"}
    (-> data :entry :content)]))
```

We've added a function that renders a bit of HTML and inserts what has
previously been parsed by the `markdown` task (the `:content`). The
rendering is done by [Hiccup][hiccup] a library to convert Clojure
data structures to HTML. A simplistic example would be:

```
[:span {:class "foo"} "bar"]    ; Clojure
<span class=\"foo\">bar</span>  ; HTML
```

Now that we have a function that we can use as `renderer` let's give it a try:

```sh
$ boot markdown render -r site.core/page  # -r is a shorthand for --renderer
```

**Error!** Our program can't find the Hiccup code because we haven't
added it to our list of dependencies. Modify `build.boot` so it looks
like this:

```clojure
(set-env!
 :source-paths #{"src" "content"}
 :dependencies '[[perun  "0.3.0" :scope "test"]
                 [hiccup "1.0.5"]])

(require '[io.perun :refer :all])
```

> Note: By adding a dependency to the list of `:dependencies` in `build.boot`
> you make it available to the rest of your program.

Now try the command from above again and see that it works:

```sh
$ boot markdown render -r site.core/page
[markdown] - parsed 1 markdown files
[render] - rendered 1 pages
```

Still we don't see any files being generated, to fix that just append
the `target` task to your command:

```sh
$ boot markdown render -r site.core/page target
[markdown] - parsed 1 markdown files
[render] - rendered 1 pages
Writing target dir(s)...
```

> Note: Remember the value we spoke about earlier that is passed from
> task to task? In Boot this value describes a directory structure and
> files inside it. Whenever the `target` task is used Boot will sync
> relevant files from this description to an actual target directory.

Now there should be a directory called `target` containing another
directory `public`, finally containing a file `index.html`.
If you open that file you should see your new website! :tada:

Admittedly it is a very basic website but let's recap what we've done:

1. We have decoupled our actual content from the rendered HTML using
   Markdown. This makes it easy to write and edit our site.
1. We have a small function rendering our page to an actual HTML file
   giving us full control over the rendered HTML and it's appearance.
1. Because an HTML file is generated for each Markdown file it's
   trivial to add new pages.

#### Adding more pages and proper serving

Let's add an about page by adding the file `content/about.markdown`:

```markdown
# About this site
This site has been made by following the Perun guides
```

Also so that our visitors can find our new about page let's change our
`index.markdown` file to look like this:

```markdown
# Hello World
We are making a website! ([about this website](/about.html))
```

After rebuilding our site by running `boot markdown render -r
site.core/page target` we can open `target/public/index.html` again
and see that there is a link to our new about page. If we click it
however there will be an error.

This is because currently we're just opening those files from our
filesystem and not retrieving them from a server as it's usually done
with websites. To get closer to the "mode of operation" of an actual
website we need to serve our website over [HTTP][http].

Add `[pandeiro/boot-http "0.7.0"]` to the list of `:dependencies` in
your `build.boot`. Also modify the `require` statement in that file to
look like this:

```clojure
(require '[io.perun :refer :all]
         '[pandeiro.boot-http :refer [serve]])
```

Now run this:

```
boot serve --resource-root public markdown render -r site.core/page wait
```

There are two new things here:

1. `serve --resource-root public` — The `serve` task starts an HTTP
    server and serves files from the JAVA classpath, which we can
    think of as an imaginary directory somewhere on our computer
    containing lots of files. When we previously used the `target`
    task we saw that our generated files were all in a directory
    `public` so we tell the server to only respond to requests for
    files in `public`. (We could have also used the shorthand `-r`
    option instead of `--resource-root` by the way.)
2. `wait` tells the task pipeline to wait even after it has finished.
    This way we ensure that even after all files are generated the server
    will keep running to serve those files.

Now after running this command there will be a line printed like this one:
```
Started Jetty on http://localhost:3000
```
Go to http://localhost:3000 — you should see the index.html file with
the link to your about page. Clicking on the link will bring you to
http://localhost:3000/about.html properly displaying your about page.

**Success!** Every good website should have a way to go back to it's
homepage however so let's modify our renderer function to always show
a link to the homepage ("root") of the site. In `src/site/core.clj` change
the `page` function to look like this

```clojure
(defn page [data]
  (hp/html5
   [:div {:style "max-width: 900px; margin: 0 auto;"}
    [:a {:href "/"} "Home"] ; <---- We added this
    (-> data :entry :content)]))
```

After restarting our site building & serving command go to
http://localhost:3000 again and view your site. On both pages there
should now be a little "Home" link at the top bringing you back to the
index page.

**One more thing before you're done:** Currently every time we make a
change we have to restart our command.  To avoid this you can adapt
the command like this:

```sh
boot serve -r public watch markdown render -r site.core/page
```
The `watch` task will rebuild your page whenever an important file
changes. (Because the watch tasks keeps the pipeline running we don't
need `wait` any longer.) **Give it a try by editing `index.markdown`
and reloading the browser!**

#### This is it!

You're at the end of the "Getting Started" tutorial. There are many
things still to be explored, proceed with whatever interests you most:

- Admittedly our site isn't very pretty right now. [CSS][css] can be
  used to influence the appearance of HTML. It can color texts and
  backgrounds and do a large number of other things. If you're
  familiar with CSS you might already know how to add more CSS to the
  page, if not you can learn all about CSS in the
  [excellent material created by CSSClasses][cssclasses-guide].
- Remembering and typing the long `boot serve ...` command can be
  annoying, learn how you can define your own tasks consisting of
  several sub-tasks in the [Boot Task Guide][task-guide]
- Everyone can have a [blog][blog], they're a good way to keep friends
  updated, talk about technical problems and their solutions or write
  about whatever you want. If you'd like to add a blog to your new
  site or are just curious how that would work
  [check out the Blog Guide][blog-guide].
- You now have a website, great! But it's not on the internet and so
  you can't show it to any of your friends. To learn more about how to
  deploy your website [check out the Deployment Guide][deploy-guide].

#### Questions?

If you have any questions please open an issue or join the `#perun`
channel in the http://clojurians.net Slack group.

[terminal-basics-mac]: http://mac.appstorm.net/how-to/utilities-how-to/how-to-use-terminal-the-basics/
[terminal-basics-linux]: http://community.linuxmint.com/tutorial/view/100
[terminal-basics-windows]: http://www.cs.princeton.edu/courses/archive/spr05/cos126/cmd-prompt.html
[perun]: https://github.com/hashobject/perun
[markdown]: https://en.wikipedia.org/wiki/Markdown
[html]: https://en.wikipedia.org/wiki/HTML
[http]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[hiccup]: https://github.com/weavejester/hiccup
[css]: https://en.wikipedia.org/wiki/Cascading_Style_Sheets
[cssclasses-guide]: http://cssclasses.cssconf.eu/materials/#css
[blog]: https://en.wikipedia.org/wiki/Blog

# Talk

- Making websites is fun, they're a form of self-expression
- There are various ways to publish content on the internet:
  1. a running program responding to requests
  2. a minimal server serving just files
  3. using commercial 3rd party services (no tinkering!)
- First is annoying, third is boring, so we'll focus on the second. (more benefits etc)
- Making a static site (great first intro to web stuff)
  - pure function f(your-files) -> your-site
  - hosting is very easy and cheap/free
  - you need to know a bit of HTML and CSS but it's easy to learn as
    you go + there are great guides (insert resources)
- Of course you can make them with Clojure
  - multiple options but I will recommend perun
  - quick example, very basic
  - more involved
