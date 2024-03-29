# gemoc-studio-eclipseforks-integration
Integration via git modules of GEMOC components in the GEMOC organization for use by continuous integration server

## CI Jobs

* **https://ci.inria.fr/gemoc/job/gemoc-studio-eclipseforks-integration/** : CI Jobs for each branches
(trigger policy: polling of branches content every 5 minutes) [![https://ci.inria.fr/gemoc/job/gemoc-studio-eclipseforks-integration/job/master/badge/icon](https://ci.inria.fr/gemoc/job/gemoc-studio-eclipseforks-integration/job/master/badge/icon?subject=Master%20build)](https://ci.inria.fr/gemoc/job/gemoc-studio-eclipseforks-integration/job/master)

* [![SyncSubModulesBranches](https://github.com/gemoc/gemoc-studio-eclipseforks-integration/actions/workflows/syncSubmodulesBranches.yml/badge.svg)](https://github.com/gemoc/gemoc-studio-eclipseforks-integration/actions/workflows/syncSubmodulesBranches.yml) : job is in charge of synchronizing the branches for all known submodules (ie. create or remove branches if they exist in the submodules)
(trigger policy: every 20 minutes)

* [![SyncForkedMasters](https://github.com/gemoc/gemoc-studio-eclipseforks-integration/actions/workflows/sync-forked-master.yml/badge.svg)](https://github.com/gemoc/gemoc-studio-eclipseforks-integration/actions/workflows/sync-forked-master.yml) : job in charge of copying all changes from master branches of the origin repositories (in eclipse organisation ) to the gemoc organisation
 (trigger policy: every 20 minutes)
 
# Maintenance tasks

In order to add a new repository in the integration build, you need to update 2 points:

* add the new git repository as a git submodule in the master branch. (default branch name must be "master")
* update the script releng-scripts/sync-forked-masters/syn_git_masters.sh to automate the copy of commit from the new forked repository master branch 

## add a new submodule

Make sure to align the folder name to the expected folder organisation if the repository fork as a different name than its origin. 

```bash
git submodule add -b master https://github.com/gemoc/gemoc-studio-eclipsefork.git gemoc-studio
git submodule add -b master https://github.com/gemoc/gemoc-studio-commons.git gemoc-studio-commons
git submodule add -b master https://github.com/gemoc/gemoc-studio-modeldebugging-eclipsefork.git gemoc-studio-modeldebugging
git submodule add -b master https://github.com/gemoc/gemoc-studio-execution-ale-eclipsefork.git gemoc-studio-execution-ale
git submodule add -b master https://github.com/gemoc/gemoc-studio-execution-moccml.git gemoc-studio-execution-moccml
git submodule add -b master https://github.com/gemoc/gemoc-studio-moccml.git gemoc-studio-moccml
git submodule add -b master https://github.com/gemoc/gemoc-studio-execution-java.git gemoc-studio-execution-java
```
