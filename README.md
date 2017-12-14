# AeroGear Proposals
This repository contains proposals, examples, PoCs for potential new mobile features to be build by the mobile team.

## How to submit a proposal
* Fork this repository 
* Create a new directory with your proposal
* Include a README.md which is either your proposal or the index of the proposal
* Send a PR to master on github.com/aerogear/proposals
* Email aerogear-dev so we know to take a look

## When to submit a proposal
Proposals aren't for everything, and they are mostly for new or large features.  If you have a bug fix, documentation improvement, etc then a PR with the fix is much more appropriate.  If you have a minor feature such as exposing a logging parameter, code cleanups, or adding tests then you could probably get by with a PR as well, but you might want to ping the mailing list (aerogear-dev) to get a sanity check.  However if you are adding a major feature like adding a new algorithm to the sync server, migrating HTTP libraries from OKHttp to volley, etc then these are they types of things proposals are for.  As a rule of thumb if what you want can be fit into a single issue then you should probably not write a proposal.

## What does a good proposal look like
When you submit a proposal it should be fairly obvious what you are trying to build.  In general you should describe what you are trying to build, its core features, any points of discussion to include, etc.  If you think something will make the proposal easier to follow or clear up any misconceptions then add that as well.  For SDKs if you have some psuedo code or POC implementation, include that in your directory.  If you want to create UI mockups then feel free to include those as well.