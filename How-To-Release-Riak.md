This doc contains the internal Basho policy for building, testing, and releasing Riak.

# Scheduling
All public releases must conform to the following schedule:
* An expected final-package delivery date from engineering on a Wednesday. Each delivery date must have at least one fallback date that is no earlier than one week after the expected delivery date. If the final-package delivery date is missed, the fallback date becomes the primary final-package delivery date, and a new fallback date must be scheduled.
* A Go/No-Go scheduled on the Thursday after the final-package delivery date to ensure that the public release can proceed as planned. During the Go/No-Go we must confirm:
  * Giddyup has been triaged, and there are no major issues
  * Smoke testing has completed successfully on all supported platforms
  * Documentation content has been merged and triaged if necessary, and release notes are ready to ship
  * Packages for all supported platforms have been built, reviewer, and are ready to stage
  * Any marketing announcements have been reviewed and are ready to ship
* A final staging date on the Friday after the Go/No-Go. During this time, all packages will be pushed to their final download locations, the `External Release` docs will be sent around internally to the Product Management and Documentation teams for review, and the indexes for the download pages will be generated.
* A public release date on the Tuesday after the final staging date. This date cannot overlap with another public release.

# Pre-Release
All releases of Riak will have at least one `Release Candidate` before the final package build. These RCs will contain all of the `features, backports, and bugfixes` required to build the Final Release, but will not have an updated link to the `Release Notes`. The Release Notes link update will be the only difference between RCs and the final version that will be released to the public.

## Pre-release Procedure
All Release Candidates will undergo the following release stages to ensure quality:
  * Package Build
  * Internal Publishing
  * riak_test (giddyup) runs
  * Smoke Testing (on at least the final RC)

The following procedure will cover all of these steps:  
* Ensure `riak_test` is configured to automatically run after the package building steps:
  * If there is no branch in `riak_test` that corresponds to `riak`, create it
  * `basho_builds.yml` must exist in the top level of the repo, and contain the list of devrels that will be used to test various upgrade / downgrade scenarios during riak_test runs. The contents of the file will be similar to:
 ```
docker_rt_riak_test_config:
  - version: current
    product_version: current
    product: riak_ee
  - version: previous
    product_version: 2.2.0
    product: riak_ee
  - version: legacy
    product_version: 2.0.8
    product: riak_ee
  - version: 2.0.2
    product_version: 2.0.2
    product: riak_ee
  - version: 2.0.4
    product_version: 2.0.4
    product: riak_ee
  - version: 2.0.5
    product_version: 2.0.5
    product: riak_ee
```
  * Tag riak_test with the same `annotated tag` that you will use for riak
    * If you are running tests against 2.0 series, see below for special instructions
  * Push the tag to github  
* Bundle Riak
  * Lock the deps: `make lock`
  * Tag riak with an `annotated tag`  
* Giddyup runs will be kicked off automatically after the package builds complete successfully, but results must be triaged after each run. Giddyup runs usually complete 4.5 hours after the tag is initially received by the system.  
* Inform the `Release Management` team that a new version of riak is being built, and schedule some time for Smoke testing.  
  
  
*Note*: Remember to follow the aforementioned procedure for EE as well, since these products are always releases in tandem.

### Special considerations for 2.0 series
There is one key difference in 2.0.x and later versions that must be accounted for when setting up riak_test: PB is locked to a specific version in Riak itself. This would cause a conflict in rebar, since `basho_bench` (built as part of riak_test) typically pulls the latest PB. To avoid this scenario in 2.0.x builds, a special branch has been created in b_b that locks PB to the same version as Riak: `riak/2.0`. An override must be created in basho-builds by setting `riak_test_bb_version` to `riak/2.0` in `basho_builds.yml` in the riak_test repo. `basho_builds.yml` for 2.0.x in riak_test should look something like this:
```
riak_test_bb_version: 'riak/2.0'
docker_rt_riak_test_config:
  - version: current
    product_version: current
    product: riak_ee
  - version: previous
    product_version: 2.0.7
    product: riak_ee
  - version: legacy
    product_version: 1.4.12
    product: riak_ee
  - version: 2.0.2
    product_version: 2.0.2
    product: riak_ee
  - version: 2.0.4
    product_version: 2.0.4
    product: riak_ee
  - version: 2.0.5
    product_version: 2.0.5
    product: riak_ee
```

# Checklists
## Planning
- [ ] Expected final-package delivery date has been scheduled
- [ ] Fallback final-package delivery date has been scheduled
- [ ] Go/No-Go has been scheduled
- [ ] Fallback Go/No-Go has been scheduled
- [ ] Public release date has been scheduled
- [ ] Fallback Public release date has been scheduled
- [ ] Product Management has issues a supported platforms list to DevOps and RelEng

## Final Packaging by Engineering
- [ ] At least one RC has been generated:
  - [ ] Packages for all supported platforms have been built
  - [ ] Giddyup has been triaged and vetted by engineering
  - [ ] Smoke testing has been run against all supported platforms
- [ ] Link to the Release Notes has been added
- [ ] Packages for all supported platforms have been built
- [ ] Smoke testing has been run against all supported platforms

## Go/No-Go
- [ ] Packages have been built and delivered by Engineering
- [ ] Giddyup has been triaged and vetted by Engineering
- [ ] Smoke testing has been completed successfully for all supported platforms
- [ ] Documentation has been merged, release notes are ready to ship
- [ ] Marketing announcements have been reviewed and are ready to ship

## Staging
- [ ] Docs pushed to staging
- [ ] Packages have been put in their final download locations and the `External Release` doc has been generated
- [ ] `External Release` doc has been sent to Product Management and Documentation for review
- [ ] Indexes for the Downloads Page have been generated

## Public Release
- [ ] Docs have been pushed live
- [ ] Packages have been uploaded to `packagecloud`
- [ ] EE package links published in Zendesk
- [ ] Marketplace AMI has been created and sent to Amazon

