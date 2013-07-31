  * Eucalyptus Source Tree
      * [VERSION](https://github.com/eucalyptus/eucalyptus/blob/master/VERSION)
          * Format is X.X.X (e.g., 3.3.0).
      * Run autoconf to regenerate the [configure](https://github.com/eucalyptus/eucalyptus/blob/master/configure) script.
      * **console/eucaconsole/version.py** which is generated when _setup.py_ is run.
      * [eucalyptus-opts.h](https://github.com/eucalyptus/eucalyptus/blob/master/clc/modules/bootstrap/src/main/native/eucalyptus-opts.h) is generated using _gengetopt_ and options are injected from _arguments.ggo_ which is created at _configure_ time.
      * [Upgrades.java] (https://github.com/eucalyptus/eucalyptus/blob/master/clc/modules/msgs/src/main/java/com/eucalyptus/upgrade/Upgrades.java) should contain an enum value for the current version.
          * Format is vX_X_X (e.g., v3_3_0).
  * Eucalyptus Datawarehouse Source Tree
      * [VERSION](https://github.com/eucalyptus/bodega/blob/master/VERSION)
  * Load Balancer Servo Source Tree
      * [VERSION](https://github.com/eucalyptus/load-balancer-servo/blob/master/VERSION)
  * Load Balancer Image Source Tree
      * [VERSION](https://github.com/eucalyptus/load-balancer-image/blob/master/VERSION)
  * RPM Spec Files
      * [Eucalyptus](https://github.com/eucalyptus/eucalyptus-rpmspec/blob/master/eucalyptus.spec)
      * Eucalyptus Enterprise (internal)
      * [Eucalyptus Datawarehouse](https://github.com/eucalyptus/bodega-rpmspec/blob/master/bodega.spec)
      * [Load Balancer Servo](https://github.com/eucalyptus/load-balancer-servo/blob/master/load-balancer-servo.spec)
      * [Load Balancer Image](https://github.com/eucalyptus/load-balancer-image/blob/master/eucalyptus-load-balancer-image.spec) (not needed as this is updated automatically)

*****

[[category.releng]]