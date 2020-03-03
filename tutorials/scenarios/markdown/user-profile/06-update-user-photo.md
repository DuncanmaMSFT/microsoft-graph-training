<!-- markdownlint-disable MD002 MD041 -->

Now let's add the ability to update the  user's photo.

1. Open **Helpers/GraphHelper.cs** and add the following function to the `GraphHelper` class.

    ```csharp
    public static async Task UpdateUserProfilePhotoAsync(Stream photoStream)
    {
        var graphClient = GetAuthenticatedClient();

        // Update the photo
        await graphClient.Me.Photo.Content
            .Request()
            .PutAsync(photoStream);

        var tokenStore = new SessionTokenStore(null,
            HttpContext.Current, ClaimsPrincipal.Current);

        var cachedUser = tokenStore.GetUserDetails();

        // Get the avatar-sized photo and save
        // it in the cache
        cachedUser.Avatar = await GetUserPhotoAsDataUriAsync(graphClient, "48x48");
        tokenStore.SaveUserDetails(cachedUser);
    }
    ```

1. Open **Controllers/ProfileController.cs** and add the following function to the `ProfileController` class.

    ```csharp
    // POST: Profile/Update
    [Authorize]
    [HttpPost]
    public async Task<ActionResult> UpdatePhoto(HttpPostedFileBase profilePhoto)
    {
        await GraphHelper.UpdateUserProfilePhotoAsync(profilePhoto.InputStream);

        return RedirectToAction("Index");
    }
    ```

1. Save all of your changes, then select **Debug** > **Start Debugging** or press **F5** to run the application.
1. Sign into the application using the **Click here to sign in** button or the **Sign In** navigation bar link.
1. Select the user's profile photo, then select **My Profile** in the drop-down menu.
1. Use the **Choose new photo** input to update the user's photo.
