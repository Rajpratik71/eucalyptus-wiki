### Configuration
1. When registering multiple OSG components, OSGs not co-located with the CLC will require a service restart to pick up the backend configuration if the service was running prior to registration. This will be resolved in the 4.0 release.

2. Bootstrapping can take longer than other components. It isn't unexpected to see OSGs listed in 'euca-describe-services' to stay in the BROKEN or LOADED state after the CLC services have reached ENABLED. They should transition successfully within a minute or two of the other services. This is normal behavior due to configuration propogation and bootstrapping of the remote component. If the OSG component stays in BROKEN for more than 30 seconds after the other services are ENABLED then a problem may exist in your configuration.

### API Features/Compatibility with S3

The following features are not enabled for the Tech-Preview but will be fully functional for the 4.0 release:

1. Object versioning ( GET bucket/object?versionId=x, PUT bucket/?versioning, DELETE bucket/object?versionId=x)

2. Bucket logging. (PUT bucket/?logging). Querying logging status is implemented, but changing it is disabled for the TP.

3. Bucket naming conventions are not current with S3 API. Specifically, '_' is not allowed. This is an issue with the version of a backend library we are using. It will be updated for the 4.0 release and this restriction will be removed. If you attempt to create a bucket with '_' in the name, you will receive a HTTP 500 Internal Error response, this will be resolved for 4.0 release.

