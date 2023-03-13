---
title: "Code template: do not reinvent the wheel"
seoTitle: "intellij,template,live template"
seoDescription: "This article explains how to use IntelliJ's 'Live Templates' feature to create secure snippets and easily share them with your team. Learn how to create, ed"
datePublished: Mon Mar 13 2023 12:35:27 GMT+0000 (Coordinated Universal Time)
cuid: clf6t55nm000f09lacwdibv5s
slug: code-template-do-not-reinvent-the-wheel
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/XJXWbfSo2f0/upload/a91bf1f14906d149b98f3f4f8a641698.jpeg
tags: templates, code, intellij, template, live-template

---

When I am coding, there is always a bunch of lines that I have to search about, or that takes time to type even though I know them, or code that needs to be included but I don't remember what, or code that I just copy/paste from a project without looking if it is still up to date.

Does it also happen to you? I think so!

### What to do?

Most of the time, those pieces of code are not the most interesting ones:

* Write in a file
    
* Create a mock
    
* Helm chart values!
    

Even your idea helps you when you want to create a main, a constructor, getter and setters, toString, loops, if, switch, etc...

For example the main method: if you start tipping main you will see this popup:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678703580224/ab060309-0c46-4fa1-8298-769ce13e56c4.png align="center")

And if you click (or press enter) then the code is injected:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678703602175/5a7f5023-73f4-4caa-b36f-6305e8aea0e1.png align="center")

### How to create a template?

In IntelliJ, we have the notion of live templates. Those snippets will be proposed when you start typing the correct abbreviation in the correct context.

To create a snippet, I will first write the code, try to improve it, and see if it works. When I am satisfied, then I select it, open the actions and select Save as Live template:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678705416090/0f5f0656-d328-4297-b69c-26d0fb0e2cfa.png align="center")

A new page opens:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678705662636/1ffe934b-f9c6-4ad6-896d-88a1e17f3eb2.png align="center")

Those are all the information you will see on this page:

1. This is your new template. As you have not defined any abbreviation (shortcut) yet, the default value is "abbreviation"
    
2. The context in which your template will show up: you can restrict a template to a type of file. This will help to only display js template in js files, java ones in java files, and so on. Use user context to have your templates always available.
    
3. The abbreviation is the text that will trigger the suggestion and the insert of the template. Try to find something that you will remember! Not something like template1, template2,...
    
4. The description of the template. Be sure to describe what the template is pasting. For example, in the case of JUnit4 / JUnit 5 templates, it is important to specify it in the description.
    
5. The template. It is what will be written in your code.
    
6. You will be able to define variables to make your template dynamic!
    

I will just create a basic template for now with abbreviation demo1

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706001120/b36593a0-b8b4-4b07-86f9-514bd65c3ab1.png align="center")

When I type demo1 in the code, I have the suggestion and when I select it, the code is written:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706141762/80a557f5-4a98-4a6e-8fa6-4fadfbb6afb0.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706176628/d7bf01f5-541b-4be3-a41d-3c1341cf5094.png align="center")

### How to create a dynamic template

It is nice to have a template, but it is nicer to have dynamic ones! Let's duplicate demo1 into demo2 and try to make it dynamic.

We can define variables by using $VARIABLE\_NAME$

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706502215/b9c315f2-2db8-494e-af96-e14bc9373dd5.png align="center")

When I start typing demo2 and use the template, I will see that my cursor is set to where I have defined $MESSAGE$ and it is asking me to type something.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706597676/c9dc17a2-28d4-4f31-af01-f2d118d2028e.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706611138/4e97803d-fe00-4006-abbe-c4057ad6baff.png align="center")

Nice! So now we have a variable. But what can we do with it? The edit variable is now available as IntelliJ detects the presence of variables in the templates. I will add a new variable to replace helloThere with a camel case of the string and use this new variable in the println:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706845596/c3fbf994-b7c2-4d81-83ac-c8b52a67f108.png align="center")

Now I will edit the variables to insert $MESSAGE$ value first and make IntelliJ do the camel case for me. When I open the Edit Template Variables, I can see the order of the variables. IntelliJ will use the order used in the template. I can change that thanks to the 2 arrows.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706896571/ddcf85a5-6893-4f9f-a099-d0cfe69f8ef4.png align="center")

I will move MESSAGE up.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678706970766/2647d46c-5a0d-4e9f-8e48-f0af8c2018cc.png align="center")

Then I will in VARIABLE Expression use one of the functions available and pass MESSAGE as a parameter

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678707012758/ec7a4045-570c-4994-9fc6-44acb55b4ea8.png align="center")

And this is when I use it:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678707497842/c505033a-6da0-4202-8d5f-8f2c04e061f5.gif align="center")

### What is the next step?

Having templates is great, sharing templates is better! Imagine that all the "approved code" could be done by templates. All the snippets from API have templates. If you could review templates, updates, and share so you are sure to always use the most up-to-date code.

Let's see how to export your templates. Just go to File &gt; Manage IDE Settings &gt; Export Settings

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678709149938/94db265b-46e5-44f9-af73-088e270858d4.png align="center")

Then select Live templates and Live templates (schemes) and export the zip to the best place for you:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678710011219/5f364fb1-d0f2-4c31-ba1b-84258d1a60a2.png align="center")

Now we can import settings! Same path but click import settings instead. Then pick your zip file:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678709567933/6d2904be-ed0b-48ac-ac4a-eabc05ca4905.png align="center")

And select what you want to import from those settings:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678709607724/cbcf82c5-c02a-49bc-8fb9-3898f3142488.png align="center")

And restart your IntelliJ to apply changes

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678709642713/cfc87281-55c9-4006-8824-3ca888c53871.png align="center")

### Looking a bit into this export

If we open the zip file, we can see the templates folder with all templates per context.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678710195030/c1ca2000-83ea-4960-b1f3-f5f309a91e40.png align="center")

If we dive into the user.xml, I can see my templates demo1 and demo2!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678710273474/8e24b51c-1cab-4777-a277-8a45ff5d5a20.png align="center")

Next step: find out a workflow to share those templates with a broader audience!

### Extra tips

When you go to a conference, and the presenter is doing a demo. And you see this kind of code :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1678707804025/229fc1af-3c93-4e45-9e4c-375b095ab884.png align="center")

And when he types function1, then a huge part of code shows up and you think: Wooooow, where does that come from ??? **TEMPLATE**

I am sure that he wrote the whole code, then extracted the part he wanted into templates, wrote down the abbreviation and when to play them and VOILA!

### In conclusion, use templates!

What is more to say, repetitive code is a source of error. If you start to copy/paste snippets, and the source starts to be old, and you apply it everywhere, and it spreads, what will it look like after some time?

By managing templates, reviewing them, and sharing them, those templates will live and that redundant, unsafe and boring piece of code will become attractive, useful and safe!