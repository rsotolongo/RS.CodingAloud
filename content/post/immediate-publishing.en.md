---
title: "Immediate publishing"
date: 2021-02-22
draft: false
slug: immediate-publishing
tags: ["Visual Studio", "MSBuild"]
categories: ["Development"]
---
[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ImmediatePublishing)

Once upon a time, a solution hosted in Team Foundation Services (TFS) team project that I downloaded for the first time. I knew that solution build and could be published seamlessly right away. But I got a surprise because both processes failed. Figure 1 shows a simplified project structure.

![Project structure](/immediate-publishing/01.png)

Figure 1: "Project structure".

Figure 2 shows how the project was configured to generate its documentation:

![Project documentation generation](/immediate-publishing/02.png)

Figure 2: "Project documentation generation".

Once we try to build it we get the error shown in figure 3.

![Project build result](/immediate-publishing/03.png)

Figure 3: "Project build result".

The error is because the documentation file: "RS.Blog.Projects.ImmediatePublishing.Web.xml" is read-only and can't be overwritten. This problem can be solved if we go through the file system and remove the attribute: "read-only" from that file. That is the short-term solution because if you or another team member download the solution for the first time, it will have the attribute again impeding that solution build. The long term solution is to exclude that file from the project as figure 4.

![Exclude documentation file from the project](/immediate-publishing/04.png)

Figure 4: "Exclude documentation file from the project".

This step adds a couple of lines into the project file (.csproj) as figure 5 shows:

![Project file content](/immediate-publishing/05.png)

Figure 5: "Project file content".

Now, the solution builds but once we try to publish it; it doesn't work. Figure 6 shows the error.

![Publishing error](/immediate-publishing/06.png)

Figure 6: "Publishing error".

In this case, the error is also related with the attribute: "read-only" but for file: "web.config". Usually, we publish applications under configuration: "Release" and configuration transformation for "web.config" can't be executed. Once again, removing the attribute from the file: "web.config" (figure 7) will solve the problem and finally we can publish the application straight away (figure 8).

![Removing 'read-only' attribute](/immediate-publishing/07.png)

Figure 7: "Removing 'read-only' attribute".

![Publishing result](/immediate-publishing/08.png)

Figure 8: "Publishing result".

### Summary ###

Publishing an application immediately after downloaded for the first time from TFS requires to remove attribute: "read-only" from file: "web.config". If the project also generates documentation is better to exclude it from the project instead of remove the same attribute from the documentation file.

[Source code of support available at GitHub](https://github.com/rsotolongo/RS.Blog.Projects/tree/main/ImmediatePublishing)
