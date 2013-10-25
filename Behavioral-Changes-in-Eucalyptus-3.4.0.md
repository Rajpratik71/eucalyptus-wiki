# What's Changed in Eucalyptus 3.4.0?

The intent of this page is to highlight as many of the changes present in Eucalyptus 3.4.0 as possible with particular emphasis on behavioral changes that do not involve API or CLI changes.

### Service Life-cycle
* 'eucalyptus-cloud' start-up is notably slower than in 3.3.x due to much more robust start-up checks, extensive logging of the start-up process, and serialization of service start-up to remove race-conditions and add stability.
* Starting a non-CLC java service, let's call it serviceA, before the CLC is running (e.g. starting a remote SC before CLC) will result in the service only partially bootstrapping and then waiting for the CLC. Once the CLC is running, the serviceA will intentionally begin a timer to wait for the system to stabilize. Logs will indicate this. serviceA will not continue bootstrapping until that timer expires, which is approximately 3 minutes. During this time any 'euca-describe-services' calls to the CLC will return the state of serviceA as 'NOTREADY'.

## HA-specific Changes

### DB Sync
* Ability to change sync strategy to dump restore (not fully tested thus not the default yet)

## DNS Changes
* DNS properties have been moved from the ```experimental.dns``` namespace to just ```dns```

## ELB Changes
* Allow NTP server to be set by euca property
* Restricted ports config property

## SANs
### Equallogic
* Allow pool name to be set explicitly. 