Qualys Cloud Agent Bosh Release
---

* Download the tile from [PivNet](https://network.pivotal.io/products/qualys-cloud-agent-tile/)
* Rename the `qualys-cloud-agent-tile-x.x.x.pivotal` file to `qualys-cloud-agent-tile-x.x.x.zip`
* Unzip the tile into a folder: 

  ```unzip qualys-cloud-agent-tile-x.x.x.zip -d qualys-boshrelease```
* Switch into the bosh release directory:

   ```cd ./qualys-boshrelease/releases```

* Create a working directory and untar the code:

  ```
  mkdir qualys-cloud-agent
  
  tar -xvf qualys-cloud-agent-1.0.1.tgz -C qualys-cloud-agent
  
  rm -f qualys-cloud-agent-1.0.1.tgz

  cd qualys-cloud-agent
  ```

* Switch into the jobs folder and untar the file under it:
  ```
  cd jobs
  
  mkdir qualys-cloud-agent-linux
  
  tar -xvf qualys-cloud-agent-linux.tgz -C qualys-cloud-agent-linux
  
  rm -f qualys-cloud-agent-linux.tgz
  
  cd ..
  ```

* Rename the `job.MF` to `spec` under jobs folder: 

  `mv jobs/qualys-cloud-agent-linux/job.MF jobs/qualys-cloud-agent-linux/spec`

* Switch into the packages folder and untar the file under it:
  ```
  cd packages
  
  mkdir qualys-cloud-agent
  
  tar -xvf qualys-cloud-agent.tgz -C qualys-cloud-agent
  
  rm -f qualys-cloud-agent.tgz
  
  cd ..
  ```

* Create a src directory to store all the `.deb` files: 

  `mkdir src`

* Move all the `.deb` files from `packges` to `src`

  `mv packages/qualys-cloud-agent/*.deb src`

* Remove the `release.MF` file

  `rm -f release.MF`

* Begin the release: 

  `bosh init-release` 

* The `config` directory is created with the following contents:

```
.
├── config
│   ├── blobs.yml
│   └── final.yml
├── jobs
│   └── qualys-cloud-agent-linux
│       ├── monit
│       ├── spec
│       └── templates
│           ├── pre-start.erb
│           ├── start-cloud-agent.sh.erb
│           └── stop-cloud-agent.sh.erb
├── packages
│   └── qualys-cloud-agent
│       └── packaging
└── src
    ├── qualys-cloud-agent.x86_64-ca_pod1.deb
    ├── qualys-cloud-agent.x86_64-eu_pod1.deb
    ├── qualys-cloud-agent.x86_64-eu_pod2.deb
    ├── qualys-cloud-agent.x86_64-in_pod1.deb
    ├── qualys-cloud-agent.x86_64-us_pod1.deb
    ├── qualys-cloud-agent.x86_64-us_pod2.deb
    ├── qualys-cloud-agent.x86_64-us_pod3.deb
    └── qualys-cloud-agent.x86_64-us_pod4.deb

7 directories, 16 files
```

* Register blobs or a new version using the following command:
```
for f in $(ls src/); do bosh add-blob src/$f $f; done
```
This populates the blobs.yml with blob filename, filename size, and SHA

* Create a `spec` file under `packages/qualys-cloud-agent`
```
---
name: qualys-cloud-agent

dependencies: []

files:
- qualys-cloud-agent.x86_64-ca_pod1.deb
- qualys-cloud-agent.x86_64-eu_pod1.deb
- qualys-cloud-agent.x86_64-eu_pod2.deb
- qualys-cloud-agent.x86_64-in_pod1.deb
- qualys-cloud-agent.x86_64-us_pod1.deb
- qualys-cloud-agent.x86_64-us_pod2.deb
- qualys-cloud-agent.x86_64-us_pod3.deb
- qualys-cloud-agent.x86_64-us_pod4.deb
```

* Create the release

  `bosh create-release --final --force --tarball=qualys-cloud-agent-linux.tgz`

* Upload the bosh Release

  `bosh upload-release qualys-cloud-agent-linux.tgz`

* Update the file under `runtime-config` > `qualys.yml` file and update the version number, from the output of the previous command

  `version: 1`

* Update the runtime config

`  bosh update-runtime-config --name=qualys runtime-config/qualys.yml`

* Trigger Apply Changes from Ops Manager