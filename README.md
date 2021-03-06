<a href="https://explosion.ai"><img src="https://explosion.ai/assets/img/logo.svg" width="125" height="125" align="right" /></a>

# displaCy.js: An open-source NLP visualiser for the modern web

[displaCy.js](https://demos.explosion.ai/displacy) is a modern and service-independent visualisation library. We hope this makes it easy to compare different services, and explore your own in-house models. If you're using [spaCy](https://spacy.io)'s syntactic parser, displaCy should be part of your regular workflow. Because spaCy's parser is statistical, it's often hard to predict how it will analyse a given sentence. Using displaCy, you can simply try and see. You can also share the page for discussion with your team, or save the SVG to use elsewhere. If you're developing your own  model, you can run the service yourself — it's 100% open source.

To read more about displaCy.js, check out the [blog post](https://explosion.ai/blog/displacy-js-nlp-visualizer).

[![npm](https://img.shields.io/npm/v/displacy.svg)](https://www.npmjs.com/package/displacy)

## Run the demo

This demo is implemented in [Jade (aka Pug)](https://www.jade-lang.org), an extensible templating language that compiles to HTML, and is built or served by [Harp](https://harpjs.com). To serve it locally on [http://localhost:9000](http://localhost:9000), simply run:

```bash
sudo npm install --global harp
git clone https://github.com/explosion/displacy
cd displacy
harp server
```

Or simply install it from npm:

```bash
npm install displacy-demo
```

The demo is written in ECMAScript 6. For full, cross-browser compatibility, make sure to use a compiler like [Babel](https://github.com/babel/babel). For more info, see this [compatibility table](https://kangax.github.io/compat-table/es6/).

## Using displacy.js

To use displaCy in your project, download [`displacy.js`](assets/js/displacy.js) from GitHub or via npm:

```bash
npm install displacy
```

Then include the file and initialize a new instance specifying the API and settings:

```javascript
const displacy = new displaCy('http://localhost:8000', {
    container: '#displacy',
    format: 'spacy',
    distance: 300,
    offsetX: 100
});
```

Our service that produces the input data is open source, too. You can find it at [spacy-services](https://github.com/explosion/spacy-services).

The following settings are available:

| Setting | Description | Default |
| --- | --- | --- |
| **container** | element to draw displaCy in, can be any query selector | `#displacy` |
| **format** | format used to generate parse (`'spacy'` or `'google'`) | `'spacy'` |
| **defaultText** | text used if displaCy is run without text specified | `'Hello World.'` |
| **defaultModel** | model used if displaCy is run without model specified | `'en'` |
| **collapsePunct** |  collapse punctuation | `true` |
| **collapsePhrase** | collapse phrases | `true` |
| **distance** | distance between words in px | `300` |
| **offsetX** | spacing on left side of the SVG in px | `50` |
| **arrowSpacing** | spacing between arrows in px to avoid overlaps | `20` |
| **arrowWidth** | width of arrow head in px | `10` |
| **arrowStroke** | width of arc in px | `2` |
| **wordSpacing** | spacing between words and arcs in px | `50` |
| **font** | font face for all text | `'inherit'` |
| **color** | text color, HEX, RGB or color names | `'#000000'` |
| **bg** | background color, HEX, RGB or color names | `'#ffffff'` |
| **onStart** | function to be executed on start of server request | `false` |
| **onSuccess** | callback function to be executed on successful server response | `false` |
| **onError** | function to be executed if request fails | `false` |

## Visualising a Parse

The `parse()` method renders a parse generated by spaCy as an SVG in the container.

```javascript
displacy.parse('This is a sentence.', 'en', {
    collapsePunct: false,
    collapsePhrase: false,
    color: '#ffffff',
    bg: '#000000'
});
```

The visual settings specified here override the global settings. The available settings are **collapsePunct**, **collapsePhrase**, **font**, **color** and **bg**.


## Rendering a Parse Manually

Alternatively, you can use `render()` to manually render a JSON-formatted set of arcs and words:

```javascript
const parse = {
    arcs: [
        { dir: 'right', end: 1, label: 'npadvmod', start: 0 }
    ],
    words: [
        { tag: 'UH', text: 'Hello' },
        { tag: 'NNP', text: 'World.' }
    ]
};

displacy.render(parse, {
    color: '#ff0000'
});
```

The visual settings specified here override the global settings.  The available settings are **font**, **color** and **bg**.

## Converting output from other formats and adding your own

By default, displaCy expects spaCy's JSON output in the following style:

```json
{
    "arcs": [
        { "dir": "left", "end": 4, "label": "nsubj", "start": 0 }
    ],

    "words": [
        { "tag": "NNS", "text": "Robots" }
    ]
}
```

If `format` is set to `'google'`, the API response is converted from [Google's format](https://cloud.google.com/natural-language/docs/basics#syntactic_analysis_responses). To add your own conversion rules, add a new case to `handleConversion()`:

```javascript
handleConversion(parse) {
    switch(this.format) {
        case 'spacy': return parse; break;
        case 'google': return({ words: ..., arcs: ... }); break;
        case 'your_format': return({ words: ..., arcs: ... }); break;
        default: return parse;
    }
}
```

You can now initialize displaCy with `format: 'your_format'`.

## Changing the theme and colours

You can find the current theme settings in [`/assets/css/_displacy-theme.sass`](assets/css/_displacy-theme.sass). All elements contained in the SVG output come with tags and data attributes and can be styled flexibly using CSS. By default, the `currentColor` of the element is used for colouring, meaning only need to change the `color` property in CSS.

The following classes are available:

| Class name | Description |
| --- | --- |
| **.displacy-word** | word text
| **.displacy-tag** | POS tag
| **.displacy-token** | container of word and POS tag
| **.displacy-arc** | arrow arc (without label or arrow head)
| **.displacy-label** | relation type (arrow label)
| **.displacy-arrowhead** | arrow head
| **.displacy-arrow** | container of arc, label and arrow head

Additionally, you can use these attributes as attribute selectors:

| Attribute | Value | On Element |
| --- | --- | --- |
| **data-tag** | POS tag value | `.displacy-token`, `.displacy-word`, `.displacy-tag` |
| **data-label** | relation type value | `.displacy-arrow`, `.displacy-arc`, `.displacy-arrowhead`, `.displacy-label` |
| **data-dir** | direction of arrow | `.displacy-arrow`, `.displacy-arc`, `.displacy-arrowhead`, `.displacy-label` |

Using a combination of those selectors and some basic CSS logic, you can create pretty powerful templates to style the elements based on their role and function in the parse. Here are a few examples:

```css
/* Format all words in 12px Helvetica and grey */

.displacy-word {
    font: 12px Helvetica, Arial, sans-serif;
    color: grey;
}


/* Make all noun phrases (tags that start with "NN") green */

.displacy-tag[data-tag^="NN"] {
    color: green;
}


/* Make all right arrows red and hide their labels */

.displacy-arc[data-dir="right"],
.displacy-arrowhead[data-dir="right"] {
    color: red;
}

.displacy-label[data-dir="right"] {
    display: none;
}


/* Hide all tags for verbs (tags that start with "VB") that are NOT the base form ("VB") */

.displacy-tag[data-tag^="VB"]:not([data-tag="VB"]) {
    display: none;
}


/* Only display tags if word is hovered (with smooth transition) */

.displacy-tag {
    opacity: 0;
    transition: opacity 0.25s ease;
}

.displacy-word:hover + .displacy-tag {
    opacity: 1;
}
```

## Adding custom data attributes

displaCy lets you define custom attributes via the JSON representation of the parse on both `words` and `arcs`:

```json
"words": [
    {
        "tag": "NNS",
        "text": "Robots",
        "data": [
            [ "custom", "your value here" ],
            [ "example", "example text here" ]
        ]
    }
]
```

Custom attributes are added as data attributes prefixed with `data-`, so their names shouldn't contain spaces or special characters. If added to `words`, the data attributes are attached to the token (`.displacy-token`), if added to `arcs`, they're attached to the arrow (`.displacy-arrow`):

```html
<text class="displacy-token" data-custom="your value here" data-example="example text here">...</text>
```
