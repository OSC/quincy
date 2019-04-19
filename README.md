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

## Architecture

- 'app.js'
    - Uses the JS 'mysql' module to connect to the database
        - Reads database configuration 'mysql.config' as described above
    - Sets some HTTP headers
    - Defines the HTTP GET route '{BASE_URI}/job/:system/:id' for JSON
        - Query: "select * from Jobs where jobid like {jid} and system={system}"
            - jid: "{job id}.{system}%"
            - system: owens, ruby, pitzer, oak
        - Sends results back in JSON format
    - Sets 'public' folder up for serving static assets
    - Sets 'public/index.html' for the GET request above
    - Starts HTTP server
- 'public/index.html'
    - Like layouts/application.html.erb in Rails in a sense
- 'public/app/'
    - 'app.component.html', 'app.component.ts'
        - Provides the search form
        - Provides an authentication form if not authenticated
        - Injects itself into index.html
        - Limits dropdown values to:
        - | Display | Value |
          | --- | --- |
          | Owens | owens |
          | Oakley | oak |
          | Ruby | ruby |
          | Pitzer | pitzer |
        - Navigates to `./:system/:jobId` on form submit
    - 'job.component.html', 'job.component.ts'
        - Defines a container for a Job
        - Provides the table layout for the information about the job
    - 'jobinfo.component.html', 'jobinfo.component.ts'
        - Provides the layout for the Ganglia links at the bottom
        - Shows an error message if the job was not found
    - 'jobinfo.service.ts'
        - Gets the nodes that were used in the job and generates the Ganglia links
    - 'jobinfo.ts'
        - Just a data structure to hold the Job itself, the nodes used, and the Ganglia URLs
    - 'app.module.ts'
        - Bundles a lot of the other components and modules together and exports them all as AppModule
    - 'app-routing.module.ts'
        - Routing stuff
    - 'job.ts'
        - Data structure for holding the job info that is shown in the table
        - Class name is 'Job'
    - 'main.ts'
        - Bootstrap starts AppModule

### Example JSON response

```
{
  "jobid": "5260000.owens-batch.ten.osc.edu",
  "system": "owens",
  "username": "kyc0045",
  "groupname": "PKS0016",
  "account": "PKS0016",
  "submithost": "owens-login04.hpc.osc.edu",
  "jobname": "fourcthreetwo",
  "nproc": 4,
  "mppe": null,
  "mppssp": null,
  "nodes": "1:ppn=4",
  "nodect": 1,
  "ngpus": 0,
  "feature": null,
  "gattr": "PKS0016",
  "gres": null,
  "queue": "serial",
  "qos": null,
  "submit_ts": 1555190148,
  "submit_date": "2019-04-13T04:00:00.000Z",
  "eligible_ts": 1555190148,
  "eligible_date": "2019-04-13T04:00:00.000Z",
  "start_ts": 1555190198,
  "start_date": "2019-04-13T04:00:00.000Z",
  "end_ts": 1555192993,
  "end_date": "2019-04-13T04:00:00.000Z",
  "start_count": 1,
  "cput_req": "00:00:00",
  "cput_req_sec": 0,
  "cput": "03:04:53",
  "cput_sec": 11093,
  "walltime_req": "24:00:00",
  "walltime_req_sec": 86400,
  "walltime": "00:46:35",
  "walltime_sec": 2795,
  "mem_req": "2000mb",
  "mem_kb": 578888,
  "vmem_req": null,
  "vmem_kb": 4663420,
  "energy": 0,
  "software": null,
  "hostlist": "o0543/1-4",
  "exit_status": 0,
  "script": "#!/bin/bash -l\n# Unified OSC submit filter $Revision: 1683 $, $Date: 2019-04-02 13:59:27 -0400 (Tue, 02 Apr 2019) $\n# ----- begin submit filter info -----\n# Time:  2019-04-13 17:15:48\n# User:  kyc0045@owens-login04.hpc.osc.edu\n# Command:  qsub /users/PKS0016/kyc0045/fourcthreetwo.script\n# Working directory:  /users/PKS0016/kyc0045\n# ----- end submit filter info -----\n# ----- begin user-specified directives -----\n#\n#PBS -N fourcthreetwo\n#PBS -A PKS0016\n#PBS -l nodes=1:ppn=4\n#PBS -l walltime=24:00:00\n#PBS -l mem=2000mb\n#PBS -j eo\n#PBS -e fourcthreetwo.err\n\n# ----- end user-specified directives -----\n# ----- begin submit filter added directives -----\n#PBS -l gattr=PKS0016\n# ----- end submit filter added directives -----\n# ----- user-submitted script follows -----\ncd $PBS_O_WORKDIR\n\nmodule load gaussian \n\ndos2unix fourcthreetwo.gjf\n\ng16 fourcthreetwo.gjf\n\n# format the chk file\nformchk fourcthreetwo.chk fourcthreetwo.fch\n\nmv fourcthreetwo.log fourcthreetwo.out\n\n",
  "sw_app": "gaussian",
  "contact": null
}
```
