## How to build your own blog with VueJs/NuxtJs

### Premise

I've been thinking about creating my own blog for some time now. Didn't want to use existing platforms like Thumblr or Wordpress, mostly because I wanted to build it myself, primary, for educational purposes. Secondary, I wanted a bit more [deeper] control over the visuals of my domain. And third - price - looking for a free and easy integrate solution. So, quickly started searching for ways to create/integrate a blogging system into an existing website or use some VueJs existing libraries. 

By the way, this website is build with NuxtJS. Not a real concern compared to VueJS in the context of this topic, except for the Server-Side-Rendering capabilities that NuxtJS is enabling.

Few minutes later the top 3 contenders started dominating the search: [Gatsby](https://www.gatsbyjs.com/), [ButterCMS](https://buttercms.com/), [Wordpress](https://wordpress.com/) (and [Wordpress API](https://developer.wordpress.org/rest-api/) in particular). This all seemed interesting, but it didn't feel like what I wanted and I am not currently ready to pay for the "advanced" features that unlocks traffic and bandwidth limitations. 

Though, started to consider Wordpress API, thinking I would create a free account with whatever template they have; would create a post that and then use their API to query my post back to my website. But how would I preserve the formatting? Presumably (and I don't really know at this point), the API would just return a plane text in the JSON format. Embedding the post url with the [iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) could be an alternative, but Wordpress might block any iframe actions. If really needed to, could figure it out, but, again, seems like too much work for a "simple" blogging page.

But, it gave me an idea of how this works and how I can build it myself relatively fast.


### What is the problem?

This is a question I ask myself every time before the research and when recognizing I'm getting too deep or already deep in the rabbit hole.

So, what do I really need to get the blog functionality going on the website:

 1) Editor tool for creating and editing posts; previewing and adding formats (text decoration, images, colors, links, and etc).
 2) Database to store Blog Entries, possibly, in plane, but formatted, text.
 3) Retrieve and Parse [formatted] Content.

The last one - "Parse Content" - is the critical piece here that drove the direction for editor tool. Because storage is easy. Plenty of options like [Firestore](https://firebase.google.com/docs/firestore/quickstart), [AWS Storage](https://aws.amazon.com/products/storage/) if you are feeling fancy, or GitHub. I've chose GitHub for now.

[GitHub Markdown](https://guides.github.com/features/mastering-markdown/) is a direction I started looking into right away. It is exactly what I need: free, works as Editor tool (has Preview capability when you edit a file on the repo page), and is a storage.
And, is easy to retrieve (no need for extra api setup and whatnot):

``` javascript
// IN THE VUEJS .vue file
mounted(): void {
  const url = 'https://raw.githubusercontent.com/PATH/helloworld.md'

  this.$axios.get(url).then((resp: any) => {
    // Set local variable to render in the template.
    // More on that later
    this.articleLoaded = resp.data
  }).catch((error: Error) => {
    console.error(error.message)
  })

} //mounted
```

Cool. Almost done. Now, it would still return a plane text - but with all the markdown formatting. So, on to searching for a VueJs library to parse Markdown.

### Implementation
