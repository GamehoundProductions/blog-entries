I've been thinking about creating a simple blog engine for my website for some time now. Didn't want to use existing platforms like Wordpress, because I would not have enough "behind the scene" control over it (including the visuals).  Plus, I just like to build things every now and then for no justifiably reason.

There are couple of options for the backend api or blogging frameworks available today: [Gatsby](https://www.gatsbyjs.com/), [ButterCMS](https://buttercms.com/), [Wordpress](https://wordpress.com/) (and [Wordpress API](https://developer.wordpress.org/rest-api/) in particular). This all seemed interesting, but they have monthly fees, that I'm not currently willing to pay for.

Though, started to consider Wordpress API, thinking I would create a free account with whatever template they have; would create a blog post and then use their API to query my post back to my website. But how would I preserve the formatting? Presumably (and I don't really know at this point), the API would just return a plane text in the JSON format. Embedding the post url with the [iframe](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) could be an alternative, but Wordpress might block any iframe actions. And I would still have not control over the visuals of the posts. If really needed to, could figure it out, but not worth it [at the time].

But, it gave me an idea of how this works and how I can build it myself.

 
### Concept

What do I think I need to get the blog functionality going and embedded into my existing website:

 1) Editor mechanism to creating and editing posts; previewing and adding formats (text decoration, images, colors, links, and etc).
 2) Database to store raw string Blog Entries and a List of available Posts (for preview).
 3) Retrieve and Parse raw blog entry on the client side.

Storage is easy. Plenty of options like [Firestore](https://firebase.google.com/docs/firestore/quickstart), [AWS Storage](https://aws.amazon.com/products/storage/), or [GitHub](https://github.com/).

[GitHub Markdown](https://guides.github.com/features/mastering-markdown/) is a direction I started looking into right away for the Editing mechanism. Easy to use and can be stored [somewhere] as a raw string, that could be parsed on the client side. And since I might use Github as an editing tool, why not use it as a storage as well. Create a file on a new branch, edit and preview it as you go and, when ready, merge it to `master` branch, where client will then be pulling from with an http request and render it on the page:

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

##### Note the raw.githubusercontent url instead of a regular github one
This would return a plane text - but with all the markdown syntax. Thus, time to find some Markdown Parsing VueJs library.

### Dealing with Markdown: parse and render

[vue-markdown](https://github.com/miaolz123/vue-markdown) is my library of choice. Does everything I needed and is [*subjectively*] easy to use. It even understands and parses the `html` tags within the string which could also have a website specific css classed.

```html
<template>
  <article class='content'>
	<!-- can parse a raw string passed as property -->
    <vue-markdown 
      :source='articleContent' 
      :html='true'
    />
    
    <!-- or some text as a child element -->
    <vue-markdown :html='true'>
	  Some text here with markdown elements and
	  html <a class='custom-example'> tags </a>
	<vue-markdown />
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

I then used an online markdown editor <a href='https://stackedit.io/' target='_blank'>stackedit.io</a> that I've used to edit and preview This article. But, even editing and previewing through github would do. 

### Github as a backend emulator - getting list of blog posts

In addition to Github being a storage, it is also can act as a backend emulator. I could create a `.json` file that will have all the fields and structure of the response I could potentially get from a backend endpoint processing DB queries.

With that setup it would be as easy to just change the request url to a real backend server in the frontend code. For example, my backend endpoint will probably be scraping the Firestore db and constructing the same `JSON` response I would create in a Github's  `.json` file containing the list of available blog posts and its metadata.

I've went on a couple existing blog websites to see how they render list of posts; looked into their network calls to get an idea for the `JSON` fields they get from their db. Eventually, created an [entrie.json](https://github.com/GamehoundProductions/blog-entries/blob/blog/blog/entries.json) file in the project with the following content:

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
A list of all the blog posts metadata to be rendered on the client side.

`id` - is a name of the blog post file that is stored on <a href='https://github.com/GamehoundProductions/blog-entries' target='_blank'>my Github's project</a>.
Fields like `title`, `subtitle`, `post_date`, `tags` are pieces of metadata to be used on the page for the post preview or search queries. 
Some other fields like `uuid` or `tags` I might not use right away, but they could be useful later on - so added them just in case.

Making an `http` request to the [`raw.githubusercontent.com/[...]`](https://raw.githubusercontent.com/GamehoundProductions/blog-entries/blog/blog/entries.json) path of this file, will return its content as a `JSON` object in the response:

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

I then save `entries` into the Vuex store to get them later in the component. And now I have enough data at hand to build a list of blog posts, each component of which would require the `id` property, that must be used to construct a url string pointing to the right [raw] file in the Github project:

```javascript
// A list of available blog posts .vue component
<template>
  <article class="media">
    <div 
      class="column is-3-desktop is-full-mobile" 
      @click="selectArticle()"
    >
      <!-- MORE HTML STUFF HERE -->
    </div>
  </article>
</template>
```

```typescript
<script lang='ts'>
  // It needs to have an id to construct the right url to that blog
  // post file
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

Clicking on the article adds a query id property to the url `gamehoundgames.com/blog?id=ARTICLE` and the http request is made to get the article data to be rendered on the page, while hiding the list of available blogs.

```
@Watch('$route', { immediate: true })
onUrlChange(newVal: any): void {
  if (newVal.query.id)
    this.getArticle()
   else
    // no article is selected, thus - show list of available posts.
}

async getArticle(): Promise<void> {
  const url = `https://raw.github.../master/blog/${id}
  this.$axios.get(url).then((resp: any) => {
    this.articleContent = resp.data
  }).catch((error: Error) => {
    ...
  })
}
```

Since I'm not doing Server Side Rendering, the dynamic routes are not available (e.g. blog/article-id-1, blog/article-id-2...). Thus, I have to set the query parameter instead and listen to the route change to decide when to trigger that http request.
