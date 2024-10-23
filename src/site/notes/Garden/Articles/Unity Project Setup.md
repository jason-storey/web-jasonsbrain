---
{"dg-publish":true,"dg-path":"Articles/Unity Project Setup.md","permalink":"/articles/unity-project-setup/","tags":["article"],"updated":"2024-10-23T12:14:00.980+01:00"}
---



<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">





I use git. Starting with installing Git and GitLFS I setup a folder for the project and use a folder structure like this:

```
MyProject
 - U.MyProject
	 - .gitignore
	 - .gitattributes
- .git
```

Why like that? As a project grows it tends to have additional context. Documentation, Fmod Files, Marketing Copy etc. If you have the LFS storage to support it I find it easier to keep as much surrounding context of a project... with the project. Otherwise you have to start hunting for different parts and it becomes a headache. 
You could also setup git submodules or use symlinks but in all honesty its not worth the pain. In 90% of cases knowing "Most stuff is in the git repo and some extra stuff is in the google drive" is good enough.

One thing to note. If you are going to use a standard GitIgnore you find online, note its starting directory. Most of them assume starting in the "Unity" directory. In my case the "U." which I write like that for that very reason. So you can tell where the Unity project starts. A git folder can have as many gitignores as you like at different depths and I set it up like this because each added folder might have its own reason to ignore different things.

Finally, I vaguely try to stick to [Conventional Commits]() pattern for commit naming. Not mandatory but makes debugging easier later.

</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">



Unity has some baffling defaults setup. 
Here is what is included in a basic "Empty" project. Apparently that empty project assumes I am going to making a VR Android game with Vehicles, Cloth and both 2d and 3d physics. 

So I take out everything I wont use right now. (Though keep in mind a number of assets try to be _helpful_ and add modules to support these components with no option to remove them so sometimes you have to re-include them when using something like a tweening asset, unless you are willing to cut it out of the code yourself which... sometimes I do.)



</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">



Each to their own but there are a few defaults that annoy me or I want changed. 

Here is the short list. 


Oh and for bonus points here are a few other things you can configure at this stage

</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">



Personally I think a lot of early unity headaches come from a downstream problem of not having an "entry point". 
A lot of work is done to ensure things exist in a lifecycle where anything can be unloaded and loaded because there is no firm footing. Well, instead of making everything singletons and trying to dance around scene loadings its infinitely easier to setup the things that need to exist for the lifetime of the project at the start of the project. 

Here is my standard bootstrapper.


This loads a "Something" Called Game from the Resources folder. That's about all you need. A place to put things that will exist as long as the game runs. 

Though a common problem with that is if multiple people want to modify it you now have a very "hot" object that will lead to maximal collisions and general pain in the ass to change. So I like to add an extra step and have it... load services. That way you can swap out the service array and/or add entries far easier than managing the nested yaml serialization of a prefab. (Not to mention serializers and mergers in general are happier merging array elements than trying to cobble together unity's custom implementation of yaml)

An extension you can add to this down the line if you start getting fancy with your dependency chain is to either auto load all things that implement a "service" interface and/or create an injection attribute. I've done both in the past and find its not super needed unless you have a lot of team members who don't understand the process. 
If you do start to go down that road though you get into a more complex place than unity's "Dual Phase" initialization with Awake and Start and may have to resort to [Topological Sorting]() to ensure that preconditions are met and you don't end up in circular loops of dependencies needing dependencies that need the first dependencies. 

</div></div>


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">



At the Start of a project there are a few services I like to setup that are almost always required. 

- Config
- Input
- Database
- Secrets
- EventBus
- Commands
- Persistence?
- Web
- Ui
- Activity


<div class="transclusion internal-embed is-loaded"><div class="markdown-embed">



Most games have some text file to configure it. The quick and dirty easiest solution is to just wrap Json.Net's JObject and be done with it. 
This will give you a handy nested config file that will do for 90% of cases and is far better than relying on storing PlayerPrefs in the registry. 

Here is a simple implementation. 


I also add a simple utility function to load and save to and from the default locations because again... It's fine for 90% of cases and if you need to store it somewhere else.. you can.

Later its a good idea to get mildly fancier and have a default config as well as a user config that stores only the changes, this can be easy to work with too because Json.Net can allow you to apply changes from one object on top of another, but for now this is more than good enough. 

</div></div>


</div></div>
