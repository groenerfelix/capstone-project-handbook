
# MkDocs Guide

This is a proof-of-concept for an interactive handbook to be handed out to students doing the capstone project. It is written using [MkDocs](https://www.mkdocs.org/).

See an example page at 

## Setup, Development, and Deployment Steps:
1. Install and set up python
2. Create a virtual environment `python -m venv venv`
3. Activate venv `venv\Scripts\activate`
4. install requirements `pip install -r requirements.txt`
5. Change directory `cd src`
6. Start a local live preview in the browser `mkdocs serve --livereload`
7. Edit files in `src/handbook/`: `md` for pages, `.yml` for config, `.css` for styling
8. Save files and build distribution with `mkdocs build`
9. Deploy files in `output/` to static hosting (e.g., Github Pages)
10. See bottom of this document for instructions how to turn into a PDF.


## Regular Markdown

Write documentation in regular markdown.

````markdown
# Title
## Header 1
### Header 2
Regular text, *italicized* and **bold**
```python
code
```
[Link text](link.url)
````
See more like tables, footnotes etc. here: [Markdown cheat sheet](https://www.markdownguide.org/cheat-sheet/)




## Text Boxes
Requires adding `pymdownx.admonitions` to the `mkdocs.yml`. [Documentation](https://squidfunk.github.io/mkdocs-material/reference/admonitions/)

```markdown
!!! info "Details"
    This box holds longer explanations.
```
- `!!!` marks it as a box. Create collapsible boxes with `???+`, or initially collapsed with `???` 
- `info` is the type (determines icon and color); available types: note, abstract, info, top, success, question, warning, failure, danger, bug, example, quote
- `Details` is the title
- the indented text is the content

Boxes can be placed inline with `!!! info inline end`.




## Code Blocks

Add titles and line numbers like this (where 1 is the starting line number):

    ```python title="<custom title>" linenums="1"
    def test():
        pass
    ```

Highlight lines by adding `hl_lines="2 3"` or `hl_lines="2-4"`.

Add buttons for selecting and copying the code in `mkdocs.yml`:

```yml
theme:
    features:
        - content.code.copy
        - content.code.select
```


## In-line Annotations
Requires adding `pymdownx.superfences` to the `mkdocs.yml`. [Documentation](https://squidfunk.github.io/mkdocs-material/reference/annotations/)

    ```python
    def add(a:int, b:int) -> int: # (1)!
        return a + b # (2)!
    ```

    1. `a:int` is the first number. `b:int` is the second number.
    2. The function returns their sum.

The icon can be changed in the config file
```yml
theme:
    icon:
        annotation: material/help-circle
```



## Images and Captions

```markdown
<figure markdown="span">
  ![Image title](https://dummyimage.com/600x400/){ width="300" }
  <figcaption>Image caption</figcaption>
</figure>
```



## Tooltips

Individual tooltios can be created for links:
```markdown
[Hover me](https://example.com "I'm a tooltip!")
```

This needs to be enabled in the `mkdocs.yml` config file.

```yml
theme:
  features:
    - content.tooltips
```


Recurring tooltips can be defined once for every appearance of the word (e.g., abbreviations):
```
The HTML specification is maintained by the W3C.

*[HTML]: Hyper Text Markup Language
*[W3C]: World Wide Web Consortium
```

This also needs to be enabled in the `mkdocs.yml` config file.

```yml
markdown_extensions:
    - abbr
```




## Icons and Emojis

Find a list of emojis and icons here: [Link](https://squidfunk.github.io/mkdocs-material/reference/icons-emojis/)

```markdown
Welcome to the handbook. :fontawesome-regular-face-laugh-wink:
```

This also needs to be enabled in the `mkdocs.yml` config file.

```yml
markdown_extensions:
  - attr_list
  - pymdownx.emoji:
      emoji_index: !!python/name:material.extensions.emoji.twemoji
      emoji_generator: !!python/name:material.extensions.emoji.to_svg
```


## Printing to PDF
The easiest way to get a PDF document from this is to open the webpage and print it using the built-in browser function (e.g., `CTRL + P`).

The `extra.css` file contains some helper classes to control page breaks in prints.
- Insert `<div class="page-break"></div>` where you want to force a break.
- Wrap content in `<div class="no-break">...</div>` to avoid splitting it across pages.