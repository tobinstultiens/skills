# Use primary and backup Git remotes

The skills backup uses `origin` as the primary remote and `backup` as the secondary mirror. `origin` is configured with push URLs for both repositories so a normal `git push origin master` updates both destinations, while the named `backup` remote still makes the secondary location explicit for inspection and maintenance.
