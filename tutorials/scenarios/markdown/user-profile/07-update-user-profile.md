<!-- markdownlint-disable MD002 MD041 -->

Now let's add the ability to update the user's profile information.

1. Open **Helpers/GraphHelper.cs** and add the following function to the `GraphHelper` class.

    ```csharp
    public static async Task UpdateUserProfileAsync(User userProfile)
    {
        var graphClient = GetAuthenticatedClient();

        await graphClient.Me.Request().UpdateAsync(userProfile);
    }
    ```

1. In the **Solution Explorer** in Visual Studio, select the **Models** folder.
1. Open the context menu by right-clicking the **Models** folder or pressing **SHIFT** + **F10**.
1. Select **Add** > **Class...**.
1. Name the class `ProfileUpdate` and choose **Add**.
1. Open **ProfileUpdate.cs** and replace its contents with the following.

    ```csharp
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Linq;

    namespace graph_tutorial.Models
    {
        public class ProfileUpdate
        {
            public string MobilePhone { get; set; }
            public string OriginalMobilePhone { get; set; }
            public string PreferredName { get; set; }
            public string OriginalPreferredName { get; set; }
            public DateTime Birthday { get; set; }
            public DateTime OriginalBirthday { get; set; }
            public string MySite { get; set; }
            public string OriginalMySite { get; set; }
            public string AboutMe { get; set; }
            public string OriginalAboutMe { get; set; }
            public string[] Schools { get; set; }
            public string OriginalSchools { get; set; }
            public string[] Skills { get; set; }
            public string OriginalSkills { get; set; }
            public string[] Interests { get; set; }
            public string OriginalInterests { get; set; }

            public User GetUserForUpdate()
            {

                var updateUser = new User
                {
                    AboutMe = string.Compare(OriginalAboutMe, AboutMe) == 0 ? null : AboutMe,
                    MySite = string.Compare(OriginalMySite, MySite) == 0 ? null : MySite,
                    PreferredName = string.Compare(OriginalPreferredName, PreferredName) == 0 ? null : PreferredName,
                    Interests = IsListModified(OriginalInterests, Interests) ? Interests : null,
                    Schools = IsListModified(OriginalSchools, Schools) ? Schools : null,
                    Skills = IsListModified(OriginalSkills, Skills) ? Skills : null,

                    ODataType = null
                };

                if (DateTime.Compare(OriginalBirthday, Birthday) != 0)
                {
                    updateUser.Birthday = Birthday;
                }

                if (updateUser.AboutMe == null &&
                    updateUser.Birthday == null &&
                    updateUser.MySite == null &&
                    updateUser.PreferredName == null &&
                    updateUser.Interests == null &&
                    updateUser.Schools == null &&
                    updateUser.Skills == null)
                {
                    return null;
                }

                return updateUser;
            }

            // Currently you cannot update mobilePhone in the same request
            // as the other properties
            public User GetUserForMobilePhoneUpdate()
            {
                if (string.Compare(OriginalMobilePhone, MobilePhone) != 0)
                {
                    return new User { MobilePhone = MobilePhone };
                }

                return null;
            }

            private bool IsListModified(string original, IEnumerable<string> updated)
            {
                var originalList = original.Split(';');

                if (originalList.Length != updated.Count())
                    return true;

                // If the same length, there must be something in updated
                // that is not in original
                var changes = updated.Except(originalList);

                return changes.Count() > 0;
            }
        }
    }
    ```

1. Open **Controllers/ProfileController.cs** and add the following function to the `ProfileController` class.

    ```csharp
    // POST: Profile/Update
    [Authorize]
    [HttpPost]
    public async Task<ActionResult> Update(ProfileUpdate profileUpdate)
    {
        var updateUser = profileUpdate.GetUserForUpdate();

        if (updateUser != null)
        {
            await GraphHelper.UpdateUserProfileAsync(updateUser);
        }

        var updatePhoneUser = profileUpdate.GetUserForMobilePhoneUpdate();
        if (updatePhoneUser != null)
        {
            await GraphHelper.UpdateUserProfileAsync(updatePhoneUser);
        }

        return RedirectToAction("Index");
    }
    ```

## Add JavaScript for handling list inputs

1. In the **Solution Explorer** in Visual Studio, select the **Scripts** folder.
1. Open the context menu by right-clicking the **Scripts** folder or pressing **SHIFT** + **F10**.
1. Select **Add** > **JavaScript File**.
1. Name the file `FormInput` and choose **OK**.
1. Open **FormInput.js** and add the following code.

    ```javascript
    // Utility function to escape user input
    function escapeUserInput(text) {
        var textarea = document.createElement('textarea');
        textarea.textContent = text;
        return textarea.innerHTML;
    }

    // Initialize Bootstrap custom file input
    $(document).ready(function () {
        bsCustomFileInput.init();
    });
    ```

1. In the **Solution Explorer** in Visual Studio, select the **Scripts** folder.
1. Open the context menu by right-clicking the **Scripts** folder or pressing **SHIFT** + **F10**.
1. Select **Add** > **JavaScript File**.
1. Name the file `FormInput` and choose **OK**.
1. Open **ModifiableList.js** and add the following code.

    ```javascript
    $(document).ready(function () {
        // Method to take the value of the input, add it to the list,
        // then clear the input.
        $('.new-item-input').bind('addItemAndClear', function (e) {
            // Get the list group
            let newItemRaw = $(this).val();
            let newItem = escapeUserInput(newItemRaw);

            if (newItem && newItem.length > 0) {
                let name = $(this).attr('data-name');
                var newInput = $(`<input type="text" class="form-control existing-item-input" name="${name}" value="${newItem}" />`);
                var removeButton = $('<div class="input-group-append"><button type="button" class="btn btn-outline-secondary remove-button"><span>&times;</span></button></div>');
                removeButton.click(removeItemFromList);
                var newInputGroup = $('<div class="input-group mb-2"></div>');

                newInputGroup.append(newInput, removeButton);

                $(this).parent().before(newInputGroup);

                // Clear the input
                $(this).val('');
            }
        });

        // Remove item when 'x' is clicked
        $('.remove-button').click(removeItemFromList);

        // Add item if '+' is clicked
        $('.add-new-item').click(function () {
            let input = $(this).closest('.input-group').children('.new-item-input');
            input.trigger('addItemAndClear');
        });

        // Prevent form submission if enter is pressed
        // in an existing item
        $('.existing-item-input').keypress(function (e) {
            if (e.keyCode === 13) { e.preventDefault(); }
        });

        // Prevent form submission if enter is pressed
        // in the new item input
        $('.new-item-input').keypress(function (e) {
            if (e.keyCode === 13) { e.preventDefault(); }
        });

        // When enter is released, add the new value to
        // the list.
        $('.new-item-input').keyup(function (e) {
            if (e.keyCode === 13) {
                $(this).trigger('addItemAndClear');
                e.preventDefault();
            }
        });
    });

    function removeItemFromList() {
        $(this).closest('.input-group').remove();
    }
    ```

1. Save all of your changes, then select **Debug** > **Start Debugging** or press **F5** to run the application.
1. Sign into the application using the **Click here to sign in** button or the **Sign In** navigation bar link.
1. Select the user's profile photo, then select **My Profile** in the drop-down menu.
1. Change the value of any of the editable fields and choose **Update my info**. The page should reload with the new value.
