---
title: Create a Help API
permalink: /help_api/
last_updated: March 2, 2015
---


You can create a help API that developers can use to pull in content. 

## Full code demo

For the full code demo, see the notes in the <a target="_blank" title="ToolTip Demo" href="{{ "/tooltip_demo.html" | prepend: site.baseurl }}">Tooltip Demo</a>.

In this demo, the popovers pull in and display content from the information in an external tooltips.json file located on a different host.

## Diagram overview

Here's a diagram showing the basic idea of the help API: 

<img src="{{site.baseurl}}/images/helpapi.svg" style="height: 500px;"/>

Is this really an API? Well, sort of. The help content is pushed out into a JSON file that other websites and applications can easily consume. The endpoints don't deliver different data based on parameters added to a URL. But the overall concept is similar to an API: you have a client requesting resources from a server.

Note that in this scenario, the help is openly accessible on the web. If you have a private system, it's more complicated.

To deliver help this way using Jekyll, follow the steps in each of the sections below.

## 1. Create a "collection" for the help content (optional)

A collection is another content type that extends Jekyll beyond the use of pages and posts. Here I'm calling the collection "tooltips." You could also just use pages, but if you have a lot of content, it will take longer to look up information in the file because the lookup will have to scan through all your site content instead of just the tooltips.

Add the following information to your configuration file to declare your collection:

```
collections:
  tooltips:
    output: true
```

In your Jekyll project, create a new folder called "_tooltips" and put every page that you want to be part of that tooltips collection inside that folder.

## 2. Create collection pages

Create pages inside your new tooltips collection. Each page needs only a unique `id` in the frontmatter. Here's an example: 

```
---
id: basketball
---

Basketball is a sport involving two teams of five players each competing to put a ball through a small circular rim 10 feet above the ground. Basketball requires players to be in top physical condition, since they spend most of the game running back and forth along a 94-foot-long floor.
```

You need to create a separate page for each resource you want to deliver. In this setup, you can't allow users to just grab the second sentence. You could, however, add some more frontmatter tags, and then also adjust the tooltips.json fields (explained later) to loop through the additional frontmatter tags. 

However, in this scenario, I'm just keeping it simple. When people get the page id, they just grab the content for that page. If you have 50 fields in your application that you want to provide tooltips for, you'll have 50 separate files inside your _tooltips folder.

{{alertinfo}} There may be a better way to do this. This method is just the first approach that came to mind. If you have suggestions, please let me know. {{end}}

## 3. Create a JSON file that contains the help content

Add the following to a file and call it tooltips.json:

{% raw %}
```
---
layout: none
---
{
	"entries": 
[
    {% for page in site.tooltips %}
    {
      "id"    : "{{ page.id }}",
      "body": "{{ page.content | strip_newlines | replace: '\', '\\\\' | replace: '"', '\\"' }}"
    } {% unless forloop.last %},{% endunless %}
  {% endfor %}
]
}
```
{% endraw %}

This code will loop through all pages in the tooltips collection and insert the id and body into key-value pairs for the JSON code.

{{alertsuccess}}<b>Tip:</b> Check out <a href="https://google-styleguide.googlecode.com/svn/trunk/jsoncstyleguide.xml">Google's style guide for JSON</a>. These best practices can help you keep your JSON file valid.{{end}}

Store this file in your root directory. You can add different fields depending on how you want the JSON to be structured. Here I just have to fields: `id` and `body`. And the JSON is looking just in the tooltips collection that I created. 

When you build your site, Jekyll will iterate through every page in your _tooltips folder and put the page id and body into this format.

You could create different JSON files that specialize in different content. For example, suppose you have some getting started information. You could put that into a different JSON file. Using the same structure, you might add an `if` tag that checks whether the page has frontmatter that says "getting_started: true" or something. Or you could put it into a separate collection entirely (different from tooltips).

By chunking up your JSON files, you can provide a quicker lookup, though honestly I'm not sure how big the JSON file can be before you experience any latency with the jQuery lookup.

## 4. Allow CORS access to your help

When people make calls to your site *from other domains*, you must allow them access to get the content. To do this, you have to enable something called CORS (cross origin resource sharing) within the server where your help resides. 

In other words, people are going to be executing calls to reach into your site and grab your content. Just like the door on your house, you have to unlock it so people can get in. Enabling CORS is unlocking it. 

How you enable CORS depends on the type of server. 

If your server setup allows htaccess files to override general server permissions, then create an .htaccess file and add the following:

```
Header set Access-Control-Allow-Origin "*"
```

Store this in the same directory as your project. This is what I've done in a directory on my web host (bluehost.com). Inside http://idratherbewriting.com/wp-content/apidemos/, I uploaded a file called htaccess with the preceding code. After I uploaded it, I renamed it to .htaccess, right-clicked the file and set the permissions to 774.

To test whether your server permissions are set correctly, open a terminal and run the following curl command pointing to your tooltips.json file:

```
curl -I http://idratherbewriting.com/wp-content/apidemos/tooltips.json
```

If the server permissions are set correctly, you should see the following line somewhere in the response:

```
Access-Control-Allow-Origin: *
```

If you don't see this response, CORS isn't allowed for the file. 

If you have an AWS S3 bucket, you can supposedly add a CORS configuration to the bucket permissions. Log into AWS S3 and click your bucket. On the right, in the Permissions section, click **Add CORS Configuration**. In that space, add the following policy:

```
<CORSConfiguration>
 <CORSRule>
   <AllowedOrigin>*</AllowedOrigin>
   <AllowedMethod>GET</AllowedMethod>
 </CORSRule>
</CORSConfiguration>
```

Although this should work, in my experiment it doesn't. And I'm not sure why...

In other server setups, you may need to edit one of your Apache configuration files. See [Enable CORS](http://enable-cors.org/server.html) or search online for ways to allow CORS for your server.

If you don't have CORS enabled, users will see a CORS error/warning message in the console of the page making the request. 

## 5. Explain how developers can access the help

Developers can access the help using the `.get` method from jQuery, among other methods. Here's an example of how to get a page with the ID of `basketball`:

{% raw %}
```html
<script type="text/javascript">
$(document).ready(function(){

var url = "{{site.baseurl}}/tooltips.json";


$.get( url, function( data ) {

        $.each(data.entries, function(i, page) {
            if (page.id == "basketball") {
				$( "#basketball" ).attr( "data-content", page.body );
            }

        });
    });
 
});
</script>
```
{% endraw %}

The `url` is where your tooltips.json file is. The `each` method looks through all the JSON content to find the item whose `page.id` is `basketball`. It then looks for an element on the page named `#basketball` and adds a `data-content` attribute to that element. 

{{alertwarning}}<b>Note:</b> Make sure your JSON file is valid. Otherwise, this method won't work. I use the <a href="https://chrome.google.com/webstore/detail/json-formatter/bcjindcccaagfpapjjmafapmmgkkhgoa?hl=en">JSON Formatter extension for Chrome</a>. When I go to the tooltips.json page in my browser, the JSON content -- if valid -- is nicely formatted (and includes some color coding). If the file isn't valid, it's not formatted and there isn't any color. You can also check the JSON formatting using <a href="http://jsonformatter.curiousconcept.com/">JSON Formatter and Validator</a>. If your JSON file isn't valid, identify the problem area using the validator and troubleshoot the file causing issues. It's usually due to some code that isn't escaping correctly. {{end}}

Why `data-content`? Well, in this case, I'm using [Bootstrap popovers](http://getbootstrap.com/javascript/#popovers) to display the tooltip content. The `data-content` attribute is how Bootstrap injects popovers.

Here's the section on the page where the popover is inserted:

```
<p>Basketball <span class="glyphicon glyphicon-info-sign" id="basketball" data-toggle="popover"></span></p>
```

Notice that I just have `id="basketball"` added to this popover element. Developers merely need to add a unique ID to each tooltip they want to pull in the help content. Either you tell developers the unique ID they should add, or ask them what IDs they added (or just tell them to use an ID that matches the field's name).

In order to use jQuery and Bootstrap, you'll need to add the appropriate references in the head tags of your page: 

```html
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/js/bootstrap.min.js"></script>

<script type="text/javascript">
$(document).ready(function(){
    $('[data-toggle="popover"]').popover({
        placement : 'right',
        trigger: 'hover',
        html: true
    });
```

By the way, Bootstrap's popovers require you to initialize them using the above code -- they aren't turned on by default.

View the source code of the {{tooltip_demo}} for the full comments. 

## 6. Create easy links to embed the help in your help site

You might also want to insert the same content into different parts of your help site. For example, if you have tooltips providing definitions for fields, you'll probably want to create a page in your help that lists those same definitions. You could use the same method developers use to pull help content into their applications. But it will probably be easier to simply use Jekyll's tags for doing it. 

Here's how you can pull in content from your _tooltip collection to your regular pages.

Use the `capture` tag to get the content for the tooltip page you want:

{% raw %}
```
{% capture basketball %}{% for page in site.tooltips %}{% if page.id == "basketball" %}{{page.content}}{% endif %}{% endfor %}{% endcapture %}
```
{% endraw %}

Note that here you use `page.content` instead of `page.body` as you did in the JSON file. I store these links inside a file called definitions.html inside the pages/reuse folder, but you can probably put them anywhere. In this theme, I also have an include called variables.html that includes my definitions.html, callouts.html, links.html, and notes.html files.

Where you want to insert the tag, make sure you include the variables.html file:

{% raw %}
```

```
{% endraw %}

Then put the term you captured inside two curly brace pairs:

{% raw %}
```
{{basketball}}
```
{% endraw %}

Here's a demo:

<h2>Reuse Demo</h2>


<table>
<thead>
<tr>
<th>Sport</th>
<th>Comments</th>
</tr>
</thead>
<tbody>

<tr>
<td>Basketball</td>
<td>{{basketball}}</td>
</tr>

<tr>
<td>Baseball</td>
<td>{{baseball}}</td>
</tr>

<tr>
<td>Football</td>
<td>{{football}}</td>
</tr>

<tr>
<td>Soccer</td>
<td>{{soccer}}</td>
</tr>
</tbody>
</table>

