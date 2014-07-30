# SharpBucket
SharpBucket is a .Net wrapper for the BitBucket's REST API. It is written in in C#. With it you can have all the data of your repositories / issues at your fingertips.

## How to get started
### Installation
To install SharpBucket, run the following command in the Package Manager Console:

    PM> Install-Package SharpBucket

### Usage
See the [Console Test Project][1] to see how to use the wrapper. Here's just a brief demo:

First lets set your entry point to the API
```CSharp
// your main entry to the BitBucket API, this one is for V1
var sharpBucket = new SharpBucketV1();
// authenticate with OAuth keys
sharpBucket.OAuth2LeggedAuthentication(ConsumerKey, ConsumerSecretKey);
```

There are various end points you can use. Lets take a look at User end point:
```CSharp
var user = sharpBucket.User();
var info = user.GetInfo();
var privileges = user.GetPrivileges();
var follows = user.ListFollows();
var userRepos = user.ListRepositories();
```

Similarly for the Issues end point:

```CSharp
var issues = sharpBucket.Repository(accountName, repository).Issues();
var issues = issues.ListIssues();
var issueComments = issues.ListIssueComments(ISSUE_ID);
```
Sending information is just as easy.

```CSharp
var newIssue = new Issue{Title = "I have this little bug", 
                         content = "that is really annoying",
                         status = "new"};
var newIssueResult = issues.PostIssue(newIssue);
```

SharpBucket uses a strict naming convention:
- methods starting with List will return a collection of items (ListIssues() returns a list of issues)
- methods starting with Get will return an item (GetIssue(10) will return an issue with the id 10)
- methods starting with Put are used for updating the objects
- methods starting with Delete will delete the objects

### End points

We might add aditional endpoints if needed. Current ones represent the BitBucket end points. But in some cases it might be better to introduce new ones. Compare the current implementation with a possible upgrade. Existing one:
```CSharp
var newComment = new Comment{content = "This bug is really annoying!"};
// Issues endpoint needs to know the id of the issue we want to comment
var newCommentResult = issues.PostIssueComment(ISSUE_ID, newComment);
```
Maybe this would be better:
```CSharp
var newComment = new Comment{content = "This bug is really annoying!"};
var issue = issues.GetIssue(ISSUE_ID);
var newCommentResult = issue.PostComment(newComment);
```
## Authentication
There are two ways you can authenticate with SharpBucket
- via the Oauth 1.0a, which is preferred
- via BitBucket's username and password

Here is how you can use them:
### Basic authentication
```CSharp
// authenticate with username and password
sharpBucket.BasicAuthentication(email, password);
```

### OAuth authentication
With OAuth you can choose between [2 legged and 3 legged authentication](http://cakebaker.42dh.com/2011/01/10/2-legged-vs-3-legged-oauth/).

**Two legged** is as simple as basic authentication:
```CSharp
// authenticate with OAuth keys
sharpBucket.OAuth2LeggedAuthentication(ConsumerKey, ConsumerSecretKey);
```
**The three legged** one requires an additional step for getting the pin / verifier from the server. If you dont supply a callback url (or use "oob") you will get an bitbucket url that will contain your pin / verifier. Here is a simple example of how you could manually copy paste the pin from the browser:
```CSharp
var authenticator = sharpBucket.OAuth3LeggedAuthentication(ConsumerKey, ConsumerSecretKey, "oob");
var uri = authenticator.StartAuthentication();
Process.Start(uri);
var pin = Console.ReadLine();
// we can now do the final step by using the pin to get our access tokens
authenticator.AuthenticateWithPin(pin);
```
If you had a server waiting from BitBucket's response, you would simply use your server's url as the callback and then wait for BitBucket to send you the pin to that address.

## How much of the API is covered?
While a complete coverage of the API is preferred SharpBucket currently does not support everything yet. But the main functionality is [covered](https://github.com/MitjaBezensek/SharpBucket/blob/master/Coverage.md) and the rest should also get covered sooner or later.

## Contributing
Contributions are always welcome. You can check the issues or look which api calls [are still missing](https://github.com/MitjaBezensek/SharpBucket/blob/master/Coverage.md). A few topics that still need to be covered:
- V2 of the api
- async calls?
- paging
- logo!

## Continuous Integration from AppVeyor
The project is using [AppVeyor's](http://www.appveyor.com/) Continuous Integration
Service that is free for open source projects. It is enabled for Pull Requests as well as the main branch. Main branch's current status is:

[![Build status](https://ci.appveyor.com/api/projects/status/jtlni3j2fq3j6pxy/branch/master)](https://ci.appveyor.com/project/MitjaBezenek/sharpbucket/branch/master)

## Licensing
SharpBucket is licensed with MIT license. It uses NServiceKit for requesting and parsing the data. NServiceKit is also [licensed under MIT](https://github.com/NServiceKit/NServiceKit.Text/blob/master/LICENSE) terms and is an open source fork of StackService. SharpBucket was influenced by ServiceStack's [Stripe api wrapper](https://github.com/ServiceStack/Stripe). It also uses Google's [OAuth protocol](https://code.google.com/p/oauth/), that is licensed under Apache License.


  [1]: https://github.com/MitjaBezensek/SharpBucket/blob/master/ConsoleTests/Program.cs