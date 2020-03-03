<!-- markdownlint-disable MD002 MD041 -->

This tutorial shows you how to add user profile features to an ASP.NET MVC web app using Microsoft Graph. Microsoft Graph provides information about the signed-in user from Azure Active Directory, Exchange Online, and the user's Office profile. You can use this information to personalize the user's experience in your app. For example, you might use the user's profile photo as an avatar, or display dates and times in the user's preferred time zone. You can also allow the user to update their profile.

For a complete code sample that includes the code from this tutorial, see [this](https://github.com/microsoftgraph/aspnet-mvc-sample).

In this tutorial, you learn how to:

> [!div class="checklist"]
>
> - Get and display the user's profile photo
> - Get and display the user's profile information
> - Get and display the user's mailbox settings
> - Request additional permissions scopes after sign-in
> - Update the user's profile photo
> - Update the user's profile information

If you don't have a Microsoft 365 subscription, join the [Office 365 developer program](https://developer.microsoft.com/office/dev-program) to get a free subscription.

## Prerequisites

- Visual Studio 2019
- Completed project from [Build ASP.NET MVC apps with Microsoft Graph](https://docs.microsoft.com/graph/tutorials/aspnet)
- A Microsoft 365 subscription
