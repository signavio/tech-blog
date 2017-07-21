---
title: "Legacy redirects in Sphinx"
description: "How to handle redirects directly in your Sphinx project"
author: Timotheus Kampik
tags: Sphinx, documentation, redirects
layout: article
image:
  feature: /2017/street-sign.jpg
  alt: "Street sign"
---

<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px;" href="http://unsplash.com/@dmpop?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Dmitri  Popov"><span style="display:inline-block;padding:2px 3px;"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white;" viewBox="0 0 32 32"><title></title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px;">Dmitri  Popov</span></a>

When you use [Sphinx](http://www.sphinx-doc.org/en/stable/) to document your software, you sometimes move files to a different location as part of a content refactoring.
Then, you need to set up redirects that point from the old to the new location of a content piece.

If you make your documentation available on one particular domain, the obvious way to handle redirects is to ask the system administrator to configure the redirects on web server side, for example in Apache [.htaccess](https://httpd.apache.org/docs/current/howto/htaccess.html) files.
But in case you release the docs on multiple platforms or even have them deployed locally within a customer's internal network, you can reduce the organizational overhead by managing the redirects yourself, directly within your Sphinx project.
This blog post explains how.

## Redirect template
First you need to create an HTML template for redirect handling.
It triggers the redirect via an HTML meta tag and also via JavaScript (to support some possible edge cases).
You find alternative approaches on [Stack Overflow](https://stackoverflow.com/questions/5411538/redirect-from-an-html-page).
In this example, `<./new_url>` is the placeholder for the actual target URL that points to the new location of the file:

```html
<html>
  <head>
    <meta http-equiv="refresh" content="1; url=<./new_url>:" />
    <script>
      window.location.href = "<./new_url>"
    </script>
  </head>
</html>
```

Every time you make a refactoring that moves an entire file, place the HTML file with the template snippet to the folder you moved the source file from and adjust the redirect target ( `<./new_url>`).
Of course, the HTML file needs to have the same name as the compiled moved file.
For example, if your source file was `folders.rst`, name the output file `folders.html`.

## Setup script in project configuration file
By default, Sphinx won't process the redirect files (it only considers source files).
That's why you need to write a couple of lines of Python code to copy the redirect files to their correct place in the build directory.
You can simply add the lines to your project's `conf.py` configuration file.

Create a list that contains all redirect files:

```python
redirect_files = ['explorer/basics/deleting_and_restoring.html']
```

Alternatively, you could write a script that collects all paths to HTML files in your source directory.

Now, define a function that copies all redirect files from the source directory to the equivalent position in the output directory.
For this, you can conveniently make use of Sphinx's [application API](http://www.sphinx-doc.org/en/stable/extdev/appapi.html):

```python
from shutil import copyfile
# copy legacy redirects
def copy_legacy_redirects(app, docname): # Sphinx expects two arguments
    if app.builder.name == 'html':
        for html_src_path in redirect_files:
            target_path = app.outdir + '/' + html_src_path
            src_path = app.srcdir + '/' + html_src_path
        if os.path.isfile(src_path):
            copyfile(src_path, target_path)
 ```

Finally, register the function on the `build-finished` [Sphinx core event](http://www.sphinx-doc.org/en/stable/extdev/appapi.html#sphinx-core-events) to execute the function when the Sphinx build finishes:

```python
def setup(app):
    app.connect('build-finished', copy_legacy_redirects)
```

That's all!
After you run your Sphinx build command, the tool will include the HTML redirect files at the desired location in the build folder.
