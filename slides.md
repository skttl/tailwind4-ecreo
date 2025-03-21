---
# You can also start simply with 'default'
theme: default
---
# Tailwind 4

---

# Agenda

- Installering
- Opsætning / build
- Opgradering
- Nye features og ændrede metoder

---

# Installering

```bash
npm install --save tailwindcss
```

main.css:
```css
@import "tailwindcss";
```

---

# Build

Kan bruge Vite og PostCSS - vi bruger blot deres CLI for at holde det enkelt.

```bash
npx @tailwindcss/cli -i ./src/main.css -o ./dist/main.css --watch --minify
```

---
layout: iframe-right
url: https://tailwindcss.com/docs/functions-and-directives#theme-directive
---

# Opsætning

tailwind.config.js er forældet!

Konfigurering via CSS:

```css
@theme {
  --color-primary: #B4D455;
}
```

---

# Plugins

Før:

```js
  plugins: [
      require("@tailwindcss/typography"),
      require("@tailwindcss/forms")({ strategy: 'class' }),
      require("tailwindcss-quantity-queries")
  ],
```

Nu:

```css
@plugin "@tailwindcss/forms" {
  strategy: class;
}

@plugin "@tailwindcss/typography";
```

---

# Egne utilities

```css
@utility scrollbar-hidden {
  -ms-overflow-style: none;

  &::-webkit-scrollbar {
    display: none;
  }
}

@utility container {
  margin-inline: auto;
  padding-inline: 2rem;
}

@utility block-size-* {
  block-size: --value(integer);
}
```

---

# Egne components

```css
@layer components {
  .btn {
    /* ... */
  }
}
```

---

# Egne varianter

Før (js):

```js
    plugins: [
        plugin(function ({ addVariant }) {
            addVariant('error', '&.input-validation-error');
            addVariant('valid', '&.input-validation-valid, &:valid')
        })
    ],
```

Efter (css):

```css
@custom-variant mouse-only (@media screen and (pointer: fine));

@custom-variant marker (&::marker);

@custom-variant sidenav (.group\/sidenav &);

@custom-variant hx-request (.htmx-request &);

```

---

# Utilities, components, varianter?

Vigtigt i forhold til specificitet:

- Utilities trumfer components, også hvis componenten har højere specificitet, fx. `.my .component`
- Varianter kan bindes sammen med utilities, fx. marker:hidden

---

# Utilities, components, varianter?

For eksempel:

main.css
```css
@layer component {
  .my .component { /* specificitet: 0 2 0 */
    color: red; /* eller @apply text-red */
  }
}
```

index.html
```html
<div class="my">
  <div class="component text-blue-500">
    Teksten er blå!
  </div>
</div>
```

---

# Opgrader eksisterende projekt

Eller rettere, migrer - nok nemmere...

---

# Auto opgradering

```bash
npx @tailwindcss/upgrade
```

Blandede erfaringer med vores projekter

---

```diff
{
  /* .. */
  "scripts": {
    /* .. */
-    "build:css": "cross-env NODE_ENV=production && npx tailwindcss -i ./UI/CSS/main.scss -o ./wwwroot/css/main.css -c ./UI/CSS/tailwind.config.js --postcss ./UI/CSS/postcss.config.js --minify",
-    "watch:css": "npx tailwindcss -i ./UI/CSS/main.scss -o ./wwwroot/css/main.css -c ./UI/CSS/tailwind.config.js --postcss ./UI/CSS/postcss.config.js --watch --minify",
+    "build:css": "cross-env NODE_ENV=production && npx --yes @tailwindcss/cli -i ./UI/CSS/main.css -o ./wwwroot/css/main.css --minify",
+    "watch:css": "npx --yes @tailwindcss/cli -i ./UI/CSS/main.css -o ./wwwroot/css/main.css --watch --minify",
    /* .. */
  },
  "author": "ecreo",
  "devDependencies": {
-    "autoprefixer": "^10.4.19",
-    "generate-tailwind-scale": "^0.1.4",
-    "postcss": "^8.4.39",
-    "postcss-import": "^16.1.0"
    /* .. */
  },
  "dependencies": {
-    "tailwindcss": "^3.4.17",
+    "tailwindcss": "^4.0.9",
    "@tailwindcss/forms": "^0.5.10",
    "@tailwindcss/typography": "^0.5.16",
    "tailwindcss-quantity-queries": "^0.1.0"
  }
}
```

---

main.css
```diff
-@import "tailwindcss/base";
-@import "tailwindcss/components";
-@import "tailwindcss/utilities";
+@import 'tailwindcss';
```

tailwind.config.js
```js
module.exports = {
  theme: {
    extend: {
      colors: {
        "primary": "#b4d455",
```

main.css
```css
@theme {
  --color-primary: #b4d455;
```

---
layout: iframe-right
url: https://tailwindcss.com/docs/upgrade-guide#changes-from-v3
---

# Gennemgå alt kode for ændringer

Der er nogle stykker, men mest kun obskure utilities.

"Søg og erstat" er din ven!

https://tailwindcss.com/docs/upgrade-guide#changes-from-v3

---

# theme()

Det anbefales at bruge CSS variabler hvorend muligt, fx:

```diff
.my-class {
-  background-color: theme(colors.red.500);
+  background-color: var(--color-red-500);
}

-@media (width >= theme(screens.xl)) {
+@media (width >= theme(--breakpoint-xl)) {
  /* ... */
}

.myclass {
-  @screen xl {
+  @variant xl {
    /* ... */
  }
}
```



---

# Mærkbare ændringer

---

# Syntaks ændringer

Parentes `()` i stedet for klammer `[]` angiver en variabel, fx.

- `w-​(--width)` bliver til `width: var(--width)`
- `w-[100cqw]` bliver til `width: 100cqw`
- Mange utilities understøtter et vilkårligt tal som værdi, fx. `w-223.75`

---
layout: iframe-right
url: https://tailwindcss.com/docs/functions-and-directives#spacing-function
---

# Spacing

- Basisværdi i themet fx. `--spacing: 0.25rem`
- Vilkårligt tal, hvor den bruges, fx. `w-223.75`
- `calc(223.75 * 0.25rem)`
- Kun .25, .5 og .75
- `--spacing()`

---
layout: iframe-right
url: https://tailwindcss.com/docs/grid-template-columns
---

# Grid

- Vilkårligt tal for kolonner / rækker, uden at definere i theme.
- Eller fx. `grid-cols-[1fr_minmax(100px,1fr)]`, nødvendige mellemrum sørger Tailwind for

---

# Container Queries

- `@container` definerer en container, som man kan lave container queries på.
  - `@@container` i cshtml!
- `@sm:hidden` bruges som normale varianter, dog med @ foran.

```html
<div class="@container">
  <div class="@sm:hidden">
    Jeg er skjult, hvis min container er bredere end 24rem
  </div>
  <div class="@min-[475px]:hidden">
    Jeg er skjult, hvis min container er bredere end 475px
  </div>
</div>
```

---

# Container Queries

- `@container` definerer en container, som man kan lave container queries på.
  - `@@container` i cshtml! (eller `@("@container")`)
- `@sm:hidden` bruges som normale varianter, dog med @ foran.

```html
<div class="@container/hest">
  <div class="@container">
    <div class="@sm/hest:hidden">
      Jeg er skjult, hvis containeren med navnet "hest" er bredere end 24 rem
    </div>
  </div>
</div>
```

---

# Container Queries

Stem på præsentationen!

https://ecreo.atlassian.net/issues/IIU-100?filter=11119


---

# Definering af egne utilities

```css
@theme {
  --tab-size-github: 8;
}
@utility tab-* {
  tab-size: --value([integer]); /* support for arbitrary value, integer / percentage */
  tab-size: --value(integer); /* support for numeric value */
  tab-size: --value(--tab-size-*); /* support for theme value */
}
```

|Class|CSS|
|-|-|
|`tab-[2px]`|N/A|
|`tab-2 `|`tab-size: 2`|
|`tab-github`|`tab-size: 8`|

---

# Definering af egne utilities, eksempler

```css
@utility block-size-* {
  block-size: --value(integer);
}

@utility block-size-auto {
  block-size: auto;
  block-size: calc-size(auto, size);
}

```

---

# Definering af egne utilities, eksempler

```css
@utility prose {
  color: inherit;
  --tw-prose-body: inherit;
  --tw-prose-headings: inherit;
  --tw-prose-lead: inherit;
  --tw-prose-links: inherit;
  --tw-prose-bold: inherit;
  --tw-prose-counters: inherit;
  --tw-prose-bullets: inherit;
  --tw-prose-hr: inherit;
  --tw-prose-quotes: inherit;
  --tw-prose-quote-borders: inherit;
  --tw-prose-captions: inherit;
  --tw-prose-kbd: inherit;
  --tw-prose-kbd-shadows: inherit;
  --tw-prose-code: inherit;
  --tw-prose-pre-code: inherit;
  --tw-prose-pre-bg: inherit;
  --tw-prose-th-borders: inherit;
  --tw-prose-td-borders: inherit;
  --tw-prose-invert-body: inherit;
  --tw-prose-invert-headings: inherit;
  --tw-prose-invert-lead: inherit;
  --tw-prose-invert-links: inherit;
  --tw-prose-invert-bold: inherit;
  --tw-prose-invert-counters: inherit;
  --tw-prose-invert-bullets: inherit;
  --tw-prose-invert-hr: inherit;
  --tw-prose-invert-quotes: inherit;
  --tw-prose-invert-quote-borders: inherit;
  --tw-prose-invert-captions: inherit;
  --tw-prose-invert-kbd: inherit;
  --tw-prose-invert-kbd-shadows: inherit;
  --tw-prose-invert-code: inherit;
  --tw-prose-invert-pre-code: inherit;
  --tw-prose-invert-pre-bg: inherit;
  --tw-prose-invert-th-borders: inherit;
  --tw-prose-invert-td-borders: inherit;
}
```

# Definering af egne varianter

```css
@custom-variant mouse-only (@media screen and (pointer: fine));

@custom-variant marker (&::marker);

@custom-variant sidenav (.group\/sidenav &);

@custom-variant hx-request (.htmx-request &);

```

|Class|CSS|
|-|-|
|`marker:hidden`|`.marker\:hidden::marker { display:none; }`|
|`sidenav:hidden `|`.group\/sidenav .sidenav\:hidden { display:none; }`|
|`mouse-only:hidden`|`@media screen and (pointer:fine) { .mouse-only\:hidden { display:none; }}`|


---

# Custom properties

- Alt er tilgængeligt via custom properties, fx. `--color-blue-500`.
- Alt kan ændres via custom properties, fx `style="--color-blue-500: green;"` eller `class="[--color-blue-500:green]"`

Fx:

```html
<div 
  class="bg-​(--selected-color)" 
  style="--selected-color: var(--color-@Model.SelectedColorName, '#b4d455')"
>
  Min baggrundsfarve er defineret i CMS'et!
</div>
```

Eller:

```html
<body style="--color-primary: var(--color-@site.SelectedColorName, --color-black)">
  <h1 class="text-primary">
    Jeg er den primære farve valgt i CMS'et!
  </h1>
</body>
```

---

# Nærmest ingen forskel

- I daglig brug (`class=""`), mærker man ingen forskel
- Kort læringskurve til ny konfiguration

---

# Mangler

## ~~Transforms til content, `@@container` til `@container`~~ 

~~Så man ikke skal skrive `@("@container")` i cshtml.~~

Er fixet

---

# Mangler

## Dynamiske plugins, ex. quantity-queries `children-[>5]:hidden`

Javascript API'et virker stadig, men ville være rart at kunne definere i CSS.

---

# Mangler

## Regex baseret safelist

Kan ikke lade sig gøre pt.
Tailwind selv har umiddelbart ingen planer

https://www.npmjs.com/package/create-tailwind-safelist

patterns.js
```js
module.exports = [
  '(sm:|md:|lg:)text-(blue|red)-500',
  '(hover:|)text-blue-500',
  'text\-blue', // Example with escaped hyphen
];
```

`npx create-tailwind-safelist@latest ./patterns.js`



---

# Extensions

Intellisense til VS Code
https://marketplace.visualstudio.com/items?itemName=bradlc.vscode-tailwindcss


---

# Extensions

Intellisense til Visual Studio
https://marketplace.visualstudio.com/items?itemName=TheronWang.TailwindCSSIntellisense

---

# Extensions

Rider 👎
