# Node Management Status

## Overview
The Open Horizon policy based, autonomous node management capability is described [here](./node_management.md). 

A Node Management Policy (NMP) status object is generated by the node management worker during node management jobs in order to track the state of the job.

## Definition
Following are the fields in the JSON representation of an NMP status:

* `agentUpgradePolicyStatus`: A JSON structure to define the status of an automatic agent upgrade job.
    * `scheduledTime`: An RFC3339 formatted timestamp for when the NMP should start execution.
    * `startTime`: An RFC3339 formatted timestamp for when the NMP was *actually* executed. 
    * `endTime`: An RFC3339 formatted timestamp for when the NMP was executed successfully. This field will not be populated if the NMP fails.
    * `upgradedVersions`: A JSON structure to define the versions being upgraded or downgraded to on a device case. Cluster agents will not use this section.
        * `softwareVersion`: The version of the agent software packages to be installed.
        * `certVersion`: The version of the certificate to be installed.
        * `configVersion`: The version of the config to be installed.
    * `status`: The state, of the upgrade job. See the section **Status Values** below for more information.
    * `errorMessage`: A short message that describes why an agent upgrade job has failed.
    * `workingDirectory`: The directory that the upgrade job will be reading and writing files to.

## Status values

* Agent Auto Upgrade status values
    * `"waiting"`: The node management worker has matched the NMP to this node and created the status object in the local database.
    * `"download started"`: The download worker has began downloading all necessary packages from the Management Hub.
    * `"downloaded"`: The download worker has finished downloading all necessary packages from the Management Hub.
    * `"initiated"`: The installation of the downloaded packages has started and is being performed by the AgentAutoUpgrade cron job script.
    * `"successful"`: The node management worker has successfully performed the upgrade job specified in the NMP.
    * `"no action required"`: The node management worker has determined that no actions need to be taken to upgrade or downgrade the agent. This typicaly means that all the files specified within the NMP's manifest are already installed, or they are a lower version than what is currently installed, and the NMP set the allowDowngrade field to false.
    * `upgrade aborted`: There was a problem during the pre-check in the AgentAutoUpgrade cron job script, so the job was cancelled before the installation.
    * `"download failed"`: The download worker was unable to download all necessary packages from the Management Hub.
    * `"failed"`: There was a problem during the installation of the downloaded packages either in the node management worker or in the AgentAutoUpgrade cron job script.
    * `"rollback started"`: If the status was set to "failed", the next time the AgentAutoUpgrade cron job waked up, it will attempt to rollback the version to the previous version, and it will set the status to this value.
    * `"rollback failed"`: There was a problem with the rollback to the previous version. The agent is most likely in an unoperable state and will need manual intervention to fix.
    * `"rollback successful"`: The agent was successfully rolled back to the previous version.
    * `"unknown"`: The NMP job is in some unrecognizable state.

## Examples

The following is an example of a NMP status json file. The status objects are nested within the node and the NMP they apply to, as this is how they are stored in the Exchange. There can be multiple NMP's running on a single node, and there can be multiple nodes running the same NMP, so this is why the structure is formatted this way.

In this case, there is one upgrade job type - agent auto upgrade. The status for this job is stored in the agentUpgradePolicyStatus field. This job was completed successfully, so the status is set to "successful" and all of the timestamps are included. It should also be noted that the errorMessage field is omitted since the job was successful and there were no errors.
```
{
  "org/sample-node": {
    "org/sample-nmp": {
      "agentUpgradePolicyStatus": {
        "scheduledTime": "2022-05-24T12:00:00Z",
        "startTime": "2022-05-24T12:01:00Z",
        "endTime": "2022-05-24T12:02:00Z",
        "upgradedVersions": {
          "softwareVersion": "2.30.0",
          "certVersion": "1.0.0",
          "configVersion": "1.0.0"
        },
        "status": "successful"
      }
    }
  }
}
```

## Listing NMP statuses currently stored in the Exchange
To list all the NMPS that exist in the Exchange, use the following command:
```
hzn exchange nmp status <nmp-name>
```

**Optional Flags**:  

* `--long, -l`: Display the entire contents of each node management policy status object.
* `--node`: Filter output to include just this one node. Use with --long flag to display entire content of a single node management policy status object.