# The Context
Most blogs you will run into on the net is just a pretty wrapper around plain text and are typically built as static websites, with the content of the page being baked directly into the HTML. This has 1 big disadvantage: you need to write in HTML. This may not be an issue for some people, but my preference is to write with as little mental overload as possible. I wanted an experience where I could write in a minimal format (such as Markdown) and have it converted into the relevant HTML.

This conversion of Markdown to HTML can either occur prior to the site being deployed or it can be done when the page is requested from the server. Projects such as [Hugo](https://gohugo.io/) fall into the first category and do the job really well, allowing for quick and easy deployment as a regular static site. However, something about not hand-rolling my own overly-complex solution didn't sit right with me, so I opted to build out the second option.

# Benefits (either than fun) and drawbacks
With the second option, the content for the site and the server serving the HTML (the "blog"), are 2 separate entities.
There are 3 main advantages to this:
- the blog did not have to be rebuilt and redeployed when changes to content occurred
- blog content was naturally separate from the blog itself and could easily be transformed and moved to any medium
- separate version control systems for both the content and the server code

Of course, this would lead to some additional latency when a user loads the page, as the server would need to:
1. fetch the content 
2. convert the content to HTML 
3. send it back to the user 

This could be circumvented by caching and converting the content periodically and not when the user fetches the page, but I wanted to keep version 0 as simple as possible. "It's better to finish than chase perfection" - Some smart guy probably. 

# Proposed solution
As I was very keen on using version control for the blog content, I opted to use git and host the [it](https://github.com/jasshanK/blog_content) on GitHub. This way, every new post could be a branch and merged into the main when it was complete. This made it easy to keep track of what posts were currently being worked on. Philosophically, the concept of tracking content changes via git would allow for a level of transparency not normally seen on news sites and social media (either than the editor tag).

Next, was choosing the language the blog would be built with. For a small project such as this, the choice is often more personal than it is professional. As I had just began dabbling with [Elixir](https://elixir-lang.org/) at the time, I decided this was a better time than any to train my Elixir muscles. The flagship web framework for Elixir is [Phoenix](https://www.phoenixframework.org/). I've enjoyed using it so far so I saw no reason to not go for it again.

For version 0, I wanted to make sure the following features were in:
1. a home page which lists all the posts
2. a post page to display the content
3. a navigation side bar on the post page to jump to specific sub-headings 
4. rudimentary mobile friendly scaling and item positioning 

# Building Pains
If you are interested at looking at some very unprofessional Elixir code, [here you go](https://github.com/jasshanK/blog). Instead of going over every single thing I did, I would like to highlight a few things from the building journey.

## Translating Markdown to HTML and the Navigation Bar
As you can imagine, the Markdown to HTML conversion would be the most crucial step in this whole endeavour. Thankfully, [Earmark](https://hexdocs.pm/earmark/1.4.47/Earmark.html) exists! It is a well documented Markdown parser for Elixir, with functions to directly convert the generated Abstract Syntax Tree (AST) to HTML. As such, getting the basic translation to work was trivial. 

The tricky part came in when I needed to grab the headings used so I can generate a navigation bar. I could either write some good 'ol JavaScript to do this, or I could modify the AST and tag the headings with custom ID attributes that I could then reference. The author thought of this too however, and exposed functions to allow you to pass in a function which can modify any node of the tree as it is walked. If you're interested in how this was implemented, the code can be found [here](https://github.com/jasshanK/blog/blob/main/lib/blog/posts.ex).

## CSS Woes
Getting the HTML elements to get placed exactly where I wanted proved a lot more challenging than anticipated. After battling it out with CSS Flex and some combinations of manually spacing, I stripped everything away and settled for a simple CSS Grid approach, which has been working very well (so far). This whole fiasco can be attributed to a lack of experience, but I find it amusing that styling ended up being as big of a problem as it was.

## Deployment
Since I was going down the hand-rolled route, I decided to do my deployment on a Virtual Private Server (VPS) as well, instead of using a service such as [Gigalixir](https://www.gigalixir.com/). I already run a NextCloud instance on a VPS via [Linode](https://www.linode.com/), so setting up the server and getting my domain to point to it was a matter of replication. Luckily, Dreams of Code released a great [video](https://youtu.be/F-9KWQByeU0) on the topic which helped me setup SSH hardening and my firewall. I did not want to rely on Docker and GitHub Actions for deployment, and opted to write a few lines of bash and run it on bare metal.

My host OS is NixOS while my VPS was running Ubuntu 24.04. I naively compiled the project and attempted to run it on the server only to be met by some cryptic error messages about the binary not being available. I eventually figured out that the issue was due to the OS mismatch, as was aptly forewarned in the Elixir documentation. However, Elixir's developer centric ergonomics saved me as mix, Elixir's build tool, provides an option to build the content in a Docker environment. To my surprise, it just worked!

The actual deployment was even simpler - I just tarball the release folder and send it over to the server via rsync. I then unpack the tarball on the server. I run the binary via a systemd service, so pointing to the new binary is just a matter of restarting my systemd service.

HTTPS was the final frontier to getting my site up and accessed to the internet, and I stumbled across this cool tool called [Caddy](https://caddyserver.com/). It auto renews the CA certificates, forces HTTP connections to HTTPS, and acts as a reverse proxy to the server running on a port not exposed on the firewall.

# Reflections
There is still a long way to go to get this site to where I envision it. The next big feature I would like to build would probably be a tag system for the posts as well as a way to search through posts. I'll hold off building it till I get a few more posts up so there is something to actually tag and search!

If you have any suggestions on how I can improve the site or questions on any part of the process, feel free to shoot me an e-mail at jasshank@gmail.com.



