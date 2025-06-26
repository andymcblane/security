# Broken access control

This is _fairly_ easy to understand and fairly easy to test. This occurs when controls in place to authenticate, authorise (or log) against resources fails.

e.g test to ensure @auth_required decorator does not allow regular users

```python

@auth_required("admin")
@route("/admin")
def login():
    return 200

```

```python
non_admin = Users.objects.create(name="regular guy")
non_admin.auth_token = AuthToken.objects.create(user=non_admin)
# no permissions granted
header = {"Authorization": non_admin.auth_token}
assert request.get("/admin", headers=header).status_code == 403
```

This can also happen in reverse e.g admin users cannot access resources that regular users are allowed to access.

All product role permutations should be unit tested and the tests should enforce that all roles have been tested (to avoid the case of a new role being added and no tests added.)

Additionally, this may occur if a resource is accessed but, for example, and error occurs during the processing of that resource and subsequent audit logging does not occur.