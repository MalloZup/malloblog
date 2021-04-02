+++ 
date = 2021-03-29T10:49:37+02:00
title = "Some fondamentals for web application and microservices in golang"
description = ""
slug = "" 
tags = []
categories = ["web", "go"]
externalLink = ""
series = []
+++


# Intro:

I will write in this post about some useful concept you need for building web application in golang.
Also I will compare such approaches to other languages and upstream approaches from my developer carreer into opensource world.

# 1) Web toolkit are the golang minimalist answer against complexity of frameworks

Choosing a framework is a safe choice. People has already solved the problem for you. Really ?? It depends.
Adopting a framework can be quite considerable risk. It offers you an huge amount of feature thaty perhaps you will even don't need.
This is why also application are built as microservice patterns, instead of monolith frameworks.

Within a framework you embark in a mental pattern, bugs and coding style that you might don't know if it fits for you. Upgrade/update becomes problem.

As pro for frameworks, it is true that one don't want  to reinvent the wheel each time. 
But as contra, you inherit a huge complexity and bugs for free.

For these reasons, in golang minimalism, has emerged the concept of "web toolkit".

Gorilla is one developed by google. https://www.gorillatoolkit.org/

Such approach is used in other contexts like "microservices", example: https://gokit.io/

A web-toolkit is a collection of tools/library you can plug to http, incrementally, without embarking on huge dependencies risk.

If you neeed a http router, my personal suggestion is to use `gorilla/mux`webtoolkit or even `chi` as router.

https://www.gorillatoolkit.org/ or https://github.com/go-chi/chi

Both are safe choices because they aren't a framework and they use the standard go handlers HTTP signature, so in case want to revert your choices, you can still swap them without much hassle.

# 2) Templating and base templates in golang:


Every modern web-application uses templating.
In short, templates allow you to combine the HTML frontend techs with the backend data. Templates are reusable part html etc, that you can program.
 In rails you have a variety of choices, like erb, and others.

In go you can use the standard `html` templating.

There is a useful article from upstream which explain the basics: https://golang.org/doc/articles/wiki/

I want to complement the article, by explaining how to make a `base` template inherit each others.

Basically you want to build a `base` template, from which each custom  "child" template inherit. 
The base template contains the basic layout, and the child one will overwrite it.

The working code, you can find it here: https://github.com/MalloZup/console-for-sap-applications/tree/chi-lib/web

Important, while I discuss the components here, check also the templates directory: https://github.com/MalloZup/console-for-sap-applications/tree/chi-lib/web/templates


We can decompose the example in 3 components:

0) The router part 
1) the template manager
2) The handler which the HTTP calls


### 0 Router

This code does following:

a) it uses the new functionality in go 1.16 to embed static files into binary. ( see embed.FS).
  So basically, you can embed the files and filesystem at compile time the binary so that you don't  have to package it or do other tipes of logic when serving file ( very useful)

b) It init the http router, chi in our case, and register the `IndexHandler`, which execute the templates, see later.

c) It serves static files we embeded

```
package web

import (
        "embed"
        "net/http"
        "strings"

        "github.com/go-chi/chi"
)

//go:embed templates
var templatesFS embed.FS

//go:embed frontend/assets
var assetsFS embed.FS

func InitRouter() chi.Router {
        r := chi.NewRouter()
        // parse all templates and return a map, which is consumed by each handler

        templs := AddAllTemplatesFromFS(templatesFS, "templates/*.tmpl")
        r.Get("/", IndexHandler(templs))

        FileServer(r, "/static", http.FS(assetsFS))
        return r
}

func FileServer(r chi.Router, path string, root http.FileSystem) {
        if strings.ContainsAny(path, "{}*") {
                panic("FileServer does not permit any URL parameters.")
        }

        if path != "/" && path[len(path)-1] != '/' {
                r.Get(path, http.RedirectHandler(path+"/", 301).ServeHTTP)
                path += "/"
        }
        path += "*"

        r.Get(path, func(w http.ResponseWriter, r *http.Request) {
                rctx := chi.RouteContext(r.Context())
                pathPrefix := strings.TrimSuffix(rctx.RoutePattern(), "/*")
                fs := http.StripPrefix(pathPrefix, http.FileServer(root))
                fs.ServeHTTP(w, r)
        })
}
```



###  1) Template Manager:

The template manager basically return a map of templates which can used by the various handlers.


```
package web

import (
	"bytes"
	"fmt"
	"html/template"
	"io/fs"
	"path/filepath"
)

const (
	base       = "templates/base.html.tmpl"
	blocksHtml = "templates/blocks/*.html.tmpl"
)

// the main motivation of this code is to parse all templates contained in templates directory,
// respecting the "base" template which is the basis, where other custom templates will inherit and override content
// we also extend the template and build the base template with blocks templated (contained in /templates/blocks directory)

// AddAllTemplateFromFs return a map of parsed templated from filesystem
// the map of parsed html templates  is consumed by http handler, accessed via key
func AddAllTemplatesFromFS(templatesFS fs.FS, templates ...string) map[string]template.Template {
	templs := make(map[string]template.Template)

	for _, pattern := range templates {
		files, err := fs.Glob(templatesFS, pattern)
		if err != nil {
			// we exceptionally hard panic in case of glob errors, these should never happen.
			panic(err)
		}
		for _, file := range files {
			if file == base {
				continue
			}
			addFileFromFS(templatesFS, file, templs)
		}
	}
	return templs
}

// addFileFromFS parses the base template with the user
func addFileFromFS(templatesFS fs.FS, file string, templs map[string]template.Template) {
	var tmpl *template.Template
	// use the base template first
	name := filepath.Base(file)
	tmpl = template.New(filepath.Base(base))

	// we "extend" the templates by adding custom functions
	tmpl = tmpl.Funcs(template.FuncMap{
		"escapedTemplate": func(name string, data interface{}) string {
			var out bytes.Buffer
			_ = tmpl.ExecuteTemplate(&out, name, data)
			return out.String()
		},
	})
	// parse all templates
	patterns := append([]string{base, file}, []string{blocksHtml}...)
	tmpl = template.Must(tmpl.ParseFS(templatesFS, patterns...))

	// add template to template map, consumed by handlers
	if tmpl == nil {
		panic("template can not be nil")
	}
	if len(name) == 0 {
		panic("template name cannot be empty")
	}
	if _, ok := templs[name]; ok {
		panic(fmt.Sprintf("template %s already exists", name))
	}
	templs[name] = *tmpl
}
```






2) The handler called:

the handler in this example does 2 things:

a) it takes the `home` templates from

```
package web

import (
	"fmt"
	"html/template"
	"net/http"
)

// Index data is used for the home template
type Index struct {
	Title     string
	Copyright string
}

func IndexHandler(templates map[string]template.Template) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		data := Index{
			Title:     "SUSE Linux",
			Copyright: "Your copyrights",
		}

		tmpl, ok := templates["home.html.tmpl"]

		if !ok {
			http.Error(w, fmt.Sprintf("The template HOME does not exist"),
				http.StatusInternalServerError)
			return
		}

		err := tmpl.ExecuteTemplate(w, "base", data)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
			return
		}

	}
}
```


