# Start multiple Node services with PM2's cluster mode

Just a sample for fun to demonstrate how we could scale nicely with just folder structures and existing tools.

## Cautions
- PM2 does not work with npm/yarn script so exec needs to be on with Node

## Monorepo on steroid
Combine https://github.com/boennemann/alle with Yarn workspaces gets us:
 - Monorepo to flatten packages and dedupe them, reducing node_modules size and faster installation is always good!

 - Independent services that could still be developed separately, with a common module to share code that could be imported like a package (see https://gitlab.com/cheshireCode/monorepo-services/blob/master/packages/node_modules/@monorepo-services/service-1/index.js#L2).

 - Tricked Node module resolution that allows modules/dependencies within `packages/node_modules` to be imported with the same `require` as Node walks up to the next available `node_modules` at the top, so either in workspaces or root folder, where all dependencies are installed. E.g. in https://gitlab.com/cheshireCode/monorepo-services/blob/master/packages/node_modules/@monorepo-services/service-2/index.js#L2 we see `service-2` importing package `semver` which is a dependency of `service-1` as shown at https://gitlab.com/cheshireCode/monorepo-services/blob/master/packages/node_modules/@monorepo-services/service-1/package.json#L6. 

 ## Run once with PM2
 - PM2 uses [cluster](https://nodejs.org/api/cluster.html) module that allows Node process to all bind to the same host port. This is good because we could use any Node versions and let PM2 handle the scaling automagically.
 - The process file for PM2 at https://gitlab.com/cheshireCode/monorepo-services/blob/master/pm2.json shows how we could:
   - Specify Node instances per service by hardware (based on how many CPUs the host have available, which is the usual scaling approach)
   - Watch and restart services with Chokidar e.g.

Current status
```
--> yarn status
```

Try to edit something
```
--> touch ./packages/node_modules/@monorepo-services/service-1/index.js
```

Check process status, restart column
```
--> yarn status
```
