#TL;DR

NIAD Data Ecosystem (NDE) QA deployment overview

## Overview

The NDE deploys three gen3 stacks to illustrate how
multiple loosely related commons can interoperate.

### https://niaiddata.org - https://qa-test3.planx-pla.net

Root domain with 2 subcommons below.
* should not hold any real data - just a catalog of sub-commons
    - just deploy the dev dictionary, sheepdog, peregrine, pidgin to keep cloud-automation happy

* runs shared fence, arborist, indexd, ssjdispatcher


### https://tb.niaiddata.org - https://qa-test2.planx-pla.net

Tuberculosis sub-commons with its own dictionary - sibling to daids.niaiddata.org

* delegates fence, indexd, arborist, ssjdispatcher, to the parent domain
* runs portal, guppy, sower, spark, etl, manifestservice, hatchery, etc

### https://daids.niaiddata.org - https://qa-test4.planx-pla.net

AIDS sub-commons with its own dictionary - sibling to tb.niaiddata.org

## Test Plan

We should test all the core commons functionality to 
make sure it all works with the screwy shared fence/arborist/indexd setup.

I think the intention is that we login to the root commons https://qa-test3.planx-pla.net (corresponds to https://niaiddata.org).  The root commons does not have any data itself, but may have tools that aggregate data from the subcommons (https://qa-test2.planx-pla.net corresponds to https://tb.niaiddata.org, and https://qa-test4.planx-pla.net corresponds to https://daids.niaiddata.org).  The root commons also handles authentication and authorization for the sub-commons, url-signing, and a shared indexd.

* Login to https://qa-test3.planx-pla.net (the root commons of the ecosystem), then you should be able to access both https://qa-test2.planx-pla.net and https://qa-test4.planx-pla.net
* logout - make sure you really logout
* give yourself permission to some programs in both sub-commons
* setup a project, and submit metadata to both sub-commons.
* run the ETL in both sub-commons
* check the explorer page in both sub-commons
* upload some data files with the `gen3-client` to both sub-commons ...?
* download an api key
* check that gen3-sdk can do something against both sub-commons with the api key
