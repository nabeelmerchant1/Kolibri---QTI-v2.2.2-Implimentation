# QTI.JS

This is the initial release of the QTI Delivery Engine for IMS Question & Test Interoperability.
It supports the latest version of QTI, Version 2.2.2.


## Preview

The build outputs of this project can be seen at:

<http://qtihub.org.s3-website-us-east-1.amazonaws.com>

This is an Amazon S3 bucket setup for static web hosting, containing
the sample QTI assessment tests built by this project.  Included in
the preview are almost all the QTI v2.2 examples distributed by IMS
Global Learning Consortium, along with some other sample tests. No
login is required, and no results will be reported anywhere.


## Build

The QTI build is done with a bash script. From the directory 
where you cloned the repository, enter the command.

```
tools/build.sh
```

This will generate:
* a *package* directory
* a zipped version of the *package* directory, 
  called *qtijs.package.zip*
* a *docroot* directory, a copy of the *test* directory
  with symlinks resolved, for uploading to a static
  web hosting service.

The Amazon S3 bucket mentioned in the previous section is just a copy
of the *docroot* generated by the build.

The *package* directory and the *qtijs.package.zip* archive file are
both already in the repository; so a build is necessary only if you
have changed the QTI.JS source code or want your own copy of *docroot*
for testing or some other purpose.


## QTI.JS Package Contents

The QTI.JS package consists of the following.
* the Javascript source files in *src*,
  bundled into a single script file, called *qti.js*.
* a Single Page Application (SPA) HTML file, called *index.html*.
* a subirectory, *theme*, containing the CSS stylesheets
  and image files for the default QTI.JS theme, *basic*.
* the QTI.JS *LICENSE*, the MIT license.
* a *VERSION* file, which gives QTI.JS version of the package
* this *README.md* file

The Single Page Application loads *qti.js*, which finds QTI
XML content by various means, and transforms it with its
referenced resources into HTML.  The generated HTML is appended to the
initially empty body of the SPA, and event handlers are set up to
manage user interactions, to do response and outcome processing, and
to submit candidate responses and outcomes.

The SPA also loads a [MathJax](http://mathjax.com) Javascript from a
Content Distribution Network, for MathML support. This is the only
external dependency of QTI.JS, and it can be removed from the SPA if
you do not need MathML support.

The name of the SPA is not required to be *index.html*, in case you
need some other file to have that name.  The SPA file is also
editable, but you should bear in mind that QTI.JS will be trying
to coexist with whatever edits you make.

If you do edit the SPA, perhaps to put a header or footer on the
page, or to load scripts to handle customInteractions, etc, 
you might need to edit the QTI.JS theme too. For example, the 
default *basic* QTI.JS theme assumes it can position fixed elements 
at the top of the browser viewport.


## Deployment

The following sections cover various scenarios for deploying QTI.JS.


### You have a QTI 2.2 Content Package with a single test

Many QTI-compliant authoring systems have the ability to export a QTI
"content package". This is just a zip archive containing a manifest
file, called *imsmanifest.xml*, along with the actual content,
consisting of XML and other files.

To deploy a QTI content package with QTI.JS, do the following:
* unzip the content package into an empty "deployment" directory.
* unzip *qtijs.package.zip* into the same directory, or copy the files
  from the *package* directory into the deployment directory.
* upload the deployment directory to a static web hosting service.

The URL for the test would then be something like
http://example.com/mytest/index.html, *mytest* being the name of the
deployment directory on the webhost. This would be omitted if the
deployment has been copied to the root of the web host.
Depending on the web host, the *index.html* part of the URL may also
be unnecessary. If you have changed the name of the QTI.JS Single Page
Application from the default of *index.html*, the URL would need to
reflect your different name.  If everything defaults right, the
minimal absolute URL for a QTI.JS test can just be the URL for a
domain, *http://example.com/*, which would make the QTI content the
top level content of the domain.

If you proceed this way, when a user invokes the URL of the SPA in a
browser, *qti.js* will load.  It will then load the manifest,
*imsmanifest.xml*, and play the first test it finds listed in the
manifest.  Currently, if this test does not exist, QTI.JS will not
continue by trying any subsequent tests in the manifest, but 
will simply display a blank screen.

The manifest-based approach is illustrated by the *tao* sample, but it
works only if there is just one test in the manifest or you can edit
the manifest to make the one test you need be first.


### You have QTI 2.2 files for a single test

Do the following:
* copy your QTI 2.2 content to an empty "deployment" directory.
* unzip *qtijs.package.zip*, or copy the files in the QTI.JS *package*
  directory into the deployment directory.
* let QTI.JS know which file is its "root" by renaming the root file
  to *index.xml*, or if you prefer your own name, and your web
  hosting service supports symbolic links, create a symbolic link
  called *index.xml* pointing to the root file.
* upload the deployment directory to a static web hosting service.

As with the previous deployment scenario, the URL for the test will
be something like: http://example.com/mytest/index.html , with *mytest*
and *index.html* possibly omitted.

This is how most of the sample tests are deployed for the preview.


### You have QTI 2.2 files for multiple tests

Do the following:
* copy your QTI 2.2 content to an empty "deployment" directory.
* unzip *qtijs.package.zip*, or copy the files in the QTI.JS *package*
  directory into the deployment directory.
* upload the deployment directory to a static web hosting service.

Since there is no manifest or *index.xml* to let QTI.JS know which
test to play, this must be specified using the *root* query string
parameter on the URL for the QTI.JS Single Page App. For
example, http://example.com/mytest/index.html?root=sometest.xml
    
This lets you run any of multiple QTI tests out of the same deployment
directory.  This approach comes at the cost of a more complex URL for
the SPA, which users probably won't be able to remember, and which
therefore can only be used in links.

The root parameter can point to a file containing any of the following:
* an *assessmentTest*
* an *assessmentSection*
* an *assessmentItem*


### Your QTI.JS package and QTI 2.2 content have different locations

The previous deployment scenarios assumed that the QTI.JS SPA
*index.html", *qti.js*, *theme* subdirectory, and the QTI 2.2
content are all deployed to the same directory on a web host. This
is not actually required and more complex cross-directory and
cross-domain deployments are possible.

The *root* query string parameter mentioned in the previous section
takes a URL as its value, either relative or absolute. This means that
the QTI.JS package and the root XML file of the QTI content
can be at different locations within the same HTTP/HTTPS domain
or even be within different HTTP/HTTPS domains.

With different domains, Cross-Origin Resource Sharing (CORS) comes
into the picture. However, CORS and security allowing, a deployment of
the QTI.JS package anywhere on the web can be used to run a QTI
*assessmentTest* which is anywhere on the web, containing QTI content
resources from anywhere on the web, even if the servers with those
resources are oblivious to QTI.JS.  For example, the following URL
will play an *assessmentTest* on example.com without any QTI.JS
presence at all on example.com:

  http://qtihub.org/package/index.html?root=http://example.com/test.xml

In this URL, the QTI.JS package is on *qtihub.org* and the root of the
QTI assessment content is on *example.com*. Possibly some of the
resources referenced via hrefs in the *assessmentTest* are on 
yet other servers. 

(Incidentally http://qtihub.org/package/index.html is a real URL and
refers to an Amazon S3 bucket. You are welcome to use it for testing,
at least for the beta/preview period. Take it easy on the hits, though.)

As powerful as cross-origin deployment may potentially be, the
need for CORS to be configured on the servers which hold the QTI
content perhaps makes this deployment scenario an advanced one.


### The QTI.JS package and/or the content are in the local file system

It is possible to have the QTI.JS package in the local file system,
accessed by the browser using the file: URL scheme. A QTI.JS SPA
loaded from the file system can access QTI content on the Web. For
example,

 *file://path-to-qtijs/package/index.html?root=http://example.com/test.xml*

This results in a cross-origin request, so CORS has to be enabled on
the servers with the QTI content (example.com, and possibly others, in
this case).

In addition, in some browsers, a QTI.JS SPA with a file: origin can
access QTI content in the local file system, also with a file: origin,
*PROVIDED* the browser is started in a special mode or the content is
in a special location. For example, the URI

 *file://path-to-qtijs/index.html?root=path-to-test/test.xml* 
 
starts QTI.JS from the local file *path-to-test/index.html* to run 
the test in the local file *path-to-test/test.xml*.

For this to work in Chrome on a desktop, the browser would need to
have been started with the *--allow-file-access-from-files*
argument. For it to work on Android in a WebView, both *path-to-qtijs*
and *path-to-test* would need to specify subdirectories of the
*android_asset* folder of the application containing the WebView. 
Other browsers and other platforms may not allow this at all or 
have different methods for enabling cross-file access.

Note that a QTI.JS package with a http: or https: origin can never
access QTI content in the local file system using a file: URL, for
obvious security reasons.

Currently the ability to run QTI.JS from the local file system is
really useful only for testing, if the platform is a desktop.  For
mobile devices, it opens up the intriguing possibility of an e-book or
hybrid mobile app in which both the QTI content and QTI.JS, as the
player, are installed on the local file system of the mobile system or
desktop.  If anyone is interested in exploring this, please contact
me.


### You wish to distribute a "self-playing" QTI.JS content package

It can be convenient to distribute a QTI content package with QTI.JS
already integrated, which in a sense will make the content package
"self-playing".

In the *tools* directory of the repository there is a bash script
which will repackage a QTI 2.2 content package so that it becomes
"self-playing".  This is done by adding the QTI.JS files to the package 
and its manifest as "webcontent".

Assuming the content package is called *content.zip*, and the new
package is called *new.zip* invoke the repackaging tool as follows:

```
tools/repackage.sh content.zip new.zip
```

A new self-playing QTI.JS *new.zip* content package will be generated.
Assuming the original content package was QTI compliant, the new one
will also be compliant, but will alos contain the files necessary for
QTI.JS to play the QTI content.

A self-playing QTI.JS content package can be deployed as follows:
* unzip the QTI.JS content package into an empty deployment directory
* upload the deployment directory, as is, to a static web hosting
  service.


## QTI.JS Themes

The instructions in the previous sections assume that you will be
using the default QTI.JS theme, which is called *basic*.  This is the
theme which the build copies to the *package/theme* directory.  But
one of the strengths of QTI.JS is that you can have your own theme.
Your tests need not look the same as everyone else's tests, and can
carry your institutional identity or commercial branding.

Only one of the sample tests uses the default theme; namely, *tao*.
Most of the other samples use the *qtijs* theme, which incorporates
*basic* but changes the colors and adds a logo. There was no technical
reason why the *qtijs* theme couldn't have also been used with the
*tao* sample; it just seemed a bit cheeky to plaster the QTI.JS logo
on TAO's content.

The *ccQTIpackage", *material*, *newsquiz*¸ and *parcc* sample tests
all have their own custom themes. Like *qtijs*, the *parcc* custom
theme is an extension of *basic*, but the others are standalone
themes, limited to styling only interactions and features found in the
specific tests.  All of these are in the *theme* subdirectories of the
respective samples.

In addition to the theme dealing with aesthetic issues such as fonts,
colors, backgrounds, and layout, the delivery engine delegates to the
theme such issues as getting items on- and off-screen in a linear
test, or controlling the visibility of elements subject to the QTI
showHide mechanism.  The *minimal* theme is one which, as the name
suggests, provides only this minimal CSS support to the delivery
engine, generally leaving all aesthetic concerns to the browser
defaults. The *minimal* theme is provided mainly as documentation, or
as the starting point for custom themes, and is not used by any of the
sample tests.

In the future, I hope to provide more documentation on how to create a
custom QTI.JS theme, and possibly will provide tools to enable someone
who isn't an HTML/CSS developer to create a new theme by "tweaking"
*basic* or *minimal* to change colors, fonts, and backgrounds, or 
add a logo.

Meanwhile, a person with some knowledge of CSS can probably figure out
how, without too much effort, to tweak the *minimal*, *basic* or
*qtijs* themes to make cosmetic changes, such as changing the logo or
using different fonts and colors. Creating a limited theme, such as
the themes for the *ccQTIpackage* or *newsquiz* samples is also
feasible.

If you do have a different theme you wish to use, you can deploy it
using any of the scenarios described above.  Just copy the theme to
the deployment directory as a subdirectory called *theme*, in place
of the *theme* directory created by the build.


## Results Reporting

QTI.JS supports QTI Results Reporting.  Upon submission of an item or
items by the candidate, QTI.JS will POST an XML message conforming to
the QTI Results Reporting schema to a HTTP/HTTPS-based "Results
Server", if enabled.  A Results Server is enabled by defining the URL
of the server endpoint in the Javascript variable, QTI.RESULTS_ENDPOINT.
Of course, there must actually be a server at this endpoint.  CORS
will need to be enabled on this server, unless it is also the origin
of the QTI.JS files.

The Javascript variable QTI.RESULTS_HEADERS can be used to
specify the request headers on the messages sent to the results
endpoint.  This can be used, for example, to send an Authorization
header to the Results Server.

One way to define these variables is in the *index.html* SPA. For
example,

```
<script>
 QTI.RESULTS_ENDPOINT = "https://example.com/results";
 QTI.RESULTS_HEADERS = {
   Authorization: "Bearer aaaaaaaa.bbbbbbbb.ccccccccc"
 };
</script>
```

Since it is a QTI delivery engine, results submission is within the
scope of QTI.JS, but a server providing results storage and reporting
is not.  Nevertheless, for testing, there is PHP code for a simple
results logging server in the *test/results_server* directory of 
the repository.


## QTI 2.2  Authoring Systems, Extensions, and Interoperability

The QTI 2.2 specification affords many opportunities for extension and
customization, both through "extension points" provided in the
specification, and by more "creative" methods.  Some QTI developers
have used these extension possibilities, both official and creative,
rather freely.

QTI.JS ignores QTI extension points and generally passes them through
verbatim to the HTML it generates, or gives them the default handling
or styling. It does provide several different ways to hook in
additional CSS stylesheets or Javascript functions to style or handle
extension points in the QTI content, like *data-*, *aria-*, and
*class* attributes, or *customInteraction*, *customOperator*, and
custom *selection* and *ordering* elements, etc.  But if you cannot
supply stylesheets and/or Javascript code to handle the extensions and
customizations in your QTI content, QTI.JS will not be able to handle
them on its own.

So, if the extensions are not merely cosmetic but essential to the
content, what this means is that your QTI 2.2 content, though
syntactically valid and nominally interoperable and portable, is in
reality locked to the delivery system or systems which understand
the extensions, and items or entire tests will not play properly or
look right in QTI.JS, or indeed any other "generic" QTI delivery
engine.

That said, as you can see from the samples, I have had much success in
playing QTI content from various systems in QTI.JS.  Every example
distributed by IMS, except for one item intended to demonstrate data-
attributes, out of some 180 assessment, section, and item files, plays
in QTI.JS without problem.

For the one sample test exported from TAO, I had to make QTI.JS more
flexible in a few minor areas before the TAO test fully worked.
This was because the folks at TAO interpreted the spec somewhat
differently than the IMS examples did, or at any rate, differently
than I did. It is not unusual for that sort of thing to happen
occasionally, and fortunately it was no more than a little bump in the
road.

The good news is that although in places it gives developers a bit too
much latitude at the cost of interoperability, QTI 2.2 is a very good
specification, and it seems generally possible to achieve
interoperability between QTI implementations, if not always right off
the bat, at least without too much effort most of the time.


## Please Report Issues

During the beta, I expect to be making changes to QTI.JS to provide
the few remaining unimplemented features, to fix the inevitable bugs,
and to handle any issues with content from other systems which might
arise from other developers having a different read of the spec than
me.  I don't discount the possibility that there will be a few cases
where other developers have bugs in their systems, or have partied a
little too hard with the QTI customization possibilities, etc, and I
won't be able to make QTI.JS play some content the same as they do
without turning QTI.JS upside down.

This is one area where I really need everybody's help.  If you have a
QTI content package or item exported from another system which doesn't
play acceptably in QTI.JS, and you would be so kind as to send it to
me, it is sure to be helpful. Just post an "Issue" here on GitHub.  If
it is a problem with QTI.JS, I will try to fix it.  If I cannot, at
least I will try to let you know what features of your authoring
system seem to be presenting interoperability issues and help you find
a workaround.


## Plans

I am aiming for a production release of QTI.JS and certification by
IMS in Q1 2022.  The production release will be a 100% implementation
of QTI 2.2.2 (this preview/beta is already close to that) and will
support recent versions of Chrome, Firefox, Safari, and Edge, on
desktops and (where applicable) mobile devices.

I hope to deliver a QTI v3.0 version of QTI.JS as soon after the
IMS release of v3.0 as possible.



Nabeel Tad Merchant<br/>
Developer
