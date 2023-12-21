# Red Hat Developer Hub with Azure Repos

This is a simple guide to get started with **Red Hat Developer Hub** and:

* Azure AD (Or Entra ID as it's now called...) for login.
* A private Azure DevOps org to hold your GPTs, and
* The ability to create new private Azure Repos with Red Hat Developer Hub.

## Getting Started

This guide is based on Red Hat Developer Hub 1.0.  To set this up for yourself you will need:

1. An OpenShift cluster (it's possible OpenShift Local will work, but I haven't tried this myself).
2. An Azure account with the ability to create a new App Registration (or re-use an existing one).
3. An Azure DevOps organization.

## Azure Details

First, create (or re-use) an App Registration.  You will need all the standard details of this App Registration later when configuring Red Hat Developer Hub, so make sure to note the following:

* AZURE_CLIENT_ID
* AZURE_CLIENT_SECRET
* AZURE_TENANT_ID

You will also need to set a callback URL to redirect a user back to Red Hat Developer Hub once they login.  The callback URL should match the following pattern that assumes you will install Red Hat Developer Hub in the namespace "rhdh":

```
https://developer-hub-rhdh.apps.<cluster url>/api/auth/microsoft/handler/frame
```

If you install Red Hat Developer Hub into a different namespace, then the URL pattern would be:

```
https://developer-hub-<namespace>.apps.<cluster url>/api/auth/microsoft/handler/frame
```

Of course, be sure to substitute the proper values before registering this callback url.

## Azure DevOps Details

In your Azure DevOps Project, click on the "Gear" icon at the top-right beside your account link.  This should be the **"User Settings"** options.

From this list, click on **Personal Access Tokens** and create a new Token.  Give it whatever name makes sense to you.  I used "rhdh-demo".

Make sure you have the correct Azure DevOps Organization selected and choose an appropriate expiration time.  Remember, you will need to update your Red Hat Developer Hub configuration with a new token once this one expires, so you might want to pick a longer expiration time.

For demo purposes, I gave my token "Full Access", however, I'm sure you can probably scope this down considerably.  I haven't tested this, but "Full Access" to just "Code" is likely good enough.  Basically, Red Hat Developer Hub needs the rights to be able to read, update, and create new repos.

Make sure to note the actual token that gets created, as you won't be able to look it up again later.

## OpenShift Preparation

First, create a new namespace for Red Hat Developer Hub.  I'm simply using "rhdh":

```
oc new-project rhdh
```

Next, update the values in `rhdh-secrets.yaml` under the `manifests` directory with your App Registration details and your new Personal Access Token.

Review the contents of `app-config-rhdh.yaml`.  You will need to update the link to the templates list (line 37) with the location of your own templates list.  A sample of templates will be added in a different repo shortly.

Apply these manifests to your namespace.

```
oc apply -k manifests -n rhdh
```

## Deploy Red Hat Developer Hub with Helm

In the OpenShift Developer View navigate to your `rhdh` project.

From here:

1. Select **+Add** From the left navigation menu.
2. Select "Helm Chart" (last entry in the "Developer Catalog" tile).
3. Search for "Red Hat Developer Hub" and click on that tile.
4. Click "Create".
5. Leave "Release name" as the default value of `developer-hub`.
6. Switch to the YAML view ("Configure via: YAML view" radio button).
7. Delete the default contents and paste in the contents of `values.yaml` in the `helm` directory of this repo.
8. Change the value of `clusterRouterBase` (line 5) to the apps url for your cluster (`apps.<your cluster domain>`).
9. Click "Create"

This will spin up a PostgreSQL StatefulSet as well as a Red Hat Developer Hub Deployment.  The process should only take a few minutes.

You're done!

## Using Red Hat Develoepr Hub

Once Red Hat Developer Hub is running, you can click on the route to launch it.  If you have configured your App Registration correctly and added the proper data to your secret, you should be able to login via Azure AD.

If you have provided a valid software templates link in your app config, when you click "Create" you should see your Golden Path Templates.

When you create a Golden Path Template instance, the new private repos should be created in your Azure DevOps project.

## Further Exploration

For more information and advanced configuration, please see the [Red Hat Developer Hub 1.0 documentation](https://access.redhat.com/documentation/en-us/red_hat_developer_hub/1.0).

