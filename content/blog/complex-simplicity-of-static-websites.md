---
title: "The complex simplicity of my static websites"
subtitle: "A deep dive into using indentation-based languages to build websites"
excerpt: |-
  My love of indentation-based languages pushed me on a year-long dive into how I could use them for website building.

  Here’s me, trying to fit the Python peg into the HTML hole, and sharing how I built (and keep building) all my websites.

description: A deep dive into using indentation-based languages like Python and Slim, to build beautiful websites and love the work of doing it till the end.

date: 2023-08-08T10:14:26+03:00
author: Alin Panaitiu
draft: false
image: "complex-simplicity-of-static-websites.png"
images:
    - "complex-simplicity-of-static-websites.png"

series:
tags:
    - web dev
    - static websites
    - one page websites
    - python
    - plim
    - hugo
    - blog
categories:
    - web dev

pageStyles:
  - file: article.sass
    media: "(prefers-color-scheme: light), (prefers-color-scheme: no-preference)"
  - file: article-dark.sass
    media: "(prefers-color-scheme: dark)"
---


It was the spring of 2014, over 9 years ago, just 6 months into my first year of college, when my Computer Architecture teacher stopped in the middle of an `assembly` exercise to tell us that [Bitdefender](https://www.bitdefender.com/company/) is hiring juniors for Malware Researcher positions.

I had no idea what that is, but boy, did it sound cool?...

I fondly remember how at that time we weren’t chasing high salaries and filtering jobs by programming languages and frameworks. We just wanted to learn something.

> As students, we needed money as well of course, but when I got the job for `1750 lei` *(~€350)*, I suddenly became the richest 18 year old in my home town, so it wasn't the top priority.

And we learnt so much in 2 years.. obscure things like [AOP](https://www.baeldung.com/aspectj), a lot of x86 assembly, reverse engineering techniques which dumped us head first into languages like Java, .NET, *ActionScript? (malware authors were creative)*.

But most of all, we did tons of *Python scripting*, and we loved every minute of it. It was my first time getting acquainted with fast tools like [Sublime Text](https://www.sublimetext.com) and [FAR Manager](https://www.farmanager.com/screenshots.php). Coming from Notepad++ and Windows Explorer, I felt like a mad hacker with the world at my fingertips.

I’m known as a macOS app dev nowadays, but 9 years ago, I actually started by writing petty Python scripts which spurred the obsessive love I have nowadays for clean *accolade-free* code and indentation based languages.

*What does all that have to do with static websites though?*

## Pythonic HTML

Well, 5 years ago, when I launched [my first macOS app](https://lunar.fyi/), I found myself needing to create a simple webpage to showcase the app and at the very least, provide a way to download it.

And HTML I did not want to write. The XML like syntax is something I always dreaded, so overfilled with unnecessary `</>` symbols that make both writing and reading much more cumbersome. I wanted Python syntax for HTML so I went looking for it.

I went through [pug](https://pugjs.org/language/attributes.html)…

```coffee
doctype html
html
  head
    title Lunar - The defacto app for controlling monitor brightness
    meta(itemprop='description' content='...')
    style.
      a.button {
        background: bisque;
        padding: 0.5rem 1rem;
        color: black;
        border-radius: 0.5rem;
      }
      body {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
      }
  body
    h1(style='color: white; font: bold 3rem monospace') Lunar
    img(src='https://files.lunar.fyi/display-page.png' style='width: 80%')
    a.button(href='https://files.lunar.fyi/releases/Lunar.dmg') Download
```
*pretty, but still needs `()` for attributes, and I still need accolades in CSS and JS*


then [haml](https://haml.info/tutorial.html)…

```coffee
!!!
%html{lang: "en"}
  %head
    %meta{content: "text/html; charset=UTF-8", "http-equiv" => "Content-Type"}/
    %title Lunar - The defacto app for controlling monitor brightness
    %meta{content: "...", itemprop: "description"}/
    :css
      a.button {
        background: bisque;
        padding: 0.5rem 1rem;
        color: black;
        border-radius: 0.5rem;
      }
      body {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
      }
  %body{style: "background: #2e2431; min-height: 90vh"}
    %h1{style: "color: white; font: bold 3rem monospace"} Lunar
    %img{src: "https://files.lunar.fyi/display-page.png", style: "width: 80%"}/
    %a.button{href: "https://files.lunar.fyi/releases/Lunar.dmg"} Download
```
*even more symbols: `%`, `:`, `=>` and `/` for self-closing tags*

...and eventually stumbled upon [Slim](https://slim-template.github.io) and its Python counterpart: [Plim](https://plim.readthedocs.io/en/latest/)

```stylus
doctype html
html lang="en"
  head
    title Lunar - The defacto app for controlling monitor brightness
    meta itemprop="description" content="..."
    -stylus
      a.button
        background bisque
        padding 0.5rem 1rem
        color black
        border-radius 0.5rem
      body
        display flex
        flex-direction column
        align-items center
        justify-content center
        text-align center

  body style="background: #2e2431; min-height: 90vh"
    h1 style="color: white; font: bold 3rem monospace" Lunar
    img src="https://files.lunar.fyi/display-page.png" style="width: 80%"
    a.button href="https://files.lunar.fyi/releases/Lunar.dmg" Download
```
*ahhh.. so clean!*

Here's how that example looks like if I would have to write it as HTML:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Lunar - The defacto app for controlling monitor brightness</title>
        <meta itemprop="description" content="...">
        <style>
            a.button {
                background: bisque;
                padding: 0.5rem 1rem;
                color: black;
                border-radius: 0.5rem;
            }

            body {
                display: flex;
                flex-direction: column;
                align-items: center;
                justify-content: center;
                text-align: center;
            }
        </style>
    </head>

    <body>
        <h1 style="color: white; font: bold 3rem monospace">Lunar</h1>
        <img src="https://files.lunar.fyi/display-page.png" style="width: 80%">
        <a class="button" href="https://files.lunar.fyi/releases/Lunar.dmg">Download</a>
    </body>
</html>
```
*not particulary hard to read, but writing would need a lot of Shift-holding and repeating tags*

The thing I like most about Plim, and why I stuck with it, is that it can parse my other favorite *symbol-hating* languages without additional configuration:

- Python for abstracting away repeating structures
- Stylus for writing `style` tags
- CoffeeScript for the `script` tags
- Markdown for long text content

Here's a more complex example to showcase the above features *[(might require sunglasses)](https://youtu.be/cVAcRH9b44w?t=55)*:

*example of writing a HDR page section, similar to the one on [lunar.fyi](https://lunar.fyi#xdr)*

```coffee
---!
  # use Python to generate the dynamic image sizes for the srcset attr

  WIDTHS = [1920, 1280, 1024, 768, 640, 320]

  def srcset(image, ext, page_fraction=1.0):
    return ','.join(
      f'/img/{image}/{width}.{ext} {width // page_fraction:.0f}w'
      for width in WIDTHS
    )

doctype html
html lang="en"
  head
    -stylus
      # use Stylus to do a readable media query that checks for wide color gamut

      @media screen and (color-gamut: p3)
        @supports (-webkit-backdrop-filter: brightness(1.5))
          section#xdr
            -webkit-backdrop-filter: brightness(1)
            filter: brightness(1.5)

  body
    section#xdr
      picture
        source type="image/webp" srcset=${srcset('xdr', 'webp', 0.3)}
        source type="image/png" srcset=${srcset('xdr', 'png', 0.3)}

      -md
        # write markdown that renders as inline HTML

        Unlock the full brightness of your XDR display

        The **2021 MacBook Pro** and the **Pro Display XDR** feature an incredibly bright panel *(1600 nits!)*,
        but which is locked by macOS to a third of its potential *(500 nits...)*.

        Lunar can **remove the brightness lock** and allow you to increase the brightness past that limit.

    -coffee
      # use CoffeeScript to detect if the browser might not support HDR

      $ = document.querySelector
      safari = /^((?!chrome|android).)*safari/i.test navigator.userAgent

      window.onload = () ->
        if not safari
          $('#xdr')?.style.filter = "none"
```

{{< rawhtml >}}
<style type="text/css">
  a[href="https://youtu.be/cVAcRH9b44w?t=55"] {
    display: none;
  }
  @media screen and (color-gamut: p3) {
    video#xdr {
      display: block !important;
    }
    a[href="https://youtu.be/cVAcRH9b44w?t=55"] {
      display: inline !important;
    }
  }
</style>

<video id="xdr" loop muted playsinline style="width: 100%; margin: 10px auto; border-radius: 8px; display: none;" >
  <source src="/video/xdr-blog-h265.mp4" type="video/mp4; codecs=hvc1">
  <source src="/video/xdr-blog.mp4" type="video/mp4; codecs=avc1">
</video>
{{< /rawhtml >}}


And best of all, there is no crazy toolchain, bundler or dependency hell involved. No project structure needed, no configuration files. I can just write a `contact.plim` file, compile it with `plimc` to a readable `contact.html` and have a `/contact` page ready!

So that’s how it went with my app: I wrote a simple `index.plim`, dropped it on Netlify and went on with my day.

#### Complexity Cost
- 1 pip install for getting the Plim CLI
- 1 npm install for getting `stylus` and `coffeescript` *(optional)*
- 1 build command for generating the HTML files

## Complex simplicity

The app managed to get quite a bit of attention, and while I kept developing it, for the next 4 years the website remained the same *heading - image - download button* single page. It was only a side project after all.

Working for US companies from Romania made good money, but it was so tiring to get through 3h of video meetings daily, standups, syntax nitpicking in PR review, SCRUM bullshit, JIRA, task writing, task assigning, estimating task time in *T-shirt sizes??*

In **April 2021** I finally got tired of writing useless code and selling my time like it was some grain silo I could always fill back up with even more work…

I bet on [developing my app further](https://alinpanaitiu.com/blog/journey-to-ddc-on-m1-macs/). Since my college days, I always chose the work that helps me learn new concepts. At some point I had to understand that I learnt enough and had to start sharing. This time I really wanted to write software that helped people, and was willing to spend my savings on it.

### Comically Stuffed Stylesheets

A more complete app also required a more complete presentation website, but the styling was getting out of hand. You would think that with `flexbox` and `grids`, you can just write vanilla CSS these days, but just adding a bit of variation requires constant jumping between the CSS and HTML files.

> A presentation page is usually only 10% HTML markup. The rest is a ton of styling and copy text, so I wanted to optimize my dev experience for that.

There’s no “go to definition” on HTML `.classes` or `#ids` because their styles can be defined **✨anywhere✨**. So you have to Cmd-F like a madman and be very rigorous on your CSS structure.

The controversial but very clever solution to this was [Tailwind CSS](https://tailwindcss.com): a large collection of short predefined classes that *mostly* style just the property they hint at.

For example in the first code block I had to write a non-reusable 5-line style to center the body contents.

```css
body {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}
```

With Tailwind, I would have written the `body` tag like so:

```sass
body.flex.flex-col.justify-center.items-center.text-center
```

That might not seem like much, some would argue that it’s even a lot less readable than the CSS one. Can’t I *just* define a `.center` class that I can reuse?

Well, think about a few things:

{{< rawhtml >}}
<div id="tailwind-usefulness"></div>
{{< /rawhtml >}}

- this might repeat on many sections of the page, but with slight variations *(what if I want a centered row, or longer paragraphs of text aligned to the left)*
- **responsive** sections might need to alter layout (e.g. vertical on mobile, horizontal on desktop) and media queries will quickly blow up the style size
	- `.md:flex-row.flex-col` is what you would write in Tailwind
- adding **dark/light mode** support is yet another media query
	- `.dark:bg-white.bg-black` looks simple enough
- interactions like **hover effects**, complex *shadows* and filters like *blur* and *brightness* is syntax that’s often forgotten
	- `.shadow.hover:shadow-xl` creates a lift off the page effect on hover by making the shadow larger
	- `.blur.active:blur-none` un-blurs an element when you click on it
- choosing **[colors](https://tailwindcss.com/docs/customizing-colors)** and reusing them needs a lot of attention
	- `.bg-red-500.text-white` sets white text on saturated red
	- `red-100` is less saturated, towards white
	- `red-900` is darker, towards black

{{< rawhtml >}}
<style type="text/css">
  #tailwind-usefulness + ul > li:nth-child(4) > ul > li > code {
    transition: box-shadow 0.55s ease-in-out, filter 0.55s ease-in-out;
  }
  #tailwind-usefulness + ul > li:nth-child(4) > ul > li > code:hover {
    transition: box-shadow 0.25s ease-in-out, filter 0.15s ease-out;
  }

  #tailwind-usefulness + ul > li:nth-child(3) > ul > li:nth-of-type(1) > code {
    background: black !important;
    color: rgb(255, 200, 149) !important;
  }

  @media (prefers-color-scheme: dark) {
    #tailwind-usefulness + ul > li:nth-child(3) > ul > li:nth-of-type(1) > code {
      background: white !important;
      color: hsl(19, 72%, 32%) !important;
    }
  }

  #tailwind-usefulness + ul > li:nth-child(4) > ul > li:nth-of-type(1) > code {
    box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  }
  #tailwind-usefulness + ul > li:nth-child(4) > ul > li:nth-of-type(1) > code:hover {
    box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  }
  @media (prefers-color-scheme: dark) {
    #tailwind-usefulness + ul > li:nth-child(4) > ul > li:nth-of-type(1) > code:hover {
      box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.5), 0 8px 10px -6px rgb(0 0 0 / 0.7);
    }
  }
  #tailwind-usefulness + ul > li:nth-child(4) > ul > li:nth-of-type(2) > code {
    filter: blur(4px);
  }
  #tailwind-usefulness + ul > li:nth-child(4) > ul > li:nth-of-type(2) > code:active {
    filter: blur(0) !important;
  }
  #tailwind-usefulness + ul > li:nth-child(5) > ul > li:nth-of-type(1) > code {
    background: #ef4444 !important;
    color: white !important;
  }
  #tailwind-usefulness + ul > li:nth-child(5) > ul > li:nth-of-type(2) > code {
    background: #fee2e2 !important;
    color: black !important;
  }
  #tailwind-usefulness + ul > li:nth-child(5) > ul > li:nth-of-type(3) > code {
    background: #7f1d1d !important;
    color: white !important;
  }
</style>
{{< /rawhtml >}}

Sure, long lines of classes might not be so readable, but neither are long files of CSS styling. At least the Tailwind classes are right there at your fingertips, and you can replace a `-lg` with a `-xl`  to quickly fine tune your style.

#### Complexity Cost:
- 1 command added for building the minimal CSS from the classes used
- 1 npm install for getting the Tailwind CLI
- 1 config file for defining custom colors, animations etc. *(optional)*

### Responsive images

So many people obsess over the size of their JS or CSS, but fail to realize that the bulk of their page is *unnecessarily large and not well compressed images*.

Of course, I was one of those people.

For years, my app’s website had a screenshot of its window as an uncompressed PNG, loading slowly from top to bottom and chugging the user’s bandwidth.

{{< img src="old-lunar-website.png" alt="Old Lunar website" sizes="(min-width: 60em) 610px, 70vw" >}}

I had no idea, but screenshots and screen recordings are most of the time up to 10x larger than their visually indistinguishable compressed counterparts.

> I even wrote an app to fix that since I’m constantly sending screenshots to people and was tired of waiting for 5MB images to upload in rapid chats.
>
> It’s called **[Clop](https://lowtechguys.com/clop)** if you want to check it out.
>
> Yes, just like [that famous ransomware](https://lite.cnn.com/2023/06/16/tech/clop-ransomware-attack-explainer/index.html), it wasn’t that famous at the time of naming the app.

I needed a lot more images to showcase the features of an app controlling monitor brightness and colors, so I had to improve on this.

Delivering the smallest image necessary to the user is quite a complex endeavour:

1. Optimize the image using [ImageOptim](https://imageoptim.com/mac)
2. Resize it to fit multiple screen sizes using [vipsthumbnail](https://www.libvips.org/API/current/Using-vipsthumbnail.html)
3. Figure out what fraction of the page width will be occupied by the image
4. Write a suitable [srcset](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images#resolution_switching_different_sizes) attribute to load the suitable image
5. *Optional:* convert the image to formats like `webp`, `avif` or JPEG XL for smallest file size

I did so much of that work manually in the past… thankfully nowadays I have [imgproxy](https://imgproxy.net) to do the encoding, optimization and resizing for me.

I just have to write the srcset, for which I defined Plim and Python functions to do the string wrangling for me.

```sass
-def image(img, ext='png', factor=0.4, mobile_factor=1)
    picture
        -call img=${img} ext=${ext} factor=${factor} mobile_factor=${mobile_factor} self:sources
        img srcset=${srcset(img, ext, factor)}

-def sources(img, ext='png', factor=0.4, mobile_factor=1)
    source type="image/avif" srcset=${srcset(img, ext, mobile_factor, convert_to="avif")} media="(max-width: 767px)"
    source type="image/avif" srcset=${srcset(img, ext, factor, convert_to="avif")} media="(min-width: 768px)"

    source type="image/webp" srcset=${srcset(img, ext, mobile_factor, convert_to="webp")} media="(max-width: 767px)"
    source type="image/webp" srcset=${srcset(img, ext, factor, convert_to="webp")} media="(min-width: 768px)"

    source type="image/${ext}" srcset=${srcset(img, ext, mobile_factor)} media="(max-width: 767px)"
    source type="image/${ext}" srcset=${srcset(img, ext, factor)} media="(min-width: 768px)"

```

```python
WIDTHS = [1920, 1280, 1024, 768, 640, 320]

def imgurl(image, width, ext="png", convert_to=""):
    conversion = f"@{convert_to}" if convert_to else ""
    return f"https://img.panaitiu.com/_/{width}/plain/https://lunar.fyi/img/{urllib.parse.quote(image)}.{ext}{conversion}"


def srcset(image, ext="png", factor=1.0, convert_to=""):
    return ",".join(
        f"{imgurl(image, width, ext, convert_to)} {width // factor:.0f}w"
        for width in WIDTHS
    )
```

#### Complexity Cost
- 1 imgproxy server that needs to run somewhere publicly available, be kept alive and secure
- some Python and Plim code for generating srcsets

### Hot reloading

After 2 weeks of editing the page, Cmd-Tab to the browser, Cmd-R to refresh, I got really tired of this routine.

I worked with Next.js before on [Noiseblend](https://noiseblend.com) and loved how each file change automatically gets refreshed in the browser. Instantly and in-place as well, not a full page refresh. I got the same experience when I worked with React Native.

There should be something for static pages too, I thought. Well it turns out there is, it’s called [LiveReload](https://nitoyon.github.io/livereloadx/) and I had to slap my forehead for not searching for it sooner.

After installing the browser extension, and running the `livereloadx --static` file watcher, I got my hot reloading dev experience back.

> Actually now that I think about it, Hugo has super fast hot reloading, how does it accomplish that? Yep, turns out [Hugo uses LiveReload](https://gohugo.io/getting-started/usage/#livereload) as well.

#### Complexity Cost
- 1 more command to run in 1 more terminal panel, [multiplex](https://pypi.org/project/multiplex/) helps with that
- 1 browser extension to install and hope it’s not compromised or sold to a data thief
- 1 npm install for getting the livereloadx CLI

### Contact pages
After releasing the new app version, many things were broken, expectedly.

People tried to reach me in so many ways: Github issues, personal email, through the app licensing provider, even Facebook Messenger. I had no idea that including an official way of contact would be so vital.

And I had no idea how to even do it. A contact form needs, like, a server to `POST` to, right? And that server needs to notify me in some way, and then I have to respond to the user in some other way… *sigh*

I thought about those chat bubbles that a lot of sites have, but I used them on Noiseblend and did not like the experience. Plus I dislike seeing them myself, they’re an eyesore and a nuisance obscuring page content and possibly violating privacy.

After long searches, *not sure why it took so long*, I stumbled upon [Formspark](https://formspark.io): a service that gives you a link to `POST` your form to, and they send you an email with the form contents. The email will contain the user email in `ReplyTo` so I can just reply normally from my mail client.

```sass
form action="https://submit-form.com/some-random-id"
    label for="name" Name
    input#from hidden="true" name="_email.from" type="text"
    input#name name="name" placeholder="John Doe" required="" type="text"

    label for="email" Email
    input#email name="email" placeholder="john@doe.com" required="" type="email"

    label for="subject" Subject
    input#email-subject hidden="true" name="_email.subject" type="text"
    input#subject name="subject" placeholder="What's this message about?" required="" type="text"

    label for="message" Message
    textarea#message name="message" placeholder="Something about our apps perhaps" required="" type="text" rows="6"

-coffee
    # Custom subject and name: https://documentation.formspark.io/customization/notification-email.html#subject

    nameInput = document.getElementById("name")
    fromInput = document.getElementById("from")
    nameInput.addEventListener 'input', (event) -> fromInput.value = event.target.value

    subjectInput = document.getElementById("subject")
    emailSubjectInput = document.getElementById("email-subject")
    subjectInput.addEventListener 'input', (event) -> emailSubjectInput.value = event.target.value
```

#### Complexity Cost
None, I guess. I just hope that the prolific but [unique Formspark dev](https://github.com/botre) doesn’t die or get kidnapped or something.

## And you call this “simple”?

It’s not. Really. It’s crazy what I had to go through to get to a productive setup that fits my needs.

One could say I could have spent all that time on writing vanilla HTML, CSS and JS and I would have had the same result in the same amount of time. I agree, if time would be all that mattered.

But for some people *(like me)*, feeling productive, seeing how easy it is to test my ideas and how code seems to flow from my fingertips at the speed of thought, is what decides if I’ll ever finish and publish something, or if I’ll lose my patience and fallback to comfort zones.

Having to write the same boilerplate code over and over again, constant context switching between files, jumping back into a project after a few days and not knowing where everything was in those thousand-lines files.. these are all detractors that will eventually make me say *”f••k this! at least my day job brings money”*.

### Reusability
So many JS frameworks were created in the name of reusable components, but they all failed for me.

I mean sure, I can *“npm install”* a React calendar, and I am now *“reusing”* and not *“reimplementing”* the hard work of someone better than me at calendar UIs. But just try to stray away a little from the happy path that the component creator envisioned, and you will find it is mind-bendingly hard to bend the component to your specific needs.

You might raise a Github issue and the creator will add a few params so you can customize that specific thing, but so will others with different and maybe clashing needs. Soon enough, that component is declared unwieldy and too complex to use, the dev will say [*“f••k this! I’d rather do furniture”*](https://github.com/docker/cli/issues/267#issuecomment-695149477) and someone else will come out and say: **here’s the *next best thing* in React calendar libraries, so much simpler to use than those behemoths!**

I never had this goal in mind but unexpectedly, the above setup is generic enough that I was able to extract it into a set of files for starting a new website. I can now duplicate that folder and start changing site-specific bits to get a new website.

Here are the websites I’ve done using this method:

- [lowtechguys.com](https://lowtechguys.com/) / where I publish my apps
- [subsol.one](https://subsol.one/) / app I made for my brother to publish parties on
- [robert.panaitiu.com](https://robert.panaitiu.com) / my brother’s personal website
- [lunar.fyi](https://lunar.fyi/) / the app I have been talking about above

> And the best thing I remember is that for each website I published a working version, good looking enough, with a contact page and small bandwidth requirements, in less than a day.

How does this solve the problem of straying away from the happy path? Well, this is not an immutable library residing in *node_modules*, or a JS script on a CDN. It is a set of files I can modify to the site’s needs.

There is no high wall to jump *(having to fork a library, figuring out its unique build system etc.)* or need to stick to a specific structure. Once the folder is duplicated, it  has its own life.

> For those interested, here is the repo containing the current state of my templates: [github.com/alin23/plim-website](https://github.com/alin23/plim-website/)

> I don't recommend using it, it's possible that I'm the only one who finds it simple because I know what went into it. But if you do, I'd love to [hear your thoughts](/contact).

## Gatsby? Jekyll? Hugo?

Weirdly, this website I’m writing on is not made with Plim. At some point I decided to start a personal website, and I thought it probably needs a blog-aware site builder.

At the time, I didn't know that RSS is an easily templatable XML file, and that all I need for a blog is to write Markdown.

I remember trying [Gatsby](https://www.gatsbyjs.com/) and not liking the JS ecosystem around it. [Jekyll](https://jekyllrb.com/) was my second choice with Github Pages, but I think I fumbled too much with `ruby` and `bundle` to get it working and lost patience.

> Both problems stemmed from my lack of familiarity with their ecosystems, but my goal was to write a blog, not learn Ruby and JS.

[Hugo](https://gohugo.io/) seemed much simpler, and it was also written in Go and distributed as a standalone binary, which I always like for my tools.

I marveled at Hugo's speed, loved the fact that it supports theming (although it's not as simple as it sounds) and that it has a lot of useful stuff built-in like syntax highlighting, image processing, RSS generator etc. But it took me sooo long to understand its structure.

There are many foreign words *(to me)* in Hugo: *archetypes*, *taxonomies*, *shortcodes*, *partials*, *layouts*, *categories*, *series*. Unfortunately, by the time I realized that I don’t need the flexibility that this structure provides, I had already finished this website and written my first article.

I also used [a theme](https://blogophonic-hugo.netlify.app/) that uses the Tachyons CSS framework, for which I can never remember the [right class to use](https://tachyons.io/docs/table-of-styles/). I thought about rewriting the website in Plim but converting everything to Tailwind or simple CSS would have been a lot of work.

I eventually started writing simple Markdown files for [my notes](https://notes.alinpanaitiu.com/How%20I%20write%20this%20blog%20on%20my%20iPhone%20in%20a%20train), and have Caddy convert and serve them on the fly. Helps me write from my phone and not have to deal with Git and Hugo.

I still keep this for longform content, where a laptop is usually needed anyway.
