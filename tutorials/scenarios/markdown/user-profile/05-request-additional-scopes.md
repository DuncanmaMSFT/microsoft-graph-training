<!-- markdownlint-disable MD002 MD041 -->

Next, let's add the ability to request additional Microsoft Graph permission scopes as we need them. Currently, the application only requests `User.Read`, which does not allow the application to update the user's profile. We could add `User.ReadWrite` to the set of permissions requested when the user first signs in, but if the user never tries to update their profile, the application would have more permissions than it needs. Instead, we'll ask the user to grant the additional permission if they choose.

1. Open **Helpers/GraphHelper.cs** and add the following function to the `GraphHelper` class. This function will check the existing access token for required permissions, and return an authorization URL if the user needs to consent.

    ```csharp
    public static async Task<Uri> GetConsentUriForScopesIfNeeded(string[] scopes, string redirect)
    {
        // Combine the requested scopes with the default set of scopes
        // requested at sign in
        var combinedScopes = graphScopes.Union(scopes);

        // Create an MSAL client and token cache
        var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                        .WithRedirectUri(redirectUri)
                        .WithClientSecret(appSecret)
                        .Build();

        var tokenStore = new SessionTokenStore(idClient.UserTokenCache,
            HttpContext.Current, ClaimsPrincipal.Current);

        var accounts = await idClient.GetAccountsAsync();

        try
        {
            // See if there is a token in the cache that has all of the required scopes
            // If so, the user has already granted the permission we need
            var result = await idClient
                .AcquireTokenSilent(combinedScopes, accounts.FirstOrDefault())
                .ExecuteAsync();

            return null;
        }
        catch (MsalUiRequiredException)
        {
            // This exception indicates that the user needs to consent
            // to one or more of the required scopes.

            // Save the page the user is on into the state parameter
            var stateParam = new Dictionary<string, string>();
            stateParam.Add("state", redirect);

            // Build the authorization URL
            var uri = await idClient.GetAuthorizationRequestUrl(scopes)
                .WithAccount(accounts.FirstOrDefault())
                .WithRedirectUri($"{redirectUri}Account/Consent")
                .WithExtraQueryParameters(stateParam)
                .ExecuteAsync();

            // Add the "prompt=consent" query parameter
            var queryParams = HttpUtility.ParseQueryString(uri.Query);
            queryParams["prompt"] = "consent";

            var builder = new UriBuilder(uri);

            builder.Query = queryParams.ToString();
            return builder.Uri;
        }
    }
    ```

1. Add the following function to the `GraphHelper` class to redeem an authorization code for an access token.

    ```csharp
    public static async Task RedeemCodeForAdditionalConsent(string code)
    {
        // Create the MSAL client with a special redirect
        var idClient = ConfidentialClientApplicationBuilder.Create(appId)
                        .WithRedirectUri($"{redirectUri}Account/Consent")
                        .WithClientSecret(appSecret)
                        .Build();

        var tokenStore = new SessionTokenStore(idClient.UserTokenCache,
            HttpContext.Current, ClaimsPrincipal.Current);

        // Exchange the code for a token
        var result = await idClient
            .AcquireTokenByAuthorizationCode(graphScopes, code)
            .ExecuteAsync();
    }
    ```

1. Open **Controllers/AccountController.cs** and add the following `using` statements at the top of the file.

    ```csharp
    using graph_tutorial.Helpers;
    using System.Threading.Tasks;
    ```

1. Add the following function to the `AccountController` class.

    ```csharp
    public async Task<ActionResult> Consent(string code, string state, string error, string error_description)
    {
        if (string.IsNullOrEmpty(code) || string.IsNullOrEmpty(state))
        {
            return RedirectToAction("Error", "Home", new
            {
                message = "Error getting consent for additional permissions",
                debug = $"Error: {(string.IsNullOrEmpty(error) ? "Unknown" : error)}\nDescription: {(string.IsNullOrEmpty(error_description) ? "None" : error_description)}"
            });
        }

        await GraphHelper.RedeemCodeForAdditionalConsent(code);

        return Redirect(state);
    }
    ```

1. Open **Controllers/ProfileController.cs** and locate the `Index` function. Replace the `ViewBag.ConsentUri = string.Empty;` line with the following code.

    ```csharp
    ViewBag.ConsentUri = await GraphHelper.GetConsentUriForScopesIfNeeded(
        new string[] { "User.ReadWrite" },
        "/Profile");
    ```

1. Save all of your changes, then select **Debug** > **Start Debugging** or press **F5** to run the application.
1. Sign into the application using the **Click here to sign in** button or the **Sign In** navigation bar link.
1. Select the user's profile photo, then select **My Profile** in the drop-down menu. Locate the **We don't have permission to update your profile. Click here to grant permission** text on the page, and follow the link to grant permission.

![A screenshot of the prompt to grant write permission](../../images/05-user-profile.png)

After granting the permission to read and update the user's profile, the browser returns to the profile page.
