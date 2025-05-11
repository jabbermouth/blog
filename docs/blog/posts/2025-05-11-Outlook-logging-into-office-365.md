---
title: Outlook logging into Office 365 Error
description: Fixing the Outlook error 4usqa / 3399614475 when trying to log in to Office 365 from Outlook
date: 2025-05-11
categories:
  - Outlook
  - Office 365
  - Email
---
# Outlook logging into Office 365 Error

If you get an error 4usqa / 3399614475 when trying to log in to Office 365 from Outlook (desktop or mobile) all of a sudden as especially if it's multiple users, try the following:

1. Go to the Entra home page: https://entra.microsoft.com/
2. Sign in as a tenant admin
3. Search for `Microsoft Information Protection API`
4. Go to "Properties"
5. Select `Yes` for "Enabled for users to sign in?"

The solution was found on [this Microsoft answers page](https://answers.microsoft.com/en-us/outlook_com/forum/all/i-am-getting-this-error-message-non-stop-in/91f30cab-7a91-48ed-9c74-8c8de8b565ea?page=2).