---
title: org-mode for Static Website Generation
---

# Some Context

This post no longer has a proper context; I am no longer using the
following approach to generate my website. That said, I think it still
contains some content that might be useful for someone else trying to
use org-mode for static website generation. 

# Summary

I was unhappy with my previous website because the overhead of
presenting content was high. It is my opinion that presenting content
on a website should require as little non-essential work as possible;
generating content (e.g., writing) is hard enough.

# The Previous Site
The previous site used [pagegen](http://pagegen.phnd.net/). Pagegen
is a static web page site generator that relies on a site template
file and a directory structure. Content for the site is stored in text
markup files. The template can contain variables referring to
components of the markup file. Explained at this level of detail,
pagegen should be sufficient for my needs.

Realizing pagegen was not the right system for my needs came from
using the markup language. Specifically, the
[textile](http://en.wikipedia.org/wiki/Textile_%28markup_language%29)
variant that pagegen uses is something I have never seen before or
since. Therefore, becoming familiar with the textile markup language
to present content on my website would not be something I would likely
find useful elsewhere.

To avoid sinking time into a technology with limited context, I
generate my content in HTML then escaped all of it in textile.  This
increased the overhead of generating and presenting knowledge;
creating simply structured content became a mess of XML. Naturally, I
just avoided it.

# Onward to [org-mode](http://orgmode.org/)

What is essential when presenting content for my site? The answer I
arrived at is some simple way to structure content and some way to
export this structure to HTML.

Org-mode has a configurable export system that will export org
documents to other formats. There is some process for implementing
conversions to target formats however, exporting to LaTeX and HTML has
already been implemented.

After using emacs org-mode for a couple months, I realized that most
of the structure I want when presenting content can be given using the
most basic org-mode markup. Structures like sections, tables, lists,
and typical text styles can all be described
intuitively. Additionally, time spent understanding the markup and
system in-depth would not be wasted; I have already committed to
integrating org-mode into my daily habits.

Assuming emacs and the org-mode module are installed, to get
org-to-HTML functionality to work add the appropriate configuration.
I will develop this org-to-HTML export configuration in a peicewise
fashion. This configuration should reside in a place such that emacs
will execute it upon configuration.

First we must require org-mode's publishing functionality.

~~~lisp
(require 'ox-publish)
~~~

Editing my site content is a three phase process consisting of
writing, editing, and publishing. During the writing and editing
phases, I want to publish to my harddrive and review my work via a
local path like `file:///path/to/local/deployment/site/` in a
browser. When I am satisfied with the content I publish it. I also
want to structure content into different directories and refer to the
content by relative paths. For example, I wanted all files to refer to
a stylesheet in the `css/` directory. I accomplished this by setting
the `base` element in the head of all generated HTML files according
to whether I want to view it locally or if I wanted to publish. How
these variables are used to construct the head will follow shortly.
Unfortunately, the solution below for toggling between the firt two
phases and the last requires I change `base-ref` and execute my
configuration via `(load-file "file")` (If you have a suggestion on
how to fix this please leave a comment!).

~~~lisp
(setf local-base-ref  "file:///home/dacosta/gitrepositories/documents/org/web/chaosape.com/build/")
(setf chaosape-base-ref  "http://www.chaosape.com/")
(setq base-ref local-base-ref)
~~~

From what I have gather, org-mode HTML exporting has two builtin
publishing functions: `org-html-publish-to-html` and
`org-publish-attachment`. The former converts org files to HTML then
places the HTML in some destination directory and the latter copies
the file to the destination directory. I have three pairs of variables
defining content source and destination for org, css, and image
files.

~~~emacs-lisp
(setf chaosape-repo-location "/home/dacosta/gitrepositories/documents/org/web/chaosape.com/")
(setf chaosape-css-location "css/")
(setf chaosape-img-location "img/")
(setf chaosape-src-location (format "%s%s" chaosape-repo-location "src/"))
(setf chaosape-src-css-location (concat chaosape-src-location chaosape-css-location))
(setf chaosape-src-img-location (concat chaosape-src-location chaosape-img-location))
(setf chaosape-dst-location (format "%s%s" chaosape-repo-location "build/"))
(setf chaosape-dst-css-location (format "%s%s" chaosape-dst-location chaosape-css-location))
(setf chaosape-dst-img-location (concat chaosape-dst-location chaosape-img-location ))
~~~

Org-modes HTML publishing functionality provides a configuration
option to set HTML header files uniformly.  Here I define a variable
containing the content I would like place in the head of all generated
HTML files. The `base href` must be set in the head so here we build
the final head content using a format string and `base-ref`.

~~~emacs-lisp
(setf chaosape-header
      (format 
       "
<base href=\"%s\">
<script
    type=\"text/javascript\"
    src=\"http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML\">
</script>
<link rel=\"stylesheet\" href=\"css/chaosape.css\">
"
       base-ref
      )
)
~~~

I also wanted a footer.

~~~emacs-lisp
(setf chaosape-footer "
<a style=\"border-bottom: none;\" href=\"http://www.twitter.com/chaosape/\">
  <img src=\"img/twitter.ico\" alt=\"chaosape on Twitter\"/>
</a>
<a style=\"border-bottom: none;\" href=\"http://www.bitbucket.com/chaosape/\">
  <img src=\"img/bitbucket.png\" alt=\"chaosape on bitbucket\">
</a>
<a style=\"border-bottom: none;\" href=\"http://www.github.com/chaosape/\">
  <img src=\"img/github.ico\" alt=\"chaosape on github\">
</a>
<a style=\"border-bottom: none;\" href=\"http://about.me/chaosape/\">
  <img style=\"height: 32px; width:32\" src=\"img/aboutme.ico\" alt=\"chaosape on about me\">
</a>
")
~~~ 

There are three components associated with the chaosape.com
project. The component chaosape.com-top recursively converts all org
files in the source directory `chaosape-src-location` and places them
in a corresponding directory with a root at
`chaosape-dst-location`. The components chaosape.com-css and
chaosape.com-img copy content from the directories
`chaosape-src-css-location` and `chaosape-src-img-location` to
`chaosape-dst-css-location` and `chaosape-dst-img-location`,
respectively. Once this configuration is in place calling
`org-publish-all` should generate content rooted at
`chaosape-dst-location`. You may find additional information about
this configuration [here](http://orgmode.org/manual/Configuration.html).

~~~emacs-lisp
(setf org-publish-project-alist
      `(("chaosape.com" :components
         ("chaosape.com-top" "chaosape.com-css" "chaosape.com-img"))
        ("chaosape.com-top"
         :base-directory ,chaosape-src-location
         :publishing-directory ,chaosape-dst-location
         :base-extension "org"
         :recursive t
         :html-head ,chaosape-header
         :publishing-function org-html-publish-to-html
         :with-author t
         :with-creator nil
         :with-latex t
         :export-with-tags t
         :export-creator-info nil
         :export-author-info nil
         :auto-postamble nil
         :table-of-contents nil
         :section-numbers nil
         :style-include-default nil
         :section-numbers nil
         :with-toc nil
         :html-postamble ,chaosape-footer
         :html-head-include-default-style nil
         :html-head-include-scripts nil
         )
        ("chaosape.com-css"
         :base-directory ,chaosape-src-css-location
         :publishing-directory ,chaosape-dst-css-location
         :base-extension "css"
         :publishing-function org-publish-attachment
         )
        ("chaosape.com-img"
         :base-directory ,chaosape-src-img-location
         :publishing-directory ,chaosape-dst-img-location
         :base-extension "jpg\\|gif\\|png\\|svg"
         :publishing-function org-publish-attachment
         )
       )
    )
~~~

I wanted my index page to contain a listing of all of my blogs ordered
by published date. The following function generates such a list based
on org files in a directory named `blog` (Much of this code was lifted
from
[steckerhalter](https://github.com/steckerhalter/org-mode-blog/blob/master/elisp/index.el)
then modified). We should be able to inline the output of this
function by
[calling](http://orgmode.org/manual/Working-With-Source-Code.html)
`generate-bloglist` in `index.org`. I was unable to get this to work
correctly. However, I was able to get the dynamic block generation
approach described
[here](http://kdr2.com/tech/emacs/orgmode-export-process.html) to
work.

~~~emacs-lisp
(defun org-dblock-write:generate-bloglist (params)
(let* ((dir "blog")
       (files (directory-files dir t "^[^\\._][^#].*\\.org$" t))
       entries)
  (dolist (file files)
    (catch 'stop
      (let* ((path (concat dir "/" (file-name-nondirectory file)))
             (env (org-combine-plists 
                   (org-babel-with-temp-filebuffer file 
                     (org-export-get-environment))))
             (date (or (apply 'encode-time    (org-parse-time-string
                               (or (car (plist-get env :date)) 
                                   (throw 'stop nil))))))
             (last-modified-date (nth 5 (file-attributes file))))
        (plist-put env :file  (file-name-nondirectory file))
        (plist-put env :path path)
        (plist-put env :parsed-date date)
        (plist-put env :last-modified-date last-modified-date)
        (push env entries))))
  (dolist (entry (sort entries 
                       (lambda (a b) 
                         (time-less-p 
                          (plist-get b :parsed-date) 
                          (plist-get a :parsed-date)))))
    (insert
    (format "* [[file:blog/%s][%s]]\n- %s\n- Last Modified on %s\n- Created on %s\n"
            (plist-get entry :file)
            (car (plist-get entry :title))
            (plist-get entry :description)
            (format-time-string "%B %e %Y" (plist-get entry :last-modified-date))
            (format-time-string "%B %e %Y" (plist-get entry :parsed-date)))))))

(add-hook 'org-export-before-processing-hook (lambda (backend) (org-update-all-dblocks)))
~~~

I found that the above code left some spurious text on the page.  I am
not exactly sure why and I am not familiar enough with org-mode to
know how to remove it; I removed this text by writing a filter that
modified the final output org-mode was to publish.

~~~emacs-lisp
(defun org-html-remove-spurious-dynamic-block-code (string backend info)
  "Remove #+BEGIN:  generate-bloglist."
       (when (and (org-export-derived-backend-p backend 'html)
                  (string-match ".\+BEGIN:  generate-bloglist" string))
         (replace-regexp-in-string ".\+BEGIN:  generate-bloglist" "" string)))

(add-to-list 'org-export-filter-final-output-functions
             'org-html-remove-spurious-dynamic-block-code)
~~~


I need the navigation menu to be placed uniformly in the
content div element (all exported org structured text ends up within
this div). I was able to get the menu placed correctly by writing
another filter.

~~~emacs-lisp
(defun org-html-add-navigation-after-content-div (string backend info)
  "Put the navigation menu inside the content div."
       (when (and (org-export-derived-backend-p backend 'html)
                  (string-match "div id=\"content\"" string))
         (replace-regexp-in-string "<div id=\"content\">" 
(format 
"<input type=\"checkbox\" id=\"nav-trigger\" class=\"nav-trigger\" />
<label for=\"nav-trigger\"></label>
<ul class=\"nav\">

        <li class=\"nav-element\"><a href=\"index.html\">home</a></li>
        <li class=\"nav-element\"><a href=\"cv.html\">cv</a></li>
</ul>
<div class=\"content\">" 
)string)))

(add-to-list 'org-export-filter-final-output-functions
             'org-html-add-navigation-after-content-div)
~~~

# Conclusion

I think this site setup will facilitate more frequent content
creation. This is something I need to make a habit of if I intend to
improve my writing skills. One gripe I have about this setup is
its dependency on this dynamic block magic that requires me to write a
filter to remove some spurious text.

# Comments

<div id="disqus_thread"></div>
<script type="text/javascript">
    /* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
    var disqus_shortname = 'chaosape'; // Required - Replace example with your forum shortname

    /* * * DON'T EDIT BELOW THIS LINE * * */
    (function() {
        var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
        dsq.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
    })();
</script>
