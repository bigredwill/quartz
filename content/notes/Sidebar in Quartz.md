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

## Summary
We will configure a dropdown style menu like Obsidian has. It will not be automatically populated by file structure, but will be auto-populated by defining some frontmatter in your markdown files.

## Demo
![[images/sidebar-28.mp4|sidebar]]

## Create `sidebar.html` partial template
Create the file: `layouts/partials/sidebar.html`

We use the [`<details>` element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details) to create a dropdown.

```html
{{- $page := .page }}
{{- $menuID := .menuID }}

{{- with index site.Menus $menuID }}
  <nav class="menu">
    <ul>
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

    <!-- Opening Tag -->
    {{- if .HasChildren }}
      <!-- https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details -->
      <!-- Details creates collapsing section -->
        <details role="menu">
    {{- else }}
      <!-- If no children, it's a list item -->
        <li>
    {{- end}}
     <!-- Summary is used to display the text for <details> -->
      <summary>
        <a tabindex="0"
          {{- range $k, $v := $attrs }}
            {{- with $v }}
              {{- printf " %s=%q" $k $v | safeHTMLAttr }}
            {{- end }}
          {{- end -}}
        >{{ or (T .Identifier) .Name | safeHTML }}</a>
      </summary>
      {{- with .Children }}
        <ul class="childMenu">
          {{- partial "inline/menu/walk.html" (dict "page" $page "menuEntries" .) }}
        </ul>
      {{- end }}
      {{- if .HasChildren }}
    </label>
    {{- end}}

    <!-- Closing Tag -->
    {{- if .HasChildren }}
      </details>
    {{- else }}
      </li>
    {{- end }}


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
    <!-- Add your sidebar here. -->
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
Here's some example CSS for the dropdown menu. It's styled similar to the Obsidian sidebar.

```scss
.sidebar {
  width: 100%;
  flex: 1 0 auto;
}

.menu {
  padding-left: 2px;
  list-style-type: disc;
  font-size: .9em;
}

.menu > ul {
  padding: 0;
  cursor: pointer;
}

li.hasChildren {
  list-style: none;
}

.childMenu {
  padding-left: 24px;
  font-size: .9em;
}

.childMenu li a[aria-current="page"] {
  color: var(--primary);
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
  height: 90%;
  position: absolute;
  left: -10px;
  background-color: var(--outlinegray);
}
```

## Add pages to Sidebar in metadata
Unfortunately, this is not an automatic process. You must add pages to the Sidebar using metadata. This is very straightforward, though.

For example, if we have a folder called "Notes", and a page called "Sidebar in Quartz", we would modify the `content/notes/Sidebar in Quartz.md` and add the following to metadata:

```md
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
