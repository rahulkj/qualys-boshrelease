Qualys Cloud Agent Bosh Release
---

* Download the tile from [pivnet](https://network.pivotal.io/products/qualys-cloud-agent-tile/)
* Rename the `.pivotal` file to `.zip`
* cd into releases > `qualys-cloud-agent`
* cd packages and untar the `qualys-cloud-agent.tgz` and `rm qualys-cloud-agent.tgz`
* cd jobs and untar the `qualys-cloud-agent-linux.tgz` and `rm qualys-cloud-agent-linux.tgz`
* `mv jobs/qualys-cloud-agent-linux/job.MF jobs/qualys-cloud-agent-linux/spec`
* `mkdir src`
* mv `mv packages/qualys-cloud-agent/*.deb src`
* `bosh init-release` 
* Add all the files to the blobs
```
for f in $(ls src/); do bosh add-blob src/$f $f; done
```
* Create a spec file under `packages/qualys-cloud-agent`
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
* Create the release `bosh create-release --final --force --tarball=qualys-cloud-agent-linux.tgz`
* `bosh upload-release qualys-cloud-agent-linux.tgz`