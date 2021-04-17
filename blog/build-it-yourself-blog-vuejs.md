
I've been thinking about creating my own blog for some time now. Didn't want to use existing platforms like Thumblr or Wordpress, mostly because I wanted to build it myself, primary, for educational purposes. Secondary, I wanted a bit more [deeper] control over the visuals of my domain. And third - price - looking for a free and easy integrate solution. So, quickly started searching for ways to create/integrate a blogging system into an existing website or use some VueJs existing libraries. 

By the way, this website is build with NuxtJS. Not a real concern compared to VueJS in the context of this topic, except for the Server-Side-Rendering capabilities that NuxtJS is enabling.

Few minutes later the top 3 contenders started dominating the search: [Gatsby](https://www.gatsbyjs.com/), [ButterCMS](https://buttercms.com/), [Wordpress](https://wordpress.com/) (and [Wordpress API](https://developer.wordpress.org/rest-api/) in particular). This all seemed interesting, but it didn't feel like what I wanted and I am not currently ready to pay for the "advanced" features that unlocks traffic and bandwidth limitations. 

Though, started to consider Wordpress API, thinking I would create a free account with whatever template they have; would create a post that and then use their API to query my post back to my website. But how would I preserve the formatting? Presumably (and I don't really know at this point), the API would just return a plane text in the JSON format. Embedding the post url with the [iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) could be an alternative, but Wordpress might block any iframe actions. If really needed to, could figure it out, but, again, seems like too much work for a "simple" blogging page.

But, it gave me an idea of how this works and how I can build it myself relatively fast.

 
### Concept

What do I think I need to get the blog functionality going and embedded into my existing website:

 1) Editor mechanism to creating and editing posts; previewing and adding formats (text decoration, images, colors, links, and etc).
 2) Database to store raw string Blog Entries and a List of available Posts (for preview).
 3) Retrieve and Parse raw blog entry on the client side.

Storage is easy. Plenty of options like [Firestore](https://firebase.google.com/docs/firestore/quickstart), [AWS Storage](https://aws.amazon.com/products/storage/) if you can afford/handle it, or GitHub - which I ended up using.

[GitHub Markdown](https://guides.github.com/features/mastering-markdown/) is a direction I started looking into right away for the Editing mechanism. Easy to use and can be stored [somewhere] as a raw string, that could be parsed on the client side. And since I might use Github as an editing tool, why not use it as a storage as well. Create a file on a new branch, edit and preview it as you go and, when ready, merge it to `master` branch, where client will then will be pulling from to render on the page:

```javascript
mounted(): void {
  const url = `https://raw.githubusercontent.com/PATH/file-name-on-github.md`

  this.$axios.get(url).then((resp: any) => {
    // Set local variable to render in the template.
    // More on that later
    this.articleContent = resp.data
  }).catch((error: Error) => {
    console.error(error.message)
  })
}
```

##### Note the the raw.githubusercontent.com url instead of a regular github.com one
This would return a plane text - but with all the markdown syntax. So, time to find some Markdown Parsing VueJs library.

### Dealing with Markdown

[vue-markdown](https://github.com/miaolz123/vue-markdown) is my library of choice. Does everything I needed and is [*subjectively*] easy to use.

```html
<template>
  <article class='content'>
    <vue-markdown 
      :source='articleContent' 
      :html='true'
    />
</template>
```

```typescript
<script lang='ts'>
import VueMarkdown from 'vue-markdown'

@Component({
  components: {
    VueMarkdown
  }
})
export default class Blog extends Vue {
 articleContent = '' 
 // make a http request to the blog post file
 // stored on Github. Example using axios above.
}
</script>
```

Additionally, it has a handy <a href='https://miaolz123.github.io/vue-markdown/' target='_blank'>live edit with preview demo</a> that I've used to write This post. But, technically, any markdown online editor would do.

### Github as a backend emulator - getting list of blog posts

In addition to Github being a storage is for it to act as a backend emulator. I could create a `.json` file that would have all the fields and structure of the response I would get from a backend endpoint processing DB queries.

With that setup it would be as easy to just change the request url to a real backend server in the frontend code, if/when I decide to build one and move away from this architecture. For example, my backend endpoint will probably be scraping the Firestore db and constructing the same `JSON` response I would create in a Github's  .json file containing the list of available blog posts and its metadata.

I've went on a couple existing blog websites to see how they render list of posts; looked into their network calls to get an idea for the `JSON` fields they get from their db. Eventually, created a `entrie.json` in the root of my project with the following content:

```json
//https://github.com/[ORG]/[PROJECT-NAME]/blob/[BRANCH]/entries.json
{
   "entries":[
      {
         "id":"THE-NAME-OF-THE-FILE",
         "uuid":null,
         "title":"Some title for the blog post.",
         "post_date":"04/12/2021",
         "reactions":{
            "heart": "0"
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

`id` - is a name of the blog post file that is stored on <a href='https://github.com/GamehoundProductions/blog-entries' target='_blank'>my Github's project</a>.

Some other fields like `uuid` or `tags` I might not use right away, but they could be useful later on - so added them just in case.

Making an `http` request to the `raw` path of this file, will return its content as a JSON object in the response:

```typescript
getBlogEntries(): void {
 axios.get(url).then((resp: any) => {
   const entries = resp.data.entries
   // ...
 }).catch((error: Error) => {
   // ...
 })
}
```

I then save `entries` into the Vuex store to get them later in the component. And with that, I now have enough information at hand to build a list of blog posts, each component of which would require the `id` property. With that, I can construct a url pointing to the right raw file in the Github project, parse it and render on the page:

```javascript
<template>
  <article class="media">
    <div 
      class="column is-3-desktop is-full-mobile" 
      @click="selectArticle()"  <--- Critical Piece. Clicking on this div will http request the right blog post.
    >
      .... MORE HTML STUFF HERE ... 
    </div>
  </article>
</template>
```

```typescript
<script lang='ts'>
  @Prop({ type: String, default: 'latest' }) readonly id!: string

  selectArticle(): void {
     const url = `https://raw.githubusercontent.com/PATH/${this.id}`

     this.$axios.get(url).then((resp: any) => {
      // Do whatever makes sense with the resp.data here:
      // set it to a local variable for render, pass it to the Vuex
      // store or anything else.
     }).catch((error: Error) => {
       ...
     })
  }
</script>
```
