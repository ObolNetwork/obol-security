22nd Mar, 23 - DKG issues in Lido

|              |                       |
| ------------ |-----------------------|
| **Status**   | Closed                |
| **Severity** | Minor |


### Summary

Users in the lido discord server faced several issues while performing DKG ceremonies amongst themselves. 
This postmortem doc is a report on what went wrong and what we could do to mitigate scenarios like this.

### Timeline

- START TIME: 21st Mar, 23
- END TIME: 22nd Mar, 23
- LENGTH of the disruption: 24 hrs

### Incident Team

|                    |                 |
| ------------------ |-----------------|
| Report Author      | Dhruv, Abhishek |
| Incident Reporter  | Dhruv           |
| Incident Commander | Dhruv, Abhishek |


### Impact

- What impacted

    - charon dkg

- Who impacted

    - Users in lido discord server trying to do DKG

- How impacted

    - Some users didn’t receive keystore files at the end of the DKG. Others had file permissions issues. In short, unsuccessful DKGs for some users.

### Detection

The incident was detected by Dhruv who was helping users in the lido discord server to conduct DKGs.


### Resolution

We discussed the final solution internally. First, we recommended users to use v0.13.0 to perform the DKG as there were bugs
in v0.14.0. But, this was a short-term solution as several users were still getting different kinds of errors. We proposed to
patch release charon [v0.14.3](https://github.com/ObolNetwork/charon/releases/tag/v0.14.3) with the fix. This was properly discussed,
tested and then [communicated](https://discord.com/channels/761182643269795850/1082703855411810394/1088085286996672564) to the users in lido discord.


### Root Cause

The root cause can be attributed to a bug in charon latest release. This was exacerbated by bad communications between the
core team and lido users about the fix.


### Prevention Actions

|            |           |            |
| ---------- | --------- | ---------- |
| **Action** | **Owner** | **Status** |
|            |           |            |
|            |           |            |


### Mitigation strategies

*Before every DKG run*:
* Ensure all the users run the same version of charon, you can now use v0.14.3.
* Ensure you have only one file in .charon/ directory i.e., only `charon-enr-private-key` and nothing else.
* Make sure to backup charon-enr-private-key in the .charon folder (without this key, we'll have to start from scratch).
* If you encounter a permissions error in DKG run on linux, then follow the post-installation steps of docker: https://docs.docker.com/engine/install/linux-postinstall/ . If the error still persists, do: `chmod -R 777 .charon`
* Ensure there is no other charon container/process running during the DKG ceremony. docker compose down all containers related to the current cluster in charon-distributed-validator-node directory.
* Don’t use the same machine to run two DKG ceremonies at the same time with the same `charon-enr-private-key`. This can have side-effects.
* Avoid running two DKG ceremonies in the same machine even with different `charon-enr-private-key` files. This can lead to mismanagement of the validator_keys directory by the user. If you still do multiple DKG ceremonies on the same machine, please ensure they are on separate directories/locations.
* After every DKG run:
  * Make sure each participant has received deposit-data.json, cluster-lock.json and the required number of keystores in validator_keys directory in the .charon directory. The directory structure should look like this:
```
* .charon
├── charon-enr-private-key
├── cluster-lock.json
├── deposit-data.json
└── validator_keys
├── keystore-0.json
└── keystore-0.txt
```
* The coordinator of the DKG MUST ensure that all the DKG participants have all the required keystore files, the cluster-lock and deposit data file.
* If there is any missing file for any participant, then the DKG has to be performed again.
* Also note that the participants should wait for confirmation from the coordinator that the DKG is done correctly. Only then the participants should go ahead and do docker compose up, ie, run the charon stack.

### DKG QA

As a side effect, we also identified a list of QA items for charon dkg. We should perform diligent QA on our DKG process before every
release. Here are some ways:
* Long Wait DKG
  * Purposefully delay a peer to perform DKG by like 4-5 minutes.
  * This will test DKG in relay re-connections.
* One peer per minute DKG
  * Start a DKG peer at the end of every minute
* Validator_Keys/ already exist
  * Have a `.charon/validator_keys/` directory already existing and then do DKG.
  * Have a `.charon/cluster-lock.json` file already existing and then do DKG.
* Large DKG
  * Perform DKG with a large set of users (>=10).
* Multi machine DKG
  * Perform DKG on multiple different machines with different OS/Arch.
  * Linux, Windows, Mac, VM combinations.
  * Make sure we also test on windows.


### Events timeline

-- 
