# ASPNET.Zero中后台管理创建的角色名，仅仅是显示名称

> 原

```C#
[AbpAuthorize(AppPermissions.Pages_Administration_Roles_Edit)]
protected virtual async Task UpdateRoleAsync(CreateOrUpdateRoleInput input)
{
    Debug.Assert(input.Role.Id != null, "input.Role.Id should be set.");

    var role = await _roleManager.GetRoleByIdAsync(input.Role.Id.Value);
    role.DisplayName = input.Role.DisplayName;
    role.IsDefault = input.Role.IsDefault;

    await UpdateGrantedPermissionsAsync(role, input.GrantedPermissionNames);
}

[AbpAuthorize(AppPermissions.Pages_Administration_Roles_Create)]
protected virtual async Task CreateRoleAsync(CreateOrUpdateRoleInput input)
{
    var role = new Role(AbpSession.TenantId, input.Role.DisplayName) { IsDefault = input.Role.IsDefault };
    CheckErrors(await _roleManager.CreateAsync(role));
    await CurrentUnitOfWork.SaveChangesAsync(); //It's done to get Id of the role.
    await UpdateGrantedPermissionsAsync(role, input.GrantedPermissionNames);
}
```

> 更改后

```C#
[AbpAuthorize(AppPermissions.Pages_Administration_Roles_Edit)]
protected virtual async Task UpdateRoleAsync(CreateOrUpdateRoleInput input)
{
    Debug.Assert(input.Role.Id != null, "input.Role.Id should be set.");

    var role = await _roleManager.GetRoleByIdAsync(input.Role.Id.Value);
    role.DisplayName = input.Role.DisplayName;
    //添加role.Name
    role.Name = role.DisplayName;
    role.IsDefault = input.Role.IsDefault;

    await UpdateGrantedPermissionsAsync(role, input.GrantedPermissionNames);
}

[AbpAuthorize(AppPermissions.Pages_Administration_Roles_Create)]
protected virtual async Task CreateRoleAsync(CreateOrUpdateRoleInput input)
{
    //使用其重载方法
    var role = new Role(AbpSession.TenantId, input.Role.DisplayName, input.Role.DisplayName) { IsDefault = input.Role.IsDefault };
    CheckErrors(await _roleManager.CreateAsync(role));
    await CurrentUnitOfWork.SaveChangesAsync(); //It's done to get Id of the role.
    await UpdateGrantedPermissionsAsync(role, input.GrantedPermissionNames);
}
```