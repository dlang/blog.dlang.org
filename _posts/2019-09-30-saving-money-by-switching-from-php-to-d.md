---
author: AndreaFontana
comments: false
date: 2019-09-30 13:03:01+00:00
layout: post
link: https://dlang.org/blog/2019/09/30/saving-money-by-switching-from-php-to-d/
slug: saving-money-by-switching-from-php-to-d
title: Saving Money by Switching from PHP to D
wordpress_id: 2201
categories:
- Code
- Companies
- Guest Posts
permalink: /saving-money-by-switching-from-php-to-d/
redirect_from: /2019/09/30/saving-money-by-switching-from-php-to-d/
---

![](http://dlang.org/blog/wp-content/uploads/2019/09/7J2w9NM36cQg7v0Nk7Z0N9QVa-200x200.png)2night was born in 2000 as [an online magazine focused on nightlife and restaurants in Italy](https://2night.it/). Over the years, we have evolved into a full-blown experiential marketing agency, keeping up our vocation of spreading what’s cool to do when you go out, but specialized in producing brand events and below-the-line unconventional marketing campaigns.

We started using D at 2night in 2012 when we developed a webservice used by our Android and iOS apps. It has worked fine since then, but it was just a small experiment. In 2019, after many other experiments, we decided to take the big step: we switched the complete website from PHP to D. The time was right; we had been planning to give our website a new look and we took this opportunity to rewrite the entire infrastructure.


### Development


The job turned out to be easier than we had imagined. We implemented a small D backend over our Mongo database in a few hundred lines. We created a [Simple Common Gateway Interface (SCGI)](https://en.wikipedia.org/wiki/Simple_Common_Gateway_Interface) library to interface with [the NGINX server](https://www.nginx.com/) and another library to work with the DOM. Using the HTML DOM instead of an obscure HTML template language helped us speed up development a lot. In this way, someone who works on HTML or JavaScript is not required to know D or any template language and can deploy plain HTML and CSS files. On the other hand, someone who works on the backend does not care so much about HTML tags since he can simply access elements by ID, class, etc.; if some HTML tags are moved around the page the whole thing still works. HTML+CSS+JavaScript on the frontend and D on the backend are totally independent.

Writing code in this way is quite simple. Let’s say we want to build a blog page. We start from a simple HTML file like this:

    <!DOCTYPE html>
    <html lang="en">
      <head><title>Test page</title></head>
      <body>
    
        <!-- Main post -->
        <h1>Post title</h1>
        <h2>The optional subheading</h2>
        <p>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit.
          Proin a velit tempus, eleifend ex non, aliquam ipsum.
          Nullam molestie enim leo, viverra finibus diam faucibus a.
          Ut dapibus a orci in eleifend.
        </p>
    
        <!-- Two more posts -->
        <div id="others">
          <h3>Other posts</h3>
    
          <div>
            <h4>Post#2</h4>
            <p>
              Morbi tempus pretium metus, et aliquet dolor.
              Duis venenatis convallis nisi, auctor elementum augue rutrum in.
              Quisque euismod vestibulum velit id pharetra.
              Morbi hendrerit faucibus sem, ac tristique libero...
            </p>
          </div>
    
          <div>
            <h4>Post #3</h4>
            <p>Sed sit amet vehicula nisl. Nulla in mi est.
              Vivamus mollis purus eu magna ullamcorper, eget posuere metus sodales.
              Vestibulum ipsum ligula, vehicula sit amet libero at, elementum vestibulum mi.
            </p>
          </div>
        </div>
    
      </body>
    </html>
This is a valid HTML5 file that can be edited by anyone who knows HTML. Now we have to fill this template with real data from a database, which we can represent as an array in this example for the sake of simplicity:

```d
// A blog post
struct SimplePost
{
  string heading;
  string subheading;
  string text;
  string uri;
}

SimplePost[] posts = [
  SimplePost("D is awesome!", "This is a real subheading", "Original content was replaced", "http://dlang.org"),
  SimplePost("Example post #1", "Example subheading #1", "Random text #1"),
  SimplePost("Example post #2", "Example subheading #2", "Random text #2"),
  SimplePost("Example post #3", "Example subheading #3", "This will never be shown")
];
```
First, we must read our HTML template just as it is and parse it [using our html5 library](https://github.com/2night/arrogant):

```d
  auto page = readText("html/test.html");

  // Parse the source
  auto dom = parser.parse(page);
```
Then we replace the content of the main article with data from the first element of our array. We use the tag name in order to select the correct HTML element:

```d
  // Take the first element from our data source
  auto mainPost = posts.front;

  // Update rendered data of main post
  dom.byTagName("h1").front.innerText = mainPost.heading;
  dom.byTagName("p").front.innerText = mainPost.text;
  dom.byTagName("a").front["href"] = mainPost.uri;
```
We want to check if our article has a subtitle. If it doesn’t we’re going to remove the related tag.

      // If we have a subtitle we show it. If not, we remove the node from our page
      if (mainPost.subheading.empty) dom.byTagName("h2").front.detach();
      else dom.byTagName("h2").front.innerText = mainPost.subheading;
If you wanted to get the same result with a template language, you'd probably need to mess up the HTML with something like this:

    <!-- We don't like this! -->
    <? if(!post.subheading.isEmpty) ?>
    <h2><?= post.subheading ?></h2>
    <? endif ?>
This mixes logic inside the view and it disrupts the whole HTML file. Anyone who works on the HTML frontend is supposed to know what `post` is, the logic behind this object, and the template language itself. Last but not least, many HTML editors would probably be driven crazy by any custom syntax. And this is still a simple case!

Going back to our example, to fill the last part of our page we must get the container from the DOM. All we need is to perform a search by ID on the DOM:

```d
auto container = dom.byId("others").front;
```
Now we use the first element inside the container as a template. So we clone it and we empty the container itself:

```d
  // Use the first children as template
  auto containerItems = container.byCssSelector(`div[id="others"] > div`);
  auto otherPostTemplate = containerItems.front.clone();

  // Remove all existing children from container
  containerItems.each!(item => item.detach);
```
Finally we add a new child to the container for each post in our data source:

```d
  // Take 2 more posts from list. We drop the first, it's the main one.
  foreach(post; posts.drop(1).take(2))
  {
    // Clone our html template
    auto newOtherPost = otherPostTemplate.clone();

    // Update it with our data
    newOtherPost.byTagName("h4").front.innerText = post.heading;
    newOtherPost.byTagName("p").front.innerText = post.text;

    // Add it to html container
    container.appendChild(newOtherPost);
  }
```
Putting it all together:

```d
import std;
import arrogant;

// Init
auto parser = Arrogant();

// A blog post
struct SimplePost
{
  string heading;
  string subheading;
  string text;
  string uri;
}

/*
  Of course real data should come from a db query.
  We're using an array for simplicity
*/
SimplePost[] posts = [
  SimplePost("D is awesome!", "This is a real subheading", "Original content was replaced", "http://dlang.org"),
  SimplePost("Example post #1", "Example subheading #1", "Random text #1"),
  SimplePost("Example post #2", "Example subheading #2", "Random text #2"),
  SimplePost("Example post #3", "Example subheading #3", "This will never be shown")
];

void main()
{
  // Our template from disk
  auto page = readText("html/test.html");

  // Parse the source
    auto dom = parser.parse(page);

  // Take the first element from our data source
  auto mainPost = posts.front;

  // Update rendered data of main post
  dom.byTagName("h1").front.innerText = mainPost.heading;
  dom.byTagName("p").front.innerText = mainPost.text;
  dom.byTagName("a").front["href"] = mainPost.uri;

  // If we have a subtitle we show it. If not, we remove the node from our page
  if (mainPost.subheading.empty) dom.byTagName("h2").front.detach();
  else dom.byTagName("h2").front.innerText = mainPost.subheading;

  // -----
  // Other articles
  // -----

  // Get the container
  auto container = dom.byId("others").front;

  // Use the first children as template
  auto containerItems = container.byCssSelector(`div[id="others"] > div`);
  auto otherPostTemplate = containerItems.front.clone();

  containerItems.each!(item => item.detach);

  // Take 2 more posts from list. We drop the first, it's the main one.
  foreach(post; posts.drop(1).take(2))
  {
    // Clone our html template
    auto newOtherPost = otherPostTemplate.clone();

    // Update it with our data
    newOtherPost.byTagName("h4").front.innerText = post.heading;
    newOtherPost.byTagName("p").front.innerText = post.text;

    // Add it to html container
    container.appendChild(newOtherPost);
  }

  writeln(dom.document);

}
```
This program will output a new valid HTML5 page like this:

    <!DOCTYPE html>
    <html lang="en">
      <head><title>Test page</title></head>
      <body>
        <h1>D is awesome!</h1>
        <h2>This is a real subheading</h2>
        <p>Original content was replaced</p>
        <a href="http://dlang.org">More...</a>
        <h3>Other posts</h3>
        <div id="others">
          <div>
            <h4>Example post #1</h4>
            <p>Random text #1</p>
          </div>
          <div>
            <h4>Example post #2</h4>
            <p>Random text #2</p>
          </div>
        </div>
      </body>
    </html>
Of course, the same results could be achieved in many other ways and in other languages too. Our library is just a wrapper over [a plain C library named Modest](https://github.com/lexborisov/Modest). But what really makes the difference is how easy it is to write and read code thanks to D’s powerful and easy-to-understand syntax. The code shown above can be easily understood by anyone has some programming experience. I’ve received pull requests for our project from colleagues who had never heard of D at all.

That’s only one part of the big picture since we’re using many different libraries for different purposes.


### Performance


Obviously, performance was a big win. The website felt like it was running on local machines, bringing a dramatic increase to speed and lower latency across the board. After the switch, at first the load on our cloud servers was so low that we thought the website was down! Switching from PHP to D meant we could cut in half the instance size of each Amazon AWS machine in our cloud. And these machines are still underloaded. Our database cloud was highly affected by this too. We now use one quarter of its original computational power. All of this brought an instantaneous and dramatic cost savings, down to more than half of what our costs used to be.


### One more thing…


A few days after launch we realized that some of our costs were rising anyway. We were relying on a third-party service to host and cut the pictures we display on the website. This is not a simple task; in order to crop a picture correctly, you need to know where the subjects of the picture are located and you must try to keep them inside the trimmed frame. On the legacy website we mostly used a fixed proportion for images and we used a third-party service for some special cases. [The new version of 2night.it](https://2night.it/) has several different possible cuts for each “master” picture, and this raised the costs by 15x! Luckily, we found that [a D binding to the OpenCV API](https://code.dlang.org/packages/opencvd) is available. We used this to develop a smart algorithm that can cut any photo while preserving the subject of the picture. And again, the performance of our service is so impressive that we do not need a new machine to host it. In a week or so the costs for pictures dropped from some thousands of euros per month to almost 0.
