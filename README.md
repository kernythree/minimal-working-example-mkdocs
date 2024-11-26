# Minimal Working Example (Mkdocs build)

> This documentation is a minimal working example of a static documentation website to be build with Mkdocs and deployed on GitHub / GitLab page or on private server.

The static website demo is available at : 

- [https://sb2-team.gitlab-pages.ifremer.fr/minimal-working-example-mkdocs/]() (if hosted on the Ifremer's GitLab)
- [link]() (If hosted on the Ifremer/CNRS GitHub organization)

## Overview

The documentation pages are written in Markdown format.

All pages are strored in folder `docs/` and written in Markdwon format.

All related figures are stored in folder `docs/assets/` and saved as PNG.

The documentation is built as a static HTML web site with Mkdocs (https://www.mkdocs.org/getting-started/)

The rendering is configured with a config file at the root folder named `mkdocs.yml`.

## Installation 

Installing mkdocs :

```
python -m pip install mkdocs
```

Installing mkdocs third-party themes

```
python -m pip install mkdocs-material mkdocs-nature mkdocs-github mkdocs-gitbook mkdocs-custommill mkdocs-windmill mkdocs-theme-bootstrap4 mkdocs-ivory mkdocs-swan mkdocs-alabaster
```

Best theme for scientific documentation are `material`, 
 `readthedocs`, `swan`, or `alabaster`

## Modify the page menu

To change the way pages are shown in the site web, modify the `nav` section of the configuration file `mkdocs.yml` (see [Mkdocs documentation](https://www.mkdocs.org/user-guide/configuration/#documentation-layout))

## Rendering LaTex equations

To render LaTex equation embeded in your markdown document with `$ ... $` (inline equation) or with `$$ ... $$` (equation blocks) follow instruction on the [Mkdocs documentation](https://squidfunk.github.io/mkdocs-material/reference/math/#mathjax-mkdocsyml)  

Step 1 : Add this to the .yml configuration file

```
markdown_extensions:
  - pymdownx.arithmatex:
      generic: true
extra_javascript:
  - javascripts/mathjax.js
  - https://unpkg.com/mathjax@3/es5/tex-mml-chtml.js
```

Step 2 : And Create a new file `mathjax.js` in folder `docs/javascripts/` with the following content

```
window.MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']],
      displayMath: [['$$', '$$'], ['\\[', '\\]']],
      processEscapes: true,
      processEnvironments: true
    },
    options: {
      ignoreHtmlClass: ".*|",
      processHtmlClass: "arithmatex"
    }
  };
  
  document$.subscribe(() => { 
    MathJax.startup.output.clearCache()
    MathJax.typesetClear()
    MathJax.texReset()
    MathJax.typesetPromise()
  })
```


## Live HTML pages

```
python -m mkdocs serve
```

Open the this link in browser : http://127.0.0.1:8000/


## Build site before deployment

```
python -m mkdocs build
```

The site ready-to-deploy is created in the `site/` folder.

Add this to to .yml file to generate local HTML documentation with proper links.

```
site_url: ""
use_directory_urls: false
plugins: []
```

## Deploying site on GitLab page

Create a `.gitlab-ci.yml` file in the root of your repository to define the GitLab CI/CD pipeline for building and deploying GitLab Pages:

```
image: python:3.9

cache:
  paths:
    - .cache/pip/

stages:
  - build
  - deploy

before_script:
  - pip install mkdocs mkdocs-material

build:
  stage: build
  script:
    - mkdocs build
  artifacts:
    paths:
      - public

pages:
  stage: deploy
  script:
    - mv site public
  artifacts:
    paths:
      - public
    expire_in: 1 week

```

Push your changes to GitLab:

```
git add .gitlab-ci.yml
git commit -m "Add CI/CD for GitLab Pages"
git push
```

Go to your repository in GitLab and navigate to CI/CD > Pipelines to ensure the pipeline runs successfully.

Open the GitLab page.

For instance for the Ifremer's Gitlab the page is at 
```
https://<username>.gitlab-pages.ifremer.fr/<projectname>
```

## Deploying site on GitHub page





## Deploying site on your server

*You must have built the site and pushed the content of `site/` folder on the Git repository hosting the source of your documentation before deployment.*

Documentation created with Mkdocs can easily be hosted out-of-the box on GitHub Pages or Read The Docs site web. Consult the page https://www.mkdocs.org/user-guide/deploying-your-docs/ for more info.

If you want to deploy it on your own server, the best way is to use Nginx (https://nginx.org/en/) to serve it.

First, install it with 
```
sudo apt-get install nginx
```

Remove the default files to avoid conflict on port 80
```
sudo rm -rf /etc/nginx/sites-available/default
sudo rm -rf /etc/nginx/sites-enabled/default
```

Clone this git repository on the server
```
git clone https://gitlab.ifremer.fr/sb2-team/lora-sensors-in-marine-scenario-mkdocs
```

Create a new Nginx site configuration file:
```
sudo nano /etc/nginx/sites-available/lora-sensors-in-marine-scenario
```

Add the following configuration
```
server {
    listen 9000; # Use a free port and not 80
    server_name ifremer.re; #  set the Name or IP address of the server

    root /home/debian/GITLAB/lora-sensors-in-marine-scenario-mkdocs/site; # set the correct path to "site/" folder
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    access_log /var/log/nginx/mycustomname-access.log;
    error_log /var/log/nginx/mycustomname-error.log;
}

```

Enable the configuration by creating a symbolic link to the sites-enabled directory
```
sudo ln -s /etc/nginx/sites-available/lora-sensors-in-marine-scenario /etc/nginx/sites-enabled/
```

Test Nginx configuration to ensure there are no syntax errors
```
sudo nginx -t
```

Restart Nginx to apply the changes
```
sudo systemctl restart nginx
```

## Export to PDF

**Option 1 : Standalone HTML page to print**

Install the [mkdocs-print-site-plugin](https://github.com/timvink/mkdocs-print-site-plugin)

```
python -m pip install mkdocs-print-site-plugin
```

Add this the `mkdocs.yml` configuration file

```yml
plugins:
  - print-site:
      add_to_navigation: false
      print_page_title: 'Print Site'
      add_print_site_banner: false
      # Table of contents
      add_table_of_contents: true
      toc_title: 'Table of Contents'
      toc_depth: 1
      # Content-related
      add_full_urls: false
      enumerate_headings: false
      enumerate_figures: true
      add_cover_page: true
      cover_page_template: ""
      path_to_pdf: ""
      include_css: true
      enabled: true
      exclude:
```

Go to folder `site/print_page` and open file `index.html` in yur browser (Firefox not recommend) and then print it to PDF file.

**Option 2 : PDF with chromium rendering for deep cutomization**

Install the [mkdocs-with-pdf](https://github.com/orzih/mkdocs-with-pdf)
```
python -m pip install first beautifulsoup4==4.9.3
python -m pip install mkdocs-with-pdf
```

Add this the `mkdocs.yml` configuration file

```yml
plugins:
  - with-pdf:
      render_js: true  # add this for math equation
      headless_chrome_path: C:/Users/mjulien/AppData/Local/Chromium/Application/chrome.exe
      output_path: ./document.pdf
      enabled_if_env: ENABLE_PDF_EXPORT 
```

To enable (=1) or disable (=0) the PDF generation use this 

```
set ENABLE_PDF_EXPORT=1
python -m mkdocs build
```

You need to install Chromium for this page [https://chromium.woolyss.com/download/fr/](https://chromium.woolyss.com/download/fr/) and modify the field `headless_chrome_path` with the installation path.
