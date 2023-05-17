---
title: Sidebar in Quartz

tags:
- code
- quartz
- obsidian

menu:
    main:
        params:
        parent: Notes
---

## Background
> [Quartz](https://github.com/jackyzha0/quartz) is a project based on [Hugo](https://gohugo.io/) (static site generator) to make deploying [Obsidian](https://obsidian.md/) (markdown note app) files easy.

Quartz helps enable publishing of notes to the web, and is an alternative to Obsidian's Publish offering.  One thing it lacks is a sidebar menu listing all content. Here I describe the process for reproducing this functionality in a minimal way.

## Demo
![[images/sidebar.mp4|sidebar]]

## Create `sidebar.html` partial template
Create the file: `layouts/partials/sidebar.html`

```html
{{- $page := .page }}
{{- $menuID := .menuID }}

{{- with index site.Menus $menuID }}
  <nav>
    <ul style="padding-left: 2px; list-style-type: disc;">
      {{- partial "inline/menu/walk.html" (dict "page" $page "menuEntries" .) }}
    </ul>
  </nav>
{{- end }}

{{- define "partials/inline/menu/walk.html" }}
  {{- $page := .page }}
  {{- range .menuEntries }}
    {{- $attrs := dict "href" .URL }}
    {{- if $page.IsMenuCurrent .Menu . }}
      {{- $attrs = merge $attrs (dict "class" "active" "aria-current" "page") }}
    {{- else if $page.HasMenuCurrent .Menu .}}
      {{- $attrs = merge $attrs (dict "class" "ancestor" "aria-current" "true") }}
    {{- end }}
    {{- if .HasChildren }}
        <li class="hasChildren">
          <label for="{{ .Name | safeHTML }}">
          <input type="checkbox" id="{{ .Name | safeHTML }}"/>
          <style>
              #{{ .Name | safeHTML }} {
                  display: none;
              }

              #{{ .Name | safeHTML }} ~ a {
                user-select: none;
              }
              
              #{{ .Name | safeHTML }}:checked ~ .childMenu {
                  /* Make height of menu height of children.
                    Would likely need to change this if has sub menu
                  */
                  max-height: {{(.Children | len)}}00px;
                  overflow: visible;
                  opacity: 1;
                  padding: 4px 14px;
                  font-size: .8em;
              }
          </style>
    {{- else }}
        <li>
    {{- end}}
    
      <a
        {{- range $k, $v := $attrs }}
          {{- with $v }}
            {{- printf " %s=%q" $k $v | safeHTMLAttr }}
          {{- end }}
        {{- end -}}
      >{{ or (T .Identifier) .Name | safeHTML }}</a>
      {{- with .Children }}
        <ul class="childMenu">
          {{- partial "inline/menu/walk.html" (dict "page" $page "menuEntries" .) }}
        </ul>
      {{- end }}
      {{- if .HasChildren }}
    </label>
{{- end}}
    </li>
  {{- end }}
{{- end }}
```


## Include `sidebar.html` partial in layout
Example modified `layouts/index.html`
```html
<!DOCTYPE html>
<html lang="{{ .Lang }}">
{{ partial "head.html" . }}

<body>
  {{partial "search.html" .}}
  <div class="singlePage">
    <div class="sidebar">
      {{ partial "menu.html" (dict "menuID" "main" "page" .) }}
    </div>
    <div class="content">
      <!-- Begin actual content -->
      {{partial "header.html" .}}
      <article>
        {{partial "toc.html" .}}
        {{partial "textprocessing.html" . }}
        {{if $.Site.Data.config.enableRecentNotes}}
        {{partial "recent.html" . }}
        {{end}}
      </article>
      {{partial "footerIndex.html" .}}
    </div>
  </div>
</body>

</html>
```


## Create `assets/styles/sidebar.scss`
```scss
li.hasChildren {
	list-style: upper-roman;
}

.childMenu {
	overflow: hidden;
	max-height: 0;
	padding: 0;
	margin: 0 auto;
	opacity: 0;
	font-size: .8em;
}

.childMenu li {
	position: relative;
	list-style-type: none;
}

.childMenu li:hover::before {
	background-color: var(--dark);
}

.childMenu li::before{
	content: '';
	width: 2px;
	height: 100%;
	position: absolute;
	left: -10px;
	background-color: var(--outlinegray);
}
```

## Add pages to Sidebar in metadata
Unfortunately, this is not an automatic process. You must add pages to the Sidebar using metadata. This is very straightforward, though.

For example, if we have a folder called "Notes", and a page called "Sidebar in Quartz", we would modify the `content/notes/Sidebar in Quartz.md` and add the following to metadata:

```
---
title: Sidebar in Quartz

tags:
- code
- quartz
- obsidian

menu:
    main:
        params:
        parent: Notes
---

# Sidebar in Quartz
blahblahblah
```

You would do this for every page that you want included in the menu. Another alternative is to just modify the layout html and hardcode a menu. 

Happy Quartz-ing.
