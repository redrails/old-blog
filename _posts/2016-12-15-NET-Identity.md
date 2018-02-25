---
title: ".NET Identity, Roles and Permissions"
categories:
  - tutorials
tags:
  - C#
  - .NET
  - ASP
  - web
header:
  overlay_image: http://www.zenesys.com/media/1068/aspnet.png
  overlay_filter: 0.4
  caption: ""
---

With my unbelievably bad experience so far with [Microsoft ASP.NET Identity 2.0](https://msdn.microsoft.com/en-us/library/mt173608(v=vs.108).aspx) I figured that I would write a small post giving a little bit of an insight as to what I've been doing in regards to setting up users, roles and permissions.

### Users

Identity provides an implementation for `IUser` which is the interface provided by Entity Framework. So in essence, the Identity classes extend the EF interfaces to give you a template and a head-start without having to go so low level yourself. Identity provides an implementation for most things like:

- `IdentityUser` 
- `IdentityRole`
- `IdentityUserRole`
- `IdentityUserStore`
- `IdentityRoleStore`
- ...

This is good because it allows for us to build on top of a nicely built stack but if you're looking to add other features like "role based authorization" then you're going to have a hard time just like I did. A very painful one at that. 

### Roles vs. Permissions

In simple terms, a role gives a user some privileges on a system. A role is therefore a collection of some permissions, but a lot of people don't see it that way to begin with. For example, an "Admin" can be deemed a role. But what's essentially behind "Admin" are permissions to access different parts of the application. So it doesn't make sense to use "Role" as a way to outline the scope of access for anything.

So what are are looking to do is something like the following (credit: : lhotka.net)
![](https://i.imgur.com/B41uNuP.png)

### Implementation

So, given this; the best thing to do is to create a `Permission` class that we will assign to roles, and then check for permissions when we reach different endpoints of the application. This is fairly simple and to keep it that way I'll just demonstrate it using a single class of constants.

```
namespace Project
{
    public class Permissions
    { 
        public const string AccessToPage1= "AccessToPage1";
        public const string AccessToPage2= "AccessToPage2";
    }
}
```

This is simple, now we need to bind these permissions to roles. This is the tricky part because roles are already implemented by [IdentityRole](https://msdn.microsoft.com/en-us/library/dn613249(v=vs.108).aspx).

So we need to override this and create out own. For my purposes I overrided all of the classes that are provided by Identity such as `IdentityUser`, `IdentityRole`, `IdentityUserRole` etc. Including the `Store` repositories because it makes it easier to work with it all.

`ApplicationRole.cs`
```
public class ApplicationRole : IdentityRole<string, ApplicationUserRole>
{
    public ApplicationRole() : base()
    {
        Id = Guid.NewGuid().ToString();
    }

    public ApplicationRole(string name, string description)
        : this()
    {
        this.Name = name;
        this.Description = description;
    }
    public string Description { get; set; }
    public virtual ICollection<RolePermission> RolePermissions { get; set; }
}
```

So we just create a simple property in `ApplicationRole` called `RolePermission` which holds a collection of `Permission`s we specify. And we create a `RolePermissions` class to do the following:

`RolePermissions.cs`
```
[Table("AspNetRolePermissions")]
public class RolePermission : ModelsBase
{
    public virtual ApplicationRole Role { get; set; }
    public string Permission { get; set; }
}
```

From here, we can now put `Permission` objects into our `Role` objects, so we can have something like the following:

![](https://i.imgur.com/CabYoMk.png)

To use these permissions we can create an [Attribute](https://msdn.microsoft.com/en-us/library/system.web.mvc.authorizeattribute(v=vs.118).aspx) that will verify if a User has a Permission to allow access to a mapping.

`HasPermissionAttribute.cs`
```
 public class HasPermissionAttribute : AuthorizeAttribute
    {
        private string _permission;

        public HasPermissionAttribute(string permission)
        {
            this._permission = permission;
        }

        public override void OnAuthorization(AuthorizationContext filterContext)
        {
            HasPermissionResult hasPermissionResult = HasPermission();
            var url = new UrlHelper(filterContext.RequestContext);
            switch (hasPermissionResult)
            {
                case HasPermissionResult.NotLoggedIn:
                    var routeValues = new RouteValueDictionary();
                    routeValues.Add("Action", "Login");
                    routeValues.Add("Controller", "Account");
                    routeValues.Add("returnUrl", filterContext.RequestContext.HttpContext.Request.RawUrl);
                    filterContext.Result = new RedirectToRouteResult(routeValues); 
                    break;
                case HasPermissionResult.DoesNotHavePermission:
                    HandleUnauthorizedRequest(filterContext);
                    break;
            }
        }
        
        protected override void HandleUnauthorizedRequest(AuthorizationContext filterContext)
        {
            var url = new UrlHelper(filterContext.RequestContext);
            var routeValues = new RouteValueDictionary();
            routeValues.Add("Action", "Unauthorized");
            routeValues.Add("Controller", "Account");
            routeValues.Add("returnUrl", filterContext.RequestContext.HttpContext.Request.RawUrl);
            filterContext.HttpContext.Response.StatusCode = 403;
            filterContext.Result = new RedirectToRouteResult(routeValues);
        }

        public virtual HasPermissionResult HasPermission()
        {
            var context = HttpContext.Current.GetOwinContext().Get<ApplicationDbContext>();

            var currentUserName = HttpContext.Current.User?.Identity.Name;

            if (string.IsNullOrEmpty(currentUserName))
            {
                return HasPermissionResult.NotLoggedIn;
            }

            ApplicationUser user = context.Users.Single(u => u.UserName == currentUserName);

            var userRoles = user.Roles;
                                
            foreach (var userRole in userRoles)
            {
                var role = userRole.Role;
                if (role.RolePermissions?.Count(r => r.Permission == _permission) >= 1)
                {
                    return HasPermissionResult.HasPermission;
                }
            }
            return HasPermissionResult.DoesNotHavePermission;
        }
    }

    public enum HasPermissionResult
    {
        NotLoggedIn,
        DoesNotHavePermission,
        HasPermission
    }
```

Then we can annotate our controllers like so:

```
[HasPermission(Permissions.AccessToPage1)]
public ActionResult Index()
{
    return View();
}
```
