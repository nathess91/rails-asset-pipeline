
## Using JS within the Asset Pipeline

The Asset Pipeline is a great tool for minifying all of your assets and speeding up page load times.  But it adds some additional challenges when working with JavaScript.  

It also gives you the chance to organize your javascript into many files and not have to worry about synchronizing which script depends on others and organizing your script tags to match.

Since all of our javascript in the app is active on every page in the app we have to be careful about how we bind to events.  For example:

```js
$('button#save').on('click', function(e) { 
  console.log('save button clicked!');
  var title = $('input#title').val();
  $.post('/posts', { title: title }).success(....
```

What happens if we have a button `<button class="btn" id="save">` on our `posts/new` page?

It works right?  No problems.  

Now what if we also have a modal on `/posts/:id` that lets users submit a comment.  It might also have a `<button class="btn" id="save">` and our JS here will bind to it as well.  We'll try to save a post when we really want to just save the comment.  Or maybe we'll do both?


### Dealing with this problem

The solution here is pretty clear, we just need to make sure we're careful with our **selectors** and make sure that they're unique across the **entire app**.

I'm going to give you **two** ways to do this.  There are of course others but the key is to be very precise in what you're binding to.

1. Use very specific complex selectors or very specific IDs.
  
  ```html
  <!-- this -->
  <button class="btn" id="save">
  <!-- can be refactored to -->
  <button class="btn" id="new-save-post">
  ```
  
  Another alternative might be to scope our jQuery better.
  
  ```js
  //this
  $('button#save').on('click', function(e) { 
  // refactors to:
  $('div#post-single-form button#save').on('click', function(e) { 
  ```
  
  In either case the key is to make our **selectors** as specific as possible.
  
1. One more idea is to add a page-specific identifier to each page.  Then we can bind our JS to that:

  ```html
  <!-- app/views/layouts/application.html.erb -->
	
	<body class="<%= controller_name %> <%= action_name %>">
	  <%= yield %>
	</body>  
  ```
  
  Then in our JS we can bind to the now modified body tag.
  
  ```js
  postNewView = 'body.posts.show';
  $(postNewView + ' #save').on('click', func..
  // or
  $(postNewView).on('click', '#save', func...
  ```
 
