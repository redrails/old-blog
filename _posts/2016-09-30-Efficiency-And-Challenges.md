---
title: "Efficiency and challenges"
categories:
  - personal
tags:
  - personal
  - blog
header:
  overlay_image: https://tmrwedition.files.wordpress.com/2016/11/efficient1.jpg
  overlay_filter: 0.4
  caption: ""
---

# Why efficiency?

Being a computer science student, with aspirations of one day becoming a computer science; most of my interests lie in theory of computation including other fields around computing. But, that isn't the only reason *why efficiency*. Efficiency is important in most, if not every case of programming and general development. Whether you're making client side applications, server side APIs or console programs; the thing you should be looking to achieve is speed. No one wants to wait for a page to load, or APIs to take time sending and receiving requests, instead we want to try and make things as fast as possible.

In computer science more specifically, efficiency is the biggest issue. Efficiency of algorithms and computations are what we measure usually to either try an achieve a better or faster solution. This is why we have the whole notion of  [complexity theory](https://en.wikipedia.org/wiki/Computational_complexity_theory) and [decidability](https://en.wikipedia.org/wiki/Decidability_(logic)).

So usually, when writing any code I try to be as efficient as possible, and try to avoid doing things that may either break my code later on or will take ages to compile.

Simple things can help so much...

```
bool CanVote(int age)
{
	if(age >= 18)
	{
		return true;
	} else {
		return false;
	}
}
```
Nope. Takes up too much space. There are better ways of doing this.
```
bool CanVote(int age)
{
	return age >=18;
}
```

That's better.

# Learning new things

## The .NET

More recently at my placement, I've started using C# and the .NET framework as well as Visual Studio. I have to say, I'm thoroughly enjoying it! Being a Java, C++ and Python developer I've never before come across an IDE that is as good as Visual Studio and while I wish that VS was supported on UNIX system like Linux; I would happily use windows just for developing applications that give me peace of mind.

## Java vs. CSHARP

I know there's a lot of controversy around this whole topic and it's an opinion based topic but I've been developing in Java for a little over a year and though it's a great language with great tools and capability; I've yet to find a flaw in C#. When looking at C#, I've found that the OOP aspects and semantics that C# allows you to use are beyond great. You can do so much and the greatest thing about C# is Visual Studio. Which by the way is probably the best IDE to exist as of yet. When I discovered features like code-stepping, breakpoints, analysis and the debugging tools; I was really gobsmacked by the whole list of tools that you have access to. Eclipse and iDEA just don't give you as many options; and when developing Spring apps for example, it get's very difficult to see where you're going wrong a lot of the times.

If I had to put my two cents on this topic; I would choose C# over Java in nearly all aspects but the fact that C# and .NET are not supported on any platform other than Windows right now prevents me from moving to the Microsoft environment; it's a shame really.

# Solving problems

I just found a [really nice problem](https://www.reddit.com/r/dailyprogrammer/comments/53ijnb/20160919_challenge_284_easy_wandering_fingers/) today on [/r/DailyProgrammer](https://www.reddit.com/r/dailyprogrammer) and I thought I'd give it a go in **C#**. Now, I've only been doing C# for a little over a week so I didn't exactly know what I was doing but I managed to get it to work very nicely. You can see my CodeReview post [here](http://codereview.stackexchange.com/questions/142784/dailyprogrammer-284-wandering-fingers) for the discussion on the efficiency and the Github repo [here](https://github.com/redrails/DailyProgrammer/tree/master/cs/284).

This was a great learning experience and a very practical one at that. From the CodeReview thread you'll be able to see that many people had great suggestions on the ways that I can evolve my idea further and again; the C# language gives you so many options to do things. 

Things like [Extension Methods](https://msdn.microsoft.com/en-gb/library/bb383977.aspx) 

# Logic is more important

As with any programming task, logic is the most important aspect of anything you write. It's important to figure out what you're trying to achieve and the best way to do this is by writing the most simplistic logic that you can achieve your goal with. From there, you can worry about optimisations, efficiency, speed and other various things. Most problems that I face when writing any code whether it's an ASP REST-API or a program like I just mentioned above; starting from some base code that helps you achieve your goals is a lot more important than going away and creating a bunch of great classes but not knowing how you'll use them. This happened to me when I was trying to do the [Multi-way merge program](https://github.com/redrails/MultiwayMerge), which I still need to finish *one day*. 

# Challenges

Programming and development always present newer and newer challenges to you. Whether you're working on a project for yourself, for somebody else or developing an enterprise application; you'll always find challenges in the process. The best way to overcome these challenges is by practicing. Practicing includes reading, researching and trying out new things. For example; in my free time I just lurk on StackOverflow and a lot of the times I just answer other people's questions; and by doing this I am learning in the process because I also face issues in solving the problems. By doing these practical challenges; you pick up things that you wouldn't really read about in a book or an article. You tend to explore outside of the scope of just your purposes and it's eternally helpful when you run into similar issues in the future on your own tasks.

# Conclusion

Efficiency is important and challenges are vital when becoming a developer and even whilst being a developer. Go out and find new things to do, and if you're not a very imaginative person like myself then go out and do challenges! There are so many websites that you can use, I'll list a few below:

- [StackOverflow#Puzzle](http://stackoverflow.com/questions/tagged/puzzle)
- [CodeGolf StackExchange](http://codegolf.stackexchange.com/)
- [/r/dailyprogrammer](https://www.reddit.com/r/dailyprogrammer/)
- [/r/codegolf](https://www.reddit.com/r/codegolf/)
