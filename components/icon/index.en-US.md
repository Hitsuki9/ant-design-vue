---
category: Components
type: General
title: Icon
cover: https://mdn.alipayobjects.com/huamei_7uahnr/afts/img/A*PdAYS7anRpoAAAAAAAAAAAAADrJ8AQ/original
coverDark: https://mdn.alipayobjects.com/huamei_7uahnr/afts/img/A*xEDOTJx2DEkAAAAAAAAAAAAADrJ8AQ/original
---

Semantic vector graphics. Before use icons, you need to install `@ant-design/icons-vue` package:

```bash
npm install --save @ant-design/icons-vue
```

## API

### Common Icon

| Property | Description | Type | Default | Version |
| --- | --- | --- | --- | --- |
| rotate | Rotate by n degrees (not working in IE9) | number | - |  |
| spin | Rotate icon with animation | boolean | false |  |
| style | Style properties of icon, like `fontSize` and `color` | CSSProperties | - |  |
| twoToneColor | Only supports the two-tone icon. Specify the primary color. | string (hex color) | - |  |

We still have three different themes for icons, icon component name is the icon name suffixed by the theme name.

```jsx
import { StarOutlined, StarFilled, StarTwoTone } from '@ant-design/icons-vue';

<star-outlined />
<star-filled />
<star-two-tone two-tone-color="#eb2f96" />
```

### Custom Icon

| Property | Description | Type | Default | Version |
| --- | --- | --- | --- | --- |
| component | The component used for the root node. | ComponentType&lt;CustomIconComponentProps> | - |  |
| rotate | Rotate degrees (not working in IE9) | number | - |  |
| spin | Rotate icon with animation | boolean | false |  |
| style | Style properties of icon, like `fontSize` and `color` | CSSProperties | - |  |

### About SVG icons

We introduced SVG icons in version `1.2.0`, replacing font icons. This has the following benefits:

- Complete offline usage of icons, without dependency on a CDN-hosted font icon file (No more empty square during downloading and no need to deploy icon font files locally either!)
- Much more display accuracy on lower-resolution screens
- The ability to choose icon color
- No need to change built-in icons with overriding styles by providing more props in component

More discussion of SVG icon reference at [#10353](https://github.com/ant-design/ant-design/issues/10353).

> ⚠️ Given the extra bundle size caused by all SVG icons imported in 1.2.0, we will provide a new API to allow developers to import icons as needed, you can track [#12011](https://github.com/ant-design/ant-design/issues/12011) for updates.
>
> While you wait, you can use [webpack plugin](https://github.com/Beven91/webpack-ant-icon-loader) from the community to chunk the icon file.

The properties `theme`, `component` and `twoToneColor` were added in `1.2.0`. The best practice is to pass the property `theme` to every `<Icon />` component.

```html
<template>
  <message-outlined :style="{fontSize: '16px', color: '#08c'}" />
</template>
<script>
  import { MessageOutlined } from '@ant-design/icons-vue';
  import { defineComponent } from 'vue';
  export default defineComponent({
    components: {
      MessageOutlined,
    },
  });
</script>
```

All the icons will render to `<svg>`. You can still set `style` and `class` for size and color of icons.

### Set TwoTone Color

When using the two-tone icons, you can use the static methods `getTwoToneColor()` and `setTwoToneColor(colorString)` to spicify the primary color.

```jsx
import { getTwoToneColor, setTwoToneColor } from '@ant-design/icons';

setTwoToneColor('#eb2f96');
getTwoToneColor(); // #eb2f96
```

### Custom Font Icon

We added a `createFromIconfontCN` function to help developer using their own icons deployed at [iconfont.cn](http://iconfont.cn/) in a convenient way.

> This method is specified for [iconfont.cn](http://iconfont.cn/).

```jsx
import { createFromIconfontCN } from '@ant-design/icons-vue';
import { defineComponent } from 'vue';
const MyIcon = createFromIconfontCN({
  scriptUrl: '//at.alicdn.com/t/font_8d5l8fzk5b87iudi.js', // generated by iconfont.cn
});

export default defineComponent({
  setup() {
    return () => <MyIcon type="icon-dianzan" />;
  },
});
```

It create a component that uses SVG sprites in essence.

The following options are available:

| Property | Description | Type | Default |
| --- | --- | --- | --- |
| extraCommonProps | Define extra properties to the component | `{ class, attrs, props, on, style }` | {} |
| scriptUrl | The URL generated by [iconfont.cn](http://iconfont.cn/) project. | string | - |

The property `scriptUrl` should be set to import the SVG sprite symbols.

See [iconfont.cn documents](http://iconfont.cn/help/detail?spm=a313x.7781069.1998910419.15&helptype=code) to learn about how to generate `scriptUrl`.

### Custom SVG Icon

#### vue cli 3

You can import SVG icon as an vue component by using `vue cli 3` and [`vue-svg-loader`](https://www.npmjs.com/package/vue-svg-loader). `vue-svg-loader`'s `options` [reference](https://github.com/visualfanatic/vue-svg-loader).

```js
// vue.config.js
module.exports = {
  chainWebpack: config => {
    const svgRule = config.module.rule('svg');

    svgRule.uses.clear();

    svgRule.use('vue-svg-loader').loader('vue-svg-loader');
  },
};
```

```jsx
import { defineComponent } from 'vue';
import Icon from '@ant-design/icons-vue';
import MessageSvg from 'path/to/message.svg'; // path to your '*.svg' file.

export default defineComponent({
  setup() {
    return () => <Icon type={MessageSvg} />;
  },
});
```

#### Rsbuild

Rsbuild is a new generation of build tool, official website https://rsbuild.dev/  
Create your own `vue-svg-loader.js` file, which allows you to customize and beautify SVG, and then configure it in `rsbuild.config.ts`

```js
// vue-svg-loader.js
/* eslint-disable */
const { optimize } = require('svgo');
const { version } = require('vue');
const semverMajor = require('semver/functions/major');

module.exports = async function (svg) {
  const callback = this.async();

  try {
    ({ data: svg } = await optimize(svg, {
      path: this.resourcePath,
      js2svg: {
        indent: 2,
        pretty: true,
      },
      plugins: [
        'convertStyleToAttrs',
        'removeDoctype',
        'removeXMLProcInst',
        'removeComments',
        'removeMetadata',
        'removeTitle',
        'removeDesc',
        'removeStyleElement',
        'removeXMLNS',
        'removeXMLProcInst',
      ],
    }));
  } catch (error) {
    callback(error);
    return;
  }

  if (semverMajor(version) === 2) {
    svg = svg.replace('<svg', '<svg v-on="$listeners"');
  }

  callback(null, `<template>${svg}</template>`);
};
```

```js
// rsbuild.config.ts
/* eslint-disable */
import { defineConfig } from '@rsbuild/core';
import { pluginVue } from '@rsbuild/plugin-vue';

export default defineConfig({
  tools: {
    bundlerChain(chain, { CHAIN_ID }) {
      chain.module.rule(CHAIN_ID.RULE.SVG).exclude.add(/\.svg$/);
    },
    rspack: {
      module: {
        rules: [
          {
            test: /\.svg$/,
            use: ['vue-loader', 'vue-svg-loader'],
          },
        ],
      },
      resolveLoader: {
        alias: {
          'vue-svg-loader': require('path').join(__dirname, './vue-svg-loader.js'),
        },
      },
    },
  },
});
```

The following properties are available for the component:

| Property | Description                                      | Type             | Default        |
| -------- | ------------------------------------------------ | ---------------- | -------------- |
| class    | The computed class name of the `svg` element     | string           | -              |
| fill     | Define the color used to paint the `svg` element | string           | 'currentColor' |
| height   | The height of the `svg` element                  | string \| number | '1em'          |
| style    | The computed style of the `svg` element          | CSSProperties    | -              |
| width    | The width of the `svg` element                   | string \| number | '1em'          |
