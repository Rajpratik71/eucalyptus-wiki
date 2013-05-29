  * Eucalyptus Source Tree
      * **./VERSION**
          * Format is X.X.X (e.g., 3.3.0).
      * **./console/eucaconsole/version.py** which is generated when _setup.py_ is run.
      * **./clc/modules/bootstrap/src/main/native/{arguments.ggo,eucalyptus-bootstrap.h}** are generated using _gengetopt_.
      * **./clc/modules/msgs/src/main/java/com/eucalyptus/upgrade/Upgrades.java** should contain an enum value for the current version.
          * Format is vX_X_X (e.g., v3_3_0).
  * Eucalyptus Datawarehouse Source Tree
      * **./VERSION**
  * Load Balancer Servo Source Tree
      * **./VERSION**
  * Load Balancer Image Source Tree
      * **./VERSION**
  * RPM Spec Files
      * Eucalyptus spec file
      * Eucalyptus Enterprise spec file
      * Eucalyptus Datawarehouse spec file
      * Load Balancer Servo spec file
      * Load Balancer Image spec file (not needed as this is updated automatically)

*****

[[category.releng]]