#TL;DR

NIAD Data Ecosystem (NDE) deployment overview

## Overview

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
