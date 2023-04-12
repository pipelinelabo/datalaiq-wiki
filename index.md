:html_theme.sidebar_secondary.remove:

# DatalaiQ Docs

This site contains documentation for DatalaiQ, plus other resources such as Changelogs.

If you're just starting out with DatalaiQ, we recommend reading the [Quick Start](quickstart/quickstart) first, then moving on to the [Search pipeline](search/search) documentation to learn more.

```{toctree}
---
hidden: true
---
Quick Start <quickstart/quickstart>
Configuration <architecture/architecture>
Ingesters <ingesters/ingesters>
Searching with DataialiQ <datalaiq>
Automation <automation>
API <api/api>
Release Notes <changelog/list>
```

::::{grid} 2

:::{grid-item-card}
:link: quickstart/quickstart
:link-type: doc

**Quick Start**  {material-twotone}`rocket;3em;sd-text-primary;`

:::
:::{grid-item-card}
:link: architecture/architecture
:link-type: doc

**Configuration**  {material-twotone}`settings;3em;sd-text-primary`

:::
::::

::::{grid} 2

:::{grid-item-card}
:link: ingesters/ingesters
:link-type: doc

**Ingesters**  {material-twotone}`upload;3em;sd-text-primary`

:::
:::{grid-item-card}
:link: gravwell
:link-type: doc

**Searching with DatalaiQ**  {material-twotone}`search;3em;sd-text-primary`

:::
::::

::::{grid} 2

:::{grid-item-card}
:link: automation
:link-type: doc

**Automation**  {material-twotone}`power;3em;sd-text-primary`

:::
:::{grid-item-card}
:link: /api/api
:link-type: doc

**API**  {material-twotone}`api;3em;sd-text-primary`

:::
::::

<script>
var url=window.location.href;
if(url.includes(".md")) {
  var split = url.split(".md");
  window.location.href= split[0].replace(/#!(.*)/g, '$1.html') + split[1].replace(/_/g, '-').toLowerCase();
}
</script>
