---
title: "Coding style using StyleCop"
date: 2021-01-04
draft: false
slug: stylecop-coding-style
tags: ["ReSharper", "Visual Studio", "StyleCop", "source code", "code analysis"]
categories: ["Development"]
---
A colleague ask me to share my experience coding with ReSharper and once he heard that I’m not use ReSharper he was even more curious about how I write source codes. Who knows me knows I love NuGet packages and be updated constantly, that’s why I found a very useful package that I install in all my projects as soon I start it.

StyleCop.MSBuild

StyleCop used to be a standalone application with integration with Visual Studio but that changed a while ago. I’ll explain through an example. Let’s start creating a console application from scratch.

![Console application](/stylecop-coding-style/01.png)

Figure 1 show a just created console application in Visual Studio.

Notice the 0 errors, 0 warnings, and 0 messages once is compiled. Let’s install the NuGet package and see what happens. Figure 2 shows the management of NuGet packages for the application.

![Solution package management](/stylecop-coding-style/02.png)

Figure 2.

Figure 3 shows the results of the solution build once that package is installed.

![First build results](/stylecop-coding-style/03.png)

Figure 3.

Notice the 11 warnings. My personal philosophy is deliver products with zero errors (that is logic because compilers and managers looks for it); zero warnings (few tools and people prevent you to deploy with warnings); and zero messages (practically nobody cares about this).

Most of the warnings received are self-explanatory and easy to fix. Let’s analyze them one by one.

The first and the last one are related with the fact that files "Program.cs" and "AssemblyInfo.cs" don’t have header and can be fixed just adding a header in XML format containing the name of the file among other information useful.

The following five warnings are related with the fact that "using" statements should be inside the namespace and not outside as the template put it. The solution is obvious, just move the statements inside the namespace.

The following four warnings are related with the lack of documentation in the different code instruments: classes and methods in this case. Once the documentation are added the warnings disappear.

Figure 4 show the build result after applied the above fixes.

![Second build results](/stylecop-coding-style/04.png)

Figure 4.

Notice the new two warnings. At this point is good to mention that this is an iterative procedure until the goal (0 errors, 0 warnings, 0 messages). Also the not used "using" statements were deleted (those gray out). Figure 5 shows the new result once we fix these two warnings adding "public" modifier to the class and the method.

![First fix](/stylecop-coding-style/05.png)

Figure 5.

Hurrah! We made it! But (there is always a but, right?), these is possible because the rule set that we have configured in the project to report warnings. So, let’s be more picky changing the rule set.

Figure 6 shows the project properties in the "Code Analysis" option. Notice the unselected checkbox: "Enable Code Analysis on Build" and the "Microsoft Managed Recommended Rules" in the Rule Set dropdown.

![Project properties settings](/stylecop-coding-style/06.png)

Figure 6.

If you enable the checkbox without install the NuGet package you will not receive any warnings and if also you change the rule set without install the NuGet package you will receive only 2 warnings also returned by the StyleCop package because what is does is add many more rules to the working set. That is the reason why I love to add it to my projects and not only use what is included in Visual Studio.

Figure 7 show the recommended options.

![Recommended options](/stylecop-coding-style/07.png)

Figure 7.

Yes, "Microsoft All Rules", more strict we are, better code we obtain if we follow the criteria to obtain at the end 0 errors, 0 warnings, and 0 messages.

Figure 8 shows the results of the build after the new options for code analysis.

![Third build results](/stylecop-coding-style/08.png)

Figure 8.

Six new warnings, time to start again. Uhmmm… The first two don’t have a file associated, how I can fix them? Well there is a solution. Right click in the code "CA2210" in this case, select the option for "Suppress" and then the option "In Suppression File" as is shown in figure 9.

![Rule fix options](/stylecop-coding-style/09.png)

Figure 9.

Once this option is selected a new file called "GlobalSuppressions.cs" is created in the root of the project as is shown in figure 10.

![Rules suppressions file](/stylecop-coding-style/10.png)

Figure 10.

At this point I would like to mention that there are occasions where you can suppress the warning in the suppression file and others directly in the source code. In the figure 10 there are still 5 more warnings to fix. My choices are suppress in global suppression file the first warning, and suppress in source code the fourth warning. To suppress in source code you should do almost the same: right click in the warning’s code, select "Suppression" option and then select option "In Source".

This option adds an attribute above the method being reported containing the code to suppress the warning, similar to the code added into "GlobalSuppressions.cs" file.

The second warning can be fixed marking the "Main" method as "static"; while the third can be fixed removing the "args" parameter (and its documentation in the method comments). Finally the fifth warning can be fixed just correcting the spelling of the word. Yes, StyleCop also include spell checking, great isn’t?

Once all remaining warnings are addressed, we can build again and see what is reported this time, figure 11 show that.

![Fourth build results](/stylecop-coding-style/11.png)

Figure 11.

What? More warnings? Yes, remember that you added a new file: "GlobalSuppressions.cs" and that file enter to the loop to reports compilation warnings too. So, is time to fix the new reported warnings.

All suppressions trough code (source code of "GlobalSuppressions.cs" file or attributes in the code instruments) must to have a justification. I use my initials as justification as a way to tracking who decided to suppress this warning. That is how I fix the first and third warnings.

The second warning can be fix adding the header to the file "GlobalSuppressions.cs".

The fourth warning ask for the use of "////" instead of "//" to start a source code comment. If you don’t like this option you can suppress globally in the "GlobalSuppressions.cs" file but I prefer to fix it changing the way of how I write comments.

Finally figure 12 show how the code looks at the end.

![Final code](/stylecop-coding-style/12.png)

Figure 12.

First that all our goal is fulfilled: 0 errors, 0 warnings, and 0 messages. Second, our resulting code is beautiful and self-explanatory for future modifications as well as less entries in other security analysis tools as HP Fortify.

Is valid to mention that because this discipline requires to add comments in all instruments write all those comments manually is a tedious task. For that reason I also use another tool called "GhostDoc" that generate very accurate documentation depending of the context for the instrument being documented. The issue with this is that should be done one by one due to license limitation. Actually I use the free version but professional license allows to generate comments for a whole class, or file, or namespace in only one click or pressing "Shift + Ctrl + D" over the instrument. In the good news is that recently "GhostDoc" distributed a version using Visual Studio extension package and not the standalone application as it was, so the installation and updates become more ease.

Team, once again these are my experience creating source codes. I know that follow this discipline is not funny if the product is not new but as Robert C. Martin say: "…no matter the state of a project, but anything you change try to do your best…" to me that means to not generate new warnings. At the end always think that the developer that touch you code in the future could be a serial killer and knows your address (Robert C. Martin quote too), so is better to leave the code as clean as possible.

All of these started with a ReSharper question and will end with a ReSharper related comment. Sometimes the warnings reported by StyleCop and those reported by ReSharper enter in conflict between them. I usually give priority to those reported by StyleCop because I think they are more strict and compatible with more tools. Also there is no money associated because is free while ReSharper is not.
