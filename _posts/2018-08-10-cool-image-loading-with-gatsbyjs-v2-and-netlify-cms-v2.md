---
layout: post
title: Cool image loading with Gatsby.js v2 and Netlify CMS v2
---

Time has come and even I got the opportunity to try out currently trending Gatsby.js static site generator. The most popular feature, at least from my point of view, is [Image Processing with gatsby-transformer-sharp](https://image-processing.gatsbyjs.org/). I thought “Everyone talks that I can have out of the box responsive images and other cool things!”. Yeah sure, but they didn't use it together with Netlify CMS while using beta of v2 :) Now with Gatsby v2 released things are a bit easier.

Getting started with something as complex as Gatsby.js is always a challenge. Looking back I would compare Gatsby to Webpack. Complexity of plugin based processing pipeline makes it a bit harder to understand than good old static site generators.

Now let's investigate what do we actually need to make this work. Here are the plugins we need.

### gatsby-plugin-sharp
Exposes several image processing functions built on the Sharp.

### gatsby-transformer-sharp
Makes it possible to query [Sharp image processing library](https://github.com/lovell/sharp) processed images using GraphQL.

### gatsby-transformer-remark

Uses [remark](https://github.com/remarkjs/remark) to parse the markdown files into usable data nodes for GraphQL.

### gatsby-remark-images
This does the magic making images linked from markdown files responsive.

###  gatsby-plugin-netlify-cms-paths
This is where the fun begins! Gatsby assumes that images are colocated with posts. This approach doesn't work with Netlify CMS, because it stores images in a single place.

I ended up with this configuration:

```js
var netlifyCmsPaths = {
  resolve: `gatsby-plugin-netlify-cms-paths`,
  options: {
    cmsConfig: `/static/admin/config.yml`,
  },
}

...

netlifyCmsPaths, // Including in your Gatsby plugins will transform any paths in your frontmatter
`gatsby-transformer-sharp`,
`gatsby-plugin-sharp`,
{
  resolve: `gatsby-transformer-remark`,
  options: {
    plugins: [
      netlifyCmsPaths, // Including in your Remark plugins will transform any paths in your markdown body
      {
        resolve: `gatsby-remark-images`,
        options: {
          // It's important to specify the maxWidth (in pixels) of
          // the content container as this plugin uses this as the
          // base for generating different widths of each image.
          maxWidth: 930,
          backgroundColor: 'transparent', // required to display blurred image first
        },
      },
    ],
},
```

These are the versions that I'm using:

```json
"gatsby-plugin-sharp": "latest",
"gatsby-remark-images": "latest",
"gatsby-transformer-remark": "latest",
"gatsby-transformer-sharp": "latest",
"gatsby-plugin-netlify-cms-paths": "^1.0.2",
```

When Gatsby is configured this way I can embed image in markdown file like this `![Chemex](/img/chemex.jpg)` and get this awesome loading “blur up” behavior.

<p class="post__image-center">
    <video width="600" src="/public/gatsby-loading.mp4" autoplay loop>
    </video>
</p>

## Working with images stored in frontmatter

The blog I was working on has image displayed in the blog post list. The cover image was defined in frontmatter as follows.

```yaml
---
templateKey: blog-post
title: The ultimate Chemex guide
cover_image: /img/chemex.jpg
```

With all important bits in place let's try running `gatsby develop`, open http://localhost:8000/___graphql and run this query. You should see that image was processed.

```graphql
{
  allMarkdownRemark {
    edges {
      node {
        frontmatter {
          title
          cover_image {
            childImageSharp {
              fluid(maxWidth: 700) {
                src
              }
            }
          }
        }
      }
    }
  }
}

```

On `index.js` page, I can then use this GraphQL query combined with [gatsby-image](https://www.gatsbyjs.org/packages/gatsby-image/#gatsby-image) package. This package provides wrapper around `img` tag to generate HTML required to display responsive images. `GatsbyImageSharpFluid` is a fragment provided by `gatsby-image` its result we will then pass to `Img` component.

```graphql
cover_image {
    childImageSharp {
        fluid(maxWidth: 630) {
            ...GatsbyImageSharpFluid
        }
    }
}
```

```jsx
import Img from 'gatsby-image'

<Img fluid={post.frontmatter.cover_image.childImageSharp.fluid} />
```

You can see entire component and website on [GitHub](https://github.com/Liduska/personal-blog/blob/master/src/pages/index.js#L45).

Overall I like Gatsby. I it's one of the two static site generators I enjoy using. The other being [Jekyll](https://jekyllrb.com/).