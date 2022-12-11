---
description: Builds on the Backend machine with updated security features.
---

# BackendTwo

## Gaining Access

Since this builds on the other Backend machine from UHC, there isn't a lot of enumeration to do.

<figure><img src="../../../.gitbook/assets/image (360).png" alt=""><figcaption></figcaption></figure>

Port 80 brings us to an API again, with the admin user still being viewable.

<figure><img src="../../../.gitbook/assets/image (358).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (356).png" alt=""><figcaption></figcaption></figure>

### Creating User

We can do the same stuff to create, signin as a user and receive the JWT token for it.&#x20;

<figure><img src="../../../.gitbook/assets/image (363).png" alt=""><figcaption></figcaption></figure>

Afterwards, we can access the `/openapi.json` endpoint to view the functionalities of this API. There was one new functionality, which was to edit the profiles of users.

<figure><img src="../../../.gitbook/assets/image (366).png" alt=""><figcaption></figcaption></figure>

This endpoint was rather interesting because it allows us to edit profiles. Checking the JWT token of our current user, we find out that our `id` is 12.

<figure><img src="../../../.gitbook/assets/image (372).png" alt=""><figcaption></figcaption></figure>

### Takeover Superuser



### Ad
