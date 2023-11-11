

##  Chapter Overview
This module reviews the constructs that allow Cloud Foundry to securely and effectively support multiple users, projects, and tenants.

 - Orgs and spaces are used to divide up a Cloud Foundry instance so  that it can be used by multiple teams, projects, tenants, etc.
 - Role-based access control (RBAC) is used to control what you can do   in Cloud Foundry.
 - Resource names provide human-readable labels to resources in Cloud  Foundry.


##  Orgs and Spaces
Organizations (orgs) and spaces are logical separations within a Cloud Foundry instance. Spaces live within orgs, and a single org can contain one or more spaces.

Orgs are often used to separate tenants or projects. For example, you may want to divide your Cloud Foundry instance into separate orgs for different areas of your business. Within an org you might have separate spaces for different lifecycle stages, like development, staging, and production.

While you're required to use orgs and spaces in Cloud Foundry, how you use them is up to you.

##  Targeting
To begin working with an org and/or space in Cloud Foundry, you must first target it. Targeting directs CLI commands to a specific org and space. The cf target command is used to change your target. It can also be used to show the org and space you are currently targeting (if any). You can see your current target by running:

```bash
cf target
```

You can change the org and space you are using by:

```bash
cf target -o <THE_ORG> -s <THE_SPACE>
```

You don't need to do this now. For the time being, we will remain in the default org and space.

##  Scope and Names
In the previous section, we deployed an application to a space. Applications are always deployed to a space, and a space is always scoped to an org (you cannot push an app directly to an org). As you will see later in the course, other elements, including service instances and routes, are also scoped to spaces. As such, you need to target a space before you can begin managing that space's resources.

When you deployed the app using cf push, you gave it a name (via the manifest) of first-push. Naming the app meant you could easily refer to it in Cloud Foundry. While names can be used as defaults by some commands (for example, in creating a route for your first-push app), they exist primarily for humans. We discuss resources names in more depth later in this module.

You can list details of your app by using its name:

cf app first-push

Because apps are scoped to a space, app names need to be unique in that space. However, they do not need to be unique outside the space. You could therefore have an app named first-push in development, staging, and production spaces. Other users may have their own app called first-push in their spaces as well.

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b09b02686e740a3845b2d77e643781f.png)
##  Role-Based Access Control (RBAC)
Cloud Foundry leverages Role-Based Access Control (RBAC) to restrict user actions that affect resources within the platform. Users can be assigned to roles globally (Cloud Foundry wide) or can be assigned to roles in specific orgs and spaces. Roles control what you can do and where you can do it.

##  Global Roles
Users can be assigned global roles and capabilities that span an entire Cloud Foundry deployment. As a developer using Cloud Foundry, it is unlikely you will be assigned one of these roles.

Click on each card to flip it and learn more about each global role.

 - Admin
Allows a user to perform operational actions on all orgs and spaces.
 - Admin-read-only
Allows visibility of all orgs and spaces without the ability to modify resources.
-  global auditor
Similar to the Admin Read-Only role, except that this role cannot see secrets such as environment variable content.

##  Org Roles
Org roles grant user access at the Org level.

Click on each card to flip it and learn more about each org role.

 - OrgManager
Can administer the org. OrgManagers can create/modify/delete spaces, org-level roles, domains, etc., in that org.
 - OrgAuditor
 Have read-only access to the org.
 - BillingManager
 Billing managers can create and manage billing account and payment information associated with an org in Cloud Foundry instances that have deployed the billing engine.

You can see the users assigned to these roles for an org via:

```bash
cf org-users <ORG>
```

OrgManagers can manage users in your org with:

```bash
cf set-org-role
```

and

```bash
cf unset-org-role
```

##  Space Roles
Space roles grant user access at the space level.

Click on each card to flip it and learn more about space roles.

 - Space Manager
 Space managers can administer roles for a space.
 - Space Devoper
 Can manage apps, services, and routes in a space. A user must have the SpaceDeveloper role to push apps.
 - Space Auditor
 Space auditors have read-only access to a space.

You can see the users assigned to roles in a space with the command:

```bash
cf space-users <ORG> <SPACE>
```

Org roles do not cascade into spaces. For example, an OrgManager cannot deploy apps to spaces in their org. However, they can grant the SpaceDeveloper role to a user (including themselves) for a particular space.

SpaceManagers can manage users in your space with:

```bash
cf set-space-role
```

and

```bash
cf unset-space-role
```

You can also see all users assigned to org roles and space roles by running:

```bash
cf org-users <ORG> -a
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/183c2211bf22490c934d28889a6d7e5c.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/9848e26bf63140e5b54c74f79525fb25.png)
##  Resource Names
You may have already noticed that the objects you create in Cloud Foundry have resource names assigned. For example, we ran cf push in a previous section, and we named the application first-push (defined in the app's manifest).

Resource names are important. You'll often refer to them when performing actions with the CLI, and in some cases, names are set according to a default. Think about the first-push example again. The app manifest includes random_route: true. This directive instructs Cloud Foundry to create a random route for our app using the resource name. If the random-route directive had not been set, a default route would have been created based on the resource name we chose i.e. http://first-push.<mydomain.io>.

Most resource names you encounter as a developer are scoped to a space. This means you can have apps with the same name in different spaces. For example, you might use the same name for apps deployed across the spaces development, staging, and production.

You'll need to be wary of routing collisions where app names are duplicated across spaces, but we'll discuss that later on.

##  GUIDs
Along with resource names, objects created in Cloud Foundry are assigned a globally unique identifier (GUID). Like resource names, an object's GUID will sometimes be passed to CLI commands. Unlike resource names, guids are globally unique.

##  Renaming
You can rename certain objects in Cloud Foundry. When an object is renamed, the GUID does not change.

Let's test this out on the first-push app. Before we do, let's look at its current GUID:

```bash
cf app first-push --guid
```

Now, let's try renaming it:

```bash
cf rename first-push renamed-app
```

Once this change succeeds, if you run cf app renamed-app --guid, you will notice that the app GUID is the same (and so is the app route). The route can be manually updated, and we'll look at mapping and unmapping routes later in the course.

##  Tidy Up
You can go ahead and delete the renamed-app now - we won't be needing it for the rest of this course.

cf delete -r renamed-app

The -r flag tells Cloud Foundry to also delete the route. We will discuss routes more in an upcoming section.

![在这里插入图片描述](https://img-blog.csdnimg.cn/85f36c0c40eb4f87bde09eb4189293f2.png)

