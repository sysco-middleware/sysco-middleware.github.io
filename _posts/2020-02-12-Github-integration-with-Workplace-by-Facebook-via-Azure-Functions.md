---
layout: post
title: Github integration with Workplace by Facebook via Azure Functions
categories: Azure functions Github Workplace
tags: [Azure functions, C#, json, Github, Workplace]
author: denzza
---

# Introduction

At my company we are using Workplace by Facebook for all of it’s possibilities and functionalities and it’s really useful for sharing information around whole organization. We have a lot of different Workplace groups with different type of content. One of those groups is related to technology and coding where we decided to share and create a post in Workplace on some action or change in one of ours Github repositories. For example get notifications on merges to the master branch and releases in Github and create a Workplace post with all necessary information. Then I thought this integration should be fairly simple and I just need a place to run my code. And that’s where the Azure Functions came into place. 

In this post I will try to show the easiest way to integrate these two systems and share my code and findings in one place. So, don’t expect anything fancy because this is still work in progress and it will definitely grow in functionality. As an easy example I decided to use Github trigger on creation of a new issue in the Repo. That being said first thing I had to do is to create a function which I need to expose and use the Azure function URL to create a webhook in the desired Github repository. I used C# for development of the integration.

*If you are just starting with Azure Functions with Java and you came upon this blog post this what I have followed and installed and eventually used and tested successfully the quickstart of: *
*[Use Java and Maven to create and publish a function to Azure.](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-java-maven)"*

*Then if you want to use C# then you need the latest version of the [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) form which you have all the support needed to develop and deploy in Azure Functions and much more.*

Ok lets start with the code of the Azure function written in C# then a setup of the Github webhook and setup of the Workplace Bot

# 1. Azure Function code

```csharp
namespace GithubWorkplaceAzure
{
    public static class GithubWebhookWorkplaceFunction
    {
        [FunctionName("GithubWebhookWorkplaceFunction")]
        public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)]HttpRequestMessage req, ILogger log)
        {          
            // Get request body
            dynamic data = await req.Content.ReadAsAsync<object>();

            // Extract github data from request
            string repoName = data?.repository?.name;
            string repoDescription = data?.repository?.description;
            string gitHubIssue = data?.issue?.title;
            string issueDescription = data?.issue?.body;
            string gitSender = data?.sender?.login;            
            string issueURL = data?.issue?.html_url;


            //Creating a new issue post content for Workplace
            string message = "New issue in repo: " + repoName + "\n" +
                             "Repo description: " + repoDescription + "\n" +
                             "\n" +
                             "Issue: " + gitHubIssue + "\n" +
                             "Comment: " + issueDescription + "\n" + "\n" +
                             "\n" +
                             "Sender: " + gitSender + "\n" +
                             "Issue url: " + issueURL;

			//Posting created message to the Workplace
            using (var httpClient = new HttpClient())
            {

                httpClient.BaseAddress = new Uri("https://graph.facebook.com/");

                var parametters = new Dictionary<string, string>
                {
                    { "access_token", "[YOUR WORKPALCE TOKEN VALUE]" },
                    { "message", message }
                };                

                var encodedContent = new FormUrlEncodedContent(parametters);
                string groupId = "[YOUR WORKPLACE GROUPID]";

                var result = await httpClient.PostAsync($"{groupId}/feed", encodedContent);
                var responseStr = await result.Content.ReadAsStringAsync();                
                var msg = result.EnsureSuccessStatusCode();
                return req.CreateResponse(HttpStatusCode.OK, "Response Workplace: " + msg);
            }
            
        }
    }
}
```

# 2. Github webhook

It’s fairly simple to create a Github webhook, choose your desired repository and create a trigger on which actions you want the Azure function to be called. In my case I created a github webhook when a new issue is created. This is how it looks in my repo:

![](/images/2020-02-12-Github-integration-with-Workplace-by-Facebook-via-Azure-Functions/GithubWebhookSetup_1.jpg)

![](/images/2020-02-12-Github-integration-with-Workplace-by-Facebook-via-Azure-Functions/GithubWebhookSetup_2.jpg)

I haven’t created a secret key in the Azure functions yet, but that is one of my next steps. This was a quick PoC where I wanted to test posability of the integration between Github and Workplace by Facebook.


# 3. Workplace setup 

I will not go into too much detail here only to explain which info we need from the Workplace bot so we could actually post the content on the desired Workplace group. This is also related if you are using Facebook groups, fun pages or just your Facebook wall. 

1. First thing you will need is your Group ID. This you can find directly from the link of the group itself. In the code replace the text: “[YOUR WORKPLACE GROUPID]” with your actual GrouId value. 
2. Next is to retrieve the Token value of the Workplace/Facebook bot. In the code replace the text “[YOUR WORKPALCE TOKEN VALUE]” with the actual token value.
3. And don’t forget that Workplace and Facebook are using the Graph API for all the third party communication and integration via link: <https://graph.facebook.com/>

More about the Facbook bots you can read here <https://developers.facebook.com/docs/workplace/bots>


# 4. Testing the integration

As a simple test here let’s create an issue in our repo:

![](/images/2020-02-12-Github-integration-with-Workplace-by-Facebook-via-Azure-Functions/GithubIssue.jpg)


As soon as we post the issue we can check the workplace group wall for the new Github issue post.

![](/images/2020-02-12-Github-integration-with-Workplace-by-Facebook-via-Azure-Functions/WorkplacePostInGroup.jpg)
