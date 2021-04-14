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

``` typescript
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

Now, it would still return a plane text - but with all the markdown formatting. So, on to Markdown parsing process.

### Implementation

[vue-markdown](https://github.com/miaolz123/vue-markdown) is my library of choice. Does everything I needed and is easy to use.

``` xml
<template>
  <article class='content'>
    <vue-markdown :source='articleLoaded' :html='true' />
</template>

<script lang='ts'>
import VueMarkdown from 'vue-markdown'

@Component({
  components: {
    VueMarkdown
  }
})
export default class Blog extends Vue { 
 ... 
}
</script>
```

##### NOTE: articleLoaded is set from the mounted() code example above

Additionally, it has a very useful <a href='https://miaolz123.github.io/vue-markdown/' target='_blank'>live edit with preview demo</a> that I've used to write This post. Will probably incorporate it to this website as well later on to make Saving, Deleting and Publishing new posts a bit easier.

Now, how would you get a list of all the available blog postings? The idea of using Github as a DB storage solution here is for it to store required data in a JSON format, as if it was a response from a real backend server scrapping real db data. Thus, I've created a JSON file that would have the fields that I need to render a list post previews. I've went on a couple existing blog websites to see how they render list of posts; looked into their network call to get an idea for the JSON fields they get from their db and that is what I've come up with for now: 

```
//https://github.com/[ORG]/[PROJECT-NAME]/blob/[BRANCH]/entries.json
{
   "entries":[
      {
         "id":"THE-NAME-OF-THE-FILE",
         "uuid":null,
         "title":"Some title for the blog post.",
         "post_date":"04/12/2021",
         "reactions":{
            "heart":0
         },
         "subtitle":"Some subtitle",
         "cover_image":"",
         "description":null,
        "tags": [
          "vuejs", "nuxtjs", "vue", "nuxt", "webdev"
        ],
         "author": {
            "name": "Zach Volchak",
            "twitter": "https://twitter.com/gamehoundgames"
         },
         "hidden":false
      }
   ]
}

```

In this setup, I've used `id` as a name of the blog post file that is stored on my Github's project. Some other fields like `uuid` or `tags` I might not use right away, but they could be useful later on - so added them just in case.

Making an `http` request to the `raw` path of this file, will return a JSON object in the response:

``` typescript
    axios.get(url).then((resp: any) => {
      const entries = get(resp, 'data.entries', []) // lodash's "get" library
      // ...
    }).catch((error: Error) => {
      // ...
    })
```

I then save `entries` into the Vuex store to get them later in the component. 
