## GitHub Enterprise Site Administration

For several years, my main responsibility at work was GitHub Enterprise Site administration.
Here is a collection of thoughts, in no particular order.

### Rate Limiting

GitHub Enterprise comes with built-in rate limiting.
Unfortunately, it isn't enabled by default, which is a shame.
If you're running an instance of any size, such as mine which had thousands of user accounts and hundreds of daily users, often with heavy workloads, you will inevitably run into performance problems. 
GitHub Enterprise is a single VM instance.
A single point of failure, and thus one can only scale it vertically.
The default rate limits are reasonable for most workloads, but it's better to start somewhere and adjust them than allow terrible user behavior to take foot.

### Hardware

Use dedicated, powerful hardware with lots of RAM and I/O.
CPUs won't hurt either because Git is computing lots of hashes.

### GitHub Apps

Prefer and promote the use of [GitHub Apps][github-apps] instead of legacy OAuth apps or personal access tokens.
It's a more secure and better scheme for integrating with the server.
It also avoids the "service account" pattern, for which a shared user account is created to perform a role.
GitHub Apps are first class actors in GitHub, but do not consume an Enterprise license.

### GitHub.com

Ask yourself if you really need GitHub Enterprise as opposed to GitHub.com.
Do you really need the headache of administering your own instance?
Also be aware that new features are always rolled out to GitHub.com first, and often do not appear in Enterprise until 6 or more months later.



[enterprise]: https://github.com/enterprise
[rate-limiting]: https://docs.github.com/en/enterprise-server@3.1/admin/configuration/configuring-your-enterprise/configuring-rate-limits
[github-apps]: https://docs.github.com/en/developers/apps/getting-started-with-apps/about-apps
