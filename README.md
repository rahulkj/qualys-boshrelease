Qualys Cloud Agent Bosh Release
---

#### Table of contents
- [Deploying Qualys Cloud Agent as a bosh release](##deploying-qualys-cloud-agent-as-a-bosh-release)
- [Installing Qualys to Ops Director using Ops Manager](#installing-qualys-to-ops-director-using-ops-manager)

## Deploying Qualys Cloud Agent as a bosh release

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

* Copy all the `.deb` files into the blobs

  ```
  for f in $(ls src/); do bosh add-blob src/$f $f; done
  ```

* Update the `final.yml` file under `config` to include the `blobstore_path` variable:

  ```
  blobstore:
    provider: local
    options:
      blobstore_path: /Users/bob/blobs
  name: qualys-cloud-agent
  ```

  where the `blobstore_path` is the current working directory: `pwd`

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

* Update the file under [runtime-config](./runtime-config/qualys.yml) file and update the version number, from the output of the previous command

  `version: 1`

* Update the runtime config

`  bosh update-runtime-config --name=qualys runtime-config/qualys.yml`

* Trigger Apply Changes from Ops Manager

## Installing Qualys to Ops Director using Ops Manager

* Modify the `spec` file under `jobs/qualys-cloud-agent-linux/spec` and add the following property to it
  ```
    qualys.version:
      description: 'Specify the qualys bosh release version'
  ```

* Next modify the file pre-start.erb under `jobs/qualys-cloud-agent-linux/templates/pre-start.erb` and replace the line `local userAgent="bosh/<%= "spec.release.version" %>"` with `local userAgent="bosh/<%= p("qualys.version") %>"`

* Create a new bosh-release version

  `bosh create-release --final --force --tarball=qualys-cloud-agent-linux.tgz`

* Create the [director.json](./runtime-config/director.json) and update the following:
  - `release_url`
  - `release_sha1`
  - `activationId`
  - `customerId`
  - `downloadUrl`
  - `podId`
  - `proxyPass`
  - `proxyServer`
  - `proxyUser`
  - `useProxyPcpDownload`
  - `version`

* Next add the job that needs to be added to bosh deployment by running the following command using `om` cli

  ```
  om -t $OM_TARGET curl -p /api/v0/staged/director/manifest_operations/add_job_to_instance_group -x POST -d '{
  "add_job_to_instance_group": {
    "instance_group": "bosh",
    "job_name": "qualys-cloud-agent-linux",
    "release_name": "qualys-cloud-agent",
    "release_url": "http://ubuntu.homelab.io:9090/vmware/addons/qualys-cloud-agent-linux.tgz",
    "release_sha1": "0fdf91e4e8eea12f8dbbebf1872161d71bb3cbe1",
    "job_properties": {
      "qualys": {
        "activationId": "1",
        "customerId": "2",
        "downloadUrl": "",
        "podId": "us_pod2",
        "proxyPass": "",
        "proxyServer": "",
        "proxyUser": "",
        "useProxyPcpDownload": "false",
        "version": 5
        }
      }
    }
  }'
  ```

  Please post the json that you have crafted to be applied here.

* Upon applying the above json, you should see the output similar to the one below

  ```
  Status: 201 Created
  Cache-Control: no-cache, no-store
  Connection: keep-alive
  Content-Type: application/json; charset=utf-8
  Date: Tue, 25 Aug 2020 22:17:20 GMT
  Etag: W/"10a6adcbc373538876e0d28b84c83858"
  Expires: Fri, 01 Jan 1990 00:00:00 GMT
  Pragma: no-cache
  Referrer-Policy: strict-origin-when-cross-origin
  Server: Ops Manager
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  X-Content-Type-Options: nosniff
  X-Download-Options: noopen
  X-Frame-Options: SAMEORIGIN
  X-Permitted-Cross-Domain-Policies: none
  X-Request-Id: 38e546a9-555f-4350-a8cc-ada643b22f81
  X-Runtime: 0.229496
  X-Xss-Protection: 1; mode=block
  {
    "add_job_to_instance_group": {
      "instance_group": "bosh",
      "job_name": "qualys-cloud-agent-linux",
      "release_name": "qualys-cloud-agent",
      "release_url": "http://ubuntu.homelab.io:9090/vmware/addons/qualys-cloud-agent-linux.tgz",
      "release_sha1": "0fdf91e4e8eea12f8dbbebf1872161d71bb3cbe1",
      "job_properties": {
        "qualys": {
          "activationId": "1",
          "customerId": "2",
          "downloadUrl": "",
          "podId": "us_pod2",
          "proxyPass": "",
          "proxyServer": "",
          "proxyUser": "",
          "useProxyPcpDownload": "false",
          "version": 5
        }
      },
      "guid": "op-5b13bf803c24",
      "product_guid": "p-bosh-3d097bc3a29680328b4a"
    }
  }
  ```

* If you need to update/delete the above job, you will need to get the `guid` from the above output and run the delete operation

  ```
  om -t $OM_TARGET curl -p /api/v0/staged/director/manifest_operations/add_job_to_instance_group/op-5b13bf803c24 -x DELETE
  ```

* Now upload the release to a link, from where Ops Manager can download the release and make it available while updating bosh director

* Finally apply changes
  `om -t $OM_TARGET apply-changes --skip-deploy-products`

* During the build of ops director, you will see a line like the following in the output
  ```  
  ...
  Compiling package 'qualys-cloud-agent/7313b295c10fcffc38d5056326c4d7595981702af281d4c12df4cd5f2a7a798f'... Finished (00:00:03)
  ...
  ```