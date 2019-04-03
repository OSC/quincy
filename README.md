# Quincy

OSC batch job post-mortem

## Installation

First clone the app from GitHub:

`git clone git@github.com:OSC/quincy.git`

Then move to the newly created 'quincy' directory:

`cd quincy`

Then do an npm install for the outer Express.js wrapper in the 'quincy' directory:

`scl enable rh-nodejs6 -- npm install`

Then navigate to the 'public' directory where the Angular stuff resides and do another npm install:

```
cd public
scl enable rh-nodejs6 -- npm install
```

The TypeScript code is in the 'app' folder from here. Compile it to JavaScript with the following command:

`scl enable rh-nodejs6 -- npm run tsc`

Then navigate back up to the 'quincy' directory:

`cd ..`

Then add a 'mysql.config' file in that directory. This contains the information to connect to the pbsacct database. It must formatted in JSON. Example:

```
{
  "host": "pbsacct.mycenter.edu",
  "user": "dbuser",
  "database": "pbsacctdb"
}
```
