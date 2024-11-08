---
description: Impersonation
---

# Impersonation 
([Enterprise](https://readonlyrest.com/enterprise))

According to [Wikipedia](https://en.wikipedia.org/wiki/Impersonator):

> An impersonator is someone who imitates or copies the behavior or actions of another.

So, an impersonation can be understood as imitating behaviors or actions.
In the context of ReadonlyREST: one user could imitate an action 
of another user. Why would we want it? Let's suppose the first user is 
an admin, who has just configured access for a new user. They would like 
to know if the rule(s) are configured correctly. And here comes the impersonation feature. The admin can impersonate a given user in Kibana and see what the user would see if they logged in themselves. 

ROR plugins support impersonation and provide UI for configuring the cluster before using it. Visit the [impersonation details page](../details/impersonation.md) to know more.
