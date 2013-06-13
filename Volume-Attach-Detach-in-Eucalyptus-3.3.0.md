# Overview of Volume Attach/Detach in Eucalyptus 3.3.0

The message flow and logic for attaching and detaching volumes to instances has changed significantly in Eucalyptus 3.3.0. Specifically, a new communication path is now used and the Storage Controller keeps more metadata about the status of a volume with regard to attachment.

