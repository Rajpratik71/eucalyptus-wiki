A lot has changed around how Eucalyptus interacts with NetApp SANs in the 3.3 release. Some of the high level changes are explained below

### Revamped NetApp Integration

Guts of the NetApp Provider in Eucalyptus have been changed to use the more recent NetApp ONTAP and NMSDK APIs. The legacy provider using an older version of NetApp API has been **deprecated**. We are planning on keeping the legacy provider around for the 3.3 release but we **DO NOT** recommend configuring it in 3.3 clouds and neither do we support it (as it has not been tested or worked on during 3.3). If you still want to use the legacy provider do so at your own risk by configuring the block storage manager to 'netapp_legacy' on a fresh install. Block storage manager cannot be changed after its set. If you want to force the change, you'll have to deregister the SC (loose all your previous data) and register the SC with a different provider. Not sure why anyone thats not testing would try this.

### CHAP User Requirement

CHAP user configuration is mandatory in 3.3 for both 7-mode and Clustered ONTAP NetApp SANs. SC will not transition to ENABLED state until CHAP user is configured. The same requirement holds good in the upgrade case too. Use euca-modify-property command to set <region>.storage.chapuser in Eucalyptus. CHAP username can be any value. SC generates the CHAP password that is used in combination with this CHAP username.

### Multiple Aggregates

Multiple NetApp aggregates can be configured for use by Eucalyptus in 3.3. Use euca-modify-property command to edit .storage.aggregate and supply a comma separated list of aggregates. SC picks an aggregate from the list based on <region>.storage.uselargestaggregate flag (true by default i.e. aggregate with the largest capacity) before creating a volume. SC can be made to choose smallest available aggregate by modifying <region>.storage.uselargestaggregate property to false. Prior to 3.3, only one aggregate could be configured using <region>.storage.aggregate and the default value was aggr1. In 3.3 there is **no default value** for <region>.storage.aggregate. So if one or more aggregates are not explicitly configured, the SC will query the SAN for all available aggregates and pick one based on <region>.storage.uselargestaggregate flag.

### New Igroup Management

In 3.3 we no longer keep track of the igroups in the SC database. Igroups are created on a per initiator (NS/SC) basis with the same name as the initiator IQN. Igroups are created and deleted when volumes are attached and detached respectively. If an Igroup already exists the create step is skipped. And if the Igroup is in use the delete step is skipped. All Igroup related operations are synchronized on the SC and the SAN is considered the source of truth for Igroups.

### Clustered Data ONTAP SAN & Multipath I/O

### Eucalyptus NetApp Properties

***
[[category.storage]]